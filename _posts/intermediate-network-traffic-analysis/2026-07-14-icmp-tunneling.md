---
layout: post
title: "ICMP Tunneling"
date: 2026-07-14 23:45:30 +0700
categories: ["Intermediate Network Traffic Analysis"]
tags: [icmp, icmp-tunneling, data-exfiltration, command-and-control, covert-channel, fragmentation, wireshark, tshark, suricata, zeek, splunk, elk, sigma, threat-hunting, incident-response, cdsa]
description: "Phân tích ICMP Tunneling: dữ liệu bị nhúng trong Echo Request/Reply, payload bất thường, fragmentation, Base64, detection bằng Wireshark/Tshark/Suricata/Zeek và quy trình ứng phó."
toc: true
---

# ICMP Tunneling

Bài này thuộc module **Intermediate Network Traffic Analysis**, tập trung vào cách attacker sử dụng ICMP như một covert channel để truyền lệnh, duy trì command-and-control hoặc exfiltrate dữ liệu.

```text
Mục tiêu: nhận diện payload ICMP bất thường, fragmentation,
data length lớn, encoding/encryption và hành vi request/reply có tính phiên.
```

> Chỉ thực hành trong HTB, lab hoặc hệ thống có ủy quyền rõ ràng. Không sử dụng ICMP tunneling để truyền dữ liệu ngoài phạm vi được phép.

---

## Timeline cập nhật

```text
Created at: 2026-07-14 23:45:30 +0700
Timezone: Asia/Ho_Chi_Minh
Module: Intermediate Network Traffic Analysis
Section: ICMP Tunneling
Focus: Covert channel, data exfiltration, fragmentation, Wireshark, threat hunting
```

---

# 1. Tunneling là gì?

Tunneling là kỹ thuật đóng gói dữ liệu bên trong một giao thức khác để vượt qua network controls.

Các loại thường gặp:

- SSH tunneling.
- HTTP/HTTPS tunneling.
- DNS tunneling.
- ICMP tunneling.
- Proxy tunneling.
- VPN-like covert channels.

Mục tiêu của attacker thường là:

- Bypass firewall.
- Duy trì command-and-control.
- Truyền lệnh tới host đã bị compromise.
- Exfiltrate dữ liệu.
- Pivot vào network segment khác.

---

# 2. ICMP Tunneling hoạt động như thế nào?

ICMP Echo Request và Echo Reply có vùng dữ liệu tùy ý.

Attacker có thể chèn dữ liệu vào:

```text
ICMP Echo Request Data
ICMP Echo Reply Data
```

Flow:

```text
Compromised host
    ↓ ICMP Echo Request + encoded data
C2 server
    ↓ ICMP Echo Reply + command/data
Compromised host
```

Do ICMP thường được cho phép để troubleshooting, lưu lượng này có thể bị bỏ sót.

---

# 3. ICMP bình thường trông như thế nào?

Một ICMP Echo Request bình thường thường có:

- Payload nhỏ.
- Pattern cố định theo hệ điều hành.
- Request/Reply tương ứng.
- Tần suất thấp hoặc ngắt quãng.
- Không chứa text nhạy cảm.
- Không fragmentation bất thường.

Ví dụ payload có thể khoảng:

```text
32 bytes
48 bytes
56 bytes
64 bytes
```

Tùy OS và công cụ.

---

# 4. Dấu hiệu ICMP Tunneling

Các dấu hiệu đáng chú ý:

- Payload ICMP lớn bất thường.
- Fragmented ICMP traffic.
- Tần suất request/reply đều như beacon.
- Payload thay đổi liên tục.
- Có text, credential hoặc file data.
- Base64/hex/encrypted-looking payload.
- Một host gửi ICMP kéo dài tới external IP hiếm gặp.
- Echo Reply mang dữ liệu lớn tương tự Request.
- ICMP hoạt động ngoài giờ hoặc từ server không có nhu cầu ping.

---

# 5. Mở PCAP trong Wireshark

```bash
wireshark icmp_tunneling.pcapng
```

Lọc toàn bộ ICMP:

```wireshark
icmp
```

Echo Request:

```wireshark
icmp.type == 8
```

Echo Reply:

```wireshark
icmp.type == 0
```

---

# 6. Phân tích Data Length

Trong packet details:

```text
Internet Control Message Protocol
→ Data
→ Length
```

Ví dụ bình thường:

```text
Data length: 48 bytes
```

Ví dụ đáng ngờ:

```text
Data length: 38000 bytes
```

Payload cực lớn thường gây fragmentation và phải được điều tra.

---

# 7. Filter Payload Lớn

Wireshark có thể dùng frame length hoặc IP length làm chỉ báo gần đúng.

Ví dụ:

```wireshark
icmp && frame.len > 200
```

Hoặc:

```wireshark
icmp && ip.len > 200
```

ICMP payload rất lớn:

```wireshark
icmp && ip.len > 1000
```

Cần baseline theo hệ điều hành và công cụ giám sát hợp lệ.

---

# 8. Phát hiện Fragmentation

ICMP tunnel có thể tạo IP fragments nếu payload vượt MTU.

Filter:

```wireshark
icmp || (ip.flags.mf == 1 || ip.frag_offset > 0)
```

Chỉ fragments:

```wireshark
ip.flags.mf == 1 || ip.frag_offset > 0
```

Theo source nghi vấn:

```wireshark
ip.src == 192.168.10.5 &&
(ip.flags.mf == 1 || ip.frag_offset > 0)
```

Nhiều fragment cùng `ip.id` cho thấy một ICMP datagram lớn đang được reassemble.

---

# 9. Xem Payload trong Wireshark

Có thể xem:

```text
Packet Bytes
ASCII pane
Data field
Follow flow theo IP pair
```

Nếu thấy dữ liệu kiểu:

```text
Username: root
Password: ...
```

đây là chỉ báo exfiltration trực tiếp.

Trong báo cáo công khai nên redacted credential thật.

---

# 10. Base64 và Encoding

Attacker thường encode payload để giảm khả năng nhìn thấy bằng mắt thường.

Ví dụ:

```text
VGhpcyBpcyBhIHNlY3VyZSBrZXk6IEtleTEyMzQ1Njc4OQo=
```

Decode trong Linux:

```bash
echo 'VGhpcyBpcyBhIHNlY3VyZSBrZXk6IEtleTEyMzQ1Njc4OQo=' | base64 -d
```

Tuy nhiên, chỉ thấy Base64 không đủ để kết luận malicious; cần correlate thêm destination, frequency và behavior.

---

# 11. Encrypted Payload

Attacker nâng cao có thể encrypt payload.

Dấu hiệu:

- Entropy cao.
- Không đọc được ASCII.
- Kích thước payload tương đối ổn định.
- Request/Reply có nhịp đều.
- Sequence/Identifier được dùng như session state.
- Destination không phù hợp business need.

Với encrypted payload, detection phải dựa vào metadata nhiều hơn nội dung.

---

# 12. Identifier và Sequence Number

ICMP Echo có:

```text
Identifier
Sequence Number
```

Attacker có thể dùng chúng để:

- Xác định session.
- Chia dữ liệu thành chunks.
- Theo dõi thứ tự.
- Đồng bộ lệnh và response.

Dấu hiệu đáng chú ý:

- Sequence tăng đều trong nhiều giờ.
- Identifier cố định nhưng payload thay đổi.
- Reply không giống hành vi ping thông thường.
- Nhiều session song song từ một endpoint.

---

# 13. Wireshark Filters Hữu ích

ICMP giữa hai host:

```wireshark
icmp &&
ip.addr == 192.168.10.5 &&
ip.addr == 192.168.10.1
```

Echo Request payload lớn:

```wireshark
icmp.type == 8 && ip.len > 200
```

Echo Reply payload lớn:

```wireshark
icmp.type == 0 && ip.len > 200
```

Fragmented ICMP:

```wireshark
icmp &&
(ip.flags.mf == 1 || ip.frag_offset > 0)
```

Một external destination:

```wireshark
icmp && ip.dst == <EXTERNAL_IP>
```

---

# 14. Tshark Hunt

Liệt kê ICMP packet length:

```bash
tshark -r icmp_tunneling.pcapng   -Y "icmp"   -T fields   -e frame.number   -e ip.src   -e ip.dst   -e icmp.type   -e icmp.ident   -e icmp.seq   -e ip.len
```

Tìm payload lớn:

```bash
tshark -r icmp_tunneling.pcapng   -Y "icmp && ip.len > 200"   -T fields   -e frame.number   -e ip.src   -e ip.dst   -e ip.len
```

Trích data:

```bash
tshark -r icmp_tunneling.pcapng   -Y "icmp.type == 8"   -T fields -e data.data
```

Decode hex thành text:

```bash
echo '<HEX_DATA>' | xxd -r -p
```

---

# 15. Detection Logic

Một detection tốt nên kết hợp:

```text
ICMP payload lớn
+ tần suất đều
+ external destination hiếm gặp
+ request/reply đều mang dữ liệu
+ fragmentation hoặc entropy cao
```

Feature nên theo dõi:

- Source/destination.
- ICMP type.
- Payload size.
- Packet rate.
- Session duration.
- Identifier/sequence pattern.
- Fragment count.
- Entropy.
- Time-of-day.
- Asset role.

---

# 16. Suricata Detection Ideas

Payload lớn:

```suricata
alert icmp $HOME_NET any -> $EXTERNAL_NET any (
  msg:"Possible ICMP tunneling large payload";
  itype:8;
  dsize:>200;
  threshold:type both, track by_src, count 5, seconds 30;
  sid:1000401;
  rev:1;
)
```

Reply lớn:

```suricata
alert icmp $EXTERNAL_NET any -> $HOME_NET any (
  msg:"Possible ICMP tunnel response";
  itype:0;
  dsize:>200;
  sid:1000402;
  rev:1;
)
```

Cần test syntax và threshold trong lab.

---

# 17. Zeek Detection Ideas

Zeek có thể theo dõi:

- ICMP volume.
- Long-lived ICMP peer relationship.
- Payload-length consistency.
- External rare destinations.
- ICMP request/reply ratio.
- Fragmentation anomalies.

Nguồn log:

```text
icmp.log
conn.log
weird.log
notice.log
```

Tùy Zeek configuration và scripts được cài đặt.

---

# 18. Splunk Detection

Ví dụ với Zeek/Suricata logs:

```spl
index=network (protocol=icmp OR proto=icmp)
| stats count
        avg(bytes_out) as avg_out
        max(bytes_out) as max_out
        dc(dest_ip) as unique_destinations
  by src_ip, dest_ip
| where max_out > 200 OR count > 100
```

Suricata alerts:

```spl
index=suricata event_type=alert
alert.signature="*ICMP tunnel*"
| table _time src_ip dest_ip alert.signature packet
```

Long-lived ICMP pair:

```spl
index=network protocol=icmp
| bin _time span=5m
| stats count sum(bytes) as total_bytes by _time src_ip dest_ip
| where total_bytes > 5000
```

---

# 19. ELK/KQL

Payload/packet-size hunting phụ thuộc schema ingest.

Ví dụ:

```kql
network.transport: "icmp"
```

Large byte count:

```kql
network.transport: "icmp" and network.bytes > 200
```

External destination:

```kql
network.transport: "icmp"
and not destination.ip: (
  10.0.0.0/8
  or 172.16.0.0/12
  or 192.168.0.0/16
)
```

---

# 20. Sigma và Network Detection

Sigma chủ yếu dành cho log-based detection, không trực tiếp parse PCAP.

Có thể viết Sigma cho:

- Firewall logs có ICMP egress bất thường.
- EDR network events.
- Suricata alert forwarding.
- Process chạy công cụ tunneling.

Ví dụ concept:

```yaml
title: Suspicious Long-Lived ICMP Communication
status: experimental
logsource:
  category: network_connection
detection:
  selection:
    Protocol: ICMP
  condition: selection
falsepositives:
  - Network monitoring
  - Approved diagnostics
level: medium
```

Cần bổ sung threshold trong SIEM.

---

# 21. Endpoint Hunt

Ngoài network telemetry, cần hunt endpoint:

- Process tạo raw socket.
- Unusual ping utilities.
- Custom binaries.
- PowerShell/Python scripts gửi ICMP.
- Network tools chạy từ temp/user profile.
- Scheduled task duy trì tunnel.

Sysmon hữu ích:

```text
Event ID 1 - Process Create
Event ID 3 - Network Connection
Event ID 11 - File Create
```

Lưu ý Sysmon Event 3 có thể không ghi đầy đủ ICMP tùy configuration.

---

# 22. Prevention

Biện pháp khả thi:

- Block outbound ICMP nếu business không cần.
- Chỉ cho phép ICMP types cần thiết.
- Rate-limit ICMP.
- Inspect payload.
- Alert payload vượt baseline.
- Segment server/user networks.
- Egress filtering.
- NDR/IDS inspection.
- Proxy hoặc gateway policy.
- Endpoint application control.

Không nên chặn toàn bộ ICMP một cách mù quáng vì ICMP cần cho:

- Path MTU Discovery.
- Network diagnostics.
- Error reporting.
- IPv6 Neighbor Discovery và operation.

---

# 23. Response

```text
[ ] Xác định source host và destination.
[ ] Đo payload size và session duration.
[ ] Trích payload nếu hợp pháp.
[ ] Kiểm tra Base64/hex/encryption.
[ ] Correlate endpoint process.
[ ] Isolate host nếu nghi C2/exfiltration.
[ ] Block destination và IOC liên quan.
[ ] Xác định dữ liệu đã bị truyền.
[ ] Reset credential nếu payload chứa secret.
[ ] Thu memory/disk/PCAP phục vụ forensics.
```

---

# 24. Mapping với CDSA

## Security Operations & Monitoring

- Baseline ICMP payload size và frequency.
- Alert external ICMP session dài.
- Correlate Suricata/Zeek/firewall.

## Incident Response & Forensics

- Trích và decode payload.
- Xác định dữ liệu bị exfiltrate.
- Triage process và persistence.
- Thu PCAP/memory.

## Threat Hunting

- Hunt ICMP payload > baseline.
- Hunt encrypted-looking payload.
- Hunt request/reply đều mang dữ liệu.
- Hunt rare external destinations.
- Hunt fragmentation và long-lived sessions.

## Detection Engineering

- Suricata large-payload rule.
- Zeek script theo payload/time-series.
- Splunk/Elastic threshold.
- Asset-role và destination-reputation enrichment.

---

# 25. Lab Setup

Môi trường tối thiểu:

```text
1 Linux attack host
1 target host
1 Wireshark/tshark sensor
Suricata hoặc Zeek
Splunk/ELK/Wazuh nhận network logs
```

Workflow:

```text
1. Mở icmp_tunneling.pcapng.
2. Lọc icmp.
3. So sánh payload bình thường và bất thường.
4. Kiểm tra fragmentation.
5. Trích và decode payload.
6. Viết detection rule.
7. Correlate endpoint process.
```

---

# Key Takeaway

```text
ICMP tunneling nhúng dữ liệu vào Echo Request/Reply để tạo covert channel.
Payload lớn, fragmentation, beaconing và dữ liệu encoded là dấu hiệu mạnh.
Payload lớn hơn baseline cần điều tra nhưng không nên dùng một threshold cố định cho mọi môi trường.
Detection hiệu quả phải kết hợp packet size, frequency, destination rarity và endpoint context.
```

ICMP Tunneling là chủ đề quan trọng của CDSA vì kết nối trực tiếp packet analysis, command-and-control detection, data exfiltration, threat hunting và incident response.

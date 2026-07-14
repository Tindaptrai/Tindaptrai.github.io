---
layout: post
title: "IP Source & Destination Spoofing Attacks"
date: 2026-07-14 23:17:18 +0700
categories: ["Intermediate Network Traffic Analysis"]
tags: [ip-spoofing, source-spoofing, destination-spoofing, decoy-scanning, random-source-ddos, smurf-attack, land-attack, fragmentation, wireshark, tshark, ids, ips, firewall, threat-hunting, incident-response, cdsa]
description: "Phân tích IP source/destination spoofing: decoy scanning, random-source DDoS, SMURF, LAND, Wireshark filters, detection logic và response."
toc: true
---

# IP Source & Destination Spoofing Attacks

Bài này thuộc module **Intermediate Network Traffic Analysis**, tập trung vào các kỹ thuật giả mạo địa chỉ nguồn/đích ở tầng IP và cách phát hiện bằng PCAP, IDS/IPS, firewall và SIEM.

```text
Mục tiêu: nhận diện source IP bất thường, decoy scanning,
random-source attacks, SMURF, LAND và các dấu hiệu fragmentation liên quan.
```

> Chỉ thực hành trong HTB, lab hoặc hệ thống có ủy quyền rõ ràng. Không tạo lưu lượng spoofed hoặc DoS ngoài phạm vi cho phép.

---

## Timeline cập nhật

```text
Created at: 2026-07-14 23:17:18 +0700
Timezone: Asia/Ho_Chi_Minh
Module: Intermediate Network Traffic Analysis
Section: IP Source & Destination Spoofing Attacks
Focus: Decoy scanning, random-source DDoS, SMURF, LAND, packet analysis
```

---

# 1. Nguyên tắc kiểm tra Source IP

Khi phân tích traffic, cần kiểm tra tính hợp lý của địa chỉ nguồn.

## Inbound traffic

Nếu packet đi **vào LAN** nhưng source IP lại thuộc chính subnet nội bộ:

```text
Source IP nội bộ xuất hiện từ interface bên ngoài
```

đây có thể là dấu hiệu spoofing hoặc packet crafting.

## Outbound traffic

Nếu packet đi **ra ngoài** nhưng source IP không thuộc subnet nội bộ:

```text
Source IP lạ xuất hiện từ endpoint nội bộ
```

đây cũng là dấu hiệu bất thường.

---

# 2. Ingress và Egress Filtering

Hai biện pháp cơ bản:

```text
Ingress filtering:
Chặn packet từ bên ngoài nhưng giả source IP nội bộ.

Egress filtering:
Chặn packet rời mạng nếu source IP không thuộc dải được cấp.
```

Đây là nền tảng chống IP spoofing ở firewall/router.

---

# 3. Decoy Scanning

Decoy scanning cố che source thật bằng cách chèn thêm nhiều source IP giả.

Pattern:

```text
Fake source IPs gửi SYN/fragment
Source thật vẫn phải tham gia để nhận kết quả
Target trả RST/ACK cho closed ports
```

Dấu hiệu:

- Nhiều source IP cùng scan một target.
- Các packet có cấu trúc gần giống nhau.
- Một source IP xuất hiện đều đặn hơn các decoy.
- Target trả RST về source thật.
- Fragmentation được dùng để né IDS/firewall.

---

# 4. Decoy Scan và Fragmentation

Một decoy scan có thể kết hợp:

```text
ICMP discovery
IPv4 fragmentation
TCP SYN probes
RST/ACK responses
```

Điểm đáng chú ý:

```text
Một host bắt đầu connection,
nhưng host khác tiếp tục hoặc nhận response.
```

Đây là behavior không bình thường trong một TCP flow hợp lệ.

---

# 5. Wireshark Filters cho Decoy Scan

Theo target:

```wireshark
ip.dst == 192.168.10.1
```

SYN scan:

```wireshark
tcp.flags.syn == 1 && tcp.flags.ack == 0
```

RST responses:

```wireshark
tcp.flags.reset == 1
```

Fragments:

```wireshark
ip.flags.mf == 1 || ip.frag_offset > 0
```

Nhiều source cùng target:

```wireshark
ip.dst == 192.168.10.1 && tcp.flags.syn == 1
```

Sau đó dùng **Statistics → Conversations** hoặc **Endpoints** để so sánh source IP.

---

# 6. Random Source Attacks

Random-source attack dùng nhiều source IP giả để gửi traffic tới cùng một service hoặc victim.

Dấu hiệu:

- Rất nhiều source IP.
- Cùng destination IP.
- Cùng destination port.
- Packet length gần như giống nhau.
- Source port tăng tuần tự hoặc theo pattern.
- TCP flags giống nhau.
- Không có TCP handshake hoàn chỉnh.

Ví dụ:

```text
Nhiều source IP → 192.168.10.1:80
```

---

# 7. Phân biệt Web Traffic thật và Random-Source Flood

Traffic web hợp lệ thường có:

- Packet sizes đa dạng.
- Source ports tương đối ngẫu nhiên.
- TCP handshake đầy đủ.
- ACK/data/FIN hợp lý.
- User-agent/application behavior khác nhau.

Random-source flood thường có:

```text
Cùng packet length
Cùng flags
Source ports tăng đều
Không hoàn tất handshake
Volume rất cao
```

---

# 8. Random Source ICMP

Một biến thể dùng ICMP:

- Nhiều destination ngẫu nhiên.
- Một host gửi ICMP replies tới các IP không có thật.
- Fragments được dùng để tăng tài nguyên reassembly.
- Payload lớn làm tăng bandwidth/resource consumption.

Filter:

```wireshark
icmp
```

Fragments ICMP:

```wireshark
icmp && (ip.flags.mf == 1 || ip.frag_offset > 0)
```

---

# 9. SMURF Attack

SMURF là amplification attack dùng ICMP.

Flow:

```text
1. Attacker gửi ICMP Echo Request tới broadcast/many hosts.
2. Source IP bị giả thành victim.
3. Nhiều hosts gửi ICMP Echo Reply về victim.
4. Victim bị resource exhaustion.
```

Dấu hiệu:

- Victim nhận nhiều ICMP replies.
- Victim không hề gửi requests tương ứng.
- Nhiều nguồn cùng trả lời trong thời gian ngắn.
- Payload/fragmentation có thể lớn bất thường.

---

# 10. Wireshark Filter cho SMURF

ICMP replies tới victim:

```wireshark
icmp.type == 0 && ip.dst == <VICTIM_IP>
```

ICMP requests có source giả:

```wireshark
icmp.type == 8
```

Volume cao:

```wireshark
icmp && ip.dst == <VICTIM_IP>
```

Dùng:

```text
Statistics → I/O Graphs
Statistics → Endpoints
```

để xem spike.

---

# 11. LAND Attack

LAND attack giả:

```text
Source IP = Destination IP
```

và thường có:

```text
Source port = Destination port
```

Mục tiêu là làm hệ thống tự phản hồi cho chính nó, gây loop hoặc cạn kiệt tài nguyên trên các hệ thống cũ.

---

# 12. Wireshark Filter cho LAND

```wireshark
ip.src == ip.dst
```

TCP LAND pattern:

```wireshark
ip.src == ip.dst && tcp.srcport == tcp.dstport
```

SYN LAND:

```wireshark
ip.src == ip.dst &&
tcp.flags.syn == 1 &&
tcp.flags.ack == 0
```

Bất kỳ packet nào thỏa `ip.src == ip.dst` đều cần được xem xét kỹ.

---

# 13. Dấu hiệu LAND Attack

- Source IP và destination IP giống nhau.
- Port reuse bất thường.
- Nhiều SYN tới cùng service.
- Không có handshake hợp lệ.
- Host bị tăng CPU/session table.
- Firewall state table tăng nhanh.

---

# 14. Initialization Vector Generation trong WEP

Trong mạng WEP cũ, attacker có thể:

- Capture packet.
- Chỉnh source/destination IP.
- Reinject packet.
- Tạo nhiều IV.
- Dùng IV để xây statistical attack.

Dấu hiệu:

```text
Rất nhiều packet lặp lại
Payload gần giống nhau
Source/destination thay đổi bất thường
IV count tăng nhanh
```

WEP không còn an toàn và nên được loại bỏ hoàn toàn.

---

# 15. Tshark Hunt

Đếm source IP theo destination:

```bash
tshark -r decoy_scanning_nmap.pcapng   -Y "tcp.flags.syn == 1 && tcp.flags.ack == 0"   -T fields -e ip.src -e ip.dst -e tcp.dstport
```

Tìm LAND:

```bash
tshark -r LAND-DoS.pcapng   -Y "ip.src == ip.dst"   -T fields -e frame.number -e ip.src -e ip.dst -e tcp.srcport -e tcp.dstport
```

Đếm ICMP replies theo source:

```bash
tshark -r ICMP_rand_source.pcapng   -Y "icmp.type == 0"   -T fields -e ip.src |
sort | uniq -c | sort -nr
```

---

# 16. Detection Logic

## Decoy Scanning

```text
Nhiều source IP
→ một target
→ nhiều destination ports
→ cùng TCP flags/payload
→ một source thật nhận response
```

## Random Source Flood

```text
Nhiều source IP
→ một destination port
→ packet length giống nhau
→ source port tăng tuần tự
```

## SMURF

```text
Nhiều ICMP replies
→ một victim
→ không có requests tương ứng từ victim
```

## LAND

```text
ip.src == ip.dst
và/hoặc
tcp.srcport == tcp.dstport
```

---

# 17. Suricata/Zeek Detection Ideas

## Suricata

Theo dõi:

- Spoofed internal source from external interface.
- Same source/destination IP.
- SYN flood từ nhiều source.
- ICMP amplification.
- Fragmentation anomalies.

## Zeek

Correlate:

- `conn.log`
- `notice.log`
- `weird.log`
- Scan detection.
- One-target-many-sources.
- Repeated identical connection attempts.

---

# 18. Firewall Hardening

- Ingress filtering.
- Egress filtering.
- Unicast Reverse Path Forwarding.
- Anti-spoofing ACL.
- Drop directed broadcast.
- Rate-limit ICMP.
- SYN cookies.
- Connection limits.
- Fragment normalization/reassembly.
- BCP38-aligned filtering.

---

# 19. SIEM Correlation

Ví dụ logic:

```text
Firewall:
Nhiều source IP → cùng target:80
        ↓
IDS:
SYN flood / spoofing alert
        ↓
Server:
Session table hoặc CPU tăng
        ↓
SOC:
Xác định DDoS hoặc spoofed scan
```

Nguồn log:

- Firewall.
- NetFlow.
- Suricata.
- Zeek.
- Router ACL logs.
- Load balancer.
- Endpoint telemetry.

---

# 20. Incident Response Checklist

```text
[ ] Xác định victim và service bị nhắm tới.
[ ] Xác định source IP có hợp lệ/routable không.
[ ] Kiểm tra interface mà packet đi vào.
[ ] So sánh source IP với subnet/CMDB.
[ ] Kiểm tra packet length, flags và source-port pattern.
[ ] Kiểm tra fragmentation.
[ ] Correlate NetFlow/firewall/IDS.
[ ] Rate-limit hoặc block tại edge.
[ ] Liên hệ ISP/upstream nếu DDoS lớn.
[ ] Thu PCAP và preserve evidence.
```

---

# 21. Mapping với CDSA

## Security Operations & Monitoring

- Anti-spoofing tại perimeter.
- Baseline source networks.
- Theo dõi one-target-many-sources.
- Correlate PCAP, NetFlow và firewall.

## Incident Response & Forensics

- Xác định attack type.
- Phân tích packet fields.
- Timeline volume spike.
- Đánh giá service impact.

## Threat Hunting

- Hunt source IP không phù hợp interface.
- Hunt identical packet lengths.
- Hunt sequential source ports.
- Hunt ICMP replies không có request.
- Hunt `ip.src == ip.dst`.

## Detection Engineering

- Suricata/Zeek rules.
- Firewall anti-spoofing ACL.
- SIEM correlation cho decoy/random-source.
- Baseline legitimate high-volume services.

---

# 22. Lab Setup

Môi trường tối thiểu:

```text
1 Linux attack host
1 target server
1 router/firewall
1 Wireshark/tshark sensor
Suricata hoặc Zeek
Splunk/ELK/Wazuh nhận network logs
```

Workflow:

```text
1. Mở PCAP.
2. Kiểm tra source/destination consistency.
3. Xác định source fan-out hoặc target concentration.
4. Kiểm tra TCP flags và packet length.
5. Correlate fragmentation.
6. Viết detection.
7. Tune false positives.
```

---

# Key Takeaway

```text
IP spoofing thường lộ qua source IP không phù hợp với interface hoặc subnet.
Decoy scanning tạo nhiều source giả nhưng source thật vẫn phải nhận kết quả.
Random-source floods có packet structure giống nhau và tập trung vào một service.
SMURF tạo nhiều ICMP replies về victim.
LAND có source IP bằng destination IP.
Ingress/egress filtering và stateful reassembly là biện pháp phòng thủ cốt lõi.
```

IP Source & Destination Spoofing Attacks là chủ đề trọng tâm của CDSA vì yêu cầu kết hợp packet analysis, firewall policy, IDS/IPS detection, threat hunting và incident response.

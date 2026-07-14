---
layout: post
title: "Fragmentation Attacks"
date: 2026-07-14 23:07:02 +0700
categories: ["Intermediate Network Traffic Analysis"]
tags: [ip-fragmentation, fragmented-packets, ids-evasion, ips-evasion, firewall-evasion, denial-of-service, nmap, wireshark, packet-analysis, threat-hunting, incident-response, cdsa]
description: "Phân tích IP Fragmentation Attacks: fragment offset, tiny fragments, IDS/IPS/firewall evasion, DoS, Wireshark reassembly và dấu hiệu one-host-to-many-ports."
toc: true
---

# Fragmentation Attacks

Bài này thuộc module **Intermediate Network Traffic Analysis**, tập trung vào cách attacker lạm dụng IP fragmentation để né IDS/IPS/firewall, gây cạn kiệt tài nguyên hoặc tạo Denial-of-Service.

```text
Mục tiêu: hiểu IPv4 fragmentation, nhận diện tiny fragments,
phân tích fragment offset và phát hiện scan phân mảnh trong Wireshark.
```

> Chỉ thực hành trong HTB, lab hoặc hệ thống có ủy quyền rõ ràng. Không chạy fragmented scan hoặc DoS test ngoài phạm vi cho phép.

---

## Timeline cập nhật

```text
Created at: 2026-07-14 23:07:02 +0700
Timezone: Asia/Ho_Chi_Minh
Module: Intermediate Network Traffic Analysis
Section: Fragmentation Attacks
Focus: IPv4 fragmentation, IDS/IPS evasion, firewall bypass, DoS, Wireshark reassembly
```

---

# 1. IPv4 Fragmentation là gì?

Khi một IP packet lớn hơn MTU của đường truyền, packet có thể được chia thành nhiều fragments nhỏ hơn.

Các field quan trọng trong IPv4 header:

| Field | Ý nghĩa |
|---|---|
| Header Length | Độ dài IPv4 header |
| Total Length | Tổng kích thước packet |
| Identification | ID chung cho các fragments cùng packet |
| Flags | Bao gồm DF và MF |
| Fragment Offset | Vị trí fragment trong packet gốc |
| Source/Destination IP | Host gửi và nhận |
| Protocol | TCP, UDP, ICMP... |

---

# 2. Các cờ fragmentation

Hai cờ quan trọng:

```text
DF - Don't Fragment
MF - More Fragments
```

Nếu `MF=1`, vẫn còn fragments phía sau.

Fragment cuối thường có:

```text
MF=0
```

Các fragments của cùng một packet thường có cùng `Identification`.

---

# 3. Vì sao attacker lạm dụng fragmentation?

## IDS/IPS Evasion

Nếu IDS/IPS không reassemble fragments giống destination host, attacker có thể chia payload hoặc TCP header thành nhiều phần để né signature.

## Firewall Evasion

Firewall chỉ nhìn fragment đầu hoặc không reassemble đầy đủ có thể bỏ sót TCP/UDP ports và flags.

## Resource Exhaustion

Attacker gửi hàng nghìn tiny fragments để buộc firewall/IDS giữ state và chờ reassembly.

## Denial-of-Service

Malformed, overlapping hoặc oversized fragment chains có thể làm crash hoặc treo hệ thống cũ.

---

# 4. Tiny Fragment Attack

Tiny fragments cố tình chia packet thành các fragment rất nhỏ.

Ví dụ lab:

```bash
nmap -f 10 <TARGET_IP>
```

Mục tiêu là tách TCP header thành nhiều fragments, khiến một số security device khó xác định:

- Source/destination port.
- TCP flags.
- Scan type.
- Payload signature.

---

# 5. Delayed Reassembly

Một network security control đúng phải:

```text
1. Thu thập các fragments cùng Identification.
2. Chờ đủ fragments.
3. Reassemble giống destination host.
4. Sau đó mới thực hiện inspection.
```

Nếu IDS, firewall và destination reassemble khác nhau, attacker có thể khai thác sự khác biệt đó.

---

# 6. Mở PCAP trong Wireshark

```bash
wireshark nmap_frag_fw_bypass.pcapng
```

Bộ lọc IPv4 fragments:

```wireshark
ip.flags.mf == 1 || ip.frag_offset > 0
```

Chỉ fragment không phải fragment đầu:

```wireshark
ip.frag_offset > 0
```

Theo một Identification cụ thể:

```wireshark
ip.id == 0xf39b
```

---

# 7. Dấu hiệu Fragmentation Scan

Các dấu hiệu chính:

- Nhiều IPv4 fragments từ một source.
- Fragment size rất nhỏ.
- Một source quét nhiều destination ports.
- Nhiều SYN packets sau reassembly.
- Target trả nhiều `RST, ACK` cho closed ports.
- Fragment IDs thay đổi liên tục.
- Scan bắt đầu bằng ICMP host discovery.

---

# 8. One Host to Many Ports

Dấu hiệu nổi bật nhất của fragmented port scan là:

```text
Một source host → một destination host → rất nhiều destination ports
```

Target sẽ trả:

```text
RST, ACK
```

cho các closed ports.

![Fragmented scan: one host to many ports](/assets/img/intermediate-network-traffic-analysis/fragmentation-attacks/fragmented-scan-single-host-many-ports.png)

Trong ảnh:

```text
Source scanner: 192.168.10.5
Destination:    192.168.10.1
Ports:          143, 53, 111, 1025, 256, 199, 139...
```

Đây là pattern rõ ràng của port enumeration.

---

# 9. Wireshark Filters hữu ích

Fragments:

```wireshark
ip.flags.mf == 1 || ip.frag_offset > 0
```

Fragments từ source cụ thể:

```wireshark
ip.src == 192.168.10.5 &&
(ip.flags.mf == 1 || ip.frag_offset > 0)
```

SYN scan:

```wireshark
tcp.flags.syn == 1 && tcp.flags.ack == 0
```

RST từ target:

```wireshark
ip.src == 192.168.10.1 &&
tcp.flags.reset == 1
```

Fragments + TCP:

```wireshark
ip.proto == 6 &&
(ip.flags.mf == 1 || ip.frag_offset > 0)
```

---

# 10. Bật Reassembly trong Wireshark

Kiểm tra:

```text
Edit
→ Preferences
→ Protocols
→ IPv4
→ Reassemble fragmented IPv4 datagrams
```

Sau khi bật, Wireshark có thể hiển thị:

```text
Reassembled IPv4 in frame ...
```

Điều này giúp xem TCP header hoặc payload hoàn chỉnh.

---

# 11. Irregular Fragment Offsets

Cần điều tra khi thấy:

- Fragment offsets không tăng hợp lý.
- Fragments chồng lấn.
- Nhiều fragments có cùng offset.
- Fragment cuối đến trước fragment đầu.
- Reassembly timeout.
- Total reassembled size bất thường.

Overlapping fragments đặc biệt quan trọng vì IDS và endpoint có thể ưu tiên fragment khác nhau.

---

# 12. Tshark Hunt

Liệt kê fragments:

```bash
tshark -r nmap_frag_fw_bypass.pcapng   -Y "ip.flags.mf == 1 || ip.frag_offset > 0"   -T fields   -e frame.number   -e ip.src   -e ip.dst   -e ip.id   -e ip.frag_offset   -e ip.flags.mf
```

Đếm fragments theo source:

```bash
tshark -r nmap_frag_fw_bypass.pcapng   -Y "ip.flags.mf == 1 || ip.frag_offset > 0"   -T fields -e ip.src |
sort | uniq -c | sort -nr
```

---

# 13. Detection Logic

Rule gợi ý:

```text
Một source gửi nhiều fragments nhỏ
tới một destination
và sau reassembly truy cập nhiều destination ports.
```

Các feature nên theo dõi:

- Fragment count.
- Average fragment size.
- Unique destination ports.
- Unique IP IDs.
- RST response count.
- Reassembly failures.
- Time window.

---

# 14. Suricata/Zeek Detection Ideas

## Suricata

Theo dõi:

- Fragmentation anomalies.
- Tiny fragments.
- Overlapping fragments.
- Excessive fragment count.
- Port-scan correlation.

## Zeek

Có thể correlate:

- Weird logs.
- Scan framework.
- Connection attempts.
- Fragment inconsistencies.

Telemetry hữu ích:

```text
weird.log
conn.log
notice.log
Suricata eve.json
Firewall session logs
```

---

# 15. Firewall và IDS Hardening

- Bật fragment reassembly.
- Drop overlapping fragments.
- Giới hạn fragment queue.
- Đặt reassembly timeout hợp lý.
- Drop tiny fragments không hợp lệ.
- Rate-limit excessive fragments.
- Normalize traffic trước inspection.
- Giám sát fragmentation từ user VLAN.

---

# 16. Incident Response Checklist

```text
[ ] Xác định source IP/MAC.
[ ] Xác định target host và ports bị scan.
[ ] Kiểm tra fragment size và offsets.
[ ] Bật reassembly để xem packet hoàn chỉnh.
[ ] Correlate firewall, IDS và endpoint logs.
[ ] Xác định scan chỉ reconnaissance hay đã có exploitation.
[ ] Isolate source nếu là host compromise.
[ ] Thu PCAP và preserve evidence.
```

---

# 17. Mapping với CDSA

## Security Operations & Monitoring

- Theo dõi fragment rate.
- Detect tiny fragments và one-to-many-ports.
- Correlate IDS/firewall/PCAP.

## Incident Response & Forensics

- Reassemble traffic.
- Xác định scan timeline.
- Kiểm tra target response và exploitation follow-up.

## Threat Hunting

- Hunt excessive fragments từ một host.
- Hunt overlapping offsets.
- Hunt RST bursts sau fragmented SYNs.
- Hunt user endpoint quét server VLAN.

## Detection Engineering

- Suricata/Sigma-like network rule.
- Zeek scan correlation.
- Threshold theo fragment count và unique ports.
- Baseline legitimate fragmentation.

---

# 18. Lab Setup

Môi trường tối thiểu:

```text
1 Linux attack host
1 target host
1 Wireshark/tshark sensor
Suricata hoặc Zeek
Firewall/IDS có thể bật và tắt reassembly để so sánh
```

Workflow:

```text
1. Capture baseline scan.
2. Capture fragmented scan.
3. So sánh packet size và offsets.
4. Bật Wireshark reassembly.
5. Xác định destination ports.
6. Viết detection rule.
7. Kiểm tra false positives.
```

---

# Key Takeaway

```text
IP fragmentation là cơ chế hợp lệ nhưng có thể bị lạm dụng để né IDS/IPS/firewall.
Tiny fragments và irregular offsets là dấu hiệu quan trọng.
Pattern một source tới nhiều ports vẫn là chỉ báo mạnh của fragmented scan.
Security controls phải reassemble giống destination host trước khi inspection.
```

Fragmentation Attacks là chủ đề điển hình của CDSA vì yêu cầu kết hợp packet analysis, IDS/IPS behavior, threat hunting và incident response ở tầng IP.

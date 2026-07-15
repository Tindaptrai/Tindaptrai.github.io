---
layout: post
title: "TCP Handshake Abnormalities"
date: 2026-07-14 23:30:49 +0700
categories: ["Intermediate Network Traffic Analysis", "Layer 2 & Network Attacks"]
tags: [tcp, tcp-handshake, syn-scan, stealth-scan, null-scan, ack-scan, fin-scan, xmas-scan, nmap, wireshark, tshark, suricata, zeek, threat-hunting, incident-response, cdsa]
description: "Phân tích bất thường TCP handshake: SYN, NULL, ACK, FIN và Xmas scans; Wireshark filters, detection logic, Suricata/Zeek hunting và incident response."
toc: true
---

# TCP Handshake Abnormalities

Bài này thuộc module **Intermediate Network Traffic Analysis**, tập trung vào các bất thường trong TCP three-way handshake và cách nhận diện nhiều kiểu port scan thông qua TCP flags.

```text
Mục tiêu: hiểu TCP flags, phân biệt SYN/NULL/ACK/FIN/Xmas scan,
nhận diện one-host-to-many-ports và xây detection bằng Wireshark, Tshark, Suricata và Zeek.
```

> Chỉ thực hành trong HTB, lab hoặc hệ thống có ủy quyền rõ ràng. Không thực hiện port scanning ngoài phạm vi được phép.

---

## Timeline cập nhật

```text
Created at: 2026-07-14 23:30:49 +0700
Timezone: Asia/Ho_Chi_Minh
Module: Intermediate Network Traffic Analysis
Section: TCP Handshake Abnormalities
Focus: TCP flags, Nmap scans, Wireshark analysis, network detection
```

---

# 1. TCP Three-Way Handshake

Một kết nối TCP bình thường được thiết lập qua ba bước:

```text
Client → Server: SYN
Server → Client: SYN, ACK
Client → Server: ACK
```

Nếu port đóng, target thường trả:

```text
RST, ACK
```

---

# 2. Các TCP Flags

| Flag | Ý nghĩa |
|---|---|
| SYN | Khởi tạo kết nối |
| ACK | Xác nhận dữ liệu/sequence |
| RST | Reset hoặc chấm dứt kết nối |
| FIN | Kết thúc kết nối |
| PSH | Đẩy dữ liệu lên application ngay |
| URG | Dữ liệu khẩn cấp |
| ECE/CWR | Explicit Congestion Notification |

Các scan thường lạm dụng TCP flags để suy luận trạng thái port hoặc né security controls.

---

# 3. Dấu hiệu chung của TCP Scanning

- Một source tới nhiều destination ports.
- Một source tới nhiều destination hosts.
- Rất nhiều SYN, ACK, FIN hoặc RST trong thời gian ngắn.
- Các flag combination bất thường.
- Không hoàn tất three-way handshake.
- Packet size và timing lặp lại.
- Source port tăng tuần tự hoặc có pattern rõ.

---

# 4. SYN Scan

SYN scan gửi SYN tới nhiều port.

Behavior:

```text
Port mở:
SYN → SYN/ACK → scanner gửi RST

Port đóng:
SYN → RST/ACK
```

Nmap thường dùng:

```bash
nmap -sS <TARGET>
```

Dấu hiệu:

- Nhiều SYN.
- Ít hoặc không có ACK hoàn tất handshake.
- RST từ scanner ngay sau SYN/ACK.
- One source → many ports.

---

# 5. Wireshark Filters cho SYN Scan

SYN không kèm ACK:

```wireshark
tcp.flags.syn == 1 && tcp.flags.ack == 0
```

SYN/ACK:

```wireshark
tcp.flags.syn == 1 && tcp.flags.ack == 1
```

RST:

```wireshark
tcp.flags.reset == 1
```

Theo source scanner:

```wireshark
ip.src == 192.168.10.5 &&
tcp.flags.syn == 1 &&
tcp.flags.ack == 0
```

---

# 6. SYN Scan và SYN Stealth Scan

## SYN Scan

Scanner gửi SYN tới nhiều ports và dựa vào response để phân loại port.

## SYN Stealth Scan

Scanner cố tránh hoàn tất handshake:

```text
SYN
→ nhận SYN/ACK
→ gửi RST hoặc im lặng
```

Điểm nhận biết chính vẫn là:

```text
Nhiều half-open connections
```

---

# 7. NULL Scan

NULL scan gửi TCP packet không đặt flag nào.

Nmap:

```bash
nmap -sN <TARGET>
```

Behavior theo RFC trên nhiều hệ thống Unix-like:

```text
Port mở:
Không phản hồi

Port đóng:
RST
```

Windows thường trả RST cho cả open và closed ports, nên scan này không luôn đáng tin.

---

# 8. Wireshark Filter cho NULL Scan

```wireshark
tcp.flags == 0x000
```

Hoặc:

```wireshark
tcp.flags.syn == 0 &&
tcp.flags.ack == 0 &&
tcp.flags.fin == 0 &&
tcp.flags.reset == 0 &&
tcp.flags.push == 0 &&
tcp.flags.urg == 0
```

Đây là pattern rất bất thường trong traffic TCP hợp lệ.

---

# 9. ACK Scan

ACK scan thường được dùng để đánh giá firewall filtering, không trực tiếp xác định port open/closed.

Nmap:

```bash
nmap -sA <TARGET>
```

Behavior thường gặp:

```text
RST trả về:
Port được xem là unfiltered

Không response hoặc ICMP unreachable:
Port được xem là filtered
```

---

# 10. Wireshark Filter cho ACK Scan

ACK không có payload:

```wireshark
tcp.flags.ack == 1 &&
tcp.len == 0 &&
tcp.flags.syn == 0 &&
tcp.flags.fin == 0 &&
tcp.flags.reset == 0
```

Nhiều ACK tới nhiều ports:

```wireshark
ip.src == 192.168.10.5 &&
tcp.flags.ack == 1 &&
tcp.len == 0
```

---

# 11. FIN Scan

FIN scan gửi packet chỉ có FIN.

Nmap:

```bash
nmap -sF <TARGET>
```

Behavior thường gặp:

```text
Port mở:
Không phản hồi

Port đóng:
RST
```

---

# 12. Wireshark Filter cho FIN Scan

```wireshark
tcp.flags.fin == 1 &&
tcp.flags.syn == 0 &&
tcp.flags.ack == 0 &&
tcp.flags.reset == 0
```

Nếu có nhiều packet FIN đơn lẻ tới nhiều ports, khả năng cao là scan.

---

# 13. Xmas Scan

Xmas scan đặt nhiều flags đồng thời, thường:

```text
FIN + PSH + URG
```

Nmap:

```bash
nmap -sX <TARGET>
```

Tên “Xmas” xuất phát từ việc packet được “thắp sáng” bởi nhiều flags.

Behavior thường gặp:

```text
Port mở:
Không phản hồi

Port đóng:
RST
```

---

# 14. Wireshark Filter cho Xmas Scan

```wireshark
tcp.flags.fin == 1 &&
tcp.flags.push == 1 &&
tcp.flags.urg == 1
```

Hoặc theo mask:

```wireshark
tcp.flags == 0x029
```

Cần nhớ giá trị mask có thể phụ thuộc cách Wireshark hiển thị các flag mở rộng.

---

# 15. So sánh các kiểu scan

| Scan | Flags gửi đi | Port mở | Port đóng |
|---|---|---|---|
| SYN | SYN | SYN/ACK | RST/ACK |
| NULL | Không flags | Không response | RST |
| ACK | ACK | RST hoặc không response | RST hoặc filtered |
| FIN | FIN | Không response | RST |
| Xmas | FIN/PSH/URG | Không response | RST |

Kết quả thực tế phụ thuộc:

- Hệ điều hành.
- Firewall.
- IDS/IPS.
- TCP stack.
- Network path.

---

# 16. One Host to Many Ports

Chỉ báo mạnh:

```text
Một source IP
→ một destination IP
→ nhiều destination ports
→ cùng flag pattern
→ trong thời gian ngắn
```

Đây là đặc điểm chung của port scanning.

Trong Wireshark:

```text
Statistics → Conversations → TCP
Statistics → Endpoints
```

---

# 17. Tshark Hunt

Đếm SYN theo source:

```bash
tshark -r nmap_syn_scan.pcapng   -Y "tcp.flags.syn == 1 && tcp.flags.ack == 0"   -T fields -e ip.src |
sort | uniq -c | sort -nr
```

Liệt kê destination ports:

```bash
tshark -r nmap_syn_scan.pcapng   -Y "tcp.flags.syn == 1 && tcp.flags.ack == 0"   -T fields -e ip.src -e ip.dst -e tcp.dstport
```

Tìm NULL scan:

```bash
tshark -r nmap_null_scan.pcapng   -Y "tcp.flags == 0x000"   -T fields -e ip.src -e ip.dst -e tcp.dstport
```

Tìm Xmas scan:

```bash
tshark -r nmap_xmas_scan.pcapng   -Y "tcp.flags.fin == 1 && tcp.flags.push == 1 && tcp.flags.urg == 1"   -T fields -e ip.src -e ip.dst -e tcp.dstport
```

---

# 18. Detection Logic

## SYN Scan

```text
Nhiều SYN không có handshake hoàn chỉnh
+ nhiều destination ports
+ nhiều RST responses
```

## NULL Scan

```text
TCP packet không có flag
+ nhiều destination ports
```

## ACK Scan

```text
Nhiều ACK không thuộc established session
+ nhiều destination ports
```

## FIN Scan

```text
Nhiều FIN không thuộc session đang đóng
+ nhiều destination ports
```

## Xmas Scan

```text
FIN + PSH + URG
+ nhiều destination ports
```

---

# 19. Suricata Detection Ideas

SYN scan pseudo-rule:

```suricata
alert tcp any any -> $HOME_NET any (
  msg:"Possible TCP SYN scan";
  flags:S;
  threshold:type both, track by_src, count 20, seconds 10;
  sid:1000201;
  rev:1;
)
```

NULL scan:

```suricata
alert tcp any any -> $HOME_NET any (
  msg:"Possible TCP NULL scan";
  flags:0;
  sid:1000202;
  rev:1;
)
```

Xmas scan:

```suricata
alert tcp any any -> $HOME_NET any (
  msg:"Possible TCP Xmas scan";
  flags:FPU;
  sid:1000203;
  rev:1;
)
```

Cần test syntax và threshold theo Suricata version/môi trường lab.

---

# 20. Zeek Detection Ideas

Zeek có thể hỗ trợ:

- Port-scan detection.
- Connection-attempt counting.
- Half-open connection tracking.
- Weird TCP flags.
- One-source-to-many-ports correlation.

Nguồn log:

```text
conn.log
notice.log
weird.log
```

---

# 21. SIEM Correlation

Ví dụ:

```text
Suricata:
Possible SYN scan
        ↓
Zeek:
One source → many ports
        ↓
Firewall:
Nhiều deny/RST
        ↓
Endpoint:
Không có process hợp lệ tạo traffic
```

Nguồn log nên correlate:

- PCAP.
- Suricata.
- Zeek.
- Firewall.
- NetFlow.
- EDR network telemetry.

---

# 22. Threat Hunting

Hunt theo:

- SYN count cao.
- Half-open connections.
- `tcp.flags == 0x000`.
- FIN-only packets.
- FIN/PSH/URG packets.
- ACK không thuộc established flow.
- One source → many ports.
- One source → many hosts.
- Repeated RST/ACK responses.

---

# 23. Incident Response Checklist

```text
[ ] Xác định source IP/MAC.
[ ] Xác định target hosts và ports.
[ ] Xác định scan type theo TCP flags.
[ ] Kiểm tra scan có đến từ approved scanner không.
[ ] Correlate firewall/IDS/NetFlow.
[ ] Triage source endpoint nếu nội bộ.
[ ] Xác định scan chỉ reconnaissance hay có exploitation.
[ ] Block/isolate nếu malicious.
[ ] Thu PCAP và preserve evidence.
```

---

# 24. Hardening

- Stateful firewall.
- IDS/IPS scan detection.
- Rate limiting.
- Network segmentation.
- Restrict management ports.
- Egress filtering.
- NAC/802.1X.
- Asset inventory cho approved scanners.
- Alert khi user VLAN quét server VLAN.

---

# 25. Mapping với CDSA

## Security Operations & Monitoring

- Theo dõi TCP flag anomalies.
- Baseline approved scanners.
- Correlate RST/SYN/ACK patterns.

## Incident Response & Forensics

- Xác định scan timeline.
- Xác định target ports.
- Phân tích PCAP và endpoint source.

## Threat Hunting

- Hunt half-open sessions.
- Hunt unusual flag combinations.
- Hunt one-source-to-many-ports.
- Hunt ACK/FIN/NULL/Xmas packets.

## Detection Engineering

- Suricata rules.
- Zeek scan framework.
- SIEM threshold/correlation.
- False-positive tuning theo scanner inventory.

---

# 26. Lab Setup

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
1. Mở từng PCAP.
2. Lọc TCP flags tương ứng.
3. Xác định source, target và ports.
4. So sánh response của open/closed ports.
5. Viết detection rule.
6. Tune false positives.
```

PCAP liên quan:

```text
nmap_syn_scan.pcapng
nmap_null_scan.pcapng
nmap_ack_scan.pcapng
nmap_fin_scan.pcapng
nmap_xmas_scan.pcapng
```

---

# Key Takeaway

```text
TCP scan thường lộ qua flag pattern bất thường và one-host-to-many-ports.
SYN scan tạo nhiều half-open connections.
NULL không đặt flag, FIN chỉ đặt FIN, Xmas thường đặt FIN/PSH/URG.
ACK scan chủ yếu được dùng để đánh giá firewall filtering.
Detection hiệu quả cần correlate TCP flags, destination-port fan-out và session state.
```

TCP Handshake Abnormalities là chủ đề nền tảng của CDSA vì kết hợp packet analysis, scan detection, IDS/IPS tuning, threat hunting và incident response.

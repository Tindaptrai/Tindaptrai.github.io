---
layout: post
title: "Intro to Network Traffic Analysis"
date: 2026-06-24 23:53:45 +0700
categories: [SOC, Network Traffic Analysis]
tags: [soc, network-traffic-analysis, nta, tcpdump, wireshark, tshark, bpf, pcap, ids, ips, blue-team]
description: "Tóm tắt Network Traffic Analysis: định nghĩa NTA, use cases, kỹ năng cần có, công cụ phân tích traffic, BPF syntax và workflow Ingest Traffic, Reduce Noise, Analyze, Detect/Alert, Fix/Monitor."
toc: true
---

# Intro to Network Traffic Analysis

**Network Traffic Analysis (NTA)** là quá trình kiểm tra, phân tích và hiểu lưu lượng mạng để xác định hành vi bình thường, phát hiện bất thường, điều tra threat và cải thiện visibility trong hệ thống.

Nói ngắn gọn:

```text
NTA = nhìn vào network traffic để hiểu ai đang nói chuyện với ai, qua port/protocol nào, có bình thường không.
```

---

## Timeline cập nhật

```text
Created at: 2026-06-24 23:53:45 +0700
Timezone: Asia/Ho_Chi_Minh
Topic: Network Traffic Analysis
Focus: NTA workflow, skills, tools, BPF syntax, traffic analysis mindset
```

---

# 1. Network Traffic Analysis là gì?

Network Traffic Analysis là việc phân tích traffic để:

- Xác định các port/protocol thường dùng.
- Xây dựng baseline cho môi trường.
- Monitor và phản ứng với threat.
- Phát hiện traffic bất thường.
- Điều tra incident trong quá khứ.
- Hỗ trợ threat hunting.
- Kiểm tra lỗi network/protocol.
- Tăng visibility vào hệ thống mạng.

---

# 2. Vì sao NTA quan trọng?

Attacker muốn xâm nhập hoặc duy trì foothold thì gần như chắc chắn phải giao tiếp với hạ tầng mạng.

Ví dụ:

- C2 communication.
- Lateral movement.
- Port scanning.
- Data exfiltration.
- Malware download.
- Credential relay.
- Beaconing.
- Exploit delivery.

Nếu ta hiểu traffic bình thường trong môi trường, ta có thể phát hiện traffic bất thường nhanh hơn.

Ví dụ:

```text
Nếu network gần như không bao giờ dùng một port lạ,
nhưng đột nhiên có nhiều packet đến port đó,
đây có thể là dấu hiệu port scan hoặc service discovery.
```

---

# 3. Use cases thường gặp của NTA

NTA thường được dùng để:

- Thu thập real-time traffic để phân tích threat.
- Xây dựng baseline network communication.
- Phát hiện non-standard ports.
- Phát hiện suspicious hosts.
- Phân tích HTTP errors.
- Phân tích TCP problems.
- Phát hiện protocol misconfiguration.
- Phát hiện malware trên wire.
- Phát hiện ransomware/exploit traffic.
- Điều tra incident cũ bằng PCAP.
- Hỗ trợ threat hunting.

---

# 4. Kỹ năng cần có

## 4.1. TCP/IP Stack và OSI Model

Cần hiểu cách traffic đi qua các layer.

Ví dụ:

```text
Application → Transport → Network → Data Link → Physical
```

Nếu hiểu layer, ta sẽ biết nên nhìn protocol/field nào khi phân tích.

---

## 4.2. Basic Network Concepts

Cần hiểu:

- Switching.
- Routing.
- VLAN.
- Broadcast domain.
- Subnet.
- Gateway.
- NAT.
- ARP.
- DNS.
- TCP/UDP.

Ví dụ: nếu capture ở backbone link, traffic sẽ khác rất nhiều so với capture ở office switch.

---

## 4.3. Common Ports and Protocols

Cần nhận diện nhanh port/protocol phổ biến:

| Port | Protocol | Ý nghĩa |
|---|---|---|
| 20/21 | FTP | File transfer |
| 22 | SSH | Remote shell |
| 25 | SMTP | Mail |
| 53 | DNS | Name resolution |
| 80 | HTTP | Web |
| 443 | HTTPS | Encrypted web |
| 445 | SMB | Windows file sharing |
| 3389 | RDP | Remote desktop |

---

## 4.4. TCP và UDP

TCP:

```text
Connection-oriented
Reliable
Stream-oriented
Dễ follow conversation
```

UDP:

```text
Connectionless
Nhanh
Không đảm bảo completeness
Khó reconstruct hơn TCP
```

---

## 4.5. Protocol Encapsulation

Mỗi layer sẽ encapsulate dữ liệu của layer trên.

Ví dụ:

```text
Ethernet frame
  → IP packet
    → TCP segment
      → HTTP data
```

Khi phân tích packet, hiểu encapsulation giúp đi từ frame đến payload nhanh hơn.

---

# 5. Công cụ phân tích traffic

| Tool | Mô tả |
|---|---|
| tcpdump | CLI tool dùng LibPcap để capture/read packet |
| TShark | CLI version của Wireshark |
| Wireshark | GUI packet analyzer mạnh, dissect nhiều protocol |
| NGrep | Grep cho network traffic, hỗ trợ regex và BPF |
| tcpick | Theo dõi và reassemble TCP streams |
| Network Taps | Thiết bị copy traffic để phân tích |
| SPAN Ports | Mirror traffic từ switch/router đến collector |
| Elastic Stack | Ingest, search, visualize network data |
| SIEM | Splunk/Elastic dùng để alert, investigate, visualize |

---

# 6. Network Tap và SPAN Port

## 6.1. Network Tap

Network Tap là thiết bị copy traffic từ một link mạng để gửi sang thiết bị phân tích.

Có thể hoạt động:

- Inline.
- Out-of-band.
- Passive.
- Active.

Ưu điểm:

- Visibility tốt.
- Ít phụ thuộc switch config.
- Phù hợp production monitoring.

---

## 6.2. SPAN Port / Port Mirroring

SPAN port copy frame từ switch/router sang port giám sát.

Dùng khi muốn gửi traffic về:

- IDS/IPS.
- SIEM collector.
- Packet capture server.
- NDR/NTA platform.

Lưu ý: nếu SPAN cấu hình sai hoặc quá tải, có thể mất packet.

---

# 7. BPF Syntax

**BPF** là viết tắt của:

```text
Berkeley Packet Filter
```

BPF giúp filter packet ở tầng thấp, thường dùng trong tcpdump, Wireshark capture filter, tshark...

Ví dụ:

```bash
tcpdump host 192.168.1.10
```

```bash
tcpdump port 80
```

```bash
tcpdump src host 10.10.10.5 and dst port 443
```

```bash
tcpdump net 192.168.1.0/24
```

```bash
tcpdump tcp
```

```bash
tcpdump udp port 53
```

---

# 8. Performing Network Traffic Analysis

Muốn phân tích traffic, tối thiểu cần capture được đúng network segment.

Trong switched network, traffic không tự động đi đến mọi port. Nếu muốn thấy traffic của VLAN nào, capture device cần nằm đúng segment hoặc dùng:

- Network tap.
- SPAN port.
- Port mirroring.
- Router/switch capture.
- Sensor đặt đúng vị trí.

---

# 9. NTA Workflow

Workflow cơ bản:

```text
Problem
  ↓
1. Ingest Traffic
  ↓
2. Reduce Noise by Filtering
  ↓
3. Analyze and Explore
  ↓
4. Detect and Alert
  ↓
5. Fix and Monitor
```

---

## 9.1. Step 1: Ingest Traffic

Bắt đầu bằng việc capture traffic.

Có thể dùng:

- tcpdump.
- Wireshark.
- TShark.
- Network tap.
- SPAN port.
- IDS/IPS sensor.
- SIEM/NDR collector.

Nếu đã biết cần tìm gì, có thể dùng capture filter từ đầu để giảm dữ liệu.

Ví dụ:

```bash
tcpdump -i eth0 host 192.168.1.10 -w capture.pcap
```

---

## 9.2. Step 2: Reduce Noise by Filtering

Traffic production rất nhiễu.

Cần filter bớt:

- Broadcast traffic.
- Multicast traffic.
- Known benign services.
- Known update traffic.
- Internal noise.
- Repeated expected traffic.

Ví dụ filter:

```text
Loại broadcast/multicast
Chỉ xem host nghi vấn
Chỉ xem port/protocol liên quan
Chỉ xem time window liên quan incident
```

---

## 9.3. Step 3: Analyze and Explore

Sau khi giảm noise, bắt đầu phân tích sâu.

Câu hỏi cần đặt:

```text
Traffic có encrypted hay plaintext?
Traffic này có nên encrypted không?
Host này có nên nói chuyện với host kia không?
Port/protocol này có bình thường trong môi trường không?
Có user nào access tài nguyên không nên access không?
Có TCP flags bất thường không?
Có retry/error/reset nhiều không?
Có DNS query lạ không?
Có beaconing pattern không?
```

---

## 9.4. Step 4: Detect and Alert

Sau khi phân tích, xác định traffic là benign hay suspicious/malicious.

Có thể dùng thêm:

- IDS.
- IPS.
- Zeek.
- Suricata.
- Snort.
- SIEM.
- NDR.
- Threat intel.
- Signatures.
- Heuristics.

Ví dụ detection:

```text
SYN scan
DNS beaconing
HTTP C2 pattern
SMB lateral movement
Large outbound data transfer
Non-standard port communication
```

---

## 9.5. Step 5: Fix and Monitor

Dù không nằm trong vòng lặp chính của sơ đồ, fix và monitor vẫn rất quan trọng.

Sau khi xử lý:

- Block IP/domain nếu cần.
- Fix misconfiguration.
- Update firewall rule.
- Tune IDS/IPS signature.
- Isolate host.
- Patch service.
- Monitor tiếp để xác nhận vấn đề đã hết.

---

# 10. Ví dụ tư duy phân tích

## Tình huống: nghi port scan

Dấu hiệu:

```text
Một host gửi nhiều SYN packet đến nhiều port khác nhau
Nhiều connection không hoàn tất TCP handshake
Traffic đến các port hiếm dùng trong môi trường
```

Hướng phân tích:

```text
Xác định source IP
Xác định destination IP/port
Đếm số port bị scan
Kiểm tra TCP flags
Kiểm tra time window
Correlate với IDS alert hoặc firewall logs
```

---

## Tình huống: nghi C2 beaconing

Dấu hiệu:

```text
Host kết nối ra ngoài theo interval đều
Destination domain/IP lạ
Traffic HTTPS nhưng SNI/domain bất thường
User-agent lạ
Dung lượng nhỏ nhưng lặp lại
```

Hướng phân tích:

```text
Xem DNS query
Xem connection frequency
Xem JA3/SNI nếu có
Xem process tạo connection nếu có EDR/Sysmon
So với threat intelligence
```

---

# 11. Bảng tóm tắt workflow

| Step | Mục tiêu | Output |
|---|---|---|
| Ingest Traffic | Thu thập traffic | PCAP/log/network telemetry |
| Reduce Noise | Lọc bớt traffic không liên quan | Dataset nhỏ hơn |
| Analyze and Explore | Phân tích host/protocol/port/payload | Finding/context |
| Detect and Alert | Xác định benign/malicious | Alert/rule/signature |
| Fix and Monitor | Khắc phục và theo dõi | Confirm issue resolved |

---

# 12. Key Takeaway

Network Traffic Analysis giúp defender nhìn thấy hành vi attacker trên network.

Điểm quan trọng nhất:

```text
Không chỉ capture packet,
mà phải hiểu baseline, lọc noise, phân tích context và chuyển finding thành detection.
```

Một analyst làm NTA tốt cần hiểu TCP/IP, OSI, port/protocol, TCP/UDP, encapsulation, BPF filter và biết dùng công cụ như tcpdump, Wireshark, TShark, Zeek/SIEM để biến traffic thành insight điều tra.

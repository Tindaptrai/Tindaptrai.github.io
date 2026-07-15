---
layout: post
title: "IP Time-to-Live Attacks"
date: 2026-07-14 23:21:58 +0700
categories: ["Intermediate Network Traffic Analysis", "Layer 2 & Network Attacks"]
tags: [ip-ttl, time-to-live, ttl-evasion, firewall-evasion, ids-evasion, ips-evasion, port-scanning, icmp-time-exceeded, wireshark, tshark, suricata, zeek, threat-hunting, incident-response, cdsa]
description: "Phân tích IP Time-to-Live attacks: TTL manipulation, firewall/IDS evasion, ICMP Time Exceeded, Wireshark filters, detection logic và hardening."
toc: true
---

# IP Time-to-Live Attacks

Bài này thuộc module **Intermediate Network Traffic Analysis**, tập trung vào việc attacker cố tình chỉnh giá trị **TTL** thấp để làm packet hết hạn trước khi tới một thiết bị kiểm soát hoặc điểm phân tích.

```text
Mục tiêu: hiểu cơ chế TTL, nhận diện TTL bất thường trong PCAP,
phát hiện port scan kết hợp TTL manipulation và xây rule phòng thủ.
```

> Chỉ thực hành trong HTB, lab hoặc hệ thống có ủy quyền rõ ràng. Không tạo traffic evasion hoặc port scan ngoài phạm vi được phép.

---

## Timeline cập nhật

```text
Created at: 2026-07-14 23:21:58 +0700
Timezone: Asia/Ho_Chi_Minh
Module: Intermediate Network Traffic Analysis
Section: IP Time-to-Live Attacks
Focus: TTL manipulation, firewall/IDS evasion, ICMP Time Exceeded, packet analysis
```

---

# 1. TTL là gì?

**Time-to-Live (TTL)** là một field trong IPv4 header dùng để giới hạn số hop mà packet được phép đi qua.

Mỗi router sẽ giảm TTL đi `1`.

Ví dụ:

```text
Attacker gửi packet với TTL = 3

Router 1 → TTL = 2
Router 2 → TTL = 1
Router 3 → TTL = 0 → packet bị drop
```

Khi TTL về `0`, router loại bỏ packet.

---

# 2. Vì sao TTL tồn tại?

TTL giúp ngăn packet chạy vô hạn trong routing loop.

Không có TTL:

```text
Packet có thể quay vòng liên tục
→ tiêu tốn bandwidth
→ làm đầy router queue
→ gây mất ổn định mạng
```

---

# 3. TTL Evasion hoạt động như thế nào?

Attacker cố tình đặt TTL rất thấp:

```text
TTL = 1
TTL = 2
TTL = 3
```

Mục tiêu có thể là:

- Làm packet hết hạn trước firewall/IDS/IPS.
- Tạo khác biệt giữa đường đi tới sensor và đường đi tới target.
- Làm một số security control không thấy đầy đủ traffic.
- Kết hợp với port scanning để dò vị trí thiết bị lọc.
- Kết hợp với decoy hoặc fragmentation.

---

# 4. Attack Flow

```text
1. Attacker gửi SYN packet với TTL thấp.
2. Packet đi qua router.
3. TTL giảm dần.
4. Packet hết hạn tại một hop trung gian.
5. Router gửi ICMP Time Exceeded về source.
6. Attacker điều chỉnh TTL để suy ra vị trí firewall/target.
```

Đây cũng là nguyên lý cơ bản được traceroute sử dụng hợp lệ.

---

# 5. ICMP Time Exceeded

Khi packet hết TTL, router thường trả:

```text
ICMP Type 11 - Time Exceeded
```

Wireshark filter:

```wireshark
icmp.type == 11
```

Theo source nghi vấn:

```wireshark
ip.dst == 192.168.10.5 && icmp.type == 11
```

Nhiều ICMP Time Exceeded liên tiếp tới một host có thể cho thấy:

- Traceroute hợp lệ.
- Network troubleshooting.
- TTL-based scanning/evasion.
- Routing loop.

Cần correlate với TCP/UDP activity.

---

# 6. Dấu hiệu TTL Manipulation

Các dấu hiệu phổ biến:

- TTL cực thấp như `1`, `2`, `3`.
- Một source gửi nhiều SYN packets với TTL thay đổi.
- Nhiều ICMP Time Exceeded trả về.
- Port scan xuất hiện cùng TTL bất thường.
- Cùng flow nhưng TTL không nhất quán.
- SYN tới nhiều ports với cùng packet length.
- Một số packets tới được target, một số hết hạn trước đó.

---

# 7. Phân tích trong Wireshark

Mở PCAP:

```bash
wireshark ip_ttl.pcapng
```

Lọc TTL thấp:

```wireshark
ip.ttl <= 5
```

Lọc SYN có TTL thấp:

```wireshark
tcp.flags.syn == 1 &&
tcp.flags.ack == 0 &&
ip.ttl <= 5
```

Theo source:

```wireshark
ip.src == 192.168.10.5 &&
ip.ttl <= 5
```

ICMP Time Exceeded:

```wireshark
icmp.type == 11
```

---

# 8. Port Scan kết hợp TTL

Một pattern đáng ngờ:

```text
192.168.10.5 → 192.168.10.1:80  SYN TTL=3
192.168.10.5 → 192.168.10.1:443 SYN TTL=3
192.168.10.5 → 192.168.10.1:22  SYN TTL=3
```

Nếu target vẫn trả SYN/ACK trên một port hợp lệ, attacker có thể đã tìm ra TTL đủ để vượt qua một hop/filter.

Dấu hiệu:

```text
Nhiều SYN
Một target
Nhiều destination ports
TTL thấp hoặc thay đổi có chủ đích
```

---

# 9. TTL Baseline

TTL bình thường phụ thuộc OS và khoảng cách hop.

Giá trị khởi tạo phổ biến:

| Hệ điều hành/thiết bị | TTL thường gặp |
|---|---:|
| Linux | 64 |
| Windows | 128 |
| Cisco/Network devices | 255 |

TTL quan sát được sẽ giảm theo số hop.

Ví dụ:

```text
Windows TTL khởi tạo 128
Đi qua 3 routers
TTL quan sát ≈ 125
```

Do đó, packet inbound với TTL `2` hoặc `3` thường rất bất thường trừ khi sensor nằm cực gần source.

---

# 10. TTL Inconsistency

Trong cùng một host/flow, cần chú ý:

```text
TTL thay đổi mạnh trong thời gian ngắn
```

Ví dụ:

```text
Packet 1 TTL=128
Packet 2 TTL=3
Packet 3 TTL=127
Packet 4 TTL=2
```

Điều này có thể chỉ ra:

- Packet crafting.
- Spoofed source.
- Multiple paths.
- Load balancing.
- NAT/proxy differences.
- Scanner điều chỉnh TTL.

Cần tránh kết luận chỉ dựa trên TTL; phải correlate thêm flow và topology.

---

# 11. Tshark Hunt

Liệt kê packets TTL thấp:

```bash
tshark -r ip_ttl.pcapng   -Y "ip.ttl <= 5"   -T fields   -e frame.number   -e ip.src   -e ip.dst   -e ip.ttl   -e ip.proto
```

TTL thấp kết hợp TCP SYN:

```bash
tshark -r ip_ttl.pcapng   -Y "tcp.flags.syn == 1 && tcp.flags.ack == 0 && ip.ttl <= 5"   -T fields   -e ip.src   -e ip.dst   -e tcp.dstport   -e ip.ttl
```

Đếm TTL theo source:

```bash
tshark -r ip_ttl.pcapng   -Y "ip"   -T fields -e ip.src -e ip.ttl |
sort | uniq -c | sort -nr
```

---

# 12. Detection Logic

Rule gợi ý:

```text
Một source gửi nhiều SYN packets
tới nhiều destination ports
với TTL <= 5
trong một khoảng thời gian ngắn.
```

Feature nên theo dõi:

- Source IP.
- Destination IP.
- Unique destination ports.
- Minimum TTL.
- Maximum TTL.
- TTL variance.
- ICMP Time Exceeded count.
- SYN count.
- Time window.

---

# 13. Suricata Detection Idea

Ví dụ concept rule:

```text
alert khi:
TCP SYN
TTL < 5
tới internal protected network
```

Pseudo-rule:

```suricata
alert tcp any any -> $HOME_NET any (
  msg:"Possible TTL-based evasion or scan";
  flags:S;
  ttl:<5;
  threshold:type both, track by_src, count 10, seconds 10;
  sid:1000101;
  rev:1;
)
```

Cần test syntax theo phiên bản Suricata đang dùng và tune threshold theo môi trường.

---

# 14. Zeek Detection Idea

Zeek có thể hỗ trợ:

- Theo dõi TTL bất thường.
- Correlate ICMP Time Exceeded.
- Phát hiện scan.
- So sánh TTL theo source.
- Ghi notice khi TTL variance quá lớn.

Nguồn log:

```text
conn.log
weird.log
notice.log
```

---

# 15. Firewall/IDS Hardening

- Drop packet có TTL thấp bất hợp lý tại perimeter.
- Bật anti-spoofing.
- Dùng stateful inspection.
- Theo dõi ICMP Time Exceeded rate.
- Reconstruct flow thay vì xử lý từng packet độc lập.
- Baseline TTL theo network segment.
- Normalize traffic tại edge.
- Correlate TTL với hop count/topology.

---

# 16. Lưu ý khi chặn TTL thấp

Không nên áp một threshold cứng cho toàn bộ mạng mà không test.

Có thể gây false positive với:

- Traceroute.
- Network diagnostics.
- MPLS/overlay.
- Tunnels.
- Asymmetric routing.
- Thiết bị ở rất gần sensor.
- Một số appliance đặc thù.

Nên áp theo:

```text
Interface
Zone
Source subnet
Expected hop count
Asset role
```

---

# 17. SIEM Correlation

Ví dụ chuỗi detection:

```text
IDS:
SYN scan với TTL <= 5
        ↓
Firewall:
Nhiều drop do TTL/hop-limit bất thường
        ↓
ICMP:
Time Exceeded trả về source
        ↓
SOC:
Xác định scanner và target ports
```

Nguồn dữ liệu:

- Suricata.
- Zeek.
- Firewall logs.
- NetFlow.
- PCAP.
- Router ICMP logs.

---

# 18. Threat Hunting

Hunt theo:

- `ip.ttl <= 5`.
- TTL variation cao từ một source.
- SYN fan-out với TTL thấp.
- Nhiều ICMP Type 11 về cùng host.
- TTL thấp kết hợp fragmentation.
- TTL thấp kết hợp decoy scanning.
- User endpoint thực hiện traceroute/scan ngoài baseline.

---

# 19. Incident Response Checklist

```text
[ ] Xác định source và target.
[ ] Xác định TTL min/max.
[ ] Kiểm tra destination ports.
[ ] Kiểm tra ICMP Time Exceeded.
[ ] Correlate firewall/IDS/NetFlow.
[ ] Xác định hoạt động hợp lệ hay scan/evasion.
[ ] Triage source host nếu là endpoint nội bộ.
[ ] Block/rate-limit nếu malicious.
[ ] Thu PCAP và preserve evidence.
```

---

# 20. Mapping với CDSA

## Security Operations & Monitoring

- Baseline TTL theo segment.
- Monitor low-TTL SYN scans.
- Correlate ICMP Type 11 với firewall/IDS.

## Incident Response & Forensics

- Xác định scan timeline.
- Tìm TTL progression.
- Xác định packet nào tới được target.

## Threat Hunting

- Hunt TTL <= 5.
- Hunt TTL variance.
- Hunt low-TTL packets tới nhiều ports.
- Hunt ICMP Time Exceeded spikes.

## Detection Engineering

- Suricata rule cho low TTL SYN.
- Zeek script cho TTL anomaly.
- SIEM correlation với topology context.
- Tune theo interface và expected hop count.

---

# 21. Lab Setup

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
1. Mở ip_ttl.pcapng.
2. Lọc ip.ttl <= 5.
3. Xác định source, target và ports.
4. Lọc ICMP Type 11.
5. So sánh TTL với traffic bình thường.
6. Viết detection.
7. Tune false positives.
```

---

# Key Takeaway

```text
TTL giảm một đơn vị qua mỗi router và packet bị drop khi TTL về 0.
Attacker có thể cố tình đặt TTL thấp để dò đường đi hoặc né thiết bị kiểm soát.
Dấu hiệu mạnh là SYN scan tới nhiều ports với TTL thấp và ICMP Time Exceeded.
Detection cần dùng topology context, không nên chỉ dựa vào một threshold TTL cố định.
```

IP Time-to-Live Attacks là chủ đề quan trọng của CDSA vì yêu cầu kết hợp IPv4 header analysis, IDS/IPS behavior, firewall policy, topology awareness và threat hunting.

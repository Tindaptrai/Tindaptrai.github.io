---
layout: post
title: "TCP Reset and Session Hijacking"
date: 2026-07-14 23:36:34 +0700
categories: ["Intermediate Network Traffic Analysis"]
tags: [tcp, tcp-reset, rst-injection, session-hijacking, sequence-number, arp-poisoning, wireshark, tshark, suricata, zeek, threat-hunting, incident-response, cdsa]
description: "Phân tích TCP RST injection và TCP session hijacking: dấu hiệu RST bất thường, IP-MAC mismatch, sequence/ACK anomalies, retransmission và quy trình phát hiện ứng phó."
toc: true
---

# TCP Reset and Session Hijacking

Bài này thuộc module **Intermediate Network Traffic Analysis**, tập trung vào hai kỹ thuật:

- **TCP RST injection** để chấm dứt kết nối.
- **TCP session hijacking** để chiếm luồng TCP đang hoạt động.

```text
Mục tiêu: nhận diện RST bất thường, IP-MAC mismatch,
sequence/ACK anomalies, retransmission và dấu hiệu MITM liên quan.
```

> Chỉ thực hành trong HTB, lab hoặc hệ thống có ủy quyền rõ ràng. Không chèn packet hoặc chiếm session ngoài phạm vi được phép.

---

## Timeline cập nhật

```text
Created at: 2026-07-14 23:36:34 +0700
Timezone: Asia/Ho_Chi_Minh
Module: Intermediate Network Traffic Analysis
Section: TCP Reset and Session Hijacking
Focus: RST injection, sequence prediction, ACK suppression, ARP poisoning, packet analysis
```

---

# 1. TCP Reset là gì?

TCP sử dụng cờ `RST` để kết thúc kết nối ngay lập tức.

RST hợp lệ có thể xuất hiện khi:

- Kết nối tới port đóng.
- Ứng dụng bị dừng đột ngột.
- Host không còn state của session.
- Firewall hoặc proxy chủ động reset flow.

Attacker có thể giả mạo RST để ép hai host đóng kết nối.

---

# 2. TCP RST Injection

Một cuộc tấn công RST injection thường cần:

```text
Source IP giả giống một endpoint hợp lệ.
Destination IP/port khớp session đang hoạt động.
TCP RST flag được đặt.
Sequence number nằm trong receive window.
```

Flow:

```text
Victim A ↔ Victim B đang có TCP session
        ↓
Attacker quan sát hoặc suy đoán flow
        ↓
Attacker gửi RST giả
        ↓
Một hoặc cả hai endpoint đóng session
```

---

# 3. Mục tiêu của RST Injection

- Gây Denial-of-Service.
- Ngắt kết nối quản trị.
- Chấm dứt ứng dụng.
- Ép client reconnect qua đường khác.
- Hỗ trợ MITM.
- Che giấu hành vi sau khi đã chèn dữ liệu.

---

# 4. Dấu hiệu TCP RST Attack

- Nhiều RST tới cùng destination port.
- RST xuất hiện giữa session đang hoạt động bình thường.
- RST không phù hợp sequence/ACK state.
- Source IP hợp lệ nhưng source MAC không khớp inventory.
- RST burst từ một MAC.
- Ngay sau RST xuất hiện retransmission hoặc reconnect.
- Endpoint báo connection reset bất thường.

---

# 5. Xác minh bằng IP-MAC Correlation

Ví dụ asset inventory:

```text
192.168.10.4 → aa:aa:aa:aa:aa:aa
```

Nhưng packet RST có:

```text
Source IP: 192.168.10.4
Source MAC: 08:00:27:53:0c:ba
```

Đây là dấu hiệu giả mạo rõ ràng.

Cần correlate với:

- DHCP lease.
- Switch CAM table.
- ARP cache.
- CMDB.
- EDR network telemetry.

---

# 6. Wireshark Filters cho RST

Tất cả RST:

```wireshark
tcp.flags.reset == 1
```

Theo source:

```wireshark
ip.src == 192.168.10.4 &&
tcp.flags.reset == 1
```

Theo destination port:

```wireshark
tcp.dstport == 80 &&
tcp.flags.reset == 1
```

RST giữa một cặp host:

```wireshark
ip.addr == 192.168.10.4 &&
ip.addr == 192.168.10.1 &&
tcp.flags.reset == 1
```

---

# 7. Phân biệt RST hợp lệ và RST giả

## RST hợp lệ

- Xuất hiện sau SYN tới port đóng.
- Sequence/ACK hợp lý.
- MAC khớp endpoint.
- Không có retransmission kéo dài.
- Endpoint thực sự không còn service/session.

## RST đáng ngờ

- Xuất hiện giữa established session.
- MAC không khớp source IP.
- TTL khác đáng kể so với traffic trước đó.
- Sequence number bất thường.
- Nhiều RST lặp lại.
- Flow tiếp tục retransmit sau RST.

---

# 8. TCP Session Hijacking

Trong session hijacking, attacker cố chèn packet vào một TCP flow đang hoạt động.

Attacker thường cần:

- Quan sát traffic.
- Biết source/destination IP.
- Biết source/destination port.
- Theo dõi sequence và acknowledgement numbers.
- Chèn packet đúng receive window.
- Ngăn endpoint thật gửi ACK làm lộ sự khác biệt.

---

# 9. Sequence Number và ACK

TCP dùng:

```text
Sequence Number
Acknowledgement Number
Receive Window
```

để đảm bảo dữ liệu đúng thứ tự.

Attacker muốn chèn dữ liệu phải tạo packet có sequence number mà receiver chấp nhận.

Nếu sai:

- Receiver gửi duplicate ACK.
- Packet bị drop.
- Retransmission tăng.
- Session bị desynchronize.

---

# 10. Hijacking Flow

```text
1. Attacker trở thành MITM hoặc sniff được session.
2. Theo dõi sequence/ACK.
3. Chèn packet giả mạo source của victim.
4. Làm chậm hoặc chặn ACK của victim thật.
5. Receiver chấp nhận dữ liệu giả.
6. Hai endpoint thật bị desynchronize.
```

ARP poisoning thường được dùng để tạo vị trí MITM.

---

# 11. Dấu hiệu TCP Hijacking

- Duplicate ACK tăng đột biến.
- Out-of-order packets.
- Retransmissions.
- ACKed unseen segment.
- Previous segment not captured.
- Sequence jumps.
- Hai MAC khác nhau dùng cùng source IP.
- Payload bất ngờ trong established flow.
- TTL/IP ID thay đổi trong cùng flow.
- Một phía gửi RST sau khi session desynchronize.

---

# 12. Wireshark Expert Indicators

Các cảnh báo hữu ích:

```text
TCP Retransmission
TCP Fast Retransmission
TCP Out-Of-Order
TCP Dup ACK
TCP ACKed unseen segment
TCP Previous segment not captured
TCP Spurious Retransmission
```

Filter tổng hợp:

```wireshark
tcp.analysis.flags
```

Chỉ retransmission:

```wireshark
tcp.analysis.retransmission ||
tcp.analysis.fast_retransmission
```

Chỉ duplicate ACK:

```wireshark
tcp.analysis.duplicate_ack
```

---

# 13. Theo dõi một TCP Stream

Chọn packet trong flow:

```text
Right click
→ Follow
→ TCP Stream
```

Sau đó kiểm tra:

- Có payload chen ngang không.
- Sequence có nhảy bất thường không.
- Có hai kiểu TTL/IP ID khác nhau không.
- Có RST ngay sau dữ liệu lạ không.
- Session có bị tách thành hai luồng không.

Filter theo stream:

```wireshark
tcp.stream eq 3
```

---

# 14. So sánh TTL và IP ID

Nếu cùng một source IP nhưng:

```text
TTL thường: 128
Packet nghi vấn: TTL 64
```

hoặc IP ID pattern khác hoàn toàn, có thể có host khác giả mạo source.

Không nên kết luận chỉ dựa vào TTL, nhưng đây là chỉ báo hỗ trợ tốt khi kết hợp với MAC mismatch.

---

# 15. Tshark Hunt

Đếm RST theo source:

```bash
tshark -r RST_Attack.pcapng   -Y "tcp.flags.reset == 1"   -T fields -e ip.src |
sort | uniq -c | sort -nr
```

Liệt kê RST và MAC:

```bash
tshark -r RST_Attack.pcapng   -Y "tcp.flags.reset == 1"   -T fields   -e frame.number   -e eth.src   -e ip.src   -e ip.dst   -e tcp.srcport   -e tcp.dstport   -e tcp.seq   -e tcp.ack
```

Tìm TCP anomalies:

```bash
tshark -r TCP-hijacking.pcap   -Y "tcp.analysis.flags"   -T fields   -e frame.number   -e ip.src   -e ip.dst   -e tcp.stream   -e _ws.col.Info
```

---

# 16. Detection Logic cho RST Injection

```text
Established TCP session
+ RST xuất hiện đột ngột
+ source MAC/TTL không khớp baseline
+ nhiều retransmissions hoặc reconnect
```

Feature nên theo dõi:

- RST count.
- Established-state context.
- Source MAC.
- TTL variance.
- Sequence validity.
- Destination service.
- RST rate theo source.

---

# 17. Detection Logic cho Session Hijacking

```text
Established session
+ sequence/ACK anomalies
+ duplicate ACK/retransmission
+ source identity inconsistency
+ payload bất thường
```

High-confidence pattern:

```text
Cùng source IP
nhưng hai source MAC
và sequence behavior khác nhau
trong cùng TCP stream.
```

---

# 18. Suricata Detection Ideas

RST burst:

```suricata
alert tcp any any -> $HOME_NET any (
  msg:"Possible TCP RST injection burst";
  flags:R;
  threshold:type both, track by_src, count 20, seconds 10;
  sid:1000301;
  rev:1;
)
```

Suricata cũng có thể theo dõi:

- Stream anomalies.
- Invalid sequence.
- Midstream pickup.
- Excessive retransmission.
- TCP evasion.

Cần test và tune rule trong lab.

---

# 19. Zeek Detection Ideas

Zeek có thể hỗ trợ qua:

```text
conn.log
weird.log
notice.log
```

Các dấu hiệu:

- Connection reset anomalies.
- Inconsistent TCP state.
- Repeated connection failures.
- Excessive resets.
- Sequence inconsistencies.
- One IP từ nhiều MAC nếu có L2 visibility.

---

# 20. SIEM Correlation

Ví dụ:

```text
Network sensor:
RST burst trong established session
        ↓
Switch telemetry:
Source MAC không thuộc source IP
        ↓
EDR:
Không có process hợp lệ tạo RST
        ↓
SOC:
Nghi TCP RST injection
```

Session hijacking:

```text
ARP anomaly
        ↓
TCP sequence/ACK anomaly
        ↓
Payload bất thường
        ↓
Session reset/reconnect
```

---

# 21. Incident Response Checklist

```text
[ ] Xác định session bị ảnh hưởng.
[ ] Xác định source/destination IP và ports.
[ ] Kiểm tra source MAC.
[ ] Kiểm tra TTL, IP ID và sequence number.
[ ] Review retransmission, duplicate ACK và out-of-order.
[ ] Correlate ARP/DHCP/switch CAM table.
[ ] Isolate nghi phạm tại switch port.
[ ] Thu PCAP và endpoint evidence.
[ ] Kiểm tra credential/session exposure.
[ ] Reset session/token nếu cần.
```

---

# 22. Prevention và Hardening

- Dùng TLS/SSH thay plaintext protocols.
- Bật anti-spoofing.
- Dynamic ARP Inspection.
- DHCP Snooping.
- Port Security.
- 802.1X/NAC.
- Network segmentation.
- Stateful firewall.
- IDS/IPS stream reassembly.
- Application-level session protection.
- Short session lifetime và reauthentication.

Encryption không ngăn RST injection hoàn toàn, nhưng giúp giảm nguy cơ attacker đọc hoặc sửa payload.

---

# 23. Mapping với CDSA

## Security Operations & Monitoring

- Monitor RST rate.
- Baseline MAC/IP/TTL.
- Correlate TCP anomalies với ARP/switch telemetry.

## Incident Response & Forensics

- Follow TCP stream.
- Phân tích sequence/ACK.
- Thu PCAP và endpoint process/network evidence.
- Xác định session/token bị ảnh hưởng.

## Threat Hunting

- Hunt RST burst giữa established sessions.
- Hunt duplicate ACK và out-of-order spikes.
- Hunt one-IP-to-many-MAC.
- Hunt TTL/IP-ID inconsistency.

## Detection Engineering

- Suricata rule cho RST burst.
- Zeek weird/notice correlation.
- SIEM correlation L2 + L3 + TCP state.
- Baseline services nhạy cảm.

---

# 24. Lab Setup

Môi trường tối thiểu:

```text
2 endpoint hosts
1 lab attacker/MITM host
1 Wireshark/tshark sensor
Suricata hoặc Zeek
Switch có SPAN port
Splunk/ELK/Wazuh nhận network logs
```

Workflow:

```text
1. Mở RST_Attack.pcapng.
2. Lọc tcp.flags.reset == 1.
3. So sánh IP/MAC/TTL.
4. Mở TCP-hijacking.pcap.
5. Follow TCP Stream.
6. Kiểm tra sequence/ACK anomalies.
7. Viết detection và IR timeline.
```

---

# Key Takeaway

```text
TCP RST injection giả mạo endpoint để chấm dứt session.
MAC mismatch, TTL inconsistency và RST burst là dấu hiệu quan trọng.
TCP hijacking cần sequence/ACK phù hợp và thường kết hợp MITM/ARP poisoning.
Duplicate ACK, retransmission, out-of-order và desynchronization là telemetry chính.
```

TCP Reset and Session Hijacking là chủ đề cốt lõi của CDSA vì yêu cầu kết hợp packet analysis, L2 identity, TCP state tracking, threat hunting và incident response.

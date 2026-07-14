---
layout: post
title: "Strange Telnet & UDP Connections"
date: 2026-07-15 00:33:14 +0700
categories: ["Intermediate Network Traffic Analysis"]
tags: [telnet, udp, tunneling, data-exfiltration, command-and-control, non-standard-port, ipv6, wireshark, tshark, suricata, zeek, splunk, elk, sigma, threat-hunting, incident-response, cdsa]
description: "Phân tích Telnet và UDP bất thường: Telnet port 23, Telnet trên cổng lạ, IPv6 tunneling, UDP exfiltration, Wireshark/Tshark hunting và SIEM detection."
toc: true
---

# Strange Telnet & UDP Connections

Bài này thuộc module **Intermediate Network Traffic Analysis**, tập trung vào việc phát hiện Telnet và UDP bị lạm dụng để tạo command-and-control, tunneling hoặc data exfiltration.

```text
Mục tiêu: nhận diện Telnet plaintext, Telnet trên cổng không chuẩn,
IPv6 communication ngoài baseline và UDP streams mang dữ liệu bất thường.
```

> Chỉ thực hành trong HTB, lab hoặc hệ thống có ủy quyền rõ ràng. Không thiết lập tunnel hoặc exfiltration channel ngoài phạm vi được phép.

---

## Timeline cập nhật

```text
Created at: 2026-07-15 00:33:14 +0700
Timezone: Asia/Ho_Chi_Minh
Module: Intermediate Network Traffic Analysis
Section: Strange Telnet & UDP Connections
Focus: Telnet tunneling, non-standard ports, IPv6 traffic, UDP exfiltration
```

---

# 1. Telnet là gì?

Telnet là giao thức remote terminal được định nghĩa từ rất sớm và thường sử dụng:

```text
TCP/23
```

Telnet cung cấp phiên giao tiếp hai chiều nhưng **không mã hóa dữ liệu**.

Điều này khiến Telnet dễ làm lộ:

- Username.
- Password.
- Command.
- File content.
- Session data.
- Exfiltrated information.

Trong môi trường hiện đại, SSH nên được ưu tiên thay Telnet.

---

# 2. Vì sao Telnet vẫn đáng chú ý?

Telnet có thể vẫn xuất hiện trong:

- Thiết bị mạng cũ.
- Hệ thống OT/ICS.
- Legacy Windows/Unix.
- Embedded devices.
- Thiết bị IoT.
- Lab environments.

Attacker cũng có thể sử dụng Telnet cho:

- Command-and-control.
- Tunneling.
- Data exfiltration.
- Remote shell.
- Persistence trên thiết bị cũ.

---

# 3. Traditional Telnet trên Port 23

PCAP liên quan:

```text
telnet_tunneling_23.pcapng
```

Wireshark filter:

```wireshark
telnet
```

Hoặc:

```wireshark
tcp.port == 23
```

Theo một cặp host:

```wireshark
ip.addr == 192.168.10.5 &&
ip.addr == 192.168.10.7 &&
tcp.port == 23
```

---

# 4. Dấu hiệu Telnet bất thường

- Telnet xuất hiện trong môi trường không cho phép.
- Server hoặc workstation user sử dụng port 23.
- Lượng Telnet traffic kéo dài.
- Command hoặc credential xuất hiện plaintext.
- Một endpoint nội bộ kết nối tới host hiếm gặp.
- Telnet traffic ra Internet.
- Telnet session xuất hiện ngoài giờ.
- Telnet dùng để truyền file hoặc chuỗi dữ liệu dài.

---

# 5. Theo dõi Telnet Stream

Trong Wireshark:

```text
Right click packet
→ Follow
→ TCP Stream
```

Có thể quan sát:

- Login prompt.
- Username/password.
- Shell commands.
- File content.
- C2 messages.
- Exfiltrated text.

Khi làm báo cáo, phải che credential hoặc token thật.

---

# 6. Telnet Có Thể Bị Encode Hoặc Mã Hóa

Telnet thường plaintext, nhưng attacker có thể:

- Base64 encode dữ liệu.
- Hex encode.
- Compress.
- Encrypt ở application layer.
- Chia dữ liệu thành nhiều chunks.

Do đó, không nên kết luận chỉ vì payload không đọc được.

Cần phân tích thêm:

```text
Packet size
Frequency
Destination
Session duration
Entropy
Endpoint process
```

---

# 7. Telnet Trên Cổng Không Chuẩn

PCAP liên quan:

```text
telnet_tunneling_9999.pcapng
```

Attacker có thể chạy Telnet-like protocol trên:

```text
TCP/9999
TCP/4444
TCP/8081
TCP/31337
```

Wireshark có thể không tự nhận diện là Telnet.

Filter:

```wireshark
tcp.port == 9999
```

Theo source:

```wireshark
ip.src == 192.168.10.5 &&
tcp.dstport == 9999
```

---

# 8. Decode As trong Wireshark

Nếu Wireshark không nhận diện giao thức:

```text
Right click packet
→ Decode As
→ Transport
→ Telnet
```

Hoặc đơn giản dùng:

```text
Follow TCP Stream
```

để xem payload.

---

# 9. Dấu hiệu Telnet Trên Port Lạ

- Một port hiếm có session dài.
- Payload chứa prompt hoặc command.
- ASCII text rõ ràng.
- Request/response tương tác giống terminal.
- Packet nhỏ, hai chiều, liên tục.
- User endpoint kết nối tới server nội bộ không rõ mục đích.
- Port không tồn tại trong service inventory.

---

# 10. Telnet Qua IPv6

PCAP liên quan:

```text
telnet_tunneling_ipv6.pcapng
```

Trong mạng không triển khai IPv6 chính thức, IPv6 traffic có thể là dấu hiệu:

- Misconfiguration.
- Rogue communication.
- Bypass IPv4-only controls.
- Tunneling.
- Shadow network path.

Filter theo địa chỉ IPv6:

```wireshark
(
  ipv6.src == fe80::c9c8:ed3:1b10:f10b ||
  ipv6.dst == fe80::c9c8:ed3:1b10:f10b
) &&
telnet
```

---

# 11. IPv6 Link-Local Traffic

Địa chỉ bắt đầu bằng:

```text
fe80::
```

là IPv6 link-local.

Chúng thường dùng cho:

- Neighbor Discovery.
- Router discovery.
- Local-segment communication.

Nếu Telnet xuất hiện giữa link-local addresses, cần kiểm tra:

- Host nào sở hữu địa chỉ.
- Interface nào đang dùng IPv6.
- IPv6 có được phép không.
- Firewall có kiểm tra IPv6 không.
- Endpoint process nào tạo traffic.

---

# 12. ICMPv6 Liên Quan

IPv6 Neighbor Discovery dùng ICMPv6.

Filter:

```wireshark
icmpv6
```

Neighbor Solicitation:

```wireshark
icmpv6.type == 135
```

Neighbor Advertisement:

```wireshark
icmpv6.type == 136
```

Attacker có thể tận dụng IPv6 nếu network controls chỉ tập trung IPv4.

---

# 13. UDP Là Gì?

UDP là giao thức connectionless:

```text
Không three-way handshake
Không sequence/retransmission mặc định
Không đảm bảo delivery
```

Ưu điểm:

- Nhanh.
- Overhead thấp.
- Phù hợp real-time traffic.

Nhược điểm cho defender:

- Dễ truyền dữ liệu ngay lập tức.
- Không có session state mạnh như TCP.
- Một số môi trường giám sát UDP kém hơn TCP.

---

# 14. Các Ứng Dụng UDP Hợp Lệ

UDP thường dùng cho:

| Giao thức/ứng dụng | Port thường gặp |
|---|---:|
| DNS | 53 |
| DHCP | 67/68 |
| SNMP | 161/162 |
| TFTP | 69 |
| NTP | 123 |
| Syslog | 514 |
| VoIP/RTP | Dynamic |
| Gaming/Streaming | Dynamic |

Không phải mọi UDP traffic đều đáng ngờ.

---

# 15. UDP Tunneling

PCAP liên quan:

```text
udp_tunneling.pcapng
```

Attacker có thể dùng UDP để:

- Exfiltrate data.
- Gửi command.
- Tạo custom C2 protocol.
- Bypass TCP-focused monitoring.
- Truyền dữ liệu nhanh.
- Tránh handshake logs.

---

# 16. Dấu Hiệu UDP Tunneling

- Một host gửi UDP đều đặn tới port lạ.
- Packet length giống nhau.
- Payload có cấu trúc hoặc ASCII.
- Destination không thuộc service inventory.
- Session kéo dài dù UDP không có connection.
- Request/response ratio bất thường.
- Nhiều data bytes nhưng không có ứng dụng hợp lệ.
- UDP traffic ra ngoài từ server nhạy cảm.

---

# 17. Wireshark Filters Cho UDP

Tất cả UDP:

```wireshark
udp
```

Theo port:

```wireshark
udp.port == 9999
```

Theo source:

```wireshark
ip.src == 192.168.10.5 && udp
```

Theo cặp host:

```wireshark
ip.addr == 192.168.10.5 &&
ip.addr == 192.168.10.7 &&
udp
```

Payload lớn:

```wireshark
udp && frame.len > 200
```

---

# 18. Follow UDP Stream

Trong Wireshark:

```text
Right click packet
→ Follow
→ UDP Stream
```

Kiểm tra:

- ASCII text.
- Base64/hex.
- File signatures.
- Command strings.
- Session identifiers.
- Sequence-like counters.
- Repeated headers.

---

# 19. UDP Payload Analysis

Các chỉ báo đáng chú ý:

```text
HTB{...}
Username=
Password=
cmd=
exec=
file=
chunk=
```

Ngoài plaintext, attacker có thể dùng:

- Base64.
- Hex.
- XOR.
- Compression.
- Encryption.

Không thực thi dữ liệu trích xuất; chỉ phân tích trong sandbox.

---

# 20. Packet-Length Analysis

Một custom UDP tunnel thường tạo:

- Packet length cố định.
- Khoảng thời gian đều.
- Payload size theo chunk.
- Source/destination port cố định.

Ví dụ:

```text
Mỗi 2 giây gửi một packet 128 bytes
```

Đây có thể là beaconing.

---

# 21. Tshark Hunt Telnet

Port 23:

```bash
tshark -r telnet_tunneling_23.pcapng   -Y "tcp.port == 23"   -T fields   -e frame.time   -e ip.src   -e ip.dst   -e tcp.srcport   -e tcp.dstport   -e tcp.len
```

Port 9999:

```bash
tshark -r telnet_tunneling_9999.pcapng   -Y "tcp.port == 9999"   -T fields   -e ip.src   -e ip.dst   -e tcp.len   -e data.data
```

---

# 22. Tshark Hunt IPv6 Telnet

```bash
tshark -r telnet_tunneling_ipv6.pcapng   -Y "ipv6 && tcp"   -T fields   -e ipv6.src   -e ipv6.dst   -e tcp.srcport   -e tcp.dstport   -e tcp.len
```

Theo host:

```bash
tshark -r telnet_tunneling_ipv6.pcapng   -Y "ipv6.addr == fe80::c9c8:ed3:1b10:f10b"   -T fields   -e frame.number   -e ipv6.src   -e ipv6.dst   -e tcp.port
```

---

# 23. Tshark Hunt UDP

```bash
tshark -r udp_tunneling.pcapng   -Y "udp"   -T fields   -e frame.time   -e ip.src   -e ip.dst   -e udp.srcport   -e udp.dstport   -e udp.length   -e data.data
```

Đếm flow:

```bash
tshark -r udp_tunneling.pcapng   -Y "udp"   -T fields   -e ip.src   -e ip.dst   -e udp.dstport |
sort | uniq -c | sort -nr
```

---

# 24. Detection Logic Cho Telnet

```text
Telnet protocol
+ source/destination không nằm trong allowlist
+ plaintext credential/command
+ session ngoài giờ
```

Non-standard Telnet:

```text
Long-lived TCP session
+ port lạ
+ interactive ASCII payload
+ endpoint không có business reason
```

---

# 25. Detection Logic Cho UDP Tunneling

```text
Một source
+ một destination/port hiếm
+ packet interval đều
+ payload length ổn định
+ session kéo dài
```

High-confidence pattern:

```text
Server nhạy cảm
→ UDP ra Internet
→ port lạ
→ bytes tăng đều
→ process không được phê duyệt
```

---

# 26. Suricata Detection Ideas

Telnet port 23:

```suricata
alert tcp $HOME_NET any -> $EXTERNAL_NET 23 (
  msg:"Outbound Telnet connection";
  flow:to_server,established;
  sid:1000501;
  rev:1;
)
```

Non-standard suspicious TCP:

```text
Long-lived connection
+ ASCII payload
+ rare destination port
```

UDP tunnel concept:

```suricata
alert udp $HOME_NET any -> $EXTERNAL_NET any (
  msg:"Possible high-volume UDP tunneling";
  threshold:type both, track by_src, count 50, seconds 60;
  sid:1000502;
  rev:1;
)
```

Cần tune theo môi trường.

---

# 27. Zeek Detection Ideas

Nguồn log:

```text
conn.log
weird.log
notice.log
```

Feature:

- Service detection khác port.
- Long-lived TCP/UDP flow.
- Orig/resp bytes.
- Rare ports.
- IPv6 connections.
- Connection frequency.
- One-host-to-one-destination beaconing.

Zeek protocol identification giúp phát hiện Telnet trên port không chuẩn nếu analyzer nhận diện được.

---

# 28. Splunk Detection

Telnet:

```spl
index=network
(dest_port=23 OR app=telnet)
| stats count sum(bytes_out) as bytes_out
  values(dest_ip) as destinations
  by src_ip, user
```

Non-standard interactive TCP:

```spl
index=network transport=tcp
| stats count
        sum(bytes_out) as out_bytes
        sum(bytes_in) as in_bytes
        values(dest_port) as ports
  by src_ip, dest_ip
| where count > 100 AND out_bytes > 1000
```

UDP tunnel:

```spl
index=network transport=udp
| bin _time span=5m
| stats count
        avg(bytes_out) as avg_bytes
        stdev(bytes_out) as byte_variance
        sum(bytes_out) as total_out
  by _time src_ip, dest_ip, dest_port
| where count > 50 AND total_out > 5000
```

IPv6 bất thường:

```spl
index=network src_ip="*:*"
| stats count values(dest_ip) values(dest_port) by src_ip
```

---

# 29. Elastic/KQL

Telnet:

```kql
destination.port: 23
```

Port 9999:

```kql
destination.port: 9999
```

UDP:

```kql
network.transport: "udp"
```

IPv6:

```kql
source.ip: "*:*" or destination.ip: "*:*"
```

Cần aggregation trong Elastic rule để phát hiện beaconing và volume.

---

# 30. Sigma Detection Ideas

Outbound Telnet:

```yaml
title: Outbound Telnet Network Connection
status: experimental
logsource:
  category: network_connection
detection:
  selection:
    DestinationPort: 23
  condition: selection
falsepositives:
  - Legacy device administration
level: high
```

Suspicious UDP:

```yaml
title: Suspicious UDP Connection to Rare Port
status: experimental
logsource:
  category: network_connection
detection:
  selection:
    Protocol: UDP
  condition: selection
falsepositives:
  - Voice, video or gaming traffic
level: medium
```

Threshold và rare-port logic cần triển khai trong SIEM.

---

# 31. Endpoint Correlation

Cần xác định process tạo traffic:

Linux:

```text
telnet
nc
socat
python
perl
custom binary
```

Windows:

```text
telnet.exe
powershell.exe
cmd.exe
python.exe
unknown executable
```

EDR/Sysmon giúp correlate:

```text
Process creation
Network connection
Command line
User
File hash
Parent process
```

---

# 32. IPv6 Security Controls

Nếu không dùng IPv6:

- Disable có kiểm soát.
- Block IPv6 tại host/firewall.
- Monitor ICMPv6.
- Kiểm tra rogue router advertisements.
- Bật IPv6 firewall rules tương đương IPv4.

Nếu có dùng IPv6:

- Asset inventory.
- IPv6 ACL.
- RA Guard.
- DHCPv6 Guard.
- NDP monitoring.
- IDS/IPS visibility.

---

# 33. Prevention và Hardening

## Telnet

- Vô hiệu hóa Telnet.
- Chuyển sang SSH.
- Block TCP/23.
- Allowlist management subnet.
- Dùng MFA/jump host.
- Network segmentation.
- Rotate credentials nếu plaintext exposure.

## UDP

- Egress allowlist.
- Block rare outbound UDP.
- Rate-limit.
- Inspect payload khi phù hợp.
- Baseline legitimate applications.
- Monitor long-lived pseudo-sessions.
- Restrict direct Internet access từ servers.

---

# 34. Incident Response Checklist

```text
[ ] Xác định source/destination/port.
[ ] Follow TCP hoặc UDP stream.
[ ] Trích payload trong sandbox.
[ ] Kiểm tra credential/command/file data.
[ ] Xác định process và user.
[ ] Kiểm tra IPv6 có được phê duyệt không.
[ ] Isolate host nếu nghi C2/exfiltration.
[ ] Block destination/port.
[ ] Rotate credential nếu dùng Telnet plaintext.
[ ] Thu PCAP, memory, process và network logs.
```

---

# 35. Threat Hunting

Hunt theo:

- TCP/23 trong môi trường hiện đại.
- Telnet-like payload trên port lạ.
- IPv6 traffic ngoài baseline.
- UDP long-lived flows.
- Packet length ổn định.
- Rare UDP destination ports.
- Server gửi UDP ra Internet.
- Interactive ASCII stream.
- Credential text trong payload.

---

# 36. Mapping với CDSA

## Security Operations & Monitoring

- Monitor Telnet và rare ports.
- Baseline UDP applications.
- Correlate IPv4/IPv6 visibility.

## Incident Response & Forensics

- Follow stream.
- Trích payload.
- Xác định process/user.
- Đánh giá exfiltration.

## Malware Analysis & Reverse Engineering

- Phân tích custom UDP protocol.
- Decode encoded payload.
- Xác định C2 message format.

## Threat Hunting

- Hunt Telnet trên non-standard ports.
- Hunt IPv6 bypass.
- Hunt UDP beaconing.
- Hunt plaintext credential exposure.

## Detection Engineering

- Suricata/Zeek service detection.
- Splunk/Elastic rare-port rules.
- Sigma network rules.
- Asset-aware allowlists.

---

# 37. Lab Setup

Môi trường tối thiểu:

```text
1 Linux client
1 Telnet/UDP test server trong lab
1 Wireshark/tshark sensor
Suricata hoặc Zeek
Splunk/ELK/Wazuh nhận network logs
Endpoint telemetry
```

Workflow:

```text
1. Mở telnet_tunneling_23.pcapng.
2. Lọc tcp.port == 23.
3. Follow TCP Stream.
4. Mở telnet_tunneling_9999.pcapng.
5. Phân tích Telnet-like payload trên port 9999.
6. Kiểm tra IPv6 PCAP.
7. Mở udp_tunneling.pcapng.
8. Follow UDP Stream.
9. Viết detection và IR timeline.
```

---

# Key Takeaway

```text
Telnet là plaintext và nên được xem là rủi ro cao trong mạng hiện đại.
Không nên chỉ tìm Telnet trên port 23 vì attacker có thể dùng cổng bất kỳ.
IPv6 có thể trở thành đường đi bị bỏ sót nếu security controls chỉ theo dõi IPv4.
UDP tunneling thường lộ qua rare ports, packet sizes ổn định, beaconing và payload bất thường.
```

Strange Telnet & UDP Connections là chủ đề quan trọng của CDSA vì kết nối packet analysis, legacy protocol risk, covert-channel detection, IPv6 visibility và incident response.

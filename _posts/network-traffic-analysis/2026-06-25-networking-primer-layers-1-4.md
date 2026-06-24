---
layout: post
title: "Networking Primer: Layers 1-4"
date: 2026-06-25 00:21:49 +0700
categories: [SOC, Network Traffic Analysis]
tags: [soc, network-traffic-analysis, osi-model, tcp-ip, pdu, mac-address, ipv4, ipv6, tcp, udp, three-way-handshake, blue-team]
description: "Tóm tắt Networking Primer Layers 1-4: OSI/TCP-IP models, PDU, encapsulation, MAC addressing, IPv4/IPv6, TCP vs UDP, TCP three-way handshake và session teardown."
toc: true
---

# Networking Primer: Layers 1-4

Bài này là phần nền tảng để hiểu **Network Traffic Analysis (NTA)**. Muốn đọc packet capture chính xác, analyst cần hiểu OSI/TCP-IP models, PDU, encapsulation, MAC/IP addressing và cơ chế TCP/UDP.

```text
Traffic analysis tốt = hiểu packet đang nằm ở layer nào, dùng protocol gì, đi từ đâu đến đâu và có hành vi bình thường không.
```

---

## Timeline cập nhật

```text
Created at: 2026-06-25 00:21:49 +0700
Timezone: Asia/Ho_Chi_Minh
Topic: Networking Primer - Layers 1-4
Focus: OSI/TCP-IP, PDU, MAC, IPv4, IPv6, TCP/UDP, handshake, teardown
```

---

# 1. OSI Model và TCP/IP Model

Hai mô hình quan trọng nhất khi học network:

```text
OSI Model
TCP/IP Model
```

## 1.1. OSI Model

OSI có 7 layer:

| Layer | Tên | Ví dụ |
|---|---|---|
| 7 | Application | FTP, HTTP |
| 6 | Presentation | JPG, PNG, SSL/TLS |
| 5 | Session | NetBIOS |
| 4 | Transport | TCP, UDP |
| 3 | Network | IP, Router, L3 Switch |
| 2 | Data-Link | Ethernet, Switch, Bridge |
| 1 | Physical | Cable, NIC, Signal |

---

## 1.2. TCP/IP Model

TCP/IP có 4 layer:

| TCP/IP Layer | Mapping với OSI |
|---|---|
| 4. Application | OSI Layer 5-7 |
| 3. Transport | OSI Layer 4 |
| 2. Internet | OSI Layer 3 |
| 1. Link | OSI Layer 1-2 |

---

## 1.3. So sánh OSI và TCP/IP

| Đặc điểm | OSI | TCP/IP |
|---|---|---|
| Số layer | 7 | 4 |
| Tính chất | Lý thuyết, chia nhỏ chức năng | Thực tế, gần với triển khai Internet |
| Độ linh hoạt | Strict | Flexible |
| Phụ thuộc protocol | Generic/protocol independent | Dựa trên protocol thực tế |

Nói dễ hiểu:

```text
OSI = mô hình lý thuyết để hiểu cách network hoạt động.
TCP/IP = mô hình thực tế hơn, gắn với cách Internet hoạt động.
```

---

# 2. PDU là gì?

**PDU** là viết tắt của:

```text
Protocol Data Unit
```

PDU là đơn vị dữ liệu ở từng layer trong mô hình mạng.

| Layer | PDU |
|---|---|
| Application / Presentation / Session | Data |
| Transport | Segment / Datagram |
| Network / Internet | Packet |
| Data-Link | Frame |
| Physical | Bit |

---

# 3. Encapsulation

Khi dữ liệu đi từ application xuống network, mỗi layer sẽ bọc thêm header riêng. Quá trình này gọi là:

```text
Encapsulation
```

Ví dụ:

```text
Data
→ TCP Segment / UDP Datagram
→ IP Packet
→ Ethernet Frame
→ Bits trên dây
```

Khi Wireshark hiển thị packet, nó thường hiển thị theo thứ tự đã được bóc tách:

```text
Frame
Ethernet II
IPv4 / IPv6
TCP / UDP
Application Data
```

Điều này giống quá trình **unencapsulation** từ dưới lên.

---

# 4. PDU Packet Breakdown

Một packet trong Wireshark thường có dạng:

```text
Frame
Ethernet II
Internet Protocol Version 4
Transmission Control Protocol / User Datagram Protocol
Application Data
```

Ví dụ với UDP:

```text
Ethernet II
  Source MAC
  Destination MAC

IPv4
  Source IP
  Destination IP
  TTL
  Protocol UDP

UDP
  Source Port
  Destination Port
  Length
  Checksum

Data
```

Khi đọc packet, cần hiểu:

- MAC address nằm ở Layer 2.
- IP address nằm ở Layer 3.
- Port nằm ở Layer 4.
- Application data nằm ở Layer 5-7.

---

# 5. MAC Addressing

**MAC address** là địa chỉ tầng 2 của interface mạng.

Đặc điểm:

```text
48-bit
6 octets
Hexadecimal format
Ví dụ: 88:66:5a:11:bb:36
```

MAC được dùng trong cùng broadcast domain để host giao tiếp ở Layer 2.

Ví dụ từ interface:

```text
ether 88:66:5a:11:bb:36
```

---

## 5.1. MAC dùng để làm gì?

MAC address giúp frame đến đúng thiết bị trong cùng mạng local.

Ví dụ:

```text
Host A muốn gửi packet ra Internet
→ Layer 3 destination là IP server ngoài Internet
→ Layer 2 destination MAC là MAC của default gateway/router
```

Router sau đó bỏ Layer 2 encapsulation cũ và tạo Layer 2 encapsulation mới cho next hop.

---

# 6. IP Addressing

**IP address** hoạt động ở Layer 3, dùng để định tuyến packet qua nhiều network.

IP chịu trách nhiệm:

- Routing.
- Packet addressing.
- Fragmentation/reassembly.
- Delivery across network boundaries.

IP là connectionless, nghĩa là bản thân IP không đảm bảo packet đến nơi. Reliability thường do upper-layer như TCP đảm nhiệm.

---

# 7. IPv4

IPv4 là địa chỉ 32-bit, thường biểu diễn dạng decimal dotted notation.

Ví dụ:

```text
192.168.86.243
```

IPv4 gồm 4 octets:

```text
192 . 168 . 86 . 243
```

Mỗi octet có giá trị từ:

```text
0 đến 255
```

Trong packet, IPv4 nằm ở:

```text
OSI Layer 3 - Network
TCP/IP Layer 2 - Internet
```

---

## 7.1. Ví dụ IPv4 trên interface

```text
inet 192.168.86.243 netmask 0xffffff00 broadcast 192.168.86.255
```

Ý nghĩa:

| Field | Ý nghĩa |
|---|---|
| inet | IPv4 address |
| netmask | Subnet mask |
| broadcast | Broadcast address của subnet |

---

# 8. IPv6

IPv6 được tạo ra để giải quyết giới hạn địa chỉ của IPv4.

Đặc điểm:

```text
128-bit
16 octets
Hexadecimal format
```

Ví dụ:

```text
fe80::49f:e3c:bf36:9bb1%en0
```

IPv6 cung cấp:

- Address space lớn hơn rất nhiều.
- Hỗ trợ multicast tốt hơn.
- Global addressing per device.
- Security support với IPSec.
- Header đơn giản hơn để xử lý hiệu quả hơn.

---

## 8.1. Các loại IPv6 address

| Type | Ý nghĩa |
|---|---|
| Unicast | Một interface nhận |
| Anycast | Nhiều interface, một interface gần/đúng nhất nhận |
| Multicast | Một gửi, nhiều interface nhận |
| Broadcast | Không tồn tại trong IPv6, thay bằng multicast |

Ghi nhớ nhanh:

```text
Unicast = one-to-one
Multicast = one-to-many
Anycast = one-to-nearest/one-of-many
Broadcast = không dùng trong IPv6
```

---

# 9. TCP và UDP

Layer Transport có hai protocol quan trọng:

```text
TCP
UDP
```

## 9.1. TCP

TCP là:

```text
Connection-oriented
Reliable
Stream-based
Có sequence/acknowledgement
Có three-way handshake
```

TCP phù hợp khi cần dữ liệu đầy đủ và đúng thứ tự.

Ví dụ:

- SSH.
- HTTP/HTTPS.
- File transfer.
- Remote management.
- Database connection.

---

## 9.2. UDP

UDP là:

```text
Connectionless
Fast
Fire-and-forget
Không đảm bảo delivery
Không có session setup như TCP
```

UDP phù hợp khi cần tốc độ hơn độ tin cậy.

Ví dụ:

- DNS.
- Streaming.
- VoIP.
- DHCP.
- NTP.

---

## 9.3. TCP vs UDP

| Đặc điểm | TCP | UDP |
|---|---|---|
| Connection | Connection-oriented | Connectionless |
| Reliability | Có ACK, sequence | Không đảm bảo |
| Speed | Chậm hơn do overhead | Nhanh hơn |
| Data model | Stream | Datagram |
| Use case | SSH, web, file transfer | DNS, streaming, VoIP |
| Handshake | Có | Không |

---

# 10. TCP Three-Way Handshake

TCP thiết lập kết nối bằng 3 bước:

```text
1. SYN
2. SYN/ACK
3. ACK
```

## 10.1. Bước 1: Client gửi SYN

Client gửi packet có flag:

```text
SYN
```

Mục tiêu:

- Yêu cầu mở kết nối.
- Đồng bộ sequence number.
- Đề xuất TCP options như window size, MSS, SACK.

---

## 10.2. Bước 2: Server trả SYN/ACK

Server phản hồi:

```text
SYN + ACK
```

Ý nghĩa:

- ACK cho SYN của client.
- Gửi SYN của server để đồng bộ sequence number chiều ngược lại.

---

## 10.3. Bước 3: Client gửi ACK

Client gửi:

```text
ACK
```

Sau bước này, TCP session được thiết lập.

---

## 10.4. Handshake summary

```text
Client → Server : SYN
Server → Client : SYN/ACK
Client → Server : ACK
```

Sau đó application data mới được gửi, ví dụ HTTP request.

---

# 11. TCP Session Teardown

Khi TCP đóng kết nối bình thường, thường thấy các flag:

```text
FIN
ACK
```

Mẫu graceful teardown:

```text
FIN, ACK
FIN, ACK
ACK
```

Ý nghĩa:

- Một bên báo đã gửi xong data bằng FIN.
- Bên kia ACK.
- Bên kia cũng gửi FIN.
- Bên đầu ACK lại và kết nối đóng.

Nếu thấy reset bất thường (`RST`) hoặc nhiều retransmission, có thể có vấn đề network hoặc ứng dụng.

---

# 12. SOC/NTA Hunting Ideas

## 12.1. Port scan

Dấu hiệu:

```text
Nhiều SYN đến nhiều port
Không hoàn tất handshake
Nhiều destination port trong thời gian ngắn
```

Cần xem:

- Source IP.
- Destination IP.
- Destination ports.
- TCP flags.
- Time window.

---

## 12.2. C2 beaconing

Dấu hiệu:

```text
Connection lặp lại theo interval đều
Destination IP/domain lạ
Dung lượng nhỏ nhưng đều
Port/protocol bất thường
```

Cần xem:

- DNS query.
- Destination IP.
- JA3/SNI nếu có.
- HTTP user-agent.
- Process tạo connection nếu correlate với endpoint logs.

---

## 12.3. UDP suspicious traffic

Dấu hiệu:

```text
UDP packet đến port lạ
DNS query bất thường
High volume UDP outbound
Không có response tương ứng
```

Cần xem:

- Source/destination.
- UDP port.
- Query name nếu DNS.
- Packet size.
- Frequency.

---

# 13. Bảng ôn nhanh

| Khái niệm | Nghĩa |
|---|---|
| OSI | Mô hình 7 layer |
| TCP/IP | Mô hình 4 layer thực tế |
| PDU | Protocol Data Unit |
| Data | PDU application layer |
| Segment | PDU của TCP |
| Datagram | PDU của UDP |
| Packet | PDU network layer |
| Frame | PDU data-link layer |
| Bit | PDU physical layer |
| MAC | Địa chỉ Layer 2 |
| IPv4 | Địa chỉ 32-bit |
| IPv6 | Địa chỉ 128-bit |
| TCP | Reliable connection-oriented transport |
| UDP | Fast connectionless transport |
| SYN | TCP flag mở kết nối |
| ACK | TCP flag xác nhận |
| FIN | TCP flag đóng kết nối |

---

# Key Takeaway

Muốn phân tích traffic tốt, phải hiểu cách dữ liệu được đóng gói qua từng layer.

```text
Application data
→ Transport segment/datagram
→ IP packet
→ Ethernet frame
→ Bits
```

Trong NTA, việc phân biệt MAC, IP, port, protocol, TCP flags và PDU giúp analyst đọc PCAP chính xác hơn, phát hiện port scan, C2, malformed traffic và các hành vi network bất thường.

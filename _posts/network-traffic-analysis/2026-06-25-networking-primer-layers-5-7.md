---
layout: post
title: "Networking Primer: Layers 5-7"
date: 2026-06-25 22:26:00 +0700
categories: [SOC, Network Traffic Analysis]
tags: [soc, network-traffic-analysis, osi-model, application-layer, http, https, tls, ftp, smb, pcap, wireshark, blue-team]
description: "Tóm tắt Networking Primer Layers 5-7: HTTP methods, HTTPS/TLS handshake, FTP active/passive mode, FTP commands và SMB traffic analysis."
toc: true
---

# Networking Primer: Layers 5-7

Bài này tiếp nối phần **Layers 1-4**, tập trung vào các protocol upper-layer thường thấy khi phân tích traffic.

Các protocol chính:

```text
HTTP
HTTPS / TLS
FTP
SMB
```

Trong Network Traffic Analysis, hiểu các protocol này giúp analyst nhận diện traffic bình thường, phát hiện lỗi, phát hiện credential exposure, lateral movement, file transfer bất thường và C2 traffic.

---

## Timeline cập nhật

```text
Created at: 2026-06-25 22:26:00 +0700
Timezone: Asia/Ho_Chi_Minh
Topic: Networking Primer - Layers 5-7
Focus: HTTP, HTTPS/TLS, FTP, SMB
```

---

# 1. Layers 5-7 trong OSI Model

Trong OSI, layers 5-7 thuộc nhóm **host/application layers**:

| OSI Layer | Tên | Vai trò |
|---|---|---|
| Layer 7 | Application | Protocol ứng dụng như HTTP, FTP, SMB |
| Layer 6 | Presentation | Mã hóa, nén, định dạng dữ liệu như TLS, JPG, PNG |
| Layer 5 | Session | Quản lý session giữa applications |

Trong TCP/IP model, ba layer này thường được gộp vào:

```text
Application Layer
```

---

# 2. HTTP

**HTTP** là viết tắt của:

```text
Hypertext Transfer Protocol
```

HTTP là protocol tầng Application, dùng để truyền dữ liệu giữa client và server.

Đặc điểm:

```text
Stateless
Clear text
Thường chạy trên TCP port 80 hoặc 8000
Client gửi request
Server trả response
```

HTTP dùng để truyền:

- HTML.
- Images.
- Hyperlinks.
- Video.
- API data.
- Web resources.

---

# 3. HTTP Methods

HTTP method xác định hành động client muốn thực hiện với resource.

| Method | Ý nghĩa |
|---|---|
| HEAD | Giống GET nhưng không lấy body |
| GET | Lấy resource/content |
| POST | Gửi dữ liệu lên server |
| PUT | Tạo hoặc cập nhật object tại URI |
| DELETE | Xóa object tại URI |
| TRACE | Server echo lại request, dùng chẩn đoán |
| OPTIONS | Hỏi server hỗ trợ method nào |
| CONNECT | Tạo tunnel qua proxy/firewall |

---

## 3.1. GET

GET là method phổ biến nhất.

Ví dụ:

```text
GET /Webserver/index.html HTTP/1.1
Host: 10.1.1.1
```

Ý nghĩa:

```text
Client yêu cầu server trả về trang index.html.
```

---

## 3.2. POST

POST dùng để gửi dữ liệu lên server.

Ví dụ:

```text
POST /login HTTP/1.1
username=admin&password=123456
```

Trong traffic clear-text HTTP, POST có thể làm lộ credential nếu không dùng HTTPS.

---

## 3.3. PUT và DELETE

PUT có thể tạo/cập nhật object tại URI.

DELETE xóa object tại URI.

Trong môi trường production, nếu thấy PUT/DELETE bất thường trên server không nên cho phép write, cần điều tra.

---

## 3.4. OPTIONS và TRACE

OPTIONS dùng để xem server hỗ trợ method nào.

TRACE có thể gây rủi ro nếu bật không cần thiết.

SOC analyst cần chú ý nếu thấy:

```text
OPTIONS scan
TRACE enabled
PUT/DELETE enabled ngoài ý muốn
```

---

# 4. HTTPS và TLS

**HTTPS** là HTTP được bảo vệ bởi TLS/SSL.

Đặc điểm:

```text
HTTP + TLS
Thường chạy TCP port 443 hoặc 8443
Dữ liệu application được mã hóa
```

HTTPS giúp bảo vệ:

- Web browsing.
- Login session.
- Search requests.
- File transfer.
- Banking transactions.
- API communications.

---

# 5. TLS Handshake qua HTTPS

Khi client kết nối HTTPS:

```text
1. TCP three-way handshake đến port 443
2. TLS ClientHello
3. TLS ServerHello
4. Certificate exchange
5. Key exchange / cryptographic negotiation
6. Tạo shared secret
7. Application data được mã hóa
```

---

## 5.1. TLS ClientHello

ClientHello gửi các thông tin như:

- TLS version hỗ trợ.
- Cipher suites.
- Random value.
- Extensions.
- SNI nếu có.

---

## 5.2. ServerHello và Certificate

Server phản hồi:

- Cipher suite được chọn.
- Server random.
- Certificate x.509.
- Thông tin cryptographic cần thiết.

---

## 5.3. TLS Application Data

Sau handshake, payload HTTP sẽ được mã hóa và trong Wireshark thường thấy là:

```text
TLS Application Data
```

Lúc này analyst thường không đọc được nội dung HTTP nếu không có key/decryption.

Tuy nhiên vẫn có thể phân tích metadata:

- Source IP.
- Destination IP.
- Port.
- SNI.
- Certificate.
- JA3/JA3S nếu có.
- Packet size.
- Timing.
- Frequency.
- Beacon pattern.

---

# 6. FTP

**FTP** là viết tắt của:

```text
File Transfer Protocol
```

FTP là protocol tầng Application dùng để truyền file.

Đặc điểm:

```text
Không an toàn nếu dùng FTP thường
Clear-text credential
Dùng TCP ports 20 và 21
Port 21: control channel
Port 20: data transfer
```

FTP có thể được dùng qua:

- Command-line.
- Browser cũ.
- FTP client như FileZilla.
- Script automation.

Hiện nay FTP thường được thay bằng SFTP/FTPS vì vấn đề bảo mật.

---

# 7. FTP Active và Passive Mode

FTP đặc biệt vì dùng nhiều connection/ports.

## 7.1. Active Mode

Trong active mode:

```text
Client kết nối server port 21 để điều khiển
Client gửi PORT command
Server kết nối ngược về client để truyền data
```

Vấn đề:

```text
Firewall/NAT có thể chặn server kết nối ngược về client.
```

---

## 7.2. Passive Mode

Trong passive mode:

```text
Client gửi PASV command
Server trả IP/port để client kết nối data channel
Client chủ động kết nối đến data port của server
```

Passive mode hữu ích khi client nằm sau NAT/firewall.

---

# 8. FTP Commands thường gặp

| Command | Ý nghĩa |
|---|---|
| USER | Gửi username |
| PASS | Gửi password |
| PORT | Chỉ định data port trong active mode |
| PASV | Chuyển sang passive mode |
| LIST | Liệt kê file/thư mục |
| CWD | Change working directory |
| PWD | Print working directory |
| SIZE | Lấy kích thước file |
| RETR | Download file từ FTP server |
| QUIT | Kết thúc session |

---

## 8.1. SOC chú ý gì với FTP?

FTP thường clear-text, nên analyst có thể thấy:

- Username.
- Password.
- File name.
- Directory.
- Command.
- Download/upload activity.

Dấu hiệu đáng chú ý:

```text
Anonymous login
Repeated failed login
Large file transfer
Sensitive file retrieved
Unusual external FTP server
Internal host acting as FTP server bất thường
```

---

# 9. SMB

**SMB** là viết tắt của:

```text
Server Message Block
```

SMB phổ biến trong Windows enterprise để chia sẻ tài nguyên.

Dùng cho:

- File shares.
- Printer sharing.
- Named pipes.
- Remote administration.
- Authentication-related activity.
- Windows domain operations.

---

## 9.1. SMB Ports

SMB hiện đại thường dùng:

| Port | Ý nghĩa |
|---|---|
| 445/TCP | SMB direct over TCP |
| 139/TCP | SMB over NetBIOS |
| 137/UDP | NetBIOS Name Service |
| 138/UDP | NetBIOS Datagram Service |

SMB hiện đại chủ yếu dùng:

```text
TCP 445
```

---

# 10. SMB on the wire

SMB là connection-oriented và thường có TCP handshake trước.

Khi phân tích SMB trong Wireshark, cần chú ý:

- TCP handshake đến port 445.
- SMB negotiation.
- Session setup.
- Authentication success/failure.
- Tree connect.
- File access.
- Named pipe activity.
- Errors lặp lại.

---

## 10.1. SMB Authentication Failures

Một vài auth failure có thể bình thường.

Nhưng nhiều failure lặp lại có thể là:

- Brute force.
- Password spraying.
- Credential misuse.
- Lateral movement attempt.
- User account bị attacker thử credential.

Ví dụ dấu hiệu:

```text
Nhiều STATUS_LOGON_FAILURE
Một host thử nhiều account
Một account thử nhiều host
SMB access từ workstation đến nhiều server lạ
```

---

## 10.2. SMB và lateral movement

SMB hấp dẫn với attacker vì có thể dùng để:

- Copy tool sang host khác.
- Truy cập share.
- Remote service execution.
- PsExec-like behavior.
- Named pipe communication.
- Credential relay.

Dấu hiệu cần chú ý:

```text
Workstation truy cập ADMIN$
Workstation truy cập C$
File executable được copy qua SMB
Named pipe bất thường
Auth failure lặp lại
Host không thường liên lạc nay lại SMB với nhau
```

---

# 11. Quick Analysis Questions

Khi xem HTTP/HTTPS/FTP/SMB traffic, tự hỏi:

```text
Protocol này có nên xuất hiện ở đây không?
Port này có đúng protocol không?
Traffic clear-text hay encrypted?
Có credential bị lộ không?
Client/server có đúng vai trò không?
Có auth failure lặp lại không?
Có file transfer bất thường không?
Có host nào nói chuyện với host không bình thường không?
```

---

# 12. Bảng tóm tắt protocol

| Protocol | Port phổ biến | Đặc điểm | SOC/NTA chú ý |
|---|---|---|---|
| HTTP | 80, 8000 | Clear-text web | Credential leak, suspicious methods |
| HTTPS | 443, 8443 | HTTP over TLS | SNI/cert/JA3/beaconing |
| FTP | 20, 21 | Clear-text file transfer | USER/PASS, RETR, LIST, large transfers |
| SMB | 445, 139 | Windows file/resource sharing | Auth failures, ADMIN$, lateral movement |

---

# 13. Wireshark mindset

Khi nhìn packet detail:

```text
Ethernet
→ IP
→ TCP/UDP
→ Application Protocol
```

Với HTTPS:

```text
TCP 443
→ TLS Handshake
→ TLS Application Data
```

Với FTP:

```text
TCP 21 control commands
TCP 20/passive data channel
```

Với SMB:

```text
TCP 445
→ SMB negotiation
→ session setup
→ tree connect / file access
```

---

# Key Takeaway

Layers 5-7 là nơi analyst thấy rõ ý nghĩa nghiệp vụ của traffic.

```text
HTTP cho biết web request.
HTTPS cho biết encrypted session và metadata.
FTP cho biết file transfer và có thể lộ credential.
SMB cho biết file share, auth, lateral movement và Windows enterprise activity.
```

Trong NTA, hiểu protocol upper-layer giúp chuyển packet thô thành câu chuyện điều tra: ai kết nối, dùng protocol gì, truyền gì, có hợp lệ không và có dấu hiệu attacker hay không.

---
layout: post
title: "Wireshark Advanced Usage"
date: 2026-06-29 22:42:27 +0700
categories: [SOC, Network Traffic Analysis]
tags: [soc, network-traffic-analysis, wireshark, tcp-stream, follow-stream, packet-analysis, pcap, file-extraction, plugins, statistics, analyze, blue-team]
description: "Tóm tắt Wireshark Advanced Usage: Statistics/Analyze tabs, plugins, Follow TCP Stream, tcp.stream filter, extract files from captures và workflow phân tích nâng cao bằng Wireshark."
toc: true
---

# Wireshark Advanced Usage

Phần này tập trung vào các tính năng nâng cao của **Wireshark**, đặc biệt là các plugin trong tab **Statistics** và **Analyze**, khả năng **Follow TCP Stream**, filter theo `tcp.stream`, và trích xuất dữ liệu/file từ PCAP.

```text
Wireshark không chỉ để xem packet.
Wireshark còn giúp analyst dựng lại conversation, follow stream, xem thống kê và extract dữ liệu từ traffic.
```

---

## Timeline cập nhật

```text
Created at: 2026-06-29 22:42:27 +0700
Timezone: Asia/Ho_Chi_Minh
Topic: Wireshark Advanced Usage
Focus: Statistics, Analyze, Follow TCP Stream, tcp.stream, file extraction
```

---

# 1. Vì sao Wireshark mạnh?

Wireshark mạnh vì có nhiều capability tích hợp sẵn:

- Theo dõi TCP conversations.
- Follow TCP Stream.
- Dựng lại stream từ nhiều packet.
- Trích xuất file/object từ capture.
- Phân tích protocol hierarchy.
- Xem top talkers.
- Xem conversations.
- Xem endpoints.
- Dùng plugin trong Statistics và Analyze.
- Hỗ trợ nhiều dissector/protocol.
- Hỗ trợ nhiều định dạng PCAP.

Với SOC/NTA, các tính năng này giúp biến packet rời rạc thành một câu chuyện điều tra rõ ràng.

---

# 2. Plugins trong Wireshark

Wireshark có nhiều plugin và tính năng phân tích nằm trong các tab:

```text
Statistics
Analyze
```

Các plugin này giúp analyst hiểu nhanh traffic trong capture.

Ví dụ:

- Host nào nói chuyện nhiều nhất.
- Protocol nào xuất hiện nhiều.
- Conversation nào đáng chú ý.
- Có lỗi protocol nào không.
- TCP stream nào chứa data quan trọng.
- Có file/object nào có thể extract không.

---

# 3. Statistics Tab

Tab **Statistics** cung cấp nhiều báo cáo về traffic.

Các mục thường dùng:

| Tính năng | Ý nghĩa |
|---|---|
| Protocol Hierarchy | Xem protocol breakdown trong PCAP |
| Conversations | Xem conversation giữa các host |
| Endpoints | Xem danh sách endpoint xuất hiện |
| I/O Graphs | Xem traffic theo thời gian |
| Packet Lengths | Phân tích kích thước packet |
| DNS | Thống kê DNS nếu có |
| HTTP | Thống kê HTTP nếu có |

---

## 3.1. Protocol Hierarchy

Protocol Hierarchy giúp trả lời:

```text
Trong PCAP có protocol nào?
Protocol nào chiếm nhiều traffic?
Có protocol nào lạ không?
```

Use case:

- Nhanh chóng nhận diện DNS/HTTP/SMB/TLS.
- Tìm protocol bất thường.
- Ưu tiên protocol cần phân tích sâu.

---

## 3.2. Conversations

Conversations giúp xem các cặp host đang giao tiếp.

Analyst có thể xem:

- IP conversations.
- TCP conversations.
- UDP conversations.
- Packet count.
- Bytes transferred.
- Direction.
- Duration.

Use case:

```text
Tìm top talkers.
Tìm host truyền nhiều data.
Tìm connection lạ giữa user workstation.
Tìm outbound traffic đáng ngờ.
```

---

## 3.3. Endpoints

Endpoints hiển thị danh sách host xuất hiện trong capture.

Use case:

- Đếm unique hosts.
- Tìm IP lạ.
- Xác định client/server.
- So sánh với asset inventory.
- Tìm external IP có traffic lớn.

---

# 4. Analyze Tab

Tab **Analyze** cung cấp các công cụ để phân tích sâu hơn.

Các tính năng đáng chú ý:

| Tính năng | Ý nghĩa |
|---|---|
| Follow | Follow TCP/UDP/TLS stream |
| Display Filters | Quản lý display filters |
| Apply as Filter | Tạo filter từ packet/field đang chọn |
| Prepare as Filter | Chuẩn bị filter |
| Enabled Protocols | Bật/tắt protocol dissector |
| Expert Information | Xem cảnh báo/lỗi Wireshark phát hiện |

---

# 5. Follow TCP Stream

**Follow TCP Stream** là một trong các tính năng quan trọng nhất.

Wireshark có thể ghép nhiều TCP packets lại để tái tạo toàn bộ conversation ở dạng dễ đọc.

Tính năng này hữu ích với các protocol chạy trên TCP như:

- HTTP.
- FTP.
- Telnet.
- SMTP.
- POP3.
- IMAP.
- Một số malware C2 clear-text.

---

## 5.1. Cách Follow TCP Stream bằng GUI

Các bước:

```text
1. Chọn một packet thuộc conversation cần xem.
2. Right-click packet.
3. Chọn Follow.
4. Chọn TCP Stream.
5. Wireshark mở cửa sổ stream đã được ghép lại.
```

Kết quả:

```text
Ta thấy toàn bộ conversation theo thứ tự giữa client và server.
```

---

## 5.2. Khi nào Follow TCP Stream hữu ích?

Follow TCP Stream hữu ích khi cần:

- Xem HTTP request/response.
- Xem credential clear-text.
- Xem file hoặc payload được truyền.
- Xem command trong Telnet/FTP.
- Xem malware callback clear-text.
- Xem server banner.
- Hiểu nội dung conversation thay vì từng packet rời rạc.

---

# 6. Filter theo tcp.stream

Mỗi TCP conversation được Wireshark đánh số bằng `tcp.stream`.

Có thể filter một stream cụ thể:

```text
tcp.stream eq 0
```

Hoặc stream khác:

```text
tcp.stream eq 1
```

Use case:

```text
Clear toàn bộ traffic không liên quan.
Chỉ xem một conversation theo thứ tự.
Dễ xác định handshake, data transfer và teardown.
```

---

# 7. Nhận diện TCP conversation

Một TCP conversation thường bắt đầu bằng full TCP handshake:

```text
SYN
SYN/ACK
ACK
```

Sau đó mới đến data transfer.

Ví dụ flow:

```text
Client → Server : SYN
Server → Client : SYN/ACK
Client → Server : ACK
Client ↔ Server : Application Data
```

Trong Wireshark, khi filter `tcp.stream eq #`, ta sẽ thấy conversation rõ ràng hơn.

---

# 8. Extracting Data and Files From a Capture

Wireshark có thể extract nhiều loại dữ liệu từ capture nếu đã capture đủ conversation.

Ví dụ có thể extract:

- HTTP objects.
- SMB files.
- DICOM objects.
- Một số file truyền qua protocol được hỗ trợ.

Điều kiện quan trọng:

```text
Phải capture được toàn bộ conversation.
Nếu capture thiếu packet, file/datagram có thể không reconstruct được.
```

---

## 8.1. Cách extract file bằng GUI

Các bước:

```text
1. Stop capture.
2. Chọn File.
3. Chọn Export Objects.
4. Chọn protocol format phù hợp.
5. Chọn object/file cần export.
6. Save file.
```

Ví dụ protocol format:

```text
HTTP
SMB
DICOM
```

---

## 8.2. Extract HTTP objects

Nếu PCAP có HTTP clear-text traffic, có thể dùng:

```text
File → Export Objects → HTTP
```

Use case:

- Extract image.
- Extract HTML.
- Extract downloaded executable.
- Extract script.
- Extract document.
- Extract suspicious payload.

---

## 8.3. Extract SMB objects

Nếu PCAP có SMB file transfer, có thể dùng:

```text
File → Export Objects → SMB
```

Use case:

- Tìm file được copy qua SMB.
- Điều tra lateral movement.
- Xác định tool được transfer trong nội bộ.
- Tìm payload được đặt lên share.

---

# 9. Lưu ý khi extract data

Khi extract file từ PCAP:

- Không mở file nghi malware trực tiếp trên máy thật.
- Nên dùng VM/sandbox.
- Hash file trước khi phân tích.
- Ghi lại packet number/stream.
- Ghi lại source/destination.
- Bảo quản PCAP như evidence.
- Không chỉnh sửa PCAP gốc.
- Đảm bảo quyền hợp lệ khi xử lý dữ liệu nhạy cảm.

---

# 10. Sample PCAP để luyện tập

Wireshark có trang sample captures để luyện phân tích nhiều protocol khác nhau.

Có thể luyện với:

- HTTP.
- DNS.
- SMB.
- TLS.
- FTP.
- ICS/SCADA.
- Malware traffic.
- Wireless traffic.
- VoIP.

Mục tiêu khi luyện:

```text
Không chỉ mở PCAP,
mà phải đặt câu hỏi:
protocol gì, ai nói chuyện với ai, data gì được truyền, có gì bất thường không.
```

---

# 11. Advanced Wireshark Workflow

Một workflow nâng cao có thể như sau:

```text
1. Mở PCAP.
2. Xem Statistics → Protocol Hierarchy.
3. Xem Statistics → Conversations.
4. Xem Statistics → Endpoints.
5. Xác định top talkers và protocol chính.
6. Dùng display filter giảm noise.
7. Chọn conversation đáng chú ý.
8. Follow TCP Stream.
9. Kiểm tra HTTP headers/server banners.
10. Extract objects nếu cần.
11. Ghi lại evidence.
12. Tóm tắt finding.
```

---

# 12. Quick filters hữu ích

Filter TCP stream:

```text
tcp.stream eq 0
```

Filter HTTP:

```text
http
```

Filter DNS:

```text
dns
```

Filter host:

```text
ip.addr == 192.168.1.10
```

Filter TCP port 80:

```text
tcp.port == 80
```

Filter TCP port 443:

```text
tcp.port == 443
```

Filter SMB:

```text
smb || smb2
```

Filter HTTP request method GET:

```text
http.request.method == "GET"
```

Filter HTTP response:

```text
http.response
```

---

# 13. SOC/NTA use cases

Wireshark advanced features hữu ích khi:

- Điều tra HTTP malware download.
- Xem server banner.
- Tìm credential clear-text.
- Extract file từ HTTP/SMB.
- Xem top talkers.
- Tìm host truyền data nhiều bất thường.
- Follow C2 clear-text stream.
- Kiểm tra TCP handshake/retransmission.
- Xem DNS query bất thường.
- Xác minh IDS alert.

---

# 14. Câu hỏi analyst nên đặt

Khi dùng Wireshark advanced features:

```text
Protocol nào chiếm nhiều nhất?
Conversation nào có nhiều bytes nhất?
Host nào là top talker?
Có TCP stream nào chứa clear-text không?
Có file nào được truyền không?
Có HTTP Server header không?
Có object nào extract được không?
Có host nội bộ nào nói chuyện bất thường với nhau không?
Traffic có khớp baseline không?
```

---

# Key Takeaway

Wireshark advanced usage giúp analyst đi xa hơn việc chỉ xem packet list.

```text
Statistics giúp nhìn tổng quan.
Analyze giúp drill down.
Follow TCP Stream giúp dựng lại conversation.
tcp.stream giúp cô lập một flow.
Export Objects giúp extract dữ liệu/file từ capture.
```

Trong SOC/Blue Team, những tính năng này giúp biến PCAP thành evidence rõ ràng để xác định server, client, file transfer, protocol behavior và dấu hiệu bất thường.

---
layout: post
title: "Interrogating Network Traffic With Capture and Display Filters"
date: 2026-06-29 21:39:03 +0700
categories: [SOC, Network Traffic Analysis]
tags: [soc, network-traffic-analysis, tcpdump, pcap, capture-filters, display-filters, dns, http, https, packet-analysis, blue-team]
description: "Tóm tắt lab phân tích PCAP bằng tcpdump: đọc capture, nhận diện protocol/port, xác định conversations, lọc DNS/HTTP/HTTPS traffic và tìm server trả lời DNS/web requests."
toc: true
---

# Interrogating Network Traffic With Capture and Display Filters

Lab này giúp thực hành **interrogating network traffic** bằng tcpdump thông qua capture filters và display filters.

Mục tiêu chính:

```text
Đọc PCAP
→ nhận diện protocol/port
→ xác định conversations
→ lọc DNS
→ lọc HTTP/HTTPS
→ xác định DNS server và web server trong local network
```

---

## Timeline cập nhật

```text
Created at: 2026-06-29 21:39:03 +0700
Timezone: Asia/Ho_Chi_Minh
Topic: Interrogating Network Traffic With Capture and Display Filters
Focus: tcpdump filters, PCAP analysis, DNS, HTTP, HTTPS, conversations
```

---

# 1. Mục tiêu của lab

Lab này tập trung vào việc dùng tcpdump để phân tích file `.pcap`.

Các filter được dùng gồm:

- `host`
- `port`
- `protocol`
- `src`
- `dst`
- `udp`
- `tcp`
- DNS filter
- HTTP/HTTPS filter

Mục tiêu điều tra:

```text
Xác định server nào đang trả lời DNS request.
Xác định server nào đang trả lời HTTP/HTTPS request.
Hiểu các conversation trong PCAP.
```

---

# 2. Bối cảnh

Sau khi chứng minh có thể capture network traffic cho Corporation, team được giao nhiệm vụ phân tích nhanh traffic đã thu thập trong quá trình khảo sát network.

Câu hỏi chính:

```text
Trong local network, server nào đang trả lời DNS?
Server nào đang trả lời HTTP/HTTPS?
Có những protocol và port nào xuất hiện?
Có conversation nào đáng chú ý?
```

---

# 3. Task 1: Read a capture without filters

Bước đầu tiên là đọc PCAP không áp dụng filter.

## Command

```bash
tcpdump -r file.pcap
```

Ý nghĩa:

```text
Đọc toàn bộ PCAP và xem traffic thô trước khi lọc.
```

Mục tiêu:

- Có cái nhìn tổng quan.
- Xem protocol nào xuất hiện.
- Xem host nào giao tiếp.
- Xem port nào được dùng.
- Chuẩn bị filter phù hợp cho bước sau.

---

# 4. Task 2: Identify traffic types

Sau khi đọc PCAP, cần ghi nhận loại traffic đang thấy.

Câu hỏi cần trả lời:

```text
Có protocol nào xuất hiện?
Có port nào được dùng?
Có DNS không?
Có HTTP/HTTPS không?
Có traffic nào bất thường không?
```

## Kết quả mong đợi

Common protocols:

```text
DNS
HTTP
HTTPS
```

Ports utilized:

```text
53
80
443
```

---

# 5. Filter giúp nhận diện traffic dễ hơn

Một số filter cơ bản:

## DNS

```bash
tcpdump -r file.pcap port 53
```

## HTTP

```bash
tcpdump -r file.pcap tcp port 80
```

## HTTPS

```bash
tcpdump -r file.pcap tcp port 443
```

## HTTP hoặc HTTPS

```bash
tcpdump -r file.pcap 'tcp port 80 or tcp port 443'
```

## Không resolve hostname/port

```bash
tcpdump -nnr file.pcap
```

---

# 6. Task 3: Identify conversations

Sau khi hiểu traffic cơ bản, cần xác định các conversation.

Cần quan sát:

- Client IP.
- Server IP.
- Client port.
- Server port.
- TCP three-way handshake.
- Well-known ports.
- Random high ports.

---

## 6.1. Client và server port

Thông thường:

```text
Server dùng well-known port.
Client dùng random high port.
```

Ví dụ:

```text
Client: 192.168.1.20:51544
Server: 93.184.216.34:80
```

Trong ví dụ này:

```text
51544 = client ephemeral port
80 = server HTTP port
```

---

## 6.2. Bật absolute sequence numbers

Để xem conversation rõ hơn:

```bash
tcpdump -Sr file.pcap
```

Hoặc kết hợp không resolve:

```bash
tcpdump -nnSr file.pcap
```

`-S` giúp hiển thị absolute sequence numbers, hữu ích khi cần đối chiếu conversation hoặc handshake.

---

# 7. TCP three-way handshake

Một TCP conversation đầy đủ thường bắt đầu bằng:

```text
SYN
SYN/ACK
ACK
```

Mẫu:

```text
Client → Server : SYN
Server → Client : SYN/ACK
Client → Server : ACK
```

Câu hỏi trong lab:

```text
Client và server port numbers trong first full TCP three-way handshake là gì?
```

Cách xác định:

- Tìm packet đầu tiên có `Flags [S]`.
- Tìm packet tiếp theo từ server có `Flags [S.]`.
- Tìm ACK từ client có `Flags [.]`.
- Xem source/destination IP và port.

---

# 8. Task 4: Interpret the capture in depth

Sau khi xác định conversation, cần trả lời:

```text
Timestamp của first established conversation là gì?
IP address của apache.org từ DNS response là gì?
Protocol được dùng trong first conversation là gì?
```

---

## 8.1. Tìm timestamp của first established conversation

Command gợi ý:

```bash
tcpdump -nnSr file.pcap
```

Cách làm:

```text
1. Tìm TCP full handshake đầu tiên.
2. Lấy timestamp ở field đầu tiên của packet SYN.
3. Xác nhận có SYN/ACK và ACK ngay sau đó.
```

---

## 8.2. Tìm IP của apache.org từ DNS response

Có thể lọc DNS:

```bash
tcpdump -nnr file.pcap port 53
```

Hoặc xem ASCII/hex để thấy domain:

```bash
tcpdump -nnXr file.pcap port 53
```

Cần tìm DNS response chứa:

```text
apache.org
A record
```

A record sẽ trả về IPv4 address của domain.

---

## 8.3. Xác định protocol trong first conversation

Nếu dùng `-nn`, tcpdump sẽ hiện port number:

```text
80
443
53
```

Nếu không dùng `-nn`, tcpdump có thể hiện service name:

```text
http
https
domain
```

Ví dụ:

```text
tcp port 80 = HTTP
tcp port 443 = HTTPS
udp/tcp port 53 = DNS
```

---

# 9. Task 5: Filter out non-DNS traffic

Để chỉ xem DNS traffic:

```bash
tcpdump -r file.pcap udp and port 53
```

Lưu ý:

```text
DNS chủ yếu dùng UDP 53.
Nhưng DNS cũng có thể dùng TCP 53 cho zone transfer hoặc response lớn.
```

Vì vậy có thể dùng:

```bash
tcpdump -r file.pcap port 53
```

để không bỏ sót TCP DNS.

---

## 9.1. Xem DNS clear-text bằng -X

```bash
tcpdump -nnXr file.pcap port 53
```

`-X` giúp xem hex và ASCII, có thể thấy domain name trong DNS query/response.

---

## 9.2. Câu hỏi cần trả lời với DNS

```text
DNS server là ai?
Domain nào được request?
Loại DNS record nào xuất hiện?
Ai request A record cho apache.org?
A record cung cấp thông tin gì?
DNS server nào phản hồi?
```

---

# 10. DNS concepts cần nhớ

## A Record

```text
A record map domain name sang IPv4 address.
```

Ví dụ:

```text
apache.org → 192.0.2.10
```

## AAAA Record

```text
AAAA record map domain name sang IPv6 address.
```

## PTR Record

```text
PTR record dùng cho reverse DNS lookup.
```

Ví dụ:

```text
8.8.8.8 → dns.google
```

---

# 11. Task 6: Filter for TCP web traffic

Để tìm HTTP/HTTPS traffic:

```bash
tcpdump -r file.pcap 'tcp port 80 or tcp port 443'
```

Hoặc:

```bash
tcpdump -nnr file.pcap 'tcp port 80 or tcp port 443'
```

Cần trả lời:

```text
Web pages nào được request?
HTTP request methods phổ biến nhất là gì?
HTTP response phổ biến nhất là gì?
```

---

## 11.1. HTTP methods thường gặp

| Method | Ý nghĩa |
|---|---|
| GET | Lấy resource |
| POST | Gửi dữ liệu |
| HEAD | Lấy header, không lấy body |
| OPTIONS | Hỏi method được hỗ trợ |
| PUT | Upload/update resource |
| DELETE | Xóa resource |

Trong PCAP web browsing bình thường, method thường gặp nhất thường là:

```text
GET
```

---

## 11.2. HTTP responses thường gặp

| Status | Ý nghĩa |
|---|---|
| 200 OK | Request thành công |
| 301/302 | Redirect |
| 304 | Not Modified |
| 400 | Bad Request |
| 403 | Forbidden |
| 404 | Not Found |
| 500 | Server Error |

Trong traffic web bình thường, response phổ biến thường là:

```text
HTTP/1.1 200 OK
```

---

# 12. Task 7: Determine information about the first webserver

Cần phân tích sâu conversation đầu tiên.

Command nên dùng:

```bash
tcpdump -nnXr file.pcap 'tcp port 80 or tcp port 443'
```

Hoặc với ASCII:

```bash
tcpdump -nnAr file.pcap 'tcp port 80'
```

Lưu ý:

```text
HTTPS được mã hóa nên không đọc được nội dung HTTP nếu không có key.
HTTP port 80 clear-text có thể đọc request/response bằng -A hoặc -X.
```

---

## 12.1. Tìm webserver application

Trong HTTP response, server có thể gửi header:

```text
Server: Apache
Server: nginx
Server: Microsoft-IIS
X-Powered-By: PHP
```

Với tcpdump, cần xem ASCII output:

```bash
tcpdump -Ar file.pcap tcp port 80
```

Cần tìm các string như:

```text
Server:
X-Powered-By:
Apache
nginx
IIS
```

---

# 13. Tips for analysis

Khi phân tích PCAP bằng tcpdump, tự hỏi:

```text
Traffic type nào đang xuất hiện?
Protocol/port nào đang được dùng?
Có bao nhiêu conversation?
Có bao nhiêu unique hosts?
Timestamp của first TCP conversation là gì?
Traffic nào có thể filter out để sạch hơn?
Server nào đang trả lời trên well-known ports?
DNS records nào được request?
HTTP methods nào được dùng?
HTTP responses nào xuất hiện?
```

---

# 14. Quick command checklist

Đọc PCAP không filter:

```bash
tcpdump -r file.pcap
```

Đọc PCAP không resolve:

```bash
tcpdump -nnr file.pcap
```

Đọc với absolute sequence:

```bash
tcpdump -nnSr file.pcap
```

Lọc DNS:

```bash
tcpdump -nnr file.pcap port 53
```

Lọc UDP DNS:

```bash
tcpdump -nnr file.pcap udp and port 53
```

Lọc HTTP/HTTPS:

```bash
tcpdump -nnr file.pcap 'tcp port 80 or tcp port 443'
```

Xem HTTP ASCII:

```bash
tcpdump -nnAr file.pcap tcp port 80
```

Xem hex/ASCII:

```bash
tcpdump -nnXr file.pcap
```

---

# 15. Analyst workflow

```text
1. Open full PCAP
2. Identify protocols and ports
3. Count/observe hosts
4. Identify conversations
5. Find first TCP handshake
6. Filter DNS traffic
7. Extract requested domains and record types
8. Filter HTTP/HTTPS traffic
9. Identify methods, responses and web servers
10. Summarize findings
```

---

# Key Takeaway

Lab này rèn kỹ năng dùng tcpdump để đọc và lọc PCAP theo hướng điều tra.

```text
Không chỉ biết chạy tcpdump,
mà phải biết đặt câu hỏi:
ai là client, ai là server, port nào được dùng, DNS trả lời gì, web server phản hồi gì và traffic nào cần lọc ra.
```

Khi làm SOC/NTA, capture filters và display filters giúp analyst chuyển một PCAP nhiều noise thành evidence rõ ràng về DNS server, web server, HTTP methods, DNS records và các conversation quan trọng.

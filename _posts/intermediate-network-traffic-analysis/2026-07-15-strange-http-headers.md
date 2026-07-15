---
layout: post
title: "Strange HTTP Headers"
date: 2026-07-15 00:00:16 +0700
categories: ["Intermediate Network Traffic Analysis", "Web Traffic Analysis"]
tags: [http, http-headers, host-header-manipulation, request-smuggling, crlf-injection, apache, reverse-proxy, wireshark, waf, splunk, elk, sigma, threat-hunting, incident-response, cdsa]
description: "Phân tích HTTP header bất thường: Host header manipulation, unusual methods, User-Agent anomalies, HTTP 400, CRLF/request smuggling, detection và hardening."
toc: true
---

# Strange HTTP Headers

Bài này thuộc module **Intermediate Network Traffic Analysis**, tập trung vào việc phát hiện các HTTP request không tạo lượng traffic lớn như fuzzing nhưng có dấu hiệu bất thường trong header, method hoặc cấu trúc request.

```text
Mục tiêu: phát hiện Host header manipulation, CRLF injection,
HTTP request smuggling, unusual methods và response-code anomalies.
```

> Chỉ thực hành trong HTB, lab hoặc ứng dụng có ủy quyền rõ ràng. Không thử request smuggling hoặc header injection trên hệ thống ngoài phạm vi được phép.

---

## Timeline cập nhật

```text
Created at: 2026-07-15 00:00:16 +0700
Timezone: Asia/Ho_Chi_Minh
Module: Intermediate Network Traffic Analysis
Section: Strange HTTP Headers
Focus: Host header manipulation, CRLF, request smuggling, HTTP 400 analysis
```

---

# 1. Vì sao HTTP Header quan trọng?

Một request HTTP không chỉ gồm URI mà còn chứa các header mô tả:

- Host.
- User-Agent.
- Content-Length.
- Transfer-Encoding.
- Cookie.
- Authorization.
- X-Forwarded-For.
- Connection.
- Referer.
- Accept.

Attacker có thể sửa các trường này để:

- Bypass virtual host controls.
- Truy cập backend nội bộ.
- Poison cache.
- Làm proxy/frontend và backend hiểu request khác nhau.
- Chèn thêm request.
- Né WAF.
- Giả client hợp lệ.

---

# 2. Những dấu hiệu header bất thường

Các dấu hiệu đáng chú ý:

- `Host` không khớp domain/IP hợp lệ.
- `Host: 127.0.0.1`.
- `Host: localhost`.
- `Host: admin`.
- Host chứa port nội bộ như `127.0.0.1:8080`.
- HTTP method hiếm hoặc không được ứng dụng sử dụng.
- User-Agent thay đổi liên tục.
- User-Agent trống hoặc tool-specific.
- Cùng request có cả `Content-Length` và `Transfer-Encoding`.
- Header lặp lại nhiều lần.
- Header chứa `%0d%0a`.
- Nhiều response `400 Bad Request`.

---

# 3. Lọc HTTP trong Wireshark

Tất cả HTTP:

```wireshark
http
```

Chỉ request:

```wireshark
http.request
```

Chỉ response:

```wireshark
http.response
```

Theo web server:

```wireshark
ip.addr == 192.168.10.7 && http
```

---

# 4. Tìm Host Header bất thường

Giả sử Host hợp lệ là:

```text
192.168.10.7
```

Lọc request có Host khác:

```wireshark
http.request &&
!(http.host == "192.168.10.7")
```

Nếu dùng domain:

```wireshark
http.request &&
!(http.host == "portal.example.local")
```

Các kết quả như sau cần điều tra:

```text
Host: 127.0.0.1
Host: localhost
Host: admin
Host: 127.0.0.1:8080
```

---

# 5. Host Header Manipulation

Attacker sửa `Host` để thử:

- Truy cập virtual host nội bộ.
- Bypass access control theo hostname.
- Truy cập admin panel chỉ bind localhost.
- Cache poisoning.
- Password-reset poisoning.
- Backend routing abuse.
- SSRF-like proxy behavior.

Ví dụ:

```http
GET /login.php HTTP/1.1
Host: 127.0.0.1:8080
```

Nếu reverse proxy tin header này, request có thể được chuyển đến backend nội bộ.

---

# 6. Virtual Host và Reverse Proxy Risk

Một cấu hình proxy không chặt chẽ có thể dẫn đến:

```text
Client request
→ Apache/Nginx frontend
→ rewrite/proxy rule
→ backend nội bộ
```

Nếu frontend và backend xử lý header khác nhau, attacker có thể lợi dụng sự không đồng nhất này.

Ví dụ cấu hình có proxy rewrite:

```apache
<VirtualHost *:80>
    RewriteEngine On
    RewriteRule "^/categories/(.*)"       "http://192.168.10.100:8080/categories.php?id=$1" [P]
    ProxyPassReverse "/categories/"       "http://192.168.10.100:8080/"
</VirtualHost>
```

Cần review kỹ:

- Rewrite rules.
- ProxyPass.
- Backend URL parsing.
- Header forwarding.
- Canonical host validation.

---

# 7. Unusual HTTP Methods

Các method phổ biến:

```text
GET
POST
HEAD
PUT
DELETE
OPTIONS
PATCH
TRACE
CONNECT
```

Trong nhiều ứng dụng, chỉ `GET` và `POST` là hợp lệ.

Filter method khác GET/POST:

```wireshark
http.request &&
!(http.request.method == "GET") &&
!(http.request.method == "POST")
```

TRACE:

```wireshark
http.request.method == "TRACE"
```

CONNECT:

```wireshark
http.request.method == "CONNECT"
```

PUT/DELETE bất thường:

```wireshark
http.request.method == "PUT" ||
http.request.method == "DELETE"
```

---

# 8. User-Agent bất thường

Filter User-Agent:

```wireshark
http.user_agent
```

Theo một User-Agent cụ thể:

```wireshark
http.user_agent contains "curl"
```

Dấu hiệu:

- Trống.
- Thay đổi sau mỗi request.
- Không phù hợp endpoint.
- Scripted client.
- Browser version rất cũ.
- User-Agent giống nhau trên distributed sources.

Tuy nhiên, User-Agent dễ giả mạo nên chỉ là một feature hỗ trợ.

---

# 9. Phân tích HTTP 400

`400 Bad Request` thường xuất hiện khi:

- Request malformed.
- Header không hợp lệ.
- CRLF injection.
- Request smuggling attempt.
- Unsupported encoding.
- Parser disagreement.
- Header quá dài.

Filter:

```wireshark
http.response.code == 400
```

Một spike `400` từ cùng source là điểm bắt đầu tốt để điều tra.

---

# 10. CRLF là gì?

CRLF gồm:

```text
CR = Carriage Return = \r = 0x0d
LF = Line Feed       = \n = 0x0a
```

Trong URL encoding:

```text
%0d%0a
```

HTTP sử dụng CRLF để phân tách các dòng header. Nếu input không được kiểm soát, attacker có thể cố chèn thêm header hoặc request.

---

# 11. Ví dụ CRLF/Request Injection

Payload encode:

```text
GET%20%2flogin.php%3fid%3d1%20HTTP%2f1.1
%0d%0aHost%3a%20192.168.10.5
%0d%0a%0d%0a
GET%20%2fuploads%2fcmd2.php%20HTTP%2f1.1
%0d%0aHost%3a%20127.0.0.1%3a8080
%0d%0a%0d%0a
```

Sau khi decode:

```http
GET /login.php?id=1 HTTP/1.1
Host: 192.168.10.5

GET /uploads/cmd2.php HTTP/1.1
Host: 127.0.0.1:8080
```

Ý tưởng là làm proxy hoặc backend hiểu thành nhiều request.

---

# 12. HTTP Request Smuggling

Request smuggling xảy ra khi các thành phần trong chuỗi xử lý HTTP không đồng ý về ranh giới request.

Ví dụ:

```text
Client
→ CDN/WAF
→ Reverse Proxy
→ Backend
```

Frontend có thể hiểu request theo `Content-Length`, còn backend hiểu theo `Transfer-Encoding`.

Kết quả có thể là:

- Chèn request thứ hai.
- Bypass WAF.
- Truy cập route nội bộ.
- Cache poisoning.
- Session confusion.
- Credential theft.

---

# 13. CL.TE và TE.CL

Hai pattern cổ điển:

## CL.TE

```text
Frontend dùng Content-Length
Backend dùng Transfer-Encoding
```

## TE.CL

```text
Frontend dùng Transfer-Encoding
Backend dùng Content-Length
```

Dấu hiệu:

```http
Content-Length: 4
Transfer-Encoding: chunked
```

Cùng request chứa cả hai header cần được điều tra.

---

# 14. Wireshark Filters cho Header Smuggling

Host khác whitelist:

```wireshark
http.request &&
!(http.host == "192.168.10.7")
```

URI chứa CRLF encoded:

```wireshark
http.request.uri contains "%0d%0a"
```

Transfer-Encoding:

```wireshark
http.transfer_encoding
```

Content-Length:

```wireshark
http.content_length_header
```

Response 400:

```wireshark
http.response.code == 400
```

Response thành công đáng ngờ:

```wireshark
http.response.code == 200
```

sau một request có header hoặc URI bất thường.

---

# 15. Follow HTTP Stream

Trong Wireshark:

```text
Right click
→ Follow
→ HTTP Stream
```

Kiểm tra:

- Có nhiều request trong một payload không.
- Host header có thay đổi không.
- Có `%0d%0a` không.
- Có cả `Content-Length` và `Transfer-Encoding` không.
- Backend trả `200` cho request thứ hai không.
- Response code thay đổi từ `400` sang `200` không.

---

# 16. Dấu hiệu Attack Thành công

Các dấu hiệu:

- Request đầu trả `400`, request sau trả `200`.
- URI nội bộ hoặc admin trả nội dung.
- Response từ backend khác bình thường.
- Access log ghi hai request nhưng frontend chỉ thấy một.
- Cache chứa response sai.
- Một user nhận response của user khác.
- Backend log có Host `127.0.0.1`.

---

# 17. Access Log Hunting

Apache:

```bash
grep '192.168.10.5' access.log
```

Host bất thường nếu log format có `%{Host}i`:

```bash
grep -Ei '127\.0\.0\.1|localhost|admin' access.log
```

Tìm 400:

```bash
awk '$9 == 400 {print $1, $7, $9}' access.log
```

Đếm 400 theo source:

```bash
awk '$9 == 400 {print $1}' access.log |
sort | uniq -c | sort -nr
```

Tìm `%0d%0a`:

```bash
grep -Ei '%0d%0a|%0a|%0d' access.log
```

---

# 18. Reverse Proxy Correlation

Cần so sánh:

```text
Frontend access log
Backend access log
WAF log
Application log
```

Dấu hiệu parser disagreement:

```text
Frontend ghi 1 request
Backend ghi 2 requests
```

Hoặc:

```text
Frontend Host hợp lệ
Backend Host là 127.0.0.1
```

Đây là evidence mạnh cho request smuggling hoặc header manipulation.

---

# 19. Splunk Detection

Host header bất thường:

```spl
index=web
| where NOT host_header IN (
    "portal.example.local",
    "192.168.10.7"
  )
| stats count values(uri_path) by src_ip, host_header
```

Nhiều HTTP 400:

```spl
index=web status=400
| stats count dc(uri_path) as unique_paths by src_ip
| where count > 20
```

CRLF encoded:

```spl
index=web
(uri_query="*%0d%0a*" OR uri_path="*%0d%0a*")
| table _time src_ip method host_header uri status
```

Host localhost:

```spl
index=web
(host_header="127.0.0.1*" OR host_header="localhost*" OR host_header="admin*")
| stats count values(uri) by src_ip, host_header
```

---

# 20. Elastic/KQL

Host bất thường:

```kql
http.request.headers.host:
  ("127.0.0.1*" or "localhost*" or "admin*")
```

HTTP 400:

```kql
http.response.status_code: 400
```

CRLF:

```kql
url.original: "*%0d%0a*"
```

Unusual methods:

```kql
http.request.method:
  ("TRACE" or "CONNECT" or "PUT" or "DELETE")
```

---

# 21. Sigma Detection Idea

```yaml
title: Suspicious HTTP Host Header Manipulation
status: experimental
logsource:
  category: webserver
detection:
  selection:
    cs-host|startswith:
      - '127.0.0.1'
      - 'localhost'
      - 'admin'
  condition: selection
falsepositives:
  - Internal health checks
  - Reverse proxy monitoring
level: high
```

CRLF concept:

```yaml
title: Encoded CRLF in HTTP Request
status: experimental
logsource:
  category: webserver
detection:
  selection:
    cs-uri-query|contains:
      - '%0d%0a'
      - '%0a'
      - '%0d'
  condition: selection
level: high
```

Field names phải điều chỉnh theo SIEM schema.

---

# 22. WAF Detection

WAF nên kiểm tra:

- Host whitelist.
- Multiple Host headers.
- Conflicting length headers.
- `%0d%0a`.
- Invalid chunked encoding.
- Header length.
- Unusual methods.
- Duplicate headers.
- Backend-only hostnames.

Các action:

- Block.
- Normalize.
- Log.
- Rate-limit.
- Challenge.
- Close connection.

---

# 23. Hardening

## Web Server

- Chỉ cho phép Host hợp lệ.
- Tạo default virtual host trả lỗi.
- Disable TRACE.
- Restrict CONNECT/PUT/DELETE.
- Cập nhật Apache/Nginx.
- Không proxy dựa trên user-controlled Host.
- Validate rewrite/proxy targets.

## Reverse Proxy

- Normalize headers.
- Reject ambiguous requests.
- Không forward duplicate Host.
- Reject simultaneous `Content-Length` và `Transfer-Encoding`.
- Đồng nhất parser giữa frontend/backend.
- Không expose backend nội bộ.

---

# 24. Apache Example

Default vhost:

```apache
<VirtualHost *:80>
    ServerName default.invalid
    Redirect 404 /
</VirtualHost>
```

Approved host:

```apache
<VirtualHost *:80>
    ServerName portal.example.local
    UseCanonicalName On
</VirtualHost>
```

Có thể bổ sung rewrite/authorization tùy kiến trúc, nhưng cần test tránh ảnh hưởng reverse proxy hợp lệ.

---

# 25. Incident Response Checklist

```text
[ ] Xác định source IP/session.
[ ] Thu request raw headers.
[ ] Kiểm tra Host, method và User-Agent.
[ ] Tìm %0d%0a hoặc duplicate headers.
[ ] Kiểm tra 400/200 sequence.
[ ] So sánh frontend và backend logs.
[ ] Xác định backend route bị truy cập.
[ ] Kiểm tra file/webshell hoặc admin action.
[ ] Block source và patch cấu hình.
[ ] Thu PCAP/logs và preserve evidence.
```

---

# 26. Threat Hunting

Hunt theo:

- Host không thuộc whitelist.
- Host `127.0.0.1`, `localhost`, `admin`.
- Nhiều HTTP 400.
- `%0d%0a`.
- Conflicting `Content-Length`/`Transfer-Encoding`.
- TRACE/CONNECT bất thường.
- Frontend/backend request-count mismatch.
- Request 400 theo sau bởi 200.
- Backend nội bộ nhận traffic ngoài baseline.

---

# 27. Mapping với CDSA

## Security Operations & Monitoring

- Monitor Host header.
- Correlate WAF/reverse proxy/backend.
- Alert 400 burst và unusual methods.

## Incident Response & Forensics

- Preserve raw request.
- Reconstruct frontend/backend path.
- Kiểm tra backend access và webshell.

## Threat Hunting

- Hunt CRLF encoding.
- Hunt localhost/admin Host.
- Hunt parser disagreement.
- Hunt response anomalies.

## Detection Engineering

- Sigma cho web logs.
- Splunk/Elastic aggregation.
- WAF rule cho host whitelist.
- Reverse-proxy normalization.

---

# 28. CVE-2023-25690 Context

Module minh họa một trường hợp request smuggling liên quan Apache HTTP Server khi sử dụng một số cấu hình `mod_proxy` và `mod_rewrite`.

Điểm phòng thủ quan trọng:

- Cập nhật Apache.
- Review rewrite rules.
- Không đưa input không tin cậy vào proxy target.
- So sánh frontend/backend logs.
- Kiểm tra các request chứa CRLF hoặc backend host nội bộ.

---

# 29. Lab Setup

Môi trường tối thiểu:

```text
1 Apache/Nginx frontend
1 backend web server
1 Wireshark/tshark sensor
WAF hoặc reverse proxy
Splunk/ELK/Wazuh nhận web logs
```

Workflow:

```text
1. Mở CRLF_and_host_header_manipulation.pcapng.
2. Lọc http.
3. Loại Host hợp lệ.
4. Kiểm tra response 400.
5. Follow HTTP Stream.
6. Tìm CRLF hoặc nhiều request.
7. So sánh frontend/backend logs.
8. Viết detection và hardening.
```

---

# Key Takeaway

```text
Không có fuzzing lớn không đồng nghĩa traffic web an toàn.
Host header khác whitelist, HTTP 400 và CRLF encoded là các dấu hiệu quan trọng.
Request smuggling thường dựa vào sự khác biệt parser giữa frontend và backend.
Detection hiệu quả cần correlate PCAP, WAF, reverse proxy và backend access logs.
```

Strange HTTP Headers là chủ đề quan trọng của CDSA vì kết hợp packet analysis, web-server telemetry, reverse-proxy behavior, detection engineering và incident response.

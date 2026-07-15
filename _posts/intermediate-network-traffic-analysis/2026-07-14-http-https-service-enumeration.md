---
layout: post
title: "HTTP/HTTPS Service Enumeration"
date: 2026-07-14 23:51:10 +0700
categories: ["Intermediate Network Traffic Analysis", "Web Traffic Analysis"]
tags: [http, https, web-enumeration, directory-fuzzing, content-discovery, idor, access-logs, wireshark, apache, nginx, waf, splunk, elk, sigma, threat-hunting, incident-response, cdsa]
description: "Phân tích HTTP/HTTPS service enumeration: directory fuzzing, 404 bursts, parameter and IDOR probing, Wireshark filters, access-log hunting, SIEM detection và WAF hardening."
toc: true
---

# HTTP/HTTPS Service Enumeration

Bài này thuộc module **Intermediate Network Traffic Analysis**, tập trung vào cách nhận diện hành vi enumeration và fuzzing nhắm vào web server thông qua PCAP, access logs, WAF và SIEM.

```text
Mục tiêu: phát hiện directory fuzzing, parameter probing,
IDOR enumeration, low-and-slow scanning và distributed web enumeration.
```

> Chỉ thực hành trong HTB, lab hoặc ứng dụng có ủy quyền rõ ràng. Không fuzz hoặc enumerate website ngoài phạm vi được phép.

---

## Timeline cập nhật

```text
Created at: 2026-07-14 23:51:10 +0700
Timezone: Asia/Ho_Chi_Minh
Module: Intermediate Network Traffic Analysis
Section: HTTP/HTTPS Service Enumeration
Focus: Directory fuzzing, IDOR probing, access-log analysis, WAF and SIEM detection
```

---

# 1. HTTP/HTTPS Enumeration là gì?

Trước khi khai thác web application, attacker thường thực hiện reconnaissance để tìm:

- Thư mục ẩn.
- File cấu hình.
- Backup files.
- Git metadata.
- Admin panels.
- API endpoints.
- Parameters.
- Numeric object identifiers.
- Virtual hosts.
- Framework fingerprints.

Hành vi này thường tạo nhiều HTTP requests trong thời gian ngắn.

---

# 2. Dấu hiệu tổng quát

Các dấu hiệu thường gặp:

- Một source tạo lượng HTTP requests lớn.
- Nhiều URI khác nhau trong thời gian ngắn.
- Tỷ lệ `404`, `403` hoặc `400` cao.
- User-Agent lặp lại trên hàng trăm requests.
- Request interval đều như automation.
- Truy cập các file nhạy cảm hoặc dotfiles.
- Một source tăng ID hoặc parameter tuần tự.
- Nhiều source cùng probe một URI pattern.

---

# 3. Directory Fuzzing

Directory fuzzing thử nhiều path để tìm tài nguyên tồn tại.

Ví dụ URI đáng ngờ:

```text
/.git/HEAD
/.env
/.bash_history
/.config
/admin
/backup.zip
/config.php
/server-status
```

Pattern thường gặp:

```text
Một host
→ nhiều path
→ rapid succession
→ phần lớn trả 404/403
```

---

# 4. Wireshark Filters

Tất cả HTTP:

```wireshark
http
```

Chỉ requests:

```wireshark
http.request
```

Chỉ GET:

```wireshark
http.request.method == "GET"
```

Theo source nghi vấn:

```wireshark
http.request &&
ip.src == 192.168.10.5
```

Theo một cặp host:

```wireshark
http.request &&
(
  ip.src == 192.168.10.5 ||
  ip.dst == 192.168.10.5
)
```

---

# 5. Lọc Response Codes

404 Not Found:

```wireshark
http.response.code == 404
```

403 Forbidden:

```wireshark
http.response.code == 403
```

Nhiều mã lỗi:

```wireshark
http.response.code == 400 ||
http.response.code == 403 ||
http.response.code == 404
```

Khi thấy nhiều `404` liên tiếp từ cùng source, cần correlate với URI và request rate.

---

# 6. HTTP và HTTPS khác nhau khi phân tích

Với HTTP, Wireshark thường nhìn thấy trực tiếp:

- Method.
- URI.
- Headers.
- User-Agent.
- Host.
- Response code.
- Payload.

Với HTTPS, nếu không có TLS decryption, PCAP thường chỉ thấy:

- Source/destination IP.
- Port.
- TLS handshake.
- SNI nếu không bị mã hóa.
- Certificate.
- Packet size/timing.
- Connection frequency.

Do đó, với HTTPS enumeration, access logs, reverse proxy, WAF và application logs thường quan trọng hơn PCAP.

---

# 7. Apache Access Log Hunting

Theo source IP:

```bash
grep '192.168.10.5' access.log
```

Dùng `awk`:

```bash
awk '$1 == "192.168.10.5"' access.log
```

Đếm status code của source:

```bash
awk '$1 == "192.168.10.5" {print $9}' access.log |
sort | uniq -c | sort -nr
```

Đếm URI được yêu cầu:

```bash
awk '$1 == "192.168.10.5" {print $7}' access.log |
sort | uniq -c | sort -nr
```

---

# 8. Tìm Source tạo nhiều 404

```bash
awk '$9 == 404 {print $1}' access.log |
sort | uniq -c | sort -nr | head
```

Liệt kê URI trả 404 nhiều nhất:

```bash
awk '$9 == 404 {print $7}' access.log |
sort | uniq -c | sort -nr | head
```

Một source có hàng trăm `404` trong vài giây là chỉ báo fuzzing mạnh.

---

# 9. Nginx Access Log Hunting

Các lệnh tương tự có thể áp dụng với Nginx nếu dùng combined log format:

```bash
grep '192.168.10.5' /var/log/nginx/access.log
```

Đếm 404:

```bash
awk '$9 == 404 {print $1}' /var/log/nginx/access.log |
sort | uniq -c | sort -nr
```

---

# 10. Parameter Fuzzing

Attacker có thể thử nhiều giá trị parameter:

```text
?id=1
?id=2
?id=3
?id=4
```

hoặc:

```text
?return=min
?return=max
?role=admin
?debug=true
```

Dấu hiệu:

- URI giống nhau.
- Chỉ parameter thay đổi.
- Giá trị tăng tuần tự.
- Request rate nhanh.
- Response code/length thay đổi theo một vài giá trị.

---

# 11. IDOR Enumeration

IDOR probing thường nhắm vào object identifiers:

```text
/users/1
/users/2
/users/3
/orders/1001
/orders/1002
```

Dấu hiệu:

```text
Một session hoặc source
→ truy cập nhiều object ID
→ ID tăng đều
→ response lengths thay đổi
```

Điều quan trọng là phân biệt:

- Pagination hợp lệ.
- API client bình thường.
- Automation được phép.
- Enumeration trái phép.

---

# 12. Wireshark Filter theo URI

Ví dụ ID parameter:

```wireshark
http.request.uri contains "id="
```

Admin path:

```wireshark
http.request.uri contains "/admin"
```

Dotfiles:

```wireshark
http.request.uri contains "/."
```

Git metadata:

```wireshark
http.request.uri contains "/.git/"
```

Theo Host header:

```wireshark
http.host == "example.local"
```

---

# 13. Follow HTTP Stream

Trong Wireshark:

```text
Right click request
→ Follow
→ HTTP Stream
```

Điều này giúp xem:

- Request đầy đủ.
- Response body.
- Status code.
- Redirect.
- Content length.
- Parameter value.
- Error message.

Cần tránh công khai token, credential hoặc session cookie trong báo cáo.

---

# 14. Response Length Analysis

Fuzzer có thể nhận diện resource dựa trên:

- Status code.
- Response size.
- Word count.
- Line count.
- Redirect location.

SOC cũng có thể dùng các đặc trưng này để phát hiện automation.

Ví dụ:

```text
Hàng trăm request
Response code chủ yếu 404
Response length giống nhau
Một số response 200/301 có length khác biệt
```

Các response khác biệt có thể là resource attacker đã tìm thấy.

---

# 15. User-Agent Analysis

Dấu hiệu:

- Một User-Agent lặp lại trên lượng requests lớn.
- User-Agent cũ hoặc bất thường.
- User-Agent rỗng.
- User-Agent tool-specific.
- Header order giống nhau trong mọi requests.

Tuy nhiên, attacker có thể giả User-Agent trình duyệt thông thường, nên không dùng riêng tiêu chí này.

---

# 16. Low-and-Slow Fuzzing

Attacker có thể tránh threshold bằng cách:

- Giãn requests trong nhiều phút hoặc giờ.
- Giảm request rate.
- Chỉ probe vào giờ làm việc.
- Chèn requests hợp lệ giữa các probe.

Detection cần cửa sổ thời gian dài hơn:

```text
Unique URI count theo source trong 1 giờ
404 ratio theo source trong 24 giờ
Rare URI attempts theo user/session
```

---

# 17. Distributed Fuzzing

Attacker có thể dùng nhiều source IP:

```text
Nhiều source
→ cùng wordlist/path pattern
→ cùng target
```

Detection không nên chỉ group theo source IP.

Có thể correlate theo:

- User-Agent.
- URI order.
- Header fingerprint.
- TLS fingerprint.
- ASN/geolocation.
- Session cookie.
- Request timing.

---

# 18. Tshark Hunt

Liệt kê HTTP requests:

```bash
tshark -r basic_fuzzing.pcapng   -Y "http.request"   -T fields   -e frame.time   -e ip.src   -e http.host   -e http.request.method   -e http.request.uri
```

Đếm request theo source:

```bash
tshark -r basic_fuzzing.pcapng   -Y "http.request"   -T fields -e ip.src |
sort | uniq -c | sort -nr
```

Liệt kê response codes:

```bash
tshark -r basic_fuzzing.pcapng   -Y "http.response"   -T fields   -e ip.dst   -e http.response.code
```

---

# 19. Detection Logic

Directory fuzzing:

```text
Một source
+ nhiều URI duy nhất
+ request rate cao
+ 404/403 ratio cao
```

IDOR enumeration:

```text
Cùng endpoint
+ object ID tăng tuần tự
+ nhiều object khác nhau
+ cùng user/session/source
```

Distributed fuzzing:

```text
Nhiều source
+ cùng URI order/pattern
+ cùng User-Agent/header fingerprint
```

---

# 20. Splunk Detection

Apache/Nginx log ví dụ:

```spl
index=web sourcetype=access_combined
| stats count
        dc(uri_path) as unique_paths
        count(eval(status=404)) as not_found
        count(eval(status=403)) as forbidden
  by src_ip
| eval error_ratio=(not_found+forbidden)/count
| where count > 50 AND unique_paths > 30 AND error_ratio > 0.7
```

Top source với 404:

```spl
index=web status=404
| stats count dc(uri_path) as unique_paths by src_ip
| sort -count
```

ID enumeration:

```spl
index=web uri_query="*id=*"
| stats count dc(uri_query) as unique_queries by src_ip, uri_path
| where unique_queries > 20
```

---

# 21. Elastic/KQL

HTTP errors:

```kql
http.response.status_code: (403 or 404)
```

Theo source:

```kql
source.ip: "192.168.10.5"
and http.request.method: "GET"
```

Sensitive paths:

```kql
url.path: (
  "/.git/*"
  or "/.env"
  or "/admin*"
  or "/server-status"
)
```

Cần dùng aggregation trong Kibana/Elastic rule để đếm unique paths và error ratio.

---

# 22. Sigma Detection Idea

```yaml
title: Excessive Web Resource Enumeration
status: experimental
logsource:
  category: webserver
detection:
  selection:
    sc-status:
      - 403
      - 404
  condition: selection
falsepositives:
  - Broken application links
  - Vulnerability scanner
  - Search engine crawler
level: medium
```

Threshold nên được triển khai tại SIEM:

```text
Nhiều 403/404
+ nhiều URI duy nhất
+ cùng source/session
```

---

# 23. WAF Detection và Prevention

WAF có thể:

- Rate-limit source.
- Block known malicious paths.
- Detect scanner behavior.
- Challenge suspicious clients.
- Correlate URI entropy.
- Block repeated 404/403 behavior.
- Apply bot management.

Nhưng WAF không thay thế:

- Access-log monitoring.
- Secure application design.
- Authorization checks.
- Asset inventory.
- Patch management.

---

# 24. Trả đúng Response Codes

Web server nên trả mã phản hồi phù hợp:

| Code | Ý nghĩa |
|---:|---|
| 200 | Resource tồn tại |
| 301/302 | Redirect |
| 400 | Bad Request |
| 401 | Authentication required |
| 403 | Forbidden |
| 404 | Not Found |
| 429 | Too Many Requests |

Không nên trả `200 OK` cho mọi path lỗi vì:

- Làm khó monitoring.
- Tạo false positives.
- Giúp attacker phân tích response length.
- Che lỗi cấu hình.

---

# 25. Incident Response Checklist

```text
[ ] Xác định source IP/session/User-Agent.
[ ] Đếm request rate và unique URIs.
[ ] Tính tỷ lệ 403/404.
[ ] Xác định resource nào trả 200/301.
[ ] Kiểm tra attacker có tìm thấy file nhạy cảm không.
[ ] Correlate WAF, reverse proxy và application logs.
[ ] Kiểm tra exploitation sau enumeration.
[ ] Block/rate-limit nếu malicious.
[ ] Thu log và preserve evidence.
[ ] Triage web server nếu có successful access.
```

---

# 26. Threat Hunting

Hunt theo:

- High 404 ratio.
- Unique URI count cao.
- Dotfile probing.
- Sequential object IDs.
- Low-and-slow enumeration.
- Distributed path pattern.
- Rare User-Agent.
- Same response length trên nhiều URI.
- Enumeration theo internal web server không có WAF.

---

# 27. Mapping với CDSA

## Security Operations & Monitoring

- Monitor WAF, access logs và reverse proxy.
- Baseline request rate.
- Alert unique URI và error ratio.

## Incident Response & Forensics

- Xây timeline enumeration.
- Xác định resource đã bị tìm thấy.
- Kiểm tra truy cập/exploitation sau fuzzing.

## Threat Hunting

- Hunt dotfiles và backup files.
- Hunt IDOR probing.
- Hunt low-and-slow hoặc distributed scanning.
- Hunt internal web enumeration.

## Detection Engineering

- Splunk/Elastic threshold.
- Sigma webserver rule.
- WAF rate-limit.
- Enrichment với approved scanner inventory.

---

# 28. Lab Setup

Môi trường tối thiểu:

```text
1 web server Apache/Nginx
1 authorized test host
1 Wireshark/tshark sensor
WAF hoặc reverse proxy
Splunk/ELK/Wazuh nhận access logs
```

Workflow:

```text
1. Mở basic_fuzzing.pcapng.
2. Lọc http.request.
3. Xác định source và URI fan-out.
4. Kiểm tra 403/404 responses.
5. Correlate access.log.
6. Tìm IDOR/parameter pattern.
7. Viết SIEM/WAF detection.
```

---

# Key Takeaway

```text
HTTP/HTTPS enumeration thường tạo nhiều URI requests với 403/404 ratio cao.
Directory fuzzing dễ thấy qua rapid path probing và dotfile requests.
IDOR enumeration thường tạo object IDs tuần tự trên cùng endpoint.
Với HTTPS, access logs và WAF thường cung cấp visibility tốt hơn PCAP chưa giải mã.
Detection hiệu quả phải kết hợp request rate, unique URI, response code và session context.
```

HTTP/HTTPS Service Enumeration là chủ đề quan trọng của CDSA vì kết nối packet analysis, web access logs, WAF telemetry, threat hunting và incident response.

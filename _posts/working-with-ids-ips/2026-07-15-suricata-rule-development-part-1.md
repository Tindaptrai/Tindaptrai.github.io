---
layout: post
title: "Suricata Rule Development Part 1"
date: 2026-07-15 23:05:33 +0700
categories: ["Working with IDS/IPS"]
tags: [suricata, rule-development, ids, ips, pcre, sticky-buffers, detection-filter, powershell-empire, covenant, sliver, detection-engineering, threat-hunting, incident-response, cdsa]
description: "Cấu trúc rule Suricata, content matching, flow, dsize, offset/depth, distance/within, PCRE và các ví dụ phát hiện C2."
toc: true
---

# Suricata Rule Development Part 1

Bài này thuộc module **Working with IDS/IPS**, tập trung vào cách viết rule Suricata chính xác, có thể kiểm thử và dễ tune trong môi trường SOC.

> Chỉ dùng các PCAP và framework trong HTB, lab hoặc hệ thống có ủy quyền. Không chuyển rule sang `drop` trước khi kiểm thử false positive và chuẩn bị rollback.

## Timeline

```text
Created at: 2026-07-15 23:05:33 +0700
Timezone: Asia/Ho_Chi_Minh
Module: Working with IDS/IPS
Section: Suricata Rule Development Part 1
Focus: Rule anatomy, content matching, PCRE, analytics, C2 detection
```

## 1. Mục tiêu của Suricata Rule

Rule là chỉ thị cho detection engine:

```text
Traffic
→ protocol/state validation
→ content hoặc behavior matching
→ alert/pass/drop/reject
```

Rule có thể phát hiện malware, nhưng cũng có thể cung cấp context cho Blue Team như giao thức chạy sai cổng, HTTP beaconing, DNS bất thường hoặc traffic vi phạm policy.

## 2. Rule Anatomy

Cấu trúc:

```suricata
action protocol source_ip source_port direction destination_ip destination_port (
  options;
)
```

Ví dụ:

```suricata
alert http $HOME_NET any -> $EXTERNAL_NET any (
  msg:"LAB Suspicious outbound HTTP";
  flow:established,to_server;
  content:"session=";
  sid:1000001;
  rev:1;
)
```

Header gồm action, protocol, IP, port và direction. Options gồm `msg`, `flow`, `content`, `pcre`, `sid`, `rev` và các modifier khác.

## 3. Action, Protocol và Direction

Action:

| Action | Ý nghĩa |
|---|---|
| `alert` | Tạo cảnh báo |
| `pass` | Bỏ qua traffic khớp rule |
| `drop` | Chặn trong inline IPS |
| `reject` | Chặn và gửi reset/error |
| `log` | Ghi nhận theo output hỗ trợ |

Protocol có thể là:

```text
tcp, udp, icmp, ip, http, tls, dns, smb, ssh
```

Direction:

```suricata
$HOME_NET any -> $EXTERNAL_NET 9090
$EXTERNAL_NET any -> $HOME_NET 8443
$EXTERNAL_NET any <> $HOME_NET any
```

## 4. Port Expressions

```suricata
9443
[8443,8080,7001:7002]
[8443,8080,7001:7002,!8443]
$UNCOMMON_PORTS
```

Port variable có thể khai báo trong `suricata.yaml`.

## 5. flow

Ví dụ:

```suricata
flow:established,to_server;
```

Điều này giới hạn detection vào TCP session đã established và traffic đi tới server.

Các modifier thường gặp:

```text
established, not_established, to_server, to_client,
from_client, from_server, stateless
```

Dùng `flow` giúp giảm false positive và tránh kiểm tra traffic sai hướng.

## 6. dsize

`dsize` kiểm tra payload size, không phải toàn bộ frame size.

```suricata
dsize:312;
dsize:>1000;
dsize:<64;
dsize:100<>500;
```

Ví dụ behavior rule:

```suricata
alert tcp $HOME_NET any -> any any (
  msg:"LAB Repeated 312-byte payload";
  dsize:312;
  detection_filter:track by_src,count 3,seconds 10;
  sid:1001001;
  rev:1;
)
```

`dsize` nên kết hợp protocol, flow, direction và additional content.

## 7. content và Hex Bytes

```suricata
content:"User-Agent";
```

```suricata
content:"User-Agent|3a 20|Go-http-client/1.1|0d 0a|";
```

Trong đó:

```text
|3a 20| = ": "
|0d 0a| = CRLF
```

Content tốt phải đủ đặc trưng, ổn định và không quá phổ biến.

## 8. Sticky Buffers

Sticky buffer giới hạn vùng dữ liệu cần tìm:

```suricata
http.accept;
content:"image/gif";
```

Các buffer hữu ích:

```text
http.method
http.uri
http.header
http.header_names
http.cookie
http.user_agent
http.request_body
http.response_body
```

Lab cũ có thể dùng dạng `http_method`, `http_uri`, `http_cookie`. Khi triển khai thực tế, kiểm tra keyword đúng phiên bản bằng tài liệu tương ứng và:

```bash
suricata --list-keywords
```

## 9. Content Modifiers

Không phân biệt hoa thường:

```suricata
content:"Mozilla";
nocase;
```

Giới hạn vị trí:

```suricata
content:"|01 02 03|";
offset:0;
depth:5;
```

Ràng buộc content thứ hai:

```suricata
content:"GET";
offset:0;
depth:4;

content:"/admin";
distance:1;
within:80;
```

Chọn fast pattern:

```suricata
content:"session=";
fast_pattern;
```

Negative content:

```suricata
content:!"Referer";
content:!"Cache";
content:!"Accept";
```

## 10. Metadata

```suricata
sid:1000001;
rev:1;
reference:url,example.org/report;
classtype:trojan-activity;
priority:1;
```

- `sid`: ID duy nhất.
- `rev`: phiên bản logic của rule.
- `reference`: nguồn threat intelligence.
- `classtype`: phân loại sự kiện.
- `priority`: mức ưu tiên.

Tăng `rev` khi thay đổi logic hoặc tuning.

## 11. PCRE

Ví dụ:

```suricata
pcre:"/^\/api\/(login|admin)\.php\?[a-z_]{1,2}=[a-z0-9]{1,10}$/i";
```

Nguyên tắc quan trọng:

```text
Không xây rule chỉ dựa vào PCRE.
```

Thứ tự tối ưu:

```text
Protocol
→ direction/flow
→ sticky buffer
→ content/fast_pattern
→ additional content
→ PCRE xác nhận cuối
```

PCRE mạnh nhưng tốn CPU và dễ tạo rule khó bảo trì.

## 12. Ba Cách Phát Triển Rule

### Signature-based

Tìm IOC hoặc chuỗi đặc trưng:

- URI.
- Cookie.
- User-Agent.
- Malware string.
- Magic bytes.

Ưu điểm: precision cao với known threat.  
Nhược điểm: dễ bị bypass khi attacker thay đổi indicator.

### Behavior/anomaly-based

Tìm hành vi:

- Beacon interval.
- Payload size lặp lại.
- Port hiếm.
- Data transfer bất thường.
- Request frequency.

Ưu điểm: có thể bắt biến thể mới.  
Nhược điểm: false positive cao hơn.

### Stateful protocol analysis

Theo dõi trạng thái TCP và application transaction:

- HTTP request/response.
- DNS query/answer.
- TLS handshake.
- SMB session.

Cách này cung cấp context tốt hơn raw packet matching.

## 13. Ví Dụ PowerShell Empire

PCAP:

```text
/home/htb-student/pcaps/psempire.pcap
```

Rule trong lab kết hợp:

```text
Outbound HTTP GET
+ URI giống login/process.php, admin/get.php hoặc news.php
+ Cookie session= có dạng Base64
+ User-Agent Windows NT 6.1
+ thiếu một số header browser thông thường
```

Điểm mạnh là không dựa vào một IOC duy nhất.

Test:

```bash
mkdir -p ~/suricata-rule-lab/empire

sudo suricata   -r /home/htb-student/pcaps/psempire.pcap   -k none   -l ~/suricata-rule-lab/empire
```

```bash
jq -c   'select(.event_type == "alert")'   ~/suricata-rule-lab/empire/eve.json
```

## 14. Ví Dụ Covenant Bằng Content

```suricata
alert tcp any any -> $HOME_NET any (
  msg:"LAB Covenant-like repeated HTML body";
  content:"<title>Hello World!</title>";
  detection_filter:track by_src,count 4,seconds 10;
  priority:1;
  sid:3000011;
  rev:1;
)
```

Logic:

```text
Content cố định
+ xuất hiện ít nhất 4 lần
+ cùng source
+ trong 10 giây
```

Hạn chế: chuỗi `Hello World` có thể xuất hiện trong ứng dụng hợp lệ. Nên thêm flow, application buffer, URI và destination context.

## 15. detection_filter

```suricata
detection_filter:track by_src,count 4,seconds 10;
```

Rule chỉ alert sau khi cùng source đạt đủ số lần match trong cửa sổ thời gian.

Các hướng theo dõi thường dùng:

```text
by_src
by_dst
both
```

Phải test semantics trên đúng phiên bản Suricata.

## 16. Covenant Bằng Analytics

```suricata
alert tcp $HOME_NET any -> any any (
  msg:"LAB Covenant-like size and counter";
  dsize:312;
  detection_filter:track by_src,count 3,seconds 10;
  priority:1;
  sid:3000001;
  rev:1;
)
```

Đây là behavior rule dựa trên payload size và frequency. Cần bổ sung flow, protocol và additional features để giảm false positive.

## 17. Ví Dụ Sliver

PCAP:

```text
/home/htb-student/pcaps/sliver.pcap
```

Rule URI tìm:

```text
POST
+ directory giả web hợp lệ
+ PHP endpoint
+ query parameter ngắn
```

Rule cookie tìm:

```text
Set-Cookie
+ tên cookie phổ biến
+ value 32 ký tự
```

Hạn chế: cookie 32 ký tự rất phổ biến. Nên dùng HTTP header buffer, `to_client`, URI pattern, frequency và endpoint context.

## 18. Quy Trình Phân Tích PCAP

```text
1. Xác định source/destination.
2. Xác định protocol thực.
3. Follow stream.
4. Xác định request/response direction.
5. Tìm feature ổn định.
6. So sánh nhiều malicious samples.
7. Kiểm tra feature trong benign traffic.
8. Chọn sticky buffer.
9. Chọn fast_pattern.
10. Chỉ dùng PCRE cho phần còn lại.
```

## 19. Test và Regression

Kiểm tra syntax:

```bash
sudo suricata -T -c /etc/suricata/suricata.yaml
```

Test PCAP:

```bash
sudo suricata   -r /home/htb-student/pcaps/covenant.pcap   -k none   -l ~/suricata-rule-lab/covenant
```

Một rule nên test với:

```text
Positive malicious PCAP
Negative benign PCAP
Near-match PCAP
Modified/evasion sample
High-volume sample
```

## 20. Tối Ưu Hiệu Năng

Rule thường tốn tài nguyên khi:

- `any any -> any any`.
- Không có flow.
- Không dùng sticky buffer.
- Content quá ngắn.
- PCRE quá rộng.
- Quét toàn payload.
- Có nhiều negative match.

Thứ tự tối ưu:

```text
Protocol → direction → flow → buffer → fast content → PCRE
```

## 21. False Positive Tuning

Trước khi sửa rule, kiểm tra:

```text
HOME_NET có đúng không?
Asset role có đúng không?
Traffic có phải scanner/lab được phép không?
Ứng dụng vừa thay đổi không?
Rule có match sai buffer không?
```

Tuning bằng:

- Refine IP/port.
- Add flow.
- Add sticky buffer.
- Add additional content.
- Add threshold.
- Suppress approved assets ở SIEM.
- Enrich alert bằng CMDB/EDR.

## 22. SIEM Correlation

```text
Suricata C2 alert
        ↓
DNS rare domain
        ↓
EDR process network event
        ↓
PowerShell/process telemetry
        ↓
High-confidence incident
```

Splunk:

```spl
index=suricata event_type=alert
| stats count
        values(alert.signature) as signatures
        values(dest_ip) as destinations
  by src_ip, flow_id
```

## 23. Threat Hunting và IR

Sau alert:

```text
[ ] Xem flow_id.
[ ] Correlate HTTP/DNS/TLS cùng flow.
[ ] Xác định endpoint và process.
[ ] Kiểm tra beacon interval.
[ ] Hunt pattern trên toàn mạng.
[ ] Kiểm tra persistence/lateral movement.
[ ] Thu PCAP, EVE, memory và process tree.
[ ] Isolate endpoint nếu true positive.
```

## 24. Mapping Với CDSA

### Security Operations & Monitoring

- Alert triage.
- Rule-quality assessment.
- False-positive tuning.
- EVE/SIEM correlation.

### Incident Response & Forensics

- PCAP reconstruction.
- Flow correlation.
- Timeline và endpoint scoping.

### Malware Analysis & Reverse Engineering

- Phân tích C2 profile.
- Tìm URI/cookie/payload ổn định.
- Viết network signature.

### Threat Hunting

- Hunt beaconing.
- Hunt rare destinations.
- Hunt cùng C2 pattern trong lịch sử.

### Detection Engineering

- Signature rules.
- Behavior rules.
- Stateful logic.
- PCRE optimization.
- Regression testing.
- IDS-to-IPS promotion.

## 25. Rule Development Checklist

```text
[ ] Đúng protocol?
[ ] Đúng direction?
[ ] HOME_NET đúng?
[ ] Có flow?
[ ] Có sticky buffer?
[ ] Content đủ đặc trưng?
[ ] fast_pattern hợp lý?
[ ] PCRE có thật sự cần?
[ ] SID có trùng?
[ ] rev đã tăng?
[ ] Đã test malicious và benign PCAP?
[ ] Đã đánh giá performance?
[ ] Rule đang alert hay drop?
[ ] Có rollback plan?
```

## Key Takeaway

```text
Rule tốt phải đúng protocol và direction,
dùng flow và sticky buffer,
có content anchor mạnh,
hạn chế PCRE,
được regression-test
và cung cấp đủ context cho analyst.
```

Suricata Rule Development Part 1 là nền tảng của Detection Engineering trong CDSA, kết nối packet analysis, malware behavior, threat intelligence, SIEM correlation và incident response.

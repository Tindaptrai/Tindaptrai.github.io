---
layout: post
title: "Suricata Rule Development Part 2 – Encrypted Traffic"
date: 2026-07-15 23:18:27 +0700
categories: ["Working with IDS/IPS", "Suricata"]
tags: [suricata, tls, ssl, encrypted-traffic, x509, ja3, dridex, sliver, detection-engineering, threat-hunting, malware-analysis, incident-response, splunk, elk, cdsa]
description: "Phát triển rule Suricata cho lưu lượng mã hóa bằng TLS certificate metadata, X.509 OID, JA3 fingerprint, beaconing và SIEM correlation."
toc: true
---

# Suricata Rule Development Part 2 – Encrypted Traffic

Bài này thuộc module **Working with IDS/IPS**, tập trung vào cách phát hiện malware/C2 khi nội dung ứng dụng đã được TLS mã hóa.

```text
Mục tiêu:
- Hiểu phần nào của TLS vẫn tạo được network telemetry.
- Phân tích certificate metadata và X.509 OID.
- Hiểu JA3 fingerprint và hạn chế của nó.
- Phân tích rule Dridex và Sliver.
- Kết hợp Suricata với SIEM, EDR và threat hunting.
```

> Chỉ kiểm thử rule và PCAP trong HTB, lab hoặc hệ thống có ủy quyền. Không dùng fingerprint đơn lẻ để chặn production nếu chưa xác thực false positive.

---

## Timeline

```text
Created at: 2026-07-15 23:18:27 +0700
Timezone: Asia/Ho_Chi_Minh
Module: Working with IDS/IPS
Section: Suricata Rule Development Part 2
Focus: TLS certificate metadata, JA3, Dridex, Sliver, encrypted C2
```

---

# 1. Thách thức của lưu lượng mã hóa

Khi HTTP chuyển thành HTTPS, Suricata không còn đọc trực tiếp:

- URI đầy đủ.
- Cookie.
- Request body.
- Response body.
- Command-and-control payload.
- File content sau khi mã hóa.

Tuy nhiên, sensor vẫn có thể quan sát nhiều metadata:

```text
Source/destination IP
Source/destination port
TLS version
Cipher suites
ClientHello extensions
SNI trong nhiều trường hợp
Certificate metadata tùy TLS version và visibility
JA3/JA3S-like fingerprint
Flow duration
Packet size
Beacon interval
Bytes transferred
TLS alerts
```

---

# 2. Lưu ý về TLS 1.2 và TLS 1.3

Trong TLS 1.2, certificate của server thường xuất hiện trong handshake trước khi application data được mã hóa hoàn toàn, nên passive sensor có thể trích metadata certificate.

Trong TLS 1.3:

```text
ClientHello
→ ServerHello
→ phần còn lại của handshake được mã hóa
```

Certificate message nằm trong phần handshake đã mã hóa sau `ServerHello`. Vì vậy, passive sensor không phải lúc nào cũng thấy certificate metadata nếu không đứng tại TLS termination point hoặc không có key material phù hợp.

Điều này làm cho các feature như:

- ClientHello fingerprint.
- SNI.
- Flow behavior.
- Destination reputation.
- Endpoint telemetry.

trở nên quan trọng hơn.

---

# 3. TLS Certificate Metadata

Các trường X.509 hữu ích:

```text
Issuer
Subject
Common Name
Subject Alternative Name
Serial Number
Validity period
Signature algorithm
Public key algorithm
Organization
Locality
Country
```

Dấu hiệu đáng ngờ:

- Self-signed certificate bất thường.
- Validity rất ngắn hoặc rất dài.
- Subject/Issuer kỳ lạ.
- CN không khớp SNI/destination.
- Organization giả hoặc không hợp lý.
- Serial pattern bất thường.
- Certificate tái sử dụng trên nhiều C2 IP.
- Weak signature/public-key algorithm.
- Domain mới hoặc hiếm.

Không nên kết luận malicious chỉ từ một trường certificate.

---

# 4. X.509 OID

Một số Object Identifier thường gặp:

| OID | Trường |
|---|---|
| `2.5.4.6` | countryName |
| `2.5.4.7` | localityName |
| `2.5.4.10` | organizationName |
| `2.5.4.3` | commonName |

Trong DER/ASN.1, các OID có thể xuất hiện dưới dạng byte pattern như:

```text
55 04 06  → countryName
55 04 07  → localityName
55 04 0a  → organizationName
55 04 03  → commonName
```

---

# 5. Ví dụ Dridex – Rule dựa trên certificate

Rule trong lab tìm một tổ hợp pattern liên quan đến certificate từng được liên kết với Dridex:

```suricata
alert tls $EXTERNAL_NET any -> $HOME_NET any (
  msg:"ET MALWARE ABUSE.CH SSL Blacklist Malicious SSL certificate detected (Dridex)";
  flow:established,from_server;

  content:"|16|";
  content:"|0b|";
  within:8;

  byte_test:3,<,1200,0,relative;

  content:"|03 02 01 02 02 09 00|";
  fast_pattern;

  content:"|30 09 06 03 55 04 06 13 02|";
  distance:0;
  pcre:"/^[A-Z]{2}/R";

  content:"|55 04 07|";
  distance:0;

  content:"|55 04 0a|";
  distance:0;

  content:"|55 04 03|";
  distance:0;

  sid:2023476;
  rev:5;
)
```

Rule thật trong lab dài hơn và dùng thêm PCRE, negative content và các kiểm tra length.

---

# 6. Giải thích các byte TLS

```suricata
content:"|16|";
```

`0x16` là TLS record content type cho handshake.

```suricata
content:"|0b|";
within:8;
```

`0x0b` là handshake type `Certificate`.

Logic:

```text
Tìm TLS handshake record
→ xác nhận certificate message gần đó
→ tiếp tục phân tích cấu trúc certificate
```

---

# 7. byte_test

Ví dụ:

```suricata
byte_test:3,<,1200,0,relative;
```

Ý nghĩa tổng quát:

- Đọc 3 byte.
- So sánh với 1200.
- Đọc tại offset tương đối.
- Chỉ tiếp tục nếu điều kiện đúng.

`byte_test` hữu ích khi rule cần kiểm tra:

- Length field.
- Version.
- Flags.
- Numeric protocol field.
- ASN.1 structure length.

Phải xác nhận endian, offset và vị trí tương đối bằng PCAP/Wireshark.

---

# 8. fast_pattern trong rule certificate

```suricata
content:"|03 02 01 02 02 09 00|";
fast_pattern;
```

Suricata dùng content này làm anchor ban đầu để giảm lượng dữ liệu cần kiểm tra bởi các condition đắt hơn.

Nguyên tắc:

```text
fast_pattern nên:
- đủ dài,
- đủ đặc trưng,
- không quá phổ biến,
- xuất hiện sớm trước PCRE.
```

---

# 9. PCRE trong certificate rule

Ví dụ kiểm tra mã quốc gia:

```suricata
pcre:"/^[A-Z]{2}/R";
```

Các modifier đáng chú ý:

- `R`: relative với vị trí match trước.
- `s`: dot khớp newline.
- `i`: không phân biệt hoa thường.

PCRE dùng để kiểm tra cấu trúc linh hoạt hơn `content`, nhưng nên đặt sau content anchor.

---

# 10. Negative Content

Rule Dridex có thể loại trừ các pattern hợp lệ như:

```suricata
content:!"www.";
content:!"GoDaddy";
```

Mục đích:

- Giảm certificate hợp lệ bị match.
- Loại bỏ issuer/subject phổ biến.
- Tăng precision.

Rủi ro:

- Malware có thể thay đổi certificate.
- CA hợp lệ có thể xuất hiện trong traffic độc hại.
- Negative content quá cứng dễ tạo false negative.

---

# 11. Test rule Dridex

PCAP:

```text
/home/htb-student/pcaps/dridex.pcap
```

Chạy:

```bash
mkdir -p ~/suricata-rule-lab/dridex

sudo suricata   -r /home/htb-student/pcaps/dridex.pcap   -k none   -l ~/suricata-rule-lab/dridex
```

Xem alert:

```bash
cat ~/suricata-rule-lab/dridex/fast.log
```

Hoặc:

```bash
jq -c   'select(.event_type == "alert")'   ~/suricata-rule-lab/dridex/eve.json
```

---

# 12. JA3 là gì?

JA3 tạo fingerprint từ trường trong TLS ClientHello:

```text
TLS version
Cipher suites
Extensions
Elliptic curves / supported groups
EC point formats
```

Chuỗi JA3 tổng quát:

```text
SSLVersion,Ciphers,Extensions,EllipticCurves,EllipticCurvePointFormats
```

Sau đó được hash MD5 thành JA3 digest.

Ví dụ:

```text
473cd7cb9faa642487833865d516e578
```

---

# 13. Ý nghĩa của JA3

JA3 fingerprint phản ánh cách một TLS client xây ClientHello.

Có thể dùng để nhận diện:

- Malware family.
- C2 implant.
- Tooling.
- TLS library.
- Browser/client implementation.
- Automation framework.

Nhưng JA3 không phải identity tuyệt đối.

Nhiều phần mềm có thể dùng cùng TLS library và tạo cùng fingerprint.

---

# 14. Hạn chế của JA3

- Có thể collision.
- Có thể bị spoof.
- Malware có thể bắt chước browser.
- TLS library update làm fingerprint đổi.
- GREASE và extension ordering ảnh hưởng cách fingerprint được tạo.
- Một JA3 có thể xuất hiện trong cả traffic benign và malicious.
- Không cho biết payload sau mã hóa.
- TLS 1.3 và hệ sinh thái mới làm fingerprinting phức tạp hơn.

Do đó:

```text
JA3 = feature
không phải verdict
```

---

# 15. Tính JA3 từ PCAP

Trong lab:

```bash
ja3 -a --json   /home/htb-student/pcaps/sliverenc.pcap
```

Output chứa:

```text
source_ip
source_port
destination_ip
destination_port
JA3 string
JA3 digest
timestamp
```

JA3 digest trong mẫu Sliver:

```text
473cd7cb9faa642487833865d516e578
```

---

# 16. Rule Sliver bằng JA3

```suricata
alert tls any any -> any any (
  msg:"LAB Sliver C2 TLS JA3";
  ja3.hash;
  content:"473cd7cb9faa642487833865d516e578";
  sid:1002002;
  rev:1;
)
```

Logic:

```text
TLS ClientHello
→ tính JA3
→ so sánh hash
→ alert nếu khớp
```

> Keyword JA3 và syntax có thể khác theo build/phiên bản. Luôn kiểm tra bằng `suricata --build-info`, `suricata --list-keywords` và `suricata -T`.

---

# 17. Test Sliver JA3 rule

PCAP:

```text
/home/htb-student/pcaps/sliverenc.pcap
```

Chạy:

```bash
mkdir -p ~/suricata-rule-lab/sliverenc

sudo suricata   -r /home/htb-student/pcaps/sliverenc.pcap   -k none   -l ~/suricata-rule-lab/sliverenc
```

Xem alert:

```bash
cat ~/suricata-rule-lab/sliverenc/fast.log
```

EVE:

```bash
jq -c   'select(.event_type == "alert")'   ~/suricata-rule-lab/sliverenc/eve.json
```

---

# 18. Cải thiện rule JA3

Rule chỉ dựa JA3 có thể quá rộng.

Nên kết hợp:

```text
JA3 hash
+ destination hiếm
+ SNI/domain bất thường
+ TLS version/cipher
+ periodic beaconing
+ low byte volume
+ endpoint process
+ threat intelligence
```

Ví dụ concept:

```suricata
alert tls $HOME_NET any -> $EXTERNAL_NET 443 (
  msg:"LAB Sliver-like TLS fingerprint to external network";
  flow:established,to_server;
  ja3.hash;
  content:"473cd7cb9faa642487833865d516e578";
  detection_filter:track by_src,count 3,seconds 300;
  sid:1002003;
  rev:1;
)
```

Rule vẫn phải được regression-test.

---

# 19. TLS Certificate Rule Hiện Đại Hơn

Thay vì parse raw DER bytes bằng nhiều `content`, khi Suricata build/phiên bản hỗ trợ metadata phù hợp, nên ưu tiên sticky buffer/keyword certificate.

Concept:

```suricata
alert tls $HOME_NET any -> $EXTERNAL_NET any (
  msg:"LAB Suspicious TLS certificate subject";
  flow:established,to_server;
  tls.cert_subject;
  content:"Example Suspicious Org";
  nocase;
  sid:1002004;
  rev:1;
)
```

Kiểm tra keyword có sẵn:

```bash
suricata --list-keywords | grep -i tls
```

---

# 20. Behavioral Detection cho TLS C2

Ngay cả khi certificate và fingerprint thay đổi, C2 vẫn có thể để lại hành vi:

- Beacon interval đều.
- Connection duration ngắn.
- Bytes out/in ổn định.
- Cùng destination lặp lại.
- SNI hiếm.
- TLS session không có traffic browser-like.
- Một process không phải browser tạo TLS.
- Kết nối xuất hiện ngoài giờ.

Detection tốt nên kết hợp signature và analytics.

---

# 21. Suricata EVE Fields Hữu Ích

Các event type:

```text
tls
flow
alert
anomaly
stats
```

Các trường có thể hữu ích tùy cấu hình/phiên bản:

```text
tls.sni
tls.version
tls.subject
tls.issuerdn
tls.serial
tls.fingerprint
tls.ja3
tls.ja3s
flow_id
src_ip
dest_ip
bytes_toserver
bytes_toclient
```

Kiểm tra TLS events:

```bash
jq -c   'select(.event_type == "tls")'   /var/log/suricata/eve.json
```

---

# 22. Splunk Detection

JA3:

```spl
index=suricata event_type=tls
tls.ja3.hash="473cd7cb9faa642487833865d516e578"
| stats count
        values(dest_ip) as destinations
        values(tls.sni) as sni
  by src_ip
```

Certificate bất thường:

```spl
index=suricata event_type=tls
| where isnotnull(tls.subject)
| stats count
        dc(dest_ip) as dest_count
        values(tls.issuerdn) as issuers
  by src_ip, tls.subject
```

Beaconing:

```spl
index=suricata event_type=flow dest_port=443
| sort 0 src_ip dest_ip _time
| streamstats current=f last(_time) as previous by src_ip dest_ip
| eval interval=_time-previous
| stats count avg(interval) stdev(interval) by src_ip dest_ip
| where count > 10 AND stdev(interval) < 5
```

---

# 23. Elastic/KQL

TLS event:

```kql
event_type: "tls"
```

JA3:

```kql
tls.client.ja3:
  "473cd7cb9faa642487833865d516e578"
```

Rare SNI hoặc certificate subject cần aggregation bằng Elastic rule/Kibana.

Field name phụ thuộc ingest pipeline và ECS mapping.

---

# 24. Endpoint Correlation

Network fingerprint mạnh hơn khi kết hợp process telemetry.

Ví dụ:

```text
Suricata JA3 match
        ↓
EDR process:
unknown.exe / powershell.exe
        ↓
Destination:
rare external IP
        ↓
Beacon:
mỗi 60 giây
        ↓
High-confidence C2
```

Windows/Sysmon:

```text
Event ID 1  - Process Create
Event ID 3  - Network Connection
Event ID 22 - DNS Query
```

---

# 25. Threat Hunting Workflow

```text
1. Xác định JA3/certificate indicator.
2. Hunt toàn bộ lịch sử EVE.
3. Group theo source host.
4. Group theo destination/SNI.
5. Kiểm tra periodicity.
6. Correlate endpoint process.
7. Kiểm tra DNS trước TLS connection.
8. Tìm các host khác cùng fingerprint.
9. Xác định first seen / last seen.
10. Đánh giá exfiltration hoặc command execution.
```

---

# 26. False Positive Tuning

Các nguồn false positive:

- Browser dùng cùng TLS library.
- Security scanner.
- Proxy/SSL inspection.
- CDN.
- Monitoring agent.
- Internal API client.
- Software update service.
- Certificate hợp lệ nhưng cấu hình lạ.

Tuning:

- Asset allowlist.
- Destination allowlist.
- Process allowlist ở EDR/SIEM.
- Thêm SNI/domain context.
- Thêm frequency threshold.
- Thêm certificate issuer/subject.
- Chỉ alert trên endpoint role nhạy cảm.

---

# 27. Incident Response Checklist

```text
[ ] Xác định source endpoint.
[ ] Xác định destination IP/SNI.
[ ] Thu flow_id và TLS event.
[ ] Xác định JA3/certificate indicator.
[ ] Correlate DNS và process telemetry.
[ ] Kiểm tra beacon interval.
[ ] Hunt cùng IOC trên toàn mạng.
[ ] Isolate host nếu true positive.
[ ] Thu memory, process tree và persistence.
[ ] Block IOC theo risk và confidence.
[ ] Cập nhật rule sau điều tra.
```

---

# 28. Rule Regression Testing

Test tối thiểu:

```text
Malicious Dridex PCAP
Malicious Sliver PCAP
Benign browser TLS PCAP
Benign updater/API client
TLS 1.2 sample
TLS 1.3 sample
Proxy-inspected traffic
High-volume PCAP
```

Đánh giá:

```text
True positives
False positives
Packets/flows inspected
Rule performance
Keyword compatibility
TLS-version visibility
```

---

# 29. Mapping Với CDSA

## Security Operations & Monitoring

- TLS metadata monitoring.
- JA3/certificate alert triage.
- SIEM correlation.
- Sensor visibility assessment.

## Incident Response & Forensics

- TLS flow reconstruction.
- First-seen/last-seen timeline.
- Endpoint/process scoping.
- C2 infrastructure identification.

## Malware Analysis & Reverse Engineering

- Phân tích ClientHello fingerprint.
- So sánh C2 profiles.
- Xác định certificate reuse.
- Trích network indicators từ sample.

## Threat Hunting

- Hunt rare JA3.
- Hunt rare SNI.
- Hunt certificate reuse.
- Hunt periodic TLS beaconing.
- Hunt unknown process tạo TLS.

## Detection Engineering

- TLS certificate signatures.
- JA3 rules.
- Behavioral analytics.
- Regression testing.
- Multi-source correlation.

---

# 30. Lab Workflow

```bash
# Kiểm tra build và keyword
suricata --version
suricata --build-info
suricata --list-keywords | grep -Ei 'ja3|tls'

# Test cấu hình
sudo suricata -T -c /etc/suricata/suricata.yaml

# Dridex
sudo suricata   -r /home/htb-student/pcaps/dridex.pcap   -k none   -l ~/suricata-rule-lab/dridex

# Sliver encrypted
sudo suricata   -r /home/htb-student/pcaps/sliverenc.pcap   -k none   -l ~/suricata-rule-lab/sliverenc

# Xem alert
jq -c   'select(.event_type == "alert")'   ~/suricata-rule-lab/sliverenc/eve.json
```

---

# Key Takeaway

```text
TLS mã hóa payload nhưng không làm biến mất toàn bộ network telemetry.

Detection hiệu quả cần kết hợp:
certificate metadata
+ ClientHello fingerprint
+ SNI/destination
+ flow behavior
+ endpoint process
+ threat intelligence.

JA3 hoặc certificate indicator đơn lẻ không đủ để kết luận malware.
```

Suricata Rule Development Part 2 là phần quan trọng của Detection Engineering trong CDSA vì kết nối encrypted traffic analysis, malware C2 profiling, SIEM analytics, threat hunting và incident response.

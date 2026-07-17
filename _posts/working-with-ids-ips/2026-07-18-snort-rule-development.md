---
layout: post
title: "Snort Rule Development"
date: 2026-07-18 00:08:35 +0700
categories: ["Working with IDS/IPS", "Snort"]
tags: [snort, snort3, rule-development, ids, ips, ursnif, cerber, patchwork, sticky-buffers, pcre, detection-filter, malware-c2, detection-engineering, threat-hunting, incident-response, cdsa]
description: "Phát triển Snort rule qua các ví dụ Ursnif, Cerber và Patchwork: content matching, sticky buffers, PCRE, detection_filter, certificate matching và quy trình kiểm thử PCAP."
toc: true
---

# Snort Rule Development

Bài này thuộc module **Working with IDS/IPS**, tập trung vào cách phát triển Snort 3 rules để phát hiện malware/C2 dựa trên signature, hành vi, HTTP transaction và TLS certificate metadata.

```text
Mục tiêu:
- Hiểu cách tối ưu rule Snort.
- Dùng flow, content, depth, distance và within.
- Dùng sticky buffers cho HTTP.
- Dùng PCRE và detection_filter hợp lý.
- Phân tích rule Ursnif, Cerber và Patchwork.
- Kiểm thử rule bằng PCAP và Wireshark.
```

> Chỉ chạy các rule và PCAP trong HTB, lab hoặc hệ thống có ủy quyền. Không chuyển rule sang hành động chặn trước khi kiểm thử false positive, performance và rollback.

---

## Timeline

```text
Created at: 2026-07-18 00:08:35 +0700
Timezone: Asia/Ho_Chi_Minh
Module: Working with IDS/IPS
Section: Snort Rule Development
Focus: Ursnif, Cerber, Patchwork, sticky buffers, PCRE, TLS certificate
```

---

# 1. Cấu trúc Snort Rule

Snort rule gồm:

```text
Rule header
+ Rule options
```

Ví dụ:

```snort
alert tcp $HOME_NET any -> $EXTERNAL_NET any (
  msg:"LAB Suspicious outbound C2";
  flow:established,to_server;
  content:"/images/";
  sid:1000001;
  rev:1;
)
```

Header xác định:

- Action.
- Protocol.
- Source/destination.
- Port.
- Direction.

Options xác định:

- Message.
- Flow state.
- Content.
- Buffer.
- PCRE.
- Threshold.
- SID/revision.
- Classification.

---

# 2. Nguyên Tắc Viết Rule Hiệu Quả

Một rule tốt nên:

```text
Giới hạn đúng protocol
→ đúng direction
→ đúng flow
→ đúng application buffer
→ content anchor mạnh
→ PCRE xác nhận cuối
```

Rule kém hiệu quả thường:

- Dùng `any any -> any any`.
- Quét toàn raw payload.
- Không có flow.
- Không dùng sticky buffer.
- Dùng PCRE quá rộng.
- Chỉ dựa vào một chuỗi phổ biến.
- Không test benign PCAP.

---

# 3. Example 1 – Ursnif Rule Không Tối Ưu

Rule lab:

```snort
alert tcp any any -> any any (
  msg:"Possible Ursnif C2 Activity";
  flow:established,to_server;
  content:"/images/";
  depth:12;
  content:"_2F";
  content:"_2B";
  content:"User-Agent|3a 20|Mozilla/4.0 (compatible|3b| MSIE 8.0|3b| Windows NT";
  content:!"Accept";
  content:!"Cookie|3a|";
  content:!"Referer|3a|";
  sid:1000002;
  rev:1;
)
```

Rule tìm:

```text
Established TCP session
+ traffic đi tới server
+ URI bắt đầu bằng /images/
+ chuỗi _2F và _2B
+ User-Agent MSIE 8.0
+ thiếu Accept/Cookie/Referer
```

---

# 4. Vì Sao Ursnif Rule Không Tối Ưu?

Rule quét raw TCP payload thay vì application buffers.

Hậu quả:

- Match có thể xảy ra ở sai vị trí.
- Tốn CPU hơn.
- Dễ gặp segmentation/reassembly edge case.
- Khó đọc và bảo trì.
- False positive cao hơn.

Các thành phần như URI, User-Agent và headers nên được kiểm tra trong HTTP buffers tương ứng.

---

# 5. Cải Thiện Ursnif Rule Bằng Sticky Buffers

Khái niệm cải tiến:

```snort
alert http $HOME_NET any -> $EXTERNAL_NET any (
  msg:"LAB Ursnif-like HTTP C2";
  flow:established,to_server;

  http_uri;
  content:"/images/";
  depth:12;
  content:"_2F";
  content:"_2B";

  http_user_agent;
  content:"Mozilla/4.0 (compatible; MSIE 8.0; Windows NT";
  nocase;

  http_header;
  content:!"Accept";
  content:!"Cookie:";
  content:!"Referer:";

  sid:1000102;
  rev:1;
)
```

> Tên sticky buffer và cú pháp phụ thuộc Snort 3 version. Luôn validate bằng Snort trước khi sử dụng.

---

# 6. Phân Tích Payload Ursnif

Trong PCAP, request có dạng:

```http
GET /images/<encoded-path>.gif HTTP/1.1
User-Agent: Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 6.1)
Host: <domain>
Connection: Keep-Alive
Cache-Control: no-cache
```

Các feature đáng chú ý:

- URI giả dạng image path.
- Encoded fragments `_2F`, `_2B`.
- User-Agent cũ.
- Thiếu một số browser headers.
- Outbound HTTP tới external host.

---

# 7. Test Ursnif Rule

PCAP:

```text
/home/htb-student/pcaps/ursnif.pcap
```

Chạy:

```bash
mkdir -p ~/snort-rule-lab/ursnif

sudo snort   -c /root/snorty/etc/snort/snort.lua   --daq-dir /usr/local/lib/daq   -R /home/htb-student/local.rules   -r /home/htb-student/pcaps/ursnif.pcap   -A cmg
```

Tải PCAP về máy phân tích:

```bash
scp htb-student@<TARGET_IP>:/home/htb-student/pcaps/ursnif.pcap .
```

---

# 8. Điều Tra Ursnif Bằng Wireshark

Filters:

```wireshark
http.request
```

```wireshark
http.request.uri contains "/images/"
```

```wireshark
http.user_agent contains "MSIE 8.0"
```

Kiểm tra:

- URI.
- Host.
- User-Agent.
- Header presence.
- Source process nếu có endpoint telemetry.
- DNS query trước HTTP connection.

---

# 9. Example 2 – Cerber UDP Check-in

Rule:

```snort
alert udp $HOME_NET any -> $EXTERNAL_NET any (
  msg:"Possible Cerber Check-in";
  dsize:9;
  content:"hi";
  depth:2;
  fast_pattern;
  pcre:"/^[af0-9]{7}$/R";
  detection_filter:track by_src,count 1,seconds 60;
  sid:2816763;
  rev:4;
)
```

Logic:

```text
Outbound UDP
+ payload đúng 9 byte
+ bắt đầu bằng "hi"
+ 7 ký tự hex-like tiếp theo
+ theo dõi theo source trong 60 giây
```

Ví dụ payload:

```text
hi0072895
```

---

# 10. dsize Trong Rule Cerber

```snort
dsize:9;
```

Chỉ match UDP payload có đúng 9 byte.

Điều này giúp giảm số packet phải kiểm tra tiếp.

Tuy nhiên:

```text
Payload size đơn lẻ không đủ để kết luận malware.
```

Rule mạnh hơn nhờ kết hợp:

- Direction.
- String prefix.
- PCRE.
- Frequency.

---

# 11. fast_pattern Trong Cerber Rule

```snort
content:"hi";
depth:2;
fast_pattern;
```

Snort kiểm tra `hi` trong 2 byte đầu và dùng làm fast-pattern anchor.

Lưu ý:

- Pattern `hi` rất ngắn và phổ biến.
- Việc giới hạn `depth:2` và `dsize:9` giúp giảm noise.
- Vẫn cần đánh giá performance trên traffic UDP lớn.

---

# 12. PCRE Trong Cerber Rule

```snort
pcre:"/^[af0-9]{7}$/R";
```

Ý nghĩa:

- `^`: bắt đầu buffer tương đối.
- `[af0-9]`: ký tự a-f hoặc số.
- `{7}`: đúng 7 ký tự.
- `$`: kết thúc.
- `R`: relative với match trước.

Toàn payload phải có dạng:

```text
hi + 7 ký tự hex
```

---

# 13. detection_filter

```snort
detection_filter:track by_src,count 1,seconds 60;
```

`detection_filter` theo dõi rule hits theo source trong cửa sổ thời gian.

Cần kiểm tra semantics chính xác trên Snort version đang dùng. Trong thực tế, rule này có thể tạo nhiều alerts nếu malware gửi check-in tới nhiều destination IP.

Tuning khả thi:

- Track theo source.
- Giới hạn destination port.
- Thêm destination reputation.
- Correlate nhiều destination cùng payload.
- Rate-limit alert trong SIEM.

---

# 14. Test Cerber Rule

PCAP:

```text
/home/htb-student/pcaps/cerber.pcap
```

Chạy:

```bash
sudo snort   -c /root/snorty/etc/snort/snort.lua   --daq-dir /usr/local/lib/daq   -R /home/htb-student/local.rules   -r /home/htb-student/pcaps/cerber.pcap   -A cmg
```

Wireshark:

```wireshark
udp && udp.length == 17
```

UDP length gồm 8-byte UDP header và 9-byte payload.

Hoặc:

```wireshark
udp contains 68:69
```

---

# 15. Cerber Detection Trong SOC

High-confidence correlation:

```text
Snort Cerber-like UDP alert
+ một endpoint gửi tới hàng trăm external IP
+ cùng destination port
+ cùng 9-byte payload
+ ransomware process/event trên endpoint
```

Nguồn bổ sung:

- Sysmon Event ID 1.
- Sysmon Event ID 3.
- DNS logs.
- EDR file activity.
- Shadow-copy deletion.
- Ransomware note creation.

---

# 16. Example 3 – Patchwork HTTP C2

Rule:

```snort
alert http $HOME_NET any -> $EXTERNAL_NET any (
  msg:"OISF TROJAN Targeted AutoIt FileStealer/Downloader CnC Beacon";
  flow:established,to_server;

  http_method;
  content:"POST";

  http_uri;
  content:".php?profile=";

  http_client_body;
  content:"ddager=";
  depth:7;

  http_client_body;
  content:"&r1=";
  distance:0;

  http_header;
  content:!"Accept";

  http_header;
  content:!"Referer|3a|";

  sid:10000006;
  rev:1;
)
```

Đây là ví dụ rule tốt hơn vì dùng sticky buffers.

---

# 17. Patchwork HTTP Features

Rule kết hợp:

```text
POST method
+ URI chứa .php?profile=
+ request body bắt đầu bằng ddager=
+ body chứa &r1=
+ thiếu Accept
+ thiếu Referer
```

Ví dụ request body:

```text
ddager=0&r1=<encoded>&r2=<value>&r3=<value>...
```

Đây là beacon có cấu trúc ổn định hơn một IOC đơn lẻ.

---

# 18. Lợi Ích Của Sticky Buffers

```snort
http_method;
http_uri;
http_client_body;
http_header;
```

Lợi ích:

- Match đúng semantic field.
- Giảm false positive.
- Không cần scan toàn raw payload.
- Tăng hiệu năng.
- Dễ bảo trì.
- Dễ hiểu khi analyst đọc rule.

---

# 19. distance:0 Trong Request Body

```snort
content:"ddager=";
depth:7;

content:"&r1=";
distance:0;
```

Logic:

```text
Body bắt đầu bằng ddager=
→ ngay sau đó tìm &r1=
```

Nếu cần giới hạn thêm, có thể kết hợp `within`.

Ví dụ concept:

```snort
content:"&r1=";
distance:0;
within:16;
```

---

# 20. Test Patchwork HTTP Rule

PCAP:

```text
/home/htb-student/pcaps/patchwork.pcap
```

Chạy:

```bash
sudo snort   -c /root/snorty/etc/snort/snort.lua   --daq-dir /usr/local/lib/daq   -R /home/htb-student/local.rules   -r /home/htb-student/pcaps/patchwork.pcap   -A cmg
```

Wireshark filters:

```wireshark
http.request.method == "POST"
```

```wireshark
http.request.uri contains ".php?profile="
```

```wireshark
http contains "ddager="
```

---

# 21. Example 4 – Patchwork TLS Certificate

Rule:

```snort
alert tcp $EXTERNAL_NET any -> $HOME_NET any (
  msg:"Patchwork SSL Cert Detected";
  flow:established,from_server;
  content:"|55 04 03|";
  content:"|08|toigetgf";
  distance:1;
  within:9;
  classtype:trojan-activity;
  sid:10000008;
  rev:1;
)
```

Rule tìm certificate Common Name:

```text
toigetgf
```

---

# 22. ASN.1 Common Name OID

```text
55 04 03
```

là DER encoding của X.509 OID:

```text
2.5.4.3 = commonName
```

Sau OID, rule tìm:

```text
08 toigetgf
```

Trong đó `08` là length byte cho chuỗi 8 ký tự.

---

# 23. distance và within Trong Certificate Rule

```snort
content:"|55 04 03|";

content:"|08|toigetgf";
distance:1;
within:9;
```

Ý nghĩa:

- Match Common Name OID.
- Bắt đầu tìm pattern thứ hai sau một khoảng tương đối.
- Giới hạn cửa sổ tìm kiếm.
- Xác nhận CN cụ thể.

Rule raw-DER kiểu này hiệu quả với mẫu cụ thể nhưng có thể dễ bị bypass khi certificate thay đổi.

---

# 24. Hạn Chế Của Certificate Signature

- Certificate có thể được thay đổi nhanh.
- C2 có thể dùng certificate hợp lệ.
- TLS 1.3 làm giảm passive visibility đối với certificate message.
- Raw byte parsing dễ phụ thuộc encoding.
- Một CN có thể xuất hiện trong benign test traffic.
- Malware có thể chuyển sang CDN/fronting/proxy.

Nên kết hợp:

- Destination IP/SNI.
- Certificate fingerprint.
- JA3/JA4-like fingerprint.
- Beaconing.
- Endpoint process.
- Threat intelligence.

---

# 25. Test Patchwork TLS Rule

Cùng PCAP:

```text
/home/htb-student/pcaps/patchwork.pcap
```

Wireshark filters:

```wireshark
tls.handshake.type == 11
```

```wireshark
x509sat.printableString contains "toigetgf"
```

Field name có thể khác theo Wireshark version.

Có thể dùng:

```wireshark
frame contains "toigetgf"
```

để xác nhận raw bytes.

---

# 26. Rule Development Workflow

```text
1. Mở PCAP trong Wireshark.
2. Xác định malicious flow.
3. Follow TCP/UDP stream.
4. Xác định feature ổn định.
5. Chọn protocol và direction.
6. Chọn sticky buffer.
7. Chọn content anchor.
8. Thêm depth/distance/within.
9. Chỉ dùng PCRE khi cần.
10. Thêm SID/rev/classtype.
11. Validate Snort config.
12. Test positive và negative PCAP.
```

---

# 27. Validation

```bash
sudo snort   -c /root/snorty/etc/snort/snort.lua   --daq-dir /usr/local/lib/daq
```

Hoặc với rule file:

```bash
sudo snort   -c /root/snorty/etc/snort/snort.lua   --daq-dir /usr/local/lib/daq   -R /home/htb-student/local.rules
```

Kiểm tra:

```text
0 warnings
Rule loaded
Không duplicate SID ngoài dự kiến
Không unsupported keyword
```

---

# 28. Regression Testing

Mỗi rule nên test với:

| Test | Kỳ vọng |
|---|---|
| Malicious PCAP | Alert |
| Benign PCAP | Không alert |
| Near-match | Theo thiết kế |
| Modified User-Agent | Đánh giá robustness |
| Modified URI | Đánh giá coverage |
| High-volume PCAP | Không gây quá tải |
| Segmented TCP | Vẫn detect nếu inspector/reassembly đúng |

---

# 29. Hiệu Năng Rule

Trong Snort output, theo dõi:

```text
raw_searches
cooked_searches
pdu_searches
qualified_events
non_qualified_events
search_engine memory
runtime
packets/sec
```

Rule raw payload có xu hướng tạo nhiều `raw_searches`.

Rule dùng HTTP inspector tạo:

```text
http_method
http_uri
http_header
http_client_body
```

và thường precise hơn.

---

# 30. False Positive Tuning

Các bước:

```text
[ ] Xác định traffic hợp lệ nào match.
[ ] Kiểm tra HOME_NET/EXTERNAL_NET.
[ ] Kiểm tra direction.
[ ] Thêm flow.
[ ] Chuyển raw content sang sticky buffer.
[ ] Thêm content thứ hai.
[ ] Giới hạn depth/within.
[ ] Thêm threshold.
[ ] Correlate asset role.
[ ] Không suppress toàn bộ khi chưa hiểu nguyên nhân.
```

---

# 31. Splunk Correlation

Snort alert:

```spl
index=snort
| stats count
        values(signature) as signatures
        values(dest_ip) as destinations
  by src_ip, sid
```

Cerber-like UDP:

```spl
index=snort sid=2816763
| stats count
        dc(dest_ip) as unique_destinations
        values(dest_port) as ports
  by src_ip
| where unique_destinations > 10
```

Patchwork:

```spl
index=snort
(sid=10000006 OR sid=10000008)
| sort _time
| table _time src_ip src_port dest_ip dest_port sid signature
```

---

# 32. Incident Response

Khi rule fire:

```text
[ ] Xác định source endpoint.
[ ] Xác định destination.
[ ] Thu PCAP và alert output.
[ ] Kiểm tra DNS trước connection.
[ ] Correlate EDR process.
[ ] Hunt cùng SID/IOC toàn mạng.
[ ] Kiểm tra persistence.
[ ] Isolate endpoint nếu true positive.
[ ] Block IOC theo confidence.
[ ] Cập nhật rule sau điều tra.
```

---

# 33. Mapping Với CDSA

## Security Operations & Monitoring

- Snort alert triage.
- HTTP/UDP/TLS visibility.
- Performance statistics.
- SIEM ingestion.

## Incident Response & Forensics

- PCAP replay.
- Packet-level reconstruction.
- Alert timeline.
- Endpoint scoping.

## Malware Analysis & Reverse Engineering

- Phân tích C2 protocol.
- Trích URI/body/certificate IOC.
- So sánh malware samples.
- Viết network signature.

## Threat Hunting

- Hunt Ursnif-like HTTP.
- Hunt Cerber UDP fan-out.
- Hunt Patchwork beaconing.
- Hunt certificate reuse.

## Detection Engineering

- Sticky buffers.
- PCRE.
- Detection filters.
- Regression testing.
- False-positive tuning.
- Passive-to-inline promotion.

---

# 34. Lab Checklist

```bash
# Validate
sudo snort   -c /root/snorty/etc/snort/snort.lua   --daq-dir /usr/local/lib/daq

# Ursnif
sudo snort   -c /root/snorty/etc/snort/snort.lua   --daq-dir /usr/local/lib/daq   -R /home/htb-student/local.rules   -r /home/htb-student/pcaps/ursnif.pcap   -A cmg

# Cerber
sudo snort   -c /root/snorty/etc/snort/snort.lua   --daq-dir /usr/local/lib/daq   -R /home/htb-student/local.rules   -r /home/htb-student/pcaps/cerber.pcap   -A cmg

# Patchwork
sudo snort   -c /root/snorty/etc/snort/snort.lua   --daq-dir /usr/local/lib/daq   -R /home/htb-student/local.rules   -r /home/htb-student/pcaps/patchwork.pcap   -A cmg
```

---

# Key Takeaway

```text
Snort rule tốt phải giới hạn đúng protocol, direction và flow.
HTTP traffic nên dùng sticky buffers thay vì quét raw payload.
dsize, depth, distance, within và PCRE chỉ hiệu quả khi được dùng có context.
Rule phải được test trên cả malicious và benign PCAP trước khi triển khai.
```

Snort Rule Development là nội dung trọng tâm của Detection Engineering trong CDSA vì kết nối malware traffic analysis, IDS/IPS rule authoring, threat hunting, SIEM correlation và incident response.

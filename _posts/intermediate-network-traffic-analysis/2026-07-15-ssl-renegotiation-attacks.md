---
layout: post
title: "SSL Renegotiation Attacks"
date: 2026-07-15 00:14:16 +0700
categories: ["Intermediate Network Traffic Analysis"]
tags: [tls, ssl, renegotiation, https, handshake, client-hello, denial-of-service, cipher-suites, wireshark, tshark, suricata, zeek, splunk, elk, threat-hunting, incident-response, cdsa]
description: "Phân tích SSL/TLS renegotiation attacks: nhiều ClientHello, handshake bất thường, resource exhaustion, Wireshark filters, SIEM detection và hardening."
toc: true
---

# SSL Renegotiation Attacks

Bài này thuộc module **Intermediate Network Traffic Analysis**, tập trung vào cách nhận diện hành vi renegotiation bất thường trong kết nối SSL/TLS và phân biệt nó với handshake hợp lệ.

```text
Mục tiêu: hiểu TLS handshake, phát hiện nhiều ClientHello,
nhận diện renegotiation sau khi session đã được thiết lập,
đánh giá DoS/resource exhaustion và xây detection.
```

> Chỉ thực hành trong HTB, lab hoặc hệ thống có ủy quyền rõ ràng. Không tạo tải renegotiation hoặc DoS trên hệ thống ngoài phạm vi được phép.

---

## Timeline cập nhật

```text
Created at: 2026-07-15 00:14:16 +0700
Timezone: Asia/Ho_Chi_Minh
Module: Intermediate Network Traffic Analysis
Section: SSL Renegotiation Attacks
Focus: TLS handshake, ClientHello repetition, renegotiation abuse, DoS detection
```

---

# 1. HTTPS và TLS

HTTPS là HTTP chạy bên trong một kênh được bảo vệ bởi TLS.

Luồng tổng quát:

```text
TCP three-way handshake
→ TLS handshake
→ xác thực server
→ thỏa thuận thuật toán mã hóa
→ tạo session keys
→ trao đổi HTTP đã mã hóa
```

TLS cung cấp:

- Confidentiality.
- Integrity.
- Server authentication.
- Client authentication tùy cấu hình.

SSL là công nghệ cũ; trong thực tế hiện đại nên ưu tiên thuật ngữ **TLS**.

---

# 2. TLS Handshake Cơ Bản

Một handshake điển hình gồm:

```text
ClientHello
ServerHello
Certificate
Key Exchange
Finished
Encrypted Application Data
```

## ClientHello

Chứa:

- TLS versions client hỗ trợ.
- Cipher suites.
- Random value.
- Extensions.
- Server Name Indication.
- Supported groups.
- Signature algorithms.

## ServerHello

Server chọn:

- TLS version.
- Cipher suite.
- Random value.
- Các extension phù hợp.

---

# 3. Certificate Exchange

Server gửi certificate để chứng minh identity.

Client sẽ kiểm tra:

- Certificate chain.
- Expiration.
- Hostname.
- Trusted CA.
- Signature.
- Revocation nếu được cấu hình.

Certificate chứa public key, nhưng cách tạo session key phụ thuộc cipher suite và TLS version.

---

# 4. Key Exchange

Trong TLS hiện đại, key exchange thường dùng:

```text
ECDHE
DHE
```

Hai bên tạo shared secret rồi dẫn xuất session keys.

Điều này cung cấp **Perfect Forward Secrecy** nếu cấu hình đúng.

Các mô tả dùng RSA-encrypted premaster secret chủ yếu áp dụng cho cipher suites TLS cũ.

---

# 5. Finished Messages

Sau khi hai bên dẫn xuất cùng key material, họ trao đổi Finished messages.

Finished message xác minh:

- Handshake không bị sửa đổi.
- Hai bên có cùng key material.
- Session có thể chuyển sang encrypted application data.

---

# 6. TLS Record Content Types

Trong Wireshark, handshake thường được nhận diện qua record content type.

Filter được module sử dụng:

```wireshark
ssl.record.content_type == 22
```

Trong Wireshark mới, có thể dùng:

```wireshark
tls.record.content_type == 22
```

Handshake messages:

```wireshark
tls.handshake
```

ClientHello:

```wireshark
tls.handshake.type == 1
```

ServerHello:

```wireshark
tls.handshake.type == 2
```

---

# 7. SSL/TLS Renegotiation Là Gì?

Renegotiation cho phép client và server thực hiện lại một phần handshake bên trong connection đã tồn tại.

Mục đích hợp lệ trước đây:

- Yêu cầu client certificate.
- Thay đổi cryptographic parameters.
- Làm mới security context.
- Tương thích với ứng dụng cũ.

Tuy nhiên, renegotiation có thể bị lạm dụng để:

- Tiêu tốn CPU server.
- Tạo nhiều handshake operations.
- Khai thác implementation lỗi thời.
- Gây session instability.

> Renegotiation không tự động đồng nghĩa với downgrade. Việc chọn cipher/version vẫn phụ thuộc cấu hình client và server.

---

# 8. Dấu Hiệu Renegotiation Attack

Các dấu hiệu chính:

- Nhiều ClientHello từ cùng client trong thời gian ngắn.
- ClientHello xuất hiện sau khi encrypted application data đã bắt đầu.
- Handshake messages lặp lại trên cùng TCP stream.
- Server thực hiện nhiều expensive cryptographic operations.
- Session tạo nhiều certificate/key-exchange cycles.
- CPU tăng nhưng lưu lượng ứng dụng rất thấp.
- Nhiều connection ngắn chỉ chứa handshake.
- TLS alert hoặc connection reset tăng.

---

# 9. Multiple ClientHello

Filter:

```wireshark
tls.handshake.type == 1
```

Theo source:

```wireshark
ip.src == 192.168.10.56 &&
tls.handshake.type == 1
```

Theo một TCP stream:

```wireshark
tcp.stream eq 3 &&
tls.handshake.type == 1
```

Nếu một stream có nhiều ClientHello sau handshake hoàn tất, cần điều tra renegotiation.

---

# 10. Out-of-Order Handshake Messages

Một số packet có thể out-of-order do:

- Packet capture loss.
- Asymmetric routing.
- TCP retransmission.
- SPAN port overload.
- Wireshark reassembly issue.

Dấu hiệu đáng ngờ hơn:

```text
ClientHello
→ ServerHello
→ Finished
→ Application Data
→ ClientHello mới
```

Đây là renegotiation hoặc một handshake bất thường trong cùng connection.

---

# 11. Phân Biệt Renegotiation và Kết Nối Mới

Không nên chỉ đếm ClientHello theo source IP.

Cần xác định:

```text
Có nằm trong cùng TCP stream không?
Source/destination port có giữ nguyên không?
Handshake mới xuất hiện sau application data không?
Connection mới hay renegotiation trong connection cũ?
```

Connection mới:

```text
SYN
→ SYN/ACK
→ ACK
→ ClientHello
```

Renegotiation:

```text
Established TCP stream
→ Application Data
→ ClientHello mới
```

---

# 12. Wireshark Filters Hữu Ích

Tất cả TLS handshake:

```wireshark
tls.record.content_type == 22
```

ClientHello:

```wireshark
tls.handshake.type == 1
```

ServerHello:

```wireshark
tls.handshake.type == 2
```

TLS alerts:

```wireshark
tls.record.content_type == 21
```

Application data:

```wireshark
tls.record.content_type == 23
```

ClientHello từ một source:

```wireshark
ip.src == 192.168.10.56 &&
tls.handshake.type == 1
```

---

# 13. Follow TCP Stream

Trong Wireshark:

```text
Right click packet
→ Follow
→ TCP Stream
```

Kiểm tra:

- Bao nhiêu ClientHello.
- Bao nhiêu ServerHello.
- Có application data giữa hai handshake không.
- Có TLS alerts không.
- Connection có reset không.
- Handshake có hoàn tất không.

---

# 14. Renegotiation và Denial-of-Service

Handshake có chi phí cao hơn application data thông thường vì có thể cần:

- Public-key operations.
- Certificate validation.
- Session state allocation.
- Key derivation.
- Random generation.
- Memory allocation.

Attacker có thể tạo nhiều renegotiation requests để làm:

```text
CPU usage tăng
Memory/session table tăng
Latency tăng
Legitimate users bị chậm hoặc timeout
```

---

# 15. Detection Logic

Một rule tốt nên kết hợp:

```text
Nhiều ClientHello
+ cùng TCP stream hoặc cùng connection tuple
+ trong thời gian ngắn
+ ít application data
+ server CPU/latency tăng
```

Feature nên theo dõi:

- ClientHello count.
- ServerHello count.
- TCP stream.
- Source IP.
- Destination service.
- Handshake duration.
- Application bytes.
- TLS alert count.
- Connection reset count.
- CPU/session metrics.

---

# 16. Tshark Hunt

Liệt kê ClientHello:

```bash
tshark -r SSL_renegotiation_edited.pcapng   -Y "tls.handshake.type == 1"   -T fields   -e frame.time   -e ip.src   -e ip.dst   -e tcp.stream   -e tls.handshake.version
```

Đếm ClientHello theo stream:

```bash
tshark -r SSL_renegotiation_edited.pcapng   -Y "tls.handshake.type == 1"   -T fields -e tcp.stream |
sort | uniq -c | sort -nr
```

Đếm theo source:

```bash
tshark -r SSL_renegotiation_edited.pcapng   -Y "tls.handshake.type == 1"   -T fields -e ip.src |
sort | uniq -c | sort -nr
```

---

# 17. Cipher Suite Analysis

ClientHello quảng bá cipher suites; ServerHello chọn một suite.

Filter:

```wireshark
tls.handshake.ciphersuite
```

Cần điều tra:

- Server chọn cipher yếu.
- Client chỉ quảng bá cipher lỗi thời.
- TLS version cũ.
- Cipher thay đổi bất thường giữa các handshake.
- Renegotiation làm thay đổi security parameters.

Hardening nên vô hiệu hóa:

- SSLv2.
- SSLv3.
- TLS 1.0.
- TLS 1.1.
- RC4.
- Export ciphers.
- Null ciphers.
- Anonymous ciphers.

---

# 18. TLS Version Notes

TLS 1.3 không hỗ trợ renegotiation theo cách của TLS 1.2 và cũ hơn.

TLS 1.3 sử dụng các cơ chế khác như:

- Key Update.
- Post-handshake authentication trong trường hợp phù hợp.

Do đó, ưu tiên TLS 1.3 giúp loại bỏ lớp rủi ro renegotiation truyền thống.

---

# 19. Heartbleed Context

Heartbleed là lỗ hổng khác với renegotiation.

```text
CVE-2014-0160
```

Nó liên quan đến OpenSSL Heartbeat extension và có thể làm rò rỉ memory.

Phân biệt:

| Kỹ thuật | Mục tiêu |
|---|---|
| Renegotiation abuse | Resource exhaustion/session manipulation |
| Heartbleed | Memory disclosure |
| Downgrade attack | Ép dùng version/cipher yếu |
| TLS stripping | Chuyển HTTPS xuống HTTP |

---

# 20. Suricata Detection Ideas

Suricata có thể log TLS metadata và handshake events.

Logic gợi ý:

```text
Nhiều TLS ClientHello
từ một source
tới một server
trong thời gian ngắn
```

Pseudo-rule/threshold concept:

```text
track by source IP
count ClientHello > N
within T seconds
```

Cần correlate với:

- `flow_id`.
- Destination IP/port.
- TLS version.
- SNI.
- Alert/reject count.

---

# 21. Zeek Detection Ideas

Nguồn log:

```text
ssl.log
conn.log
notice.log
weird.log
```

Feature:

- Handshake count.
- Version.
- Cipher.
- SNI.
- Established connections.
- Duration.
- Bytes transferred.
- Repeated short TLS sessions.

Zeek script có thể cảnh báo khi một source tạo nhiều TLS handshakes nhưng gần như không trao đổi application data.

---

# 22. Splunk Detection

Ví dụ với TLS logs:

```spl
index=network tls_event=client_hello
| bin _time span=10s
| stats count
        dc(flow_id) as flows
        values(tls_version) as versions
  by _time src_ip dest_ip dest_port
| where count > 20
```

Theo flow:

```spl
index=network tls_event=client_hello
| stats count by src_ip dest_ip flow_id
| where count > 1
```

Kết hợp server metrics:

```spl
index=network tls_event=client_hello
| stats count by _time, dest_ip
| join dest_ip _time [
    search index=metrics metric_name=cpu_usage
  ]
| where count > 20 AND cpu_usage > 80
```

---

# 23. Elastic/KQL

TLS ClientHello:

```kql
tls.client.x509.subject: *
```

Tùy ingest pipeline, có thể dùng:

```kql
network.protocol: "tls"
and event.action: "client_hello"
```

TLS version cũ:

```kql
tls.version:
  ("1.0" or "1.1" or "SSLv3")
```

Cần aggregation theo source, destination và flow để đếm nhiều handshake.

---

# 24. SIEM Correlation

High-confidence correlation:

```text
Nhiều ClientHello
+ cùng destination
+ ít application bytes
+ server CPU tăng
+ TLS alerts/resets tăng
```

Nguồn dữ liệu:

- Suricata.
- Zeek.
- Load balancer.
- Reverse proxy.
- Web server.
- TLS termination device.
- Infrastructure metrics.

---

# 25. Hardening

- Ưu tiên TLS 1.3.
- Tắt client-initiated renegotiation nếu không cần.
- Cập nhật OpenSSL/web server/load balancer.
- Chỉ cho phép cipher suites mạnh.
- Giới hạn handshake rate.
- Bật connection/request rate limiting.
- Dùng reverse proxy/CDN chống DoS.
- Bật session resumption phù hợp.
- Monitor CPU và TLS handshake latency.
- Tắt SSL/TLS versions lỗi thời.

---

# 26. Apache/Nginx Considerations

## Apache

- Cập nhật `mod_ssl`.
- Tắt legacy protocols.
- Review renegotiation requirements.
- Giới hạn connection/handshake rate.

## Nginx

- Dùng TLS versions hiện đại.
- Cấu hình cipher phù hợp.
- Rate-limit connections tại edge.
- Dùng upstream/load balancer metrics.

Cấu hình cụ thể phải test theo phiên bản phần mềm đang triển khai.

---

# 27. Incident Response Checklist

```text
[ ] Xác định source IP và destination service.
[ ] Đếm ClientHello theo flow và thời gian.
[ ] Phân biệt connection mới với renegotiation.
[ ] Kiểm tra TLS version/cipher suite.
[ ] Kiểm tra TLS alerts và resets.
[ ] Correlate CPU, memory và latency.
[ ] Rate-limit/block source nếu malicious.
[ ] Kiểm tra version và patch status.
[ ] Thu PCAP, proxy logs và metrics.
[ ] Xác định legitimate users bị ảnh hưởng.
```

---

# 28. Threat Hunting

Hunt theo:

- Nhiều ClientHello trong cùng TCP stream.
- ClientHello sau application data.
- TLS alerts tăng.
- Handshake-to-application-byte ratio cao.
- Nhiều short-lived TLS sessions.
- Legacy TLS versions.
- Cipher suite thay đổi bất thường.
- Một source nhắm nhiều TLS services.

---

# 29. Mapping với CDSA

## Security Operations & Monitoring

- Theo dõi TLS handshake rate.
- Correlate PCAP với proxy/load balancer metrics.
- Detect legacy TLS và cipher suites.

## Incident Response & Forensics

- Xác định attack timeline.
- Kiểm tra server resource impact.
- Phân biệt DoS với configuration lỗi.

## Threat Hunting

- Hunt repeated ClientHello.
- Hunt renegotiation trong established streams.
- Hunt handshake-only connections.
- Hunt TLS alerts và reset spikes.

## Detection Engineering

- Suricata/Zeek TLS correlation.
- Splunk/Elastic thresholds.
- Asset-role-aware baselines.
- Alert theo flow, không chỉ source IP.

---

# 30. Lab Setup

Môi trường tối thiểu:

```text
1 TLS-enabled web server
1 authorized test client
1 Wireshark/tshark sensor
Suricata hoặc Zeek
Splunk/ELK/Wazuh nhận TLS/network logs
Server CPU/latency metrics
```

Workflow:

```text
1. Mở SSL_renegotiation_edited.pcapng.
2. Lọc tls.record.content_type == 22.
3. Đếm ClientHello theo tcp.stream.
4. Xác định handshake xuất hiện sau application data.
5. Kiểm tra TLS version và cipher.
6. Correlate server metrics.
7. Viết detection và hardening.
```

---

# Key Takeaway

```text
Dấu hiệu nổi bật của SSL/TLS renegotiation abuse là nhiều ClientHello
trong cùng connection hoặc sau khi encrypted application data đã bắt đầu.
Không nên chỉ đếm ClientHello theo source vì connection mới hợp lệ cũng tạo ClientHello.
Detection hiệu quả cần correlate flow state, handshake count, application bytes và server resource usage.
TLS 1.3 và việc vô hiệu hóa legacy renegotiation giúp giảm đáng kể rủi ro.
```

SSL Renegotiation Attacks là chủ đề quan trọng của CDSA vì kết hợp TLS packet analysis, server telemetry, DoS detection, threat hunting và incident response.

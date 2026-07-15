---
layout: post
title: "Peculiar DNS Traffic"
date: 2026-07-15 00:26:35 +0700
categories: ["Intermediate Network Traffic Analysis", "DNS Analysis"]
tags: [dns, dns-enumeration, dns-tunneling, data-exfiltration, command-and-control, dga, txt-records, wireshark, tshark, suricata, zeek, splunk, elk, sigma, threat-hunting, incident-response, cdsa]
description: "Phân tích DNS traffic bất thường: DNS enumeration, TXT-based tunneling, encoded exfiltration, DGA, Wireshark/Tshark filters, SIEM detection và incident response."
toc: true
---

# Peculiar DNS Traffic

Bài này thuộc module **Intermediate Network Traffic Analysis**, tập trung vào các dấu hiệu bất thường trong DNS như enumeration, tunneling, data exfiltration, command-and-control và Domain Generation Algorithms.

```text
Mục tiêu: hiểu DNS lookup, nhận diện query volume bất thường,
phát hiện TXT tunneling, encoded payload, DGA và hành vi DNS hiếm gặp.
```

> Chỉ thực hành trong HTB, lab hoặc hệ thống có ủy quyền rõ ràng. Không triển khai DNS tunneling hay exfiltration ngoài phạm vi được phép.

---

## Timeline cập nhật

```text
Created at: 2026-07-15 00:26:35 +0700
Timezone: Asia/Ho_Chi_Minh
Module: Intermediate Network Traffic Analysis
Section: Peculiar DNS Traffic
Focus: DNS enumeration, TXT tunneling, DGA, exfiltration, threat hunting
```

---

# 1. DNS là gì?

DNS ánh xạ tên miền sang địa chỉ IP và ngược lại.

Ví dụ forward lookup:

```text
academy.hackthebox.com
→ 192.168.10.6
```

Ví dụ reverse lookup:

```text
192.168.10.6
→ academy.hackthebox.com
```

DNS thường dùng UDP/53, nhưng cũng có thể dùng TCP/53 trong các trường hợp như:

- Response lớn.
- Zone transfer.
- DNSSEC.
- Truncated UDP response.
- Một số triển khai tunneling.

---

# 2. Forward Lookup

Luồng cơ bản:

```text
Client
→ local cache
→ recursive resolver
→ root server
→ TLD server
→ authoritative server
→ client nhận A/AAAA record
```

Các record thường gặp:

| Record | Mục đích |
|---|---|
| A | Domain → IPv4 |
| AAAA | Domain → IPv6 |
| CNAME | Alias |
| MX | Mail server |
| NS | Authoritative nameserver |
| PTR | Reverse lookup |
| TXT | Text metadata |
| SOA | Zone authority metadata |

---

# 3. Reverse Lookup

Reverse DNS dùng PTR record.

IPv4 reverse zone:

```text
192.0.2.1
→ 1.2.0.192.in-addr.arpa
```

Reverse lookup hữu ích trong:

- Asset identification.
- Threat hunting.
- Network troubleshooting.
- Correlation IP ↔ hostname.

---

# 4. Vì sao DNS khó phân tích?

DNS volume thường rất lớn do:

- Browser.
- Operating system.
- EDR.
- Cloud services.
- CDNs.
- Software update.
- Telemetry.
- Background applications.

Malicious DNS có thể bị chìm trong lượng query hợp lệ.

Do đó, detection nên dựa trên:

```text
Volume
Query type
Length
Entropy
Frequency
Destination resolver
Domain age/context
Asset role
```

---

# 5. DNS Enumeration

DNS enumeration nhằm thu thập:

- Subdomains.
- Mail servers.
- Authoritative nameservers.
- TXT records.
- Internal hostnames.
- Service records.
- Zone metadata.

Dấu hiệu:

- Một source gửi nhiều query.
- Nhiều query type khác nhau.
- Query `ANY`.
- Nhiều subdomain guesses.
- NXDOMAIN tăng cao.
- Query tuần tự theo wordlist.

---

# 6. Wireshark Filters Cơ Bản

Tất cả DNS:

```wireshark
dns
```

Chỉ query:

```wireshark
dns.flags.response == 0
```

Chỉ response:

```wireshark
dns.flags.response == 1
```

Theo source:

```wireshark
ip.src == 192.168.10.5 && dns
```

Theo resolver:

```wireshark
ip.addr == 192.168.10.1 && dns
```

---

# 7. Query Types Trong Wireshark

A record:

```wireshark
dns.qry.type == 1
```

AAAA:

```wireshark
dns.qry.type == 28
```

TXT:

```wireshark
dns.qry.type == 16
```

PTR:

```wireshark
dns.qry.type == 12
```

MX:

```wireshark
dns.qry.type == 15
```

ANY:

```wireshark
dns.qry.type == 255
```

Query `ANY` xuất hiện lặp lại có thể là dấu hiệu enumeration hoặc abuse.

---

# 8. NXDOMAIN Enumeration

Attacker thử nhiều subdomains không tồn tại thường tạo:

```text
NXDOMAIN
```

Filter:

```wireshark
dns.flags.rcode == 3
```

Theo source:

```wireshark
ip.src == 192.168.10.5 &&
dns.flags.rcode == 3
```

Dấu hiệu:

- NXDOMAIN ratio cao.
- Nhiều unique subdomains.
- Query rate cao.
- Naming pattern giống wordlist.

---

# 9. DNS Tunneling Là Gì?

DNS tunneling nhúng dữ liệu vào DNS query hoặc response.

Attacker có thể dùng:

- Subdomain labels.
- TXT records.
- NULL/private record types.
- CNAME chains.
- Long query names.
- Encoded chunks.

Ví dụ:

```text
<encoded-data>.attacker-domain.com
```

hoặc TXT response chứa:

```text
Command/Data chunk
```

---

# 10. DNS Tunneling Flow

```text
Compromised host
→ query chứa encoded data
→ recursive resolver
→ attacker authoritative DNS

Attacker DNS
→ TXT/CNAME response chứa command
→ compromised host
```

Mục tiêu:

- Data exfiltration.
- Command-and-control.
- Bypass proxy/firewall.
- Beaconing.
- Malware configuration retrieval.

---

# 11. Dấu Hiệu DNS Tunneling

- Nhiều TXT queries.
- Query name rất dài.
- Subdomain entropy cao.
- Nhiều labels bất thường.
- Unique subdomain count cao.
- Query/response sizes lớn.
- Periodic beaconing.
- Một host query trực tiếp DNS ngoài whitelist.
- Payload Base32/Base64/hex-like.
- Domain hiếm hoặc không phù hợp business.

---

# 12. TXT Record Tunneling

Filter TXT:

```wireshark
dns.qry.type == 16
```

Theo source:

```wireshark
ip.src == 192.168.10.5 &&
dns.qry.type == 16
```

Response TXT:

```wireshark
dns.flags.response == 1 &&
dns.resp.type == 16
```

Nếu TXT chứa dữ liệu dài, encoded hoặc lặp theo chunks, cần điều tra.

---

# 13. Tìm Payload Trong Wireshark

Trong packet details:

```text
Domain Name System
→ Answers
→ TXT
```

Có thể:

```text
Right click field
→ Copy
→ Value
```

Sau đó decode trong môi trường phân tích.

Ví dụ Base64:

```bash
echo '<BASE64_VALUE>' | base64 -d
```

Hex:

```bash
echo '<HEX_VALUE>' | xxd -r -p
```

---

# 14. Double/Triple Encoding

Attacker có thể encode nhiều lớp:

```bash
echo '<VALUE>' | base64 -d | base64 -d | base64 -d
```

Nhưng cần thận trọng:

- Payload có thể là binary.
- Dữ liệu có thể encrypted.
- Output có thể chứa control characters.
- Không chạy file/payload được decode.

Luôn phân tích trong sandbox.

---

# 15. Query Name Length

Một DNS label tối đa 63 ký tự và FQDN tối đa khoảng 253 ký tự hiển thị.

Dấu hiệu tunneling:

- Nhiều labels gần 63 ký tự.
- Query name gần giới hạn.
- Labels có entropy cao.
- Không giống từ ngôn ngữ tự nhiên.

Wireshark filter gần đúng:

```wireshark
dns.qry.name.len > 50
```

Field support có thể khác theo phiên bản Wireshark; nếu không dùng được, trích query names bằng Tshark rồi tính độ dài.

---

# 16. Tshark Hunt

Liệt kê query:

```bash
tshark -r dns_enum_detection.pcapng   -Y "dns.flags.response == 0"   -T fields   -e frame.time   -e ip.src   -e ip.dst   -e dns.qry.name   -e dns.qry.type
```

Đếm query theo source:

```bash
tshark -r dns_enum_detection.pcapng   -Y "dns.flags.response == 0"   -T fields -e ip.src |
sort | uniq -c | sort -nr
```

TXT queries:

```bash
tshark -r dns_tunneling.pcapng   -Y "dns.qry.type == 16"   -T fields   -e ip.src   -e dns.qry.name
```

NXDOMAIN:

```bash
tshark -r dns_enum_detection.pcapng   -Y "dns.flags.rcode == 3"   -T fields -e ip.dst |
sort | uniq -c | sort -nr
```

---

# 17. Tìm Query Dài Bằng Shell

```bash
tshark -r dns_tunneling.pcapng   -Y "dns.flags.response == 0"   -T fields -e ip.src -e dns.qry.name |
awk '{print length($2), $1, $2}' |
sort -nr |
head
```

Unique subdomains:

```bash
tshark -r dns_tunneling.pcapng   -Y "dns.flags.response == 0"   -T fields -e dns.qry.name |
sort -u |
wc -l
```

---

# 18. Entropy và Encoded Labels

Các label đáng ngờ:

```text
ajd83kd92kslq0x...
4d5a900003000000...
U0dWc2JHOGdWMjl5...
```

Có thể là:

- Base32.
- Base64 URL-safe.
- Hex.
- Compressed/encrypted binary.

Entropy cao không tự động là malicious vì CDN, tracking và security products cũng dùng labels dài.

Cần correlate:

- Asset role.
- Destination domain.
- Query rate.
- Resolver path.
- Domain reputation/context.

---

# 19. DGA Detection

Domain Generation Algorithm tạo nhiều domain pseudo-random để malware tìm C2.

Dấu hiệu:

- Nhiều domain không tồn tại.
- Domain entropy cao.
- Query burst theo thời gian.
- Cùng suffix/TLD.
- Một host query hàng trăm domain lạ.
- Khi một domain resolve thành công, traffic C2 bắt đầu.

Ví dụ pattern:

```text
ajskd82jd.com
qowieur91.net
zxcmnq771.org
```

---

# 20. Direct-to-External DNS

Trong enterprise network, endpoint thường phải dùng resolver nội bộ.

Đáng ngờ khi:

```text
Client trực tiếp query 8.8.8.8
Client query DNS server ngoài whitelist
Client dùng TCP/53 kéo dài
Client dùng DoH/DoT ngoài policy
```

Filter external DNS:

```wireshark
dns &&
!(ip.dst == <INTERNAL_DNS_IP>)
```

Có thể chặn outbound UDP/TCP 53 ngoại trừ resolver được phê duyệt.

---

# 21. DoH và DoT

DNS over HTTPS và DNS over TLS mã hóa nội dung DNS.

Visibility giảm nếu không có:

- Endpoint telemetry.
- Proxy logs.
- TLS inspection.
- Resolver policy.
- Browser policy.
- SNI/destination monitoring.

Threat hunting nên theo dõi:

- Known DoH endpoints.
- Browser/process dùng resolver ngoài policy.
- Encrypted DNS connections từ server không cần thiết.

---

# 22. Suricata Detection Ideas

Theo dõi:

- DNS query length.
- TXT query volume.
- NXDOMAIN rate.
- External resolver usage.
- Rare domains.
- DNS tunneling signatures.

Pseudo-logic:

```text
TXT queries > N
from one source
to one domain
within T seconds
```

Suricata `eve.json` cung cấp:

- Query.
- RR type.
- Rcode.
- Answer.
- Flow metadata.

---

# 23. Zeek Detection Ideas

Nguồn log:

```text
dns.log
conn.log
notice.log
weird.log
```

Các feature hữu ích:

- Query length.
- Query type.
- Rcode.
- Unique subdomain count.
- Resolver.
- TTL.
- Answer count.
- Query interval.
- Domain entropy.

Zeek phù hợp để xây behavioral detection DNS.

---

# 24. Splunk Detection

High TXT volume:

```spl
index=dns query_type=TXT
| bin _time span=5m
| stats count
        dc(query) as unique_queries
        values(query) as samples
  by _time src_ip, registered_domain
| where count > 20 OR unique_queries > 10
```

NXDOMAIN ratio:

```spl
index=dns
| stats count
        count(eval(rcode="NXDOMAIN")) as nxdomain
        dc(query) as unique_queries
  by src_ip
| eval nx_ratio=nxdomain/count
| where count > 50 AND nx_ratio > 0.7
```

Long queries:

```spl
index=dns
| eval qlen=len(query)
| where qlen > 60
| stats count avg(qlen) max(qlen) by src_ip, registered_domain
```

External resolver:

```spl
index=network dest_port=53
| where NOT cidrmatch("192.168.0.0/16", dest_ip)
| stats count values(dest_ip) by src_ip
```

---

# 25. Elastic/KQL

TXT:

```kql
dns.question.type: "TXT"
```

NXDOMAIN:

```kql
dns.response_code: "NXDOMAIN"
```

Long query cần runtime/scripted field hoặc ingest pipeline tính:

```text
dns.question.name_length
```

External resolver:

```kql
destination.port: 53
and not destination.ip: (
  192.168.10.1
)
```

---

# 26. Sigma Detection Idea

Sigma cho DNS logs:

```yaml
title: Suspicious High Volume DNS TXT Queries
status: experimental
logsource:
  category: dns
detection:
  selection:
    QueryType: TXT
  condition: selection
falsepositives:
  - SPF/DKIM/DMARC lookups
  - Legitimate service discovery
level: medium
```

Threshold phải triển khai ở SIEM:

```text
TXT count cao
+ cùng source
+ cùng domain
+ query length lớn
```

---

# 27. IPFS và DNS/HTTP Traffic

IPFS là hệ thống phân tán theo mô hình content-addressed.

Threat actors có thể lợi dụng public IPFS gateways để lưu hoặc tải payload.

Dấu hiệu:

```text
HTTP/HTTPS tới public IPFS gateway
URI chứa /ipfs/<CID>
DNS resolve tới gateway hiếm gặp
Download binary từ content hash
```

Ví dụ URI pattern:

```text
/ipfs/Qm...
```

Không phải mọi IPFS traffic đều malicious; cần correlate với:

- Endpoint process.
- User role.
- File hash.
- Downloaded content.
- Subsequent execution.

---

# 28. Endpoint Correlation

DNS anomaly mạnh hơn khi correlate với process:

- `powershell.exe`.
- `python`.
- `curl`.
- Unknown executable.
- Browser extension.
- Malware service.
- Scheduled task.

Sysmon/EDR nên cho biết:

```text
Process nào tạo DNS query?
Query domain nào?
Sau đó process kết nối IP nào?
Có file nào được tải/chạy không?
```

---

# 29. Incident Response Checklist

```text
[ ] Xác định source host và process.
[ ] Xác định resolver được dùng.
[ ] Đếm query volume và unique names.
[ ] Kiểm tra query type, length và entropy.
[ ] Trích TXT/encoded payload trong sandbox.
[ ] Kiểm tra NXDOMAIN/DGA pattern.
[ ] Block domain/resolver nếu malicious.
[ ] Isolate host nếu nghi C2/exfiltration.
[ ] Thu PCAP, DNS logs, memory và endpoint evidence.
[ ] Xác định dữ liệu đã bị exfiltrate.
```

---

# 30. Prevention và Hardening

- Chỉ cho phép DNS qua resolver nội bộ.
- Block direct outbound TCP/UDP 53.
- Kiểm soát DoH/DoT theo policy.
- DNS logging tập trung.
- RPZ/sinkhole.
- Threat intelligence enrichment.
- Rate-limit bất thường.
- Monitor TXT/NULL/CNAME abuse.
- EDR process-to-DNS visibility.
- DNSSEC validation khi phù hợp.
- Segment endpoints và servers.

---

# 31. Mapping với CDSA

## Security Operations & Monitoring

- Theo dõi DNS query types.
- Baseline NXDOMAIN/TXT.
- Correlate resolver, process và domain.

## Incident Response & Forensics

- Trích payload.
- Decode trong sandbox.
- Xác định C2/exfiltration timeline.
- Thu DNS cache và endpoint evidence.

## Malware Analysis & Reverse Engineering

- Phân tích DGA.
- Decode DNS payload.
- Xác định command format.
- Reverse tunnel protocol.

## Threat Hunting

- Hunt long/encoded queries.
- Hunt rare TXT traffic.
- Hunt external resolvers.
- Hunt DGA/NXDOMAIN bursts.
- Hunt IPFS gateway downloads.

## Detection Engineering

- Zeek DNS analytics.
- Suricata DNS signatures.
- Splunk/Elastic thresholds.
- Sigma DNS rules.
- Asset-aware baselines.

---

# 32. Lab Setup

Môi trường tối thiểu:

```text
1 DNS resolver
1 client endpoint
1 Wireshark/tshark sensor
Suricata hoặc Zeek
Splunk/ELK/Wazuh nhận DNS/network logs
Endpoint telemetry
```

Workflow:

```text
1. Mở dns_enum_detection.pcapng.
2. Lọc dns và query ANY/NXDOMAIN.
3. Xác định source có volume cao.
4. Mở dns_tunneling.pcapng.
5. Lọc TXT queries/responses.
6. Trích và decode payload.
7. Kiểm tra query length và entropy.
8. Viết SIEM detection và IR timeline.
```

---

# Key Takeaway

```text
DNS enumeration thường tạo nhiều query, ANY requests và NXDOMAIN.
DNS tunneling thường lộ qua TXT volume, query names dài, entropy cao và beaconing.
Encoded payload có thể cần decode nhiều lớp nhưng không được thực thi trực tiếp.
Detection hiệu quả phải kết hợp DNS metadata, resolver policy, domain context và endpoint process.
```

Peculiar DNS Traffic là chủ đề quan trọng của CDSA vì kết nối packet analysis, data exfiltration, command-and-control detection, malware analysis và incident response.

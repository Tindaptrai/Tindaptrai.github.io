---
layout: post
title: "Zeek Fundamentals"
date: 2026-07-18 00:16:52 +0700
categories: ["Working with IDS/IPS", "Zeek"]
tags: [zeek, network-security-monitoring, nsm, packet-analysis, conn-log, dns-log, http-log, zeek-cut, scripting, anomaly-detection, threat-hunting, incident-response, splunk, elk, cdsa]
description: "Tổng quan Zeek: kiến trúc event engine và script interpreter, phân tích PCAP/live traffic, các log quan trọng, zeek-cut, scripting và ứng dụng trong SOC, Threat Hunting và DFIR."
toc: true
---

# Zeek Fundamentals

Bài này thuộc module **Working with IDS/IPS**, tập trung vào Zeek như một nền tảng **Network Security Monitoring**, phân tích giao thức và xây dựng logic phát hiện dựa trên hành vi.

```text
Mục tiêu:
- Hiểu Zeek khác IDS signature truyền thống như thế nào.
- Nắm kiến trúc event engine và script interpreter.
- Phân tích PCAP và live traffic.
- Đọc conn.log, dns.log, http.log và các log liên quan.
- Dùng zeek-cut, grep, zgrep và jq.
- Áp dụng Zeek vào SOC, Threat Hunting và Incident Response.
```

> Chỉ phân tích PCAP hoặc giám sát traffic trong HTB, lab hay hệ thống có ủy quyền rõ ràng.

---

## Timeline

```text
Created at: 2026-07-18 00:16:52 +0700
Timezone: Asia/Ho_Chi_Minh
Module: Working with IDS/IPS
Section: Zeek Fundamentals
Focus: NSM, protocol analysis, logs, scripting, threat hunting
```

---

# 1. Zeek Là Gì?

Zeek là một **network traffic analyzer** mã nguồn mở.

Khác với IDS thuần signature, Zeek tập trung biến network traffic thành dữ liệu có cấu trúc:

```text
Packets
→ protocol parsing
→ events
→ scripts/policy
→ structured logs
→ alerts/notices
```

Zeek phù hợp cho:

- Network Security Monitoring.
- Threat Hunting.
- Incident Response.
- Protocol analysis.
- Behavioral detection.
- Network troubleshooting.
- Traffic measurement.

---

# 2. Zeek Không Chỉ Là IDS Signature

Zeek không chỉ hỏi:

```text
Packet có khớp signature hay không?
```

Nó còn hỏi:

```text
Host nào kết nối tới đâu?
Bao lâu?
Bao nhiêu byte?
Dùng protocol gì?
Có đúng thứ tự giao thức không?
Hành vi có bất thường theo thời gian không?
```

Điều này giúp Zeek hỗ trợ:

- Semantic misuse detection.
- Anomaly detection.
- Behavioral analysis.
- Stateful tracking.
- Long-term network baselining.

---

# 3. Các Mode Vận Hành

Zeek hỗ trợ:

## Passive Traffic Analysis

Zeek quan sát traffic từ:

- SPAN port.
- Network TAP.
- Mirror interface.
- Packet broker.

Zeek không chặn traffic theo mặc định.

## Live Capture

Dùng libpcap để đọc từ interface:

```bash
sudo zeek -i ens160
```

## Offline PCAP Analysis

```bash
zeek -r suspicious.pcap
```

Log được tạo trong thư mục hiện tại.

## Cluster Mode

Dùng cho môi trường lớn:

```text
Manager
Proxy
Worker
Logger
```

Cluster giúp phân phối capture, parsing và logging.

---

# 4. Kiến Trúc Zeek

Zeek có hai thành phần chính:

```text
Event Engine
Script Interpreter
```

## Event Engine

Nhận packet và biến thành event mức cao.

Ví dụ:

```text
HTTP request packet stream
→ http_request event
```

Event engine cung cấp thông tin trung lập:

```text
Điều gì đã xảy ra?
```

Nó chưa kết luận:

```text
Hành vi đó có malicious hay không?
```

## Script Interpreter

Thực thi Zeek scripts để:

- Áp policy.
- Theo dõi state.
- Tạo notice.
- Ghi log tùy chỉnh.
- Correlate nhiều event.
- Phát hiện behavior.

---

# 5. Event Queue

Events được đưa vào queue và xử lý theo thứ tự.

Ví dụ:

```text
new_connection
→ protocol confirmation
→ dns_request
→ dns_response
→ connection_state_remove
```

Điều này cho phép script theo dõi network state theo thời gian.

Nhiều event được định nghĩa trong:

```text
scripts/base/bif/plugins/
```

---

# 6. Zeek Logs Quan Trọng

## conn.log

Ghi mọi connection:

- TCP.
- UDP.
- ICMP.

Các field thường gặp:

```text
ts
uid
id.orig_h
id.orig_p
id.resp_h
id.resp_p
proto
service
duration
orig_bytes
resp_bytes
conn_state
history
```

## dns.log

Ghi:

- Query.
- Query type.
- Response.
- RCODE.
- Answers.
- TTL.

## http.log

Ghi:

- Host.
- URI.
- Method.
- Referrer.
- User-Agent.
- Status code.
- Request/response body length.
- File IDs.

## ftp.log

Ghi FTP command và response.

## smtp.log

Ghi mail transaction:

- Sender.
- Recipient.
- Subject metadata.
- Server response.

---

# 7. Một Số Log Khác

Zeek có thể tạo:

```text
ssl.log / tls.log
ssh.log
files.log
notice.log
weird.log
x509.log
smtp.log
ftp.log
dhcp.log
ntp.log
smb_files.log
smb_mapping.log
kerberos.log
rdp.log
tunnel.log
```

Tên log và field phụ thuộc:

- Zeek version.
- Protocol analyzer.
- Loaded scripts.
- Package.
- Site policy.

---

# 8. UID Trong Zeek

Mỗi connection thường có một `uid`.

Ví dụ:

```text
C8Tful1TvM3Zf5x8l
```

UID giúp correlate:

```text
conn.log
↕
dns.log
↕
http.log
↕
files.log
↕
notice.log
```

Ví dụ:

```bash
grep 'C8Tful1TvM3Zf5x8l' *.log
```

Trong SIEM, `uid` là field quan trọng để dựng timeline theo flow.

---

# 9. Phân Tích PCAP

Chạy:

```bash
mkdir -p ~/zeek-lab
cd ~/zeek-lab

zeek -r /path/to/suspicious.pcap
```

Kiểm tra log:

```bash
ls -lah
```

Có thể xuất hiện:

```text
conn.log
dns.log
http.log
ssl.log
files.log
notice.log
weird.log
```

---

# 10. Live Analysis

```bash
sudo zeek -i ens160
```

Trong production thường dùng `zeekctl`:

```bash
sudo zeekctl deploy
sudo zeekctl status
sudo zeekctl diag
```

Không nên chạy live capture trên production nếu chưa kiểm tra:

- Interface.
- Packet loss.
- CPU.
- Disk.
- Log rotation.
- Cluster design.

---

# 11. zeek-cut

`zeek-cut` trích các cột từ Zeek ASCII logs.

Ví dụ:

```bash
cat conn.log |
zeek-cut ts id.orig_h id.resp_h id.resp_p service
```

HTTP:

```bash
cat http.log |
zeek-cut ts id.orig_h host uri status_code user_agent
```

DNS:

```bash
cat dns.log |
zeek-cut ts id.orig_h query qtype_name rcode_name answers
```

---

# 12. Top Talkers

```bash
cat conn.log |
zeek-cut id.orig_h |
sort |
uniq -c |
sort -nr |
head
```

Top destinations:

```bash
cat conn.log |
zeek-cut id.resp_h |
sort |
uniq -c |
sort -nr |
head
```

Top destination ports:

```bash
cat conn.log |
zeek-cut id.resp_p |
sort |
uniq -c |
sort -nr |
head
```

---

# 13. Tìm HTTP Bất Thường

Top host:

```bash
cat http.log |
zeek-cut host |
sort |
uniq -c |
sort -nr |
head
```

Status 404:

```bash
cat http.log |
zeek-cut ts id.orig_h host uri status_code |
awk '$5 == 404'
```

Suspicious URI:

```bash
cat http.log |
zeek-cut ts id.orig_h host uri |
grep -Ei '/\.git|/\.env|cmd=|exec=|powershell'
```

---

# 14. DNS Hunting

NXDOMAIN:

```bash
cat dns.log |
zeek-cut ts id.orig_h query rcode_name |
awk '$4 == "NXDOMAIN"'
```

TXT queries:

```bash
cat dns.log |
zeek-cut ts id.orig_h query qtype_name |
awk '$4 == "TXT"'
```

Long query:

```bash
cat dns.log |
zeek-cut id.orig_h query |
awk '{print length($2), $1, $2}' |
sort -nr |
head
```

---

# 15. conn.log States

Các `conn_state` phổ biến:

| State | Ý nghĩa |
|---|---|
| `S0` | SYN thấy nhưng không có response |
| `S1` | Established nhưng chưa thấy close |
| `SF` | Established và đóng bình thường |
| `REJ` | Connection bị reject |
| `RSTO` | Originator reset |
| `RSTR` | Responder reset |
| `OTH` | Không khớp state bình thường |

Nhiều `S0` có thể chỉ ra:

- SYN scan.
- Firewall drop.
- Host down.
- Asymmetric capture.

---

# 16. history Field

`history` ghi chuỗi event TCP.

Ví dụ ký tự:

```text
S = SYN
h = SYN-ACK
A = ACK
D = data
F = FIN
R = RST
```

Uppercase thường là originator, lowercase là responder.

Ví dụ:

```text
ShADadFf
```

giúp analyst hiểu flow mà không cần mở PCAP ngay.

---

# 17. weird.log

`weird.log` ghi protocol anomalies:

- Invalid checksum.
- Truncated packet.
- Unexpected message order.
- Malformed protocol.
- TCP overlap.
- DNS inconsistency.
- Protocol violation.

Không phải mọi weird event đều malicious.

Cần correlate:

```text
Weird type
+ source
+ frequency
+ service
+ asset role
+ packet loss
```

---

# 18. notice.log

`notice.log` chứa event mà Zeek policy đánh giá là đáng chú ý.

Ví dụ:

- Port scan.
- SSH brute force.
- Certificate anomaly.
- Malware indicator.
- Policy violation.
- Custom script alert.

Các field:

```text
note
msg
sub
src
dst
p
actions
suppress_for
```

---

# 19. files.log

Zeek có thể theo dõi file được truyền qua protocol.

Field thường gặp:

```text
fuid
tx_hosts
rx_hosts
source
mime_type
filename
total_bytes
md5
sha1
sha256
extracted
```

Workflow DFIR:

```text
files.log
→ fuid
→ protocol log
→ connection UID
→ source/destination
→ extracted file
→ YARA/static analysis
```

---

# 20. File Extraction

Zeek script có thể cấu hình trích xuất file.

Concept:

```zeek
@load base/files/extract
```

Có thể thêm policy theo:

- MIME type.
- File size.
- Protocol.
- Source/destination.
- Hash.
- Indicator.

Không nên trích tất cả file trong production nếu chưa capacity-plan disk.

---

# 21. Log Rotation và Gzip

Zeek thường rotate log theo giờ trong deployment tiêu chuẩn.

Log cũ có thể nằm trong thư mục:

```text
YYYY-MM-DD/
```

và được gzip:

```text
conn.12:00:00-13:00:00.log.gz
```

Đọc:

```bash
zcat conn.log.gz
```

Tìm kiếm:

```bash
zgrep '192.168.10.5' conn.log.gz
```

Nhiều hệ thống dùng:

```bash
gzcat
```

thay cho `zcat`.

---

# 22. JSON Output

Zeek có thể xuất JSON để dễ ingest SIEM.

Chạy offline:

```bash
zeek -C -r suspicious.pcap LogAscii::use_json=T
```

`-C` bỏ qua checksum verification, hữu ích với PCAP có checksum offloading artifacts.

JSON phù hợp cho:

- Splunk.
- ELK.
- Wazuh.
- Logstash.
- jq.

---

# 23. jq Với Zeek JSON

HTTP status 404:

```bash
jq -c   'select(.status_code == 404)'   http.log
```

DNS TXT:

```bash
jq -c   'select(.qtype_name == "TXT")'   dns.log
```

Top source cần dùng shell aggregation hoặc SIEM.

---

# 24. Zeek Scripting Cơ Bản

Ví dụ event handler:

```zeek
event zeek_init()
    {
    print "Zeek script loaded";
    }
```

HTTP request:

```zeek
event http_request(
    c: connection,
    method: string,
    original_URI: string,
    unescaped_URI: string,
    version: string)
    {
    print fmt(
        "HTTP %s %s -> %s%s",
        method,
        c$id$orig_h,
        c$http$host,
        unescaped_URI
    );
    }
```

---

# 25. Tạo Notice Đơn Giản

Concept:

```zeek
@load base/frameworks/notice

module Local;

export
    {
    redef enum Notice::Type += {
        Suspicious_HTTP_URI
    };
    }

event http_request(
    c: connection,
    method: string,
    original_URI: string,
    unescaped_URI: string,
    version: string)
    {
    if ( /cmd=/ in unescaped_URI )
        NOTICE([
            $note=Suspicious_HTTP_URI,
            $msg=fmt("Suspicious URI from %s", c$id$orig_h),
            $conn=c
        ]);
    }
```

Script phải được kiểm thử trong lab.

---

# 26. Zeek Package Manager

Zeek packages mở rộng khả năng:

```bash
zkg search <keyword>
zkg install <package>
zkg list
```

Use cases:

- JA3/JA4.
- Threat intelligence.
- Detection scripts.
- Log enrichment.
- Protocol analyzers.

Chỉ cài package từ nguồn tin cậy và review code trước production.

---

# 27. Intel Framework

Zeek có thể ingest IOC:

- IP.
- Domain.
- URL.
- File hash.
- Email.
- Certificate.

Flow:

```text
Threat intelligence feed
→ Zeek Intel Framework
→ network event match
→ intel.log / notice.log
→ SIEM alert
```

Intel match không tự động là true positive; cần context.

---

# 28. Tunnel Detection

Zeek hỗ trợ phân tích:

- Teredo.
- GRE.
- VXLAN.
- IP-in-IP.
- Encapsulation.
- Nested protocol.

`tunnel.log` giúp thấy:

- Tunnel type.
- Outer/inner addresses.
- Encapsulated traffic.
- Unexpected tunnel use.

Đây là use case quan trọng khi attacker né network controls.

---

# 29. Protocol Detection Không Phụ Thuộc Port

Zeek có thể nhận diện application protocol ngoài port mặc định.

Ví dụ:

```text
HTTP trên 8088
SSH trên 443
DNS trên port lạ
TLS trên 8443
```

Điều này giúp phát hiện:

- Protocol tunneling.
- Service hiding.
- Misconfiguration.
- Malware C2.

---

# 30. Splunk Integration

Pipeline:

```text
Zeek logs
→ Filebeat/Logstash/Universal Forwarder
→ Splunk
→ hunting/correlation
```

Top connections:

```spl
index=zeek sourcetype=zeek:conn
| stats count
        sum(orig_bytes) as bytes_out
        sum(resp_bytes) as bytes_in
  by id_orig_h, id_resp_h, id_resp_p, service
| sort -count
```

NXDOMAIN:

```spl
index=zeek sourcetype=zeek:dns
rcode_name=NXDOMAIN
| stats count dc(query) as unique_queries by id_orig_h
| sort -count
```

---

# 31. Elastic/KQL

Connections:

```kql
event.dataset: "zeek.conn"
```

DNS:

```kql
event.dataset: "zeek.dns"
```

HTTP:

```kql
event.dataset: "zeek.http"
```

Field mapping phụ thuộc ingest pipeline/ECS.

---

# 32. Threat Hunting Use Cases

- Rare destination.
- High NXDOMAIN ratio.
- Long DNS query.
- HTTP enumeration.
- Beaconing.
- Large outbound transfer.
- Protocol on unusual port.
- TLS certificate anomaly.
- Repeated failed SSH.
- Suspicious tunnel.
- Internal lateral movement.

---

# 33. Incident Response Workflow

```text
1. Xác định alert hoặc IOC.
2. Tìm UID trong conn.log.
3. Correlate DNS/HTTP/TLS/file logs.
4. Xác định source/destination.
5. Xây timeline.
6. Xác định data transfer.
7. Correlate endpoint process.
8. Hunt các host khác.
9. Thu PCAP nếu có.
10. Preserve logs.
```

---

# 34. Zeek và Suricata

| Tiêu chí | Zeek | Suricata |
|---|---|---|
| Mục tiêu chính | NSM và behavioral analysis | IDS/IPS và signature detection |
| Output | Rich structured logs | EVE JSON, alerts, flows |
| Scripting | Zeek language | Rules + Lua |
| Inline blocking | Không phải mục tiêu chính | Có |
| Protocol metadata | Rất mạnh | Mạnh |
| Signature detection | Có pattern/framework | Rất mạnh |
| Hunting | Rất phù hợp | Phù hợp |

Trong SOC, hai công cụ bổ trợ nhau:

```text
Suricata alert
+ Zeek context
+ EDR telemetry
= incident có độ tin cậy cao hơn
```

---

# 35. Sensor Health

Zeek cần được theo dõi:

- Packet loss.
- Capture drops.
- Log backlog.
- CPU.
- Memory.
- Disk.
- Worker imbalance.
- Script errors.
- Reporter warnings.

Trong cluster:

```bash
zeekctl status
zeekctl diag
```

Blind spot có thể xảy ra nếu sensor drop packet.

---

# 36. Mapping Với CDSA

## Security Operations & Monitoring

- Structured network logs.
- SIEM ingestion.
- Baseline traffic.
- Network visibility.

## Incident Response & Forensics

- UID correlation.
- Timeline reconstruction.
- File/network evidence.
- Session-level analysis.

## Malware Analysis & Reverse Engineering

- C2 protocol profiling.
- File extraction.
- Behavioral network indicators.
- Beacon analysis.

## Threat Hunting

- Rare destinations.
- DNS anomalies.
- Tunnels.
- Protocol misuse.
- Lateral movement.

## Detection Engineering

- Zeek scripts.
- Notice framework.
- Intel framework.
- Behavioral rules.
- Correlation logic.

---

# 37. Lab Checklist

```bash
# Offline PCAP
mkdir -p ~/zeek-lab
cd ~/zeek-lab
zeek -C -r suspicious.pcap

# Xem log
ls -lah

# Connections
cat conn.log |
zeek-cut ts uid id.orig_h id.resp_h id.resp_p service conn_state

# DNS
cat dns.log |
zeek-cut ts id.orig_h query qtype_name rcode_name answers

# HTTP
cat http.log |
zeek-cut ts id.orig_h host uri status_code user_agent

# Tìm UID
grep '<UID>' *.log
```

---

# Key Takeaway

```text
Zeek không chỉ tìm signature.
Nó biến network traffic thành event và log có cấu trúc,
sau đó dùng scripting để áp policy và phát hiện behavior.

Giá trị lớn nhất của Zeek là:
protocol-aware metadata
+ state tracking
+ UID correlation
+ scripting
+ historical hunting.
```

Zeek Fundamentals là nền tảng quan trọng trong CDSA vì kết nối Network Security Monitoring, Threat Hunting, Incident Response, malware traffic analysis và Detection Engineering.

---

# Tài Liệu Tham Khảo

- [Zeek Events](https://docs.zeek.org/en/stable/scripts/base/bif/)
- [Zeek Logs](https://docs.zeek.org/en/master/logs/index.html)
- [Zeek Examples](https://docs.zeek.org/en/stable/examples/index.html)
- [Zeek Quick Start](https://docs.zeek.org/en/stable/quickstart/index.html)

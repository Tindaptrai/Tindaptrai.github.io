---
layout: post
title: "Intrusion Detection With Zeek"
date: 2026-07-18 00:41:52 +0700
categories: ["Working with IDS/IPS", "Zeek"]
tags: [zeek, intrusion-detection, beaconing, dns-exfiltration, tls-exfiltration, psexec, smb, dce-rpc, network-security-monitoring, threat-hunting, incident-response, splunk, elk, cdsa]
description: "Phát hiện xâm nhập với Zeek qua các use case beaconing, DNS exfiltration, TLS exfiltration và PsExec bằng conn.log, dns.log, smb_files.log, dce_rpc.log và smb_mapping.log."
toc: true
---

# Intrusion Detection With Zeek

Bài này thuộc module **Working with IDS/IPS**, tập trung vào cách dùng Zeek để phát hiện hành vi xâm nhập từ network metadata thay vì chỉ dựa vào signature.

```text
Mục tiêu:
- Nhận diện C2 beaconing trong conn.log.
- Phát hiện DNS exfiltration trong dns.log.
- Phát hiện TLS exfiltration bằng flow volume.
- Nhận diện PsExec qua SMB và DCE/RPC.
- Correlate log bằng uid.
- Xây workflow Threat Hunting và Incident Response.
```

> Chỉ phân tích PCAP hoặc giám sát traffic trong HTB, lab hay hệ thống có ủy quyền rõ ràng.

---

## Timeline

```text
Created at: 2026-07-18 00:41:52 +0700
Timezone: Asia/Ho_Chi_Minh
Module: Working with IDS/IPS
Section: Intrusion Detection With Zeek
Focus: Beaconing, DNS/TLS exfiltration, PsExec, SMB/DCE-RPC
```

---

# 1. Vì Sao Zeek Phù Hợp Cho Intrusion Detection?

Zeek biến traffic thành telemetry có cấu trúc:

```text
Packet stream
→ protocol event
→ connection/application logs
→ behavioral analytics
→ notice/hunt/incident
```

Điểm mạnh:

- Theo dõi state theo thời gian.
- Correlate nhiều protocol.
- Phát hiện hành vi lặp lại.
- Quan sát traffic kể cả khi payload bị mã hóa.
- Dễ đưa vào SIEM.

---

# 2. Use Case 1 – Beaconing Malware

Beaconing là hành vi malware/C2 kết nối định kỳ tới cùng server để:

- Nhận lệnh.
- Gửi trạng thái.
- Exfiltrate dữ liệu.
- Duy trì session.

Dấu hiệu thường gặp:

```text
Cùng source
→ cùng destination
→ khoảng thời gian gần cố định
→ kích thước request/response gần giống
→ lặp lại nhiều lần
```

---

# 3. Phân Tích PowerShell Empire PCAP

PCAP:

```text
/home/htb-student/pcaps/psempire.pcap
```

Chạy Zeek:

```bash
mkdir -p ~/zeek-lab/psempire
cd ~/zeek-lab/psempire

/usr/local/zeek/bin/zeek   -C   -r /home/htb-student/pcaps/psempire.pcap
```

`-C` bỏ qua checksum verification, hữu ích khi PCAP có checksum offloading artifacts.

---

# 4. Beaconing Trong conn.log

Trong `conn.log`, cần tập trung:

```text
ts
id.orig_h
id.resp_h
id.resp_p
service
duration
orig_bytes
resp_bytes
conn_state
```

Ví dụ pattern:

```text
192.168.56.14
→ 51.15.197.127:80
→ mỗi khoảng 5 giây
→ response gần 399 byte
```

Đây là tín hiệu beaconing mạnh khi kết hợp:

- Destination hiếm.
- Interval đều.
- Flow nhỏ.
- User/process không bình thường.
- Nhiều connection ngắn.

---

# 5. Trích Flow Bằng zeek-cut

```bash
cat conn.log |
/usr/local/zeek/bin/zeek-cut   ts id.orig_h id.resp_h id.resp_p service orig_bytes resp_bytes
```

Lọc theo destination:

```bash
cat conn.log |
/usr/local/zeek/bin/zeek-cut   ts id.orig_h id.resp_h id.resp_p orig_bytes resp_bytes |
awk '$3 == "51.15.197.127"'
```

---

# 6. Tính Beacon Interval

```bash
cat conn.log |
/usr/local/zeek/bin/zeek-cut   ts id.orig_h id.resp_h id.resp_p |
awk '$3 == "51.15.197.127" {print $1}' |
awk 'NR==1 {prev=$1; next} {print $1-prev; prev=$1}'
```

Nếu nhiều interval quanh:

```text
5.2
5.3
5.2
5.3
```

thì cần điều tra thêm.

---

# 7. Beacon Detection Concept

Một behavioral rule có thể dựa trên:

```text
connection_count > threshold
AND interval_stdev thấp
AND same src/dst
AND low bytes per flow
AND destination rarity cao
```

Không nên chỉ dựa vào interval vì:

- Monitoring agent.
- Health check.
- NTP.
- Software updater.
- Telemetry service.

cũng có thể tạo periodic traffic.

---

# 8. Splunk Beaconing Query

```spl
index=zeek sourcetype=zeek:conn
| sort 0 id_orig_h id_resp_h id_resp_p _time
| streamstats current=f last(_time) as previous
  by id_orig_h id_resp_h id_resp_p
| eval interval=_time-previous
| stats count
        avg(interval) as avg_interval
        stdev(interval) as interval_stdev
        avg(orig_bytes) as avg_bytes_out
        avg(resp_bytes) as avg_bytes_in
  by id_orig_h id_resp_h id_resp_p
| where count > 10 AND interval_stdev < 2
| sort interval_stdev
```

Field name phụ thuộc ingest pipeline.

---

# 9. Use Case 2 – DNS Exfiltration

DNS exfiltration mã hóa/chia nhỏ dữ liệu vào:

- Subdomain labels.
- TXT records.
- Query names.
- Response data.

Dấu hiệu:

```text
Long subdomains
High subdomain cardinality
High entropy labels
Sequential labels
Nhiều query tới cùng parent domain
TXT query bất thường
NXDOMAIN ratio cao
```

---

# 10. Phân Tích DNS Exfil PCAP

PCAP:

```text
/home/htb-student/pcaps/dnsexfil.pcapng
```

Chạy:

```bash
mkdir -p ~/zeek-lab/dnsexfil
cd ~/zeek-lab/dnsexfil

/usr/local/zeek/bin/zeek   -C   -r /home/htb-student/pcaps/dnsexfil.pcapng
```

---

# 11. dns.log Fields

Các field quan trọng:

```text
ts
uid
id.orig_h
id.resp_h
query
qtype_name
rcode_name
answers
TTLs
```

Trong mẫu lab, nhiều query có dạng:

```text
www.<data>.<sequence>.456c54f2.blue.letsgohunt.online
cdn.<data>.456c54f2.blue.letsgohunt.online
post.<data>.<sequence>.456c54f2.blue.letsgohunt.online
```

Số lượng lớn subdomain động là dấu hiệu cần điều tra.

---

# 12. Trích DNS Queries

```bash
cat dns.log |
/usr/local/zeek/bin/zeek-cut query
```

Giới hạn số label:

```bash
cat dns.log |
/usr/local/zeek/bin/zeek-cut query |
cut -d . -f1-7
```

Tìm domain cụ thể:

```bash
cat dns.log |
/usr/local/zeek/bin/zeek-cut   ts id.orig_h query qtype_name rcode_name |
grep 'letsgohunt.online'
```

---

# 13. Top Parent Domains

Cách đơn giản:

```bash
cat dns.log |
/usr/local/zeek/bin/zeek-cut query |
awk -F. 'NF>=2 {print $(NF-1)"."$NF}' |
sort |
uniq -c |
sort -nr |
head
```

Cần lưu ý public suffix như:

```text
co.uk
com.au
```

Khi production nên dùng parser hỗ trợ Public Suffix List thay vì cắt hai label cuối một cách cứng nhắc.

---

# 14. Subdomain Cardinality

```bash
cat dns.log |
/usr/local/zeek/bin/zeek-cut id.orig_h query |
awk -F'	' '{print $1, $2}' |
sort -u |
awk '{count[$1]++} END {for (h in count) print count[h], h}' |
sort -nr
```

Trong SIEM, đếm:

```text
distinct query
distinct subdomain
per source
per parent domain
per time window
```

---

# 15. Long DNS Queries

```bash
cat dns.log |
/usr/local/zeek/bin/zeek-cut id.orig_h query |
awk '{print length($2), $1, $2}' |
sort -nr |
head
```

Query dài không tự động là exfiltration. CDN, DKIM, tracking và security products cũng tạo domain dài.

---

# 16. DNS Exfil Detection Logic

Một detection mạnh hơn:

```text
Same source
+ same parent domain
+ nhiều unique subdomain
+ label dài/entropy cao
+ tốc độ query cao
+ response pattern lạ
+ domain hiếm
```

Có thể thêm:

- Domain age.
- Threat intelligence.
- Asset role.
- User/process telemetry.
- Query type.

---

# 17. Splunk DNS Exfil Query

```spl
index=zeek sourcetype=zeek:dns
| eval query_len=len(query)
| rex field=query "(?<parent_domain>[^.]+\.[^.]+)$"
| stats count
        dc(query) as unique_queries
        avg(query_len) as avg_query_len
        max(query_len) as max_query_len
  by id_orig_h parent_domain
| where unique_queries > 50 AND avg_query_len > 40
| sort -unique_queries
```

Cần điều chỉnh parent-domain extraction theo Public Suffix List.

---

# 18. Use Case 3 – TLS Exfiltration

TLS mã hóa payload nhưng `conn.log` vẫn cho thấy:

- Source/destination.
- Port/service.
- Duration.
- Bytes sent.
- Bytes received.
- Flow count.
- Timing.

Điều này đủ để phát hiện:

- Upload lớn.
- Unusual destination.
- Long-lived connection.
- Repeated sessions.
- Asymmetric byte ratio.

---

# 19. Phân Tích TLS Exfil PCAP

PCAP:

```text
/home/htb-student/pcaps/tlsexfil.pcap
```

Chạy:

```bash
mkdir -p ~/zeek-lab/tlsexfil
cd ~/zeek-lab/tlsexfil

/usr/local/zeek/bin/zeek   -C   -r /home/htb-student/pcaps/tlsexfil.pcap
```

---

# 20. Tổng Hợp Bytes Theo Host Pair

```bash
cat conn.log |
/usr/local/zeek/bin/zeek-cut   id.orig_h id.resp_h orig_bytes |
sort |
grep -v -e '^$' |
grep -v '-' |
datamash -g 1,2 sum 3 |
sort -k 3 -rn |
head -10
```

Logic:

```text
Zeek conn.log
→ chọn source, destination, bytes sent
→ group theo cặp host
→ cộng tổng orig_bytes
→ xếp hạng giảm dần
```

---

# 21. Ý Nghĩa Kết Quả TLS Exfil

Ví dụ:

```text
10.0.10.100 → 192.168.151.181 → ~270 MB
```

Tín hiệu đáng chú ý:

- Một source gửi volume lớn.
- Destination không phổ biến.
- Nhiều TLS flow lặp lại.
- Một số connection có `orig_bytes` rất cao.
- Traffic xuất hiện ngoài baseline.

---

# 22. Không Nhầm Hướng Bytes

Trong Zeek:

```text
orig_bytes = bytes từ originator tới responder
resp_bytes = bytes từ responder về originator
```

Nếu host nội bộ là `id.orig_h`:

```text
orig_bytes cao
→ có thể là upload/exfiltration
```

Nếu host nội bộ là `id.resp_h`, phải xem `resp_bytes` hoặc đảo hướng phân tích.

---

# 23. TLS Exfil Detection Logic

```text
Internal source
+ external/rare destination
+ high orig_bytes
+ unusual byte ratio
+ repeated TLS sessions
+ sensitive asset
+ process không hợp lệ
```

Ví dụ ratio:

```text
orig_bytes / (resp_bytes + 1)
```

Ratio cao có thể chỉ ra outbound transfer.

---

# 24. Splunk TLS Exfil Query

```spl
index=zeek sourcetype=zeek:conn service=ssl
| stats sum(orig_bytes) as bytes_out
        sum(resp_bytes) as bytes_in
        count as connections
  by id_orig_h id_resp_h
| eval outbound_ratio=bytes_out/(bytes_in+1)
| where bytes_out > 100000000 AND outbound_ratio > 3
| sort -bytes_out
```

Ngưỡng phải dựa trên baseline thực tế.

---

# 25. Use Case 4 – PsExec Detection

PsExec thường tạo chuỗi hoạt động:

```text
SMB ADMIN$ upload
→ PSEXESVC.exe
→ IPC$ named pipe
→ svcctl RPC
→ CreateService
→ StartService
→ DeleteService
→ file cleanup
```

PsExec có thể là:

- Quản trị hợp lệ.
- Red Team activity.
- Lateral movement.
- Remote execution sau credential compromise.

---

# 26. Phân Tích PsExec PCAP

PCAP:

```text
/home/htb-student/pcaps/psexec_add_user.pcap
```

Chạy:

```bash
mkdir -p ~/zeek-lab/psexec
cd ~/zeek-lab/psexec

/usr/local/zeek/bin/zeek   -C   -r /home/htb-student/pcaps/psexec_add_user.pcap
```

Logs cần kiểm tra:

```text
smb_files.log
smb_mapping.log
dce_rpc.log
conn.log
```

---

# 27. smb_files.log

Dấu hiệu:

```text
path: \\dc1\ADMIN$
name: PSEXESVC.exe
action: SMB::FILE_OPEN
```

Sau đó có thể thấy:

```text
SMB::FILE_DELETE
```

Điều này phù hợp với PsExec:

```text
Upload service binary
→ chạy
→ xóa
```

---

# 28. smb_mapping.log

Các share quan trọng:

```text
ADMIN$
IPC$
C$
```

Trong mẫu:

```text
\\dc1\ADMIN$ → DISK
\\dc1\IPC$   → PIPE
```

`ADMIN$` thường dùng để copy file vào Windows directory.

`IPC$` thường dùng cho named-pipe/RPC communication.

---

# 29. dce_rpc.log

Operations đáng chú ý:

```text
OpenSCManagerW
CreateServiceWOW64W
OpenServiceW
StartServiceW
QueryServiceStatus
ControlService
DeleteService
```

Endpoint/named pipe:

```text
svcctl
\pipe\ntsvcs
```

Chuỗi này rất phù hợp với remote service execution.

---

# 30. Correlate Bằng uid

Trong mẫu, cùng `uid` có thể xuất hiện trong:

```text
smb_files.log
dce_rpc.log
smb_mapping.log
```

Ví dụ:

```bash
grep 'CgPykN2qCki9kzhoh6' *.log
```

Điều này dựng lại toàn bộ sequence:

```text
Share mapping
→ file upload
→ service creation
→ service start
→ service deletion
→ file deletion
```

---

# 31. Zeek-Cut Cho PsExec

Files:

```bash
cat smb_files.log |
/usr/local/zeek/bin/zeek-cut   ts uid id.orig_h id.resp_h action path name size
```

Mappings:

```bash
cat smb_mapping.log |
/usr/local/zeek/bin/zeek-cut   ts uid id.orig_h id.resp_h path share_type
```

RPC:

```bash
cat dce_rpc.log |
/usr/local/zeek/bin/zeek-cut   ts uid id.orig_h id.resp_h named_pipe endpoint operation
```

---

# 32. Detection Logic Cho PsExec

High-confidence pattern:

```text
SMB upload to ADMIN$
AND filename PSEXESVC.exe
AND IPC$ mapping
AND svcctl CreateService
AND StartService
```

Ngay cả khi attacker đổi filename, vẫn có thể phát hiện:

```text
Executable uploaded to ADMIN$
+ service control RPC
+ temporary service lifecycle
```

---

# 33. False Positive Với PsExec

Nguồn hợp lệ:

- Sysinternals PsExec.
- SCCM.
- Remote administration.
- Software deployment.
- Backup/management tools.
- Incident response team.
- Red Team.

Tuning:

- Approved admin hosts.
- Approved service accounts.
- Change window.
- Destination role.
- File hash/signature.
- Process telemetry.
- Ticket/change context.

---

# 34. Splunk PsExec Correlation

SMB file:

```spl
index=zeek sourcetype=zeek:smb_files
(name="PSEXESVC.exe" OR path="*ADMIN$*")
| table _time uid id_orig_h id_resp_h action path name size
```

DCE/RPC:

```spl
index=zeek sourcetype=zeek:dce_rpc
(operation="CreateService*" OR operation="StartServiceW" OR operation="DeleteService")
| table _time uid id_orig_h id_resp_h named_pipe endpoint operation
```

Correlation:

```spl
index=zeek
(sourcetype=zeek:smb_files OR sourcetype=zeek:dce_rpc OR sourcetype=zeek:smb_mapping)
| stats values(action) as file_actions
        values(path) as shares
        values(name) as files
        values(operation) as rpc_operations
  by uid id_orig_h id_resp_h
| where mvfind(rpc_operations,"CreateService")>=0
```

---

# 35. Endpoint Correlation

Network detection mạnh hơn khi kết hợp Windows telemetry.

Các event hữu ích:

```text
7045 - A service was installed
4697 - A service was installed
5140 - Network share accessed
5145 - Detailed share access
4688 - Process creation
Sysmon 1 - Process Create
Sysmon 3 - Network Connection
Sysmon 11 - File Create
```

High-confidence sequence:

```text
Zeek ADMIN$ upload
+ DCE/RPC CreateService
+ Windows 7045
+ PSEXESVC.exe execution
```

---

# 36. Threat Hunting Checklist

## Beaconing

```text
[ ] Same source/destination.
[ ] Stable interval.
[ ] Stable bytes.
[ ] Rare destination.
[ ] Non-browser process.
```

## DNS Exfil

```text
[ ] Long labels.
[ ] High unique-subdomain count.
[ ] High entropy.
[ ] Rare parent domain.
[ ] Unusual query volume.
```

## TLS Exfil

```text
[ ] High outbound bytes.
[ ] Rare destination.
[ ] High outbound ratio.
[ ] Sensitive source asset.
[ ] Process correlation.
```

## PsExec

```text
[ ] ADMIN$ upload.
[ ] IPC$ mapping.
[ ] svcctl operations.
[ ] Temporary service.
[ ] File/service cleanup.
```

---

# 37. Incident Response Workflow

```text
1. Xác định source host.
2. Xác định destination/target.
3. Correlate Zeek uid.
4. Dựng timeline.
5. Xác định process/account.
6. Hunt cùng behavior toàn mạng.
7. Isolate host nếu true positive.
8. Preserve PCAP và logs.
9. Rotate credentials nếu lateral movement.
10. Cập nhật detection.
```

---

# 38. Mapping Với MITRE ATT&CK

| Use case | ATT&CK |
|---|---|
| Beaconing/C2 | T1071 – Application Layer Protocol |
| DNS exfiltration | T1048.003 – Exfiltration Over Alternative Protocol: DNS |
| TLS exfiltration | T1041 hoặc T1048 tùy kênh |
| PsExec remote service | T1569.002 – Service Execution |
| SMB transfer | T1021.002 – SMB/Windows Admin Shares |

Mapping phải dựa trên evidence thực tế, không chỉ dựa tên công cụ.

---

# 39. Mapping Với CDSA

## Security Operations & Monitoring

- Behavioral detection.
- Zeek log triage.
- SIEM dashboards.
- Network baseline.

## Incident Response & Forensics

- UID correlation.
- Flow reconstruction.
- Exfiltration scoping.
- Lateral movement timeline.

## Malware Analysis & Reverse Engineering

- Beacon profile.
- C2 interval.
- DNS encoding pattern.
- Network indicator extraction.

## Threat Hunting

- Beaconing hunt.
- DNS tunneling hunt.
- TLS volume analytics.
- Remote service execution hunt.

## Detection Engineering

- Statistical thresholds.
- Zeek scripts.
- Multi-log correlation.
- SIEM detections.
- False-positive tuning.

---

# 40. Lab Checklist

```bash
# Beaconing
mkdir -p ~/zeek-lab/psempire
cd ~/zeek-lab/psempire
zeek -C -r /home/htb-student/pcaps/psempire.pcap

# DNS exfiltration
mkdir -p ~/zeek-lab/dnsexfil
cd ~/zeek-lab/dnsexfil
zeek -C -r /home/htb-student/pcaps/dnsexfil.pcapng

# TLS exfiltration
mkdir -p ~/zeek-lab/tlsexfil
cd ~/zeek-lab/tlsexfil
zeek -C -r /home/htb-student/pcaps/tlsexfil.pcap

# PsExec
mkdir -p ~/zeek-lab/psexec
cd ~/zeek-lab/psexec
zeek -C -r /home/htb-student/pcaps/psexec_add_user.pcap
```

---

# Key Takeaway

```text
Zeek phát hiện intrusion tốt nhất khi tập trung vào behavior:

C2 beaconing
+ DNS subdomain anomalies
+ outbound byte volume
+ SMB/DCE-RPC activity sequence.

Một event đơn lẻ thường chưa đủ.
Giá trị lớn nhất đến từ correlation theo uid, host, destination và thời gian.
```

Intrusion Detection With Zeek là nội dung cốt lõi của CDSA vì kết nối Network Security Monitoring, Threat Hunting, Detection Engineering, Incident Response và phân tích lateral movement.

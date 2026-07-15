---
layout: post
title: "Snort Fundamentals"
date: 2026-07-15 23:47:53 +0700
categories: ["Working with IDS/IPS"]
tags: [snort, snort3, ids, ips, nsm, daq, libdaq, snort-lua, appid, packet-inspection, alerting, rule-management, detection-engineering, threat-hunting, incident-response, cdsa]
description: "Tổng quan Snort 3: kiến trúc IDS/IPS, DAQ, snort.lua, preprocessors/inspectors, PCAP và live input, rule loading, alert outputs, statistics và quy trình kiểm thử."
toc: true
---

# Snort Fundamentals

Bài này thuộc module **Working with IDS/IPS**, tập trung vào Snort 3 như một nền tảng IDS/IPS, packet logger và Network Security Monitoring engine.

```text
Mục tiêu:
- Hiểu các mode vận hành của Snort.
- Nắm kiến trúc packet decoder, inspectors, detection và outputs.
- Cấu hình Snort 3 bằng snort.lua.
- Phân tích PCAP và live interface.
- Nạp custom rules.
- Đọc alert và performance statistics.
- So sánh ngắn với Suricata.
```

> Chỉ chạy inline IPS hoặc replay PCAP trong HTB, lab hay hệ thống có ủy quyền. Không triển khai rule chặn trước khi kiểm thử false positive, hiệu năng và rollback.

---

## Timeline

```text
Created at: 2026-07-15 23:47:53 +0700
Timezone: Asia/Ho_Chi_Minh
Module: Working with IDS/IPS
Section: Snort Fundamentals
Focus: Snort 3 architecture, DAQ, configuration, rules, outputs
HTB example version: Snort++ 3.1.64.0
```

---

# 1. Snort Là Gì?

Snort là công cụ mã nguồn mở có thể hoạt động như:

- Network IDS.
- Inline IPS.
- Packet sniffer.
- Packet logger.
- Network Security Monitoring sensor.

Luồng xử lý tổng quát:

```text
Network Interface / PCAP
→ DAQ
→ Packet Decoder
→ Inspectors / Preprocessors
→ Detection Engine
→ Logging / Alerting / Output Plugins
```

---

# 2. Các Mode Vận Hành

## Passive IDS

Snort quan sát traffic và tạo alert nhưng không chặn.

```text
SPAN/TAP
→ Snort
→ alert/log
```

## Inline IPS

Traffic phải đi xuyên qua Snort thông qua DAQ hỗ trợ inline.

```text
Internet
→ Snort inline
→ LAN
```

Rule có action chặn mới có tác dụng trong inline mode.

## Network-Based IDS

Sensor giám sát traffic của một network segment hoặc nhiều VLAN.

## Host-Based Use

Snort có thể chạy trên một host, nhưng không phải công cụ HIDS tối ưu. Với endpoint telemetry, nên dùng công cụ như:

- Wazuh.
- Sysmon.
- osquery.
- EDR/XDR.
- auditd.

---

# 3. Passive và Inline Qua DAQ

DAQ là **Data Acquisition Library** của Snort.

Vai trò:

```text
Snort detection engine
↕
DAQ abstraction layer
↕
PCAP file / interface / inline adapter
```

Passive mode thường được kích hoạt khi dùng:

```bash
-r <pcap>
-i <interface>
```

Inline mode thường dùng:

```bash
-Q
```

với DAQ hỗ trợ inline, ví dụ `afpacket`.

---

# 4. Kiến Trúc Snort 3

## Packet Decoder

Nhận raw packets và xác định:

- Ethernet.
- IPv4/IPv6.
- TCP/UDP/ICMP.
- Header length.
- Flags.
- Packet validity.

## Inspectors / Preprocessors

Snort 3 dùng inspectors/modules để hiểu protocol và behavior:

- HTTP.
- DNS.
- TLS/SSL.
- SSH.
- SMB.
- FTP.
- SMTP.
- Port scan.
- ARP spoofing.
- AppID.
- File inspection.

## Detection Engine

So sánh traffic với rule set.

## Logging và Alerting

Tạo:

- Fast alerts.
- Full alerts.
- JSON.
- CSV.
- Syslog.
- Unified2.
- PCAP logs.
- Runtime/performance statistics.

---

# 5. snort.lua

File cấu hình chính:

```text
snort.lua
```

Các nhóm cấu hình chính:

```text
1. Defaults
2. Inspection
3. Bindings
4. Performance
5. Detection
6. Filters
7. Outputs
8. Tweaks
```

Ví dụ:

```lua
HOME_NET = 'any'
EXTERNAL_NET = 'any'

include 'snort_defaults.lua'

stream = { }
stream_ip = { }
stream_icmp = { }
stream_tcp = { }
stream_udp = { }
stream_file = { }
```

Một module khai báo bằng table rỗng thường dùng default settings.

---

# 6. HOME_NET và EXTERNAL_NET

Không nên giữ:

```lua
HOME_NET = 'any'
EXTERNAL_NET = 'any'
```

trong production nếu có thể xác định rõ subnet.

Ví dụ:

```lua
HOME_NET = '192.168.10.0/24'
EXTERNAL_NET = '!$HOME_NET'
```

Với nhiều subnet, syntax phụ thuộc cách cấu hình biến trong Snort 3. Cần kiểm tra chính xác bằng tài liệu và validate config.

Sai network variable có thể gây:

- Alert sai hướng.
- False positive.
- False negative.
- Rule không match.
- Tăng chi phí xử lý.

---

# 7. Xem Danh Sách Modules

```bash
snort --help-modules
```

Ví dụ module:

```text
alert_fast
alert_json
appid
arp_spoof
dns
http_inspect
http2_inspect
port_scan
ssl
ssh
stream_tcp
stream_udp
```

Xem cấu hình module:

```bash
snort --help-config arp_spoof
```

Hoặc:

```bash
snort --help-module http_inspect
```

---

# 8. Validation Cấu Hình

```bash
sudo snort   -c /root/snorty/etc/snort/snort.lua   --daq-dir /usr/local/lib/daq
```

Kết quả mong đợi:

```text
Snort successfully validated the configuration
0 warnings
```

Trong một số hệ thống, `--daq-dir` không cần thiết nếu DAQ đã nằm trong path chuẩn.

Validation cần thực hiện sau khi thay đổi:

- Network variables.
- Inspectors.
- Rules.
- Outputs.
- DAQ.
- IPS mode.
- Performance settings.

---

# 9. Phân Tích PCAP

PCAP liên quan:

```text
/home/htb-student/pcaps/icmp.pcap
```

Lệnh:

```bash
sudo snort   -c /root/snorty/etc/snort/snort.lua   --daq-dir /usr/local/lib/daq   -r /home/htb-student/pcaps/icmp.pcap
```

`-r` đưa Snort vào chế độ đọc PCAP/passive analysis.

Output cuối phiên có thể gồm:

- Packets received.
- Packets analyzed.
- Codec counters.
- Detection counters.
- Stream sessions.
- AppID statistics.
- Runtime.
- Packets per second.

---

# 10. Live Interface

```bash
sudo snort   -c /root/snorty/etc/snort/snort.lua   --daq-dir /usr/local/lib/daq   -i ens160
```

Trong live mode, Snort sẽ:

- Capture packet từ interface.
- Decode protocol.
- Track flows.
- Chạy inspectors.
- Đánh giá rules.
- Ghi alerts và statistics.

Dừng bằng:

```text
Ctrl+C
```

---

# 11. Inline IPS Concept

Một deployment inline có thể dùng:

```bash
sudo snort   -c /path/to/snort.lua   --daq afpacket   -Q   -i eth0:eth1
```

Cú pháp cụ thể phụ thuộc:

- DAQ build.
- Interface pairing.
- Bridge/router design.
- Snort version.
- Host networking.

Trước production cần test:

```text
Fail-open/fail-close
Throughput
Latency
Packet drops
Rule action
Rollback
```

---

# 12. Snort Rules

Rule Snort có cấu trúc tương tự Suricata:

```text
Header
+ Options
```

Ví dụ ICMP:

```snort
alert icmp any any -> any any (
  msg:"ICMP test";
  sid:1000001;
  rev:1;
)
```

Các phần:

- Action.
- Protocol.
- Source/destination.
- Ports.
- Direction.
- Message.
- Detection conditions.
- SID/revision.
- Classification/priority.

---

# 13. Nạp local.rules Trong snort.lua

Ví dụ:

```lua
ips =
{
    variables = default_variables,
    include = '/home/htb-student/local.rules'
}
```

Khi Snort load config, rule file sẽ được nạp tự động.

Kiểm tra output:

```text
Loading /home/htb-student/local.rules
Finished /home/htb-student/local.rules
```

---

# 14. Nạp Rule Qua Command Line

Một rule file:

```bash
sudo snort   -c /root/snorty/etc/snort/snort.lua   -R /home/htb-student/local.rules   -r /home/htb-student/pcaps/icmp.pcap
```

Một thư mục rules:

```bash
sudo snort   -c /root/snorty/etc/snort/snort.lua   --rule-path /path/to/rules   -r /path/to/test.pcap
```

Điều này hữu ích khi:

- Test rule độc lập.
- Regression-test.
- Không muốn chỉnh file cấu hình chính.
- So sánh nhiều ruleset.

---

# 15. Alert Output

Snort 3 hỗ trợ nhiều logger:

```bash
snort --list-plugins | grep logger
```

Các output thường gặp:

```text
alert_fast
alert_full
alert_json
alert_csv
alert_syslog
unified2
log_pcap
```

---

# 16. `-A cmg`

```bash
sudo snort   -c /root/snorty/etc/snort/snort.lua   --daq-dir /usr/local/lib/daq   -r /home/htb-student/pcaps/icmp.pcap   -A cmg
```

`cmg` hiển thị:

- Alert.
- Ethernet header.
- IP/ICMP/TCP/UDP header.
- Packet payload.
- Classification.
- Priority.
- Source/destination.

Rất phù hợp để:

- Debug rule.
- Phân tích PCAP nhỏ.
- Xác định packet kích hoạt.
- Kiểm tra content match.

Không phù hợp làm output duy nhất cho SIEM production.

---

# 17. Các Alert Mode Khác

## Fast

```bash
-A fast
```

Alert ngắn, dễ triage.

## Full

```bash
-A full
```

Thông tin chi tiết hơn.

## JSON

Phù hợp ingest SIEM:

```text
alert_json
```

## CSV

```bash
-A csv
```

Dễ phân tích bằng spreadsheet hoặc script, nhưng cần chọn field phù hợp.

## Unified2

Binary format dùng cho tooling kế thừa.

---

# 18. Statistics

Snort hiển thị nhiều nhóm thống kê khi dừng.

## Packet Statistics

Ví dụ:

```text
received
analyzed
allow
rx_bytes
```

## Codec Statistics

```text
eth
ipv4
ipv6
tcp
udp
icmp4
icmp6
discards
bad checksums
```

## Module Statistics

```text
appid
binder
detection
port_scan
stream
stream_tcp
stream_udp
arp_spoof
```

## Summary Statistics

```text
runtime
seconds
packets/sec
signals
profiling
```

---

# 19. Sensor Health

Các dấu hiệu cần theo dõi:

- Packets received nhưng không analyzed.
- Discards tăng.
- Bad checksums tăng.
- Flow/session table bất thường.
- Rule evaluation quá cao.
- Throughput thấp hơn traffic thực.
- CPU/memory saturation.
- DAQ drops.
- Inspector errors.

Sensor không khỏe có thể tạo blind spot dù rules vẫn đúng.

---

# 20. AppID

AppID giúp xác định ứng dụng/service thay vì chỉ dựa port.

Ví dụ:

```text
TCP/443 không nhất thiết chỉ là HTTPS.
TCP/80 không nhất thiết luôn là HTTP.
```

AppID hỗ trợ:

- Protocol identification.
- Application visibility.
- Policy enforcement.
- Non-standard port detection.
- Threat hunting.

---

# 21. Inspectors và Stateful Analysis

Các inspectors theo dõi trạng thái giao thức.

Ví dụ:

```text
stream_tcp
→ reassembly
→ http_inspect
→ request/response transaction
→ detection rule
```

Lợi ích:

- Không chỉ match từng packet riêng lẻ.
- Chống evasion qua fragmentation/segmentation.
- Hiểu protocol state.
- Cung cấp context chính xác hơn.

---

# 22. Port Scan Inspector

`port_scan` theo dõi pattern scan dựa trên:

- Source.
- Destination.
- Port count.
- Protocol.
- Time window.
- Scan type.

False positive có thể đến từ:

- Vulnerability scanners.
- Monitoring.
- Load balancers.
- Service discovery.
- Asset inventory tools.

Nên allowlist scanner được phê duyệt.

---

# 23. ARP Spoof Inspector

Module:

```text
arp_spoof
```

Có thể cấu hình mapping IP/MAC hợp lệ.

Ví dụ query cấu hình:

```bash
snort --help-config arp_spoof
```

Use case:

- ARP spoofing.
- Gateway MAC change.
- Duplicate IP/MAC anomaly.
- Local MITM detection.

Cần baseline network topology trước khi bật blocking.

---

# 24. Snort và Suricata

| Tiêu chí | Snort 3 | Suricata |
|---|---|---|
| Configuration | Lua | YAML |
| Capture abstraction | DAQ/LibDAQ | AF_PACKET, NFQ, PCAP... |
| Application detection | Inspectors/AppID | App-layer parsers |
| Main structured output | JSON plugin tùy config | EVE JSON |
| Rule syntax | Snort rule language | Gần Snort nhưng có khác biệt |
| File extraction | Có | Có |
| IDS/IPS | Có | Có |
| NSM metadata | Có | Rất mạnh qua EVE |

Không nên sao chép rule giữa hai engine mà không kiểm tra compatibility.

---

# 25. Rule Compatibility

Một rule Snort có thể không chạy nguyên vẹn trên Suricata và ngược lại do khác:

- Keyword.
- Sticky buffer.
- HTTP inspector.
- Flow semantics.
- PCRE modifier.
- File handling.
- Threshold syntax.
- Metadata.
- Protocol parser.

Quy trình:

```text
Đọc rule
→ xác định engine mục tiêu
→ validate syntax
→ test PCAP
→ test benign traffic
→ đánh giá performance
```

---

# 26. PCAP Lab Workflow

```bash
# 1. Validate config
sudo snort   -c /root/snorty/etc/snort/snort.lua   --daq-dir /usr/local/lib/daq

# 2. Chạy PCAP với local rule
sudo snort   -c /root/snorty/etc/snort/snort.lua   --daq-dir /usr/local/lib/daq   -r /home/htb-student/pcaps/icmp.pcap   -R /home/htb-student/local.rules   -A cmg

# 3. Ghi nhận
# - rule count
# - alert count
# - packet count
# - source/destination
# - payload
# - performance
```

---

# 27. Detection Engineering Workflow

```text
1. Xác định behavior hoặc IOC.
2. Viết local rule.
3. Validate snort.lua.
4. Test malicious PCAP.
5. Test benign PCAP.
6. Kiểm tra packet gây alert.
7. Tune rule.
8. Đánh giá performance.
9. Deploy passive IDS.
10. Chỉ chuyển inline khi đủ confidence.
```

---

# 28. SIEM Integration

Pipeline:

```text
Snort alert_json/syslog/unified2
→ Collector
→ Logstash/Filebeat/Fluent Bit
→ Splunk/ELK/Wazuh
→ Detection correlation
```

Các field nên chuẩn hóa:

```text
timestamp
gid
sid
rev
signature
classification
priority
src_ip
src_port
dest_ip
dest_port
protocol
action
interface
```

---

# 29. Splunk Hunting

Concept query:

```spl
index=snort
| stats count
        values(signature) as signatures
        values(dest_ip) as destinations
  by src_ip, priority
| sort -count
```

Top signatures:

```spl
index=snort
| stats count by sid, signature
| sort -count
```

ICMP alerts:

```spl
index=snort protocol=ICMP
| table _time src_ip dest_ip signature priority
```

Field name phụ thuộc parser/ingest pipeline.

---

# 30. Incident Response Checklist

```text
[ ] Xác định SID/signature.
[ ] Kiểm tra source/destination.
[ ] Xem packet/payload kích hoạt.
[ ] Xác định alert có true positive không.
[ ] Correlate firewall, DNS, proxy và EDR.
[ ] Kiểm tra endpoint process.
[ ] Hunt cùng IOC trên toàn mạng.
[ ] Thu PCAP và alert logs.
[ ] Tune rule nếu false positive.
[ ] Chuyển rule thành correlation detection nếu cần.
```

---

# 31. Threat Hunting Checklist

```text
[ ] Protocol chạy ngoài cổng chuẩn.
[ ] Port scan từ host không được phép.
[ ] ICMP payload bất thường.
[ ] ARP spoof anomalies.
[ ] Rare source/destination.
[ ] AppID khác port expectation.
[ ] Nhiều alerts cùng source.
[ ] Alert sau DNS query đáng ngờ.
[ ] Alert trùng với EDR process event.
```

---

# 32. Mapping Với CDSA

## Security Operations & Monitoring

- Snort alert triage.
- Sensor-health monitoring.
- AppID và inspector visibility.
- SIEM ingestion.

## Incident Response & Forensics

- PCAP replay.
- Packet-level evidence.
- Timeline và source/destination correlation.
- Alert-to-endpoint scoping.

## Malware Analysis & Reverse Engineering

- Tìm network IOC.
- Phân tích payload.
- Viết Snort signature.
- So sánh malware traffic profiles.

## Threat Hunting

- Historical rule hits.
- Port scan.
- Protocol anomaly.
- Rare applications.
- ICMP/ARP behavior.

## Detection Engineering

- Rule development.
- Regression testing.
- Performance profiling.
- Passive-to-inline promotion.
- Cross-engine compatibility assessment.

---

# 33. Lab Setup Khuyến Nghị

```text
1 Ubuntu/Kali VM chạy Snort 3
1 PCAP repository
1 isolated network interface
Wireshark/tshark
Splunk/ELK/Wazuh
Optional: tcpreplay trong lab
```

Kiểm tra phiên bản:

```bash
snort -V
snort --help-modules
snort --list-plugins
```

---

# 34. Những Lỗi Thường Gặp

## Không thấy alert

Kiểm tra:

- Rule đã được include chưa?
- `-R` có đúng path không?
- SID có hợp lệ không?
- Protocol/direction có đúng không?
- Alert output đã bật chưa?
- PCAP có traffic cần match không?

## Config không load

Kiểm tra:

- Lua syntax.
- Include path.
- DAQ path.
- Module name.
- Rule syntax.

## Quá nhiều false positive

- HOME_NET quá rộng.
- Rule quá chung.
- Không có flow/state.
- Threshold sai.
- Scanner hợp lệ chưa allowlist.
- AppID/inspector chưa phù hợp.

---

# Key Takeaway

```text
Snort 3 gồm:
DAQ
+ packet decoder
+ inspectors/AppID
+ detection engine
+ alert/output modules.

Một deployment tốt cần:
snort.lua chính xác
+ HOME_NET đúng
+ DAQ phù hợp
+ rule được regression-test
+ alert output phù hợp SIEM
+ sensor-health monitoring.
```

Snort Fundamentals là nền tảng quan trọng trong CDSA vì kết nối Network Security Monitoring, packet analysis, IDS/IPS operations, Detection Engineering, Threat Hunting và Incident Response.

---
layout: post
title: "Introduction to Splunk and SPL"
date: 2026-06-23 00:02:00 +0700
categories: [SOC, Splunk]
tags: [soc, splunk, spl, siem, log-sources, search-processing-language, sysmon, windows-logs, data-discovery, blue-team]
description: "Tóm tắt Splunk fundamentals: kiến trúc Splunk, Forwarder/Indexer/Search Head, Splunk as SIEM, SPL cơ bản, các command quan trọng và cách xác định dữ liệu/field có sẵn trong Splunk."
toc: true
---

# Introduction to Splunk and SPL

**Splunk** là nền tảng phân tích dữ liệu máy dùng để ingest, index, search, analyze và visualize log từ nhiều nguồn khác nhau.

Trong SOC, Splunk thường được dùng như SIEM để hỗ trợ:

- Log management.
- Security monitoring.
- Incident response.
- Threat hunting.
- Compliance.
- Dashboard, alert và report.

```text
Splunk = ingest log → index dữ liệu → search bằng SPL → phân tích/visualize/alert.
```

---

## Timeline cập nhật

```text
Created at: 2026-06-23 00:02:00 +0700
Timezone: Asia/Ho_Chi_Minh
Module: Understanding Log Sources & Investigating with Splunk
Section: Introduction To Splunk & SPL
```

---

# 1. Kiến trúc Splunk

Kiến trúc Splunk Enterprise có thể tóm tắt như sau:

```text
Forwarder → Indexer → Search Head
```

| Thành phần | Vai trò |
|---|---|
| Forwarder | Thu thập dữ liệu và gửi về indexer |
| Indexer | Nhận, xử lý, nén, lưu và lập chỉ mục dữ liệu |
| Search Head | Giao diện search, dashboard, report, alert |
| Deployment Server | Quản lý cấu hình/app cho forwarders |
| Cluster Master / Manager | Điều phối indexer cluster |
| License Master | Quản lý license Splunk |
| HEC | Nhận dữ liệu qua HTTP token/API |

---

## 1.1. Forwarders

**Universal Forwarder (UF)** là agent nhẹ, chủ yếu collect log và forward về indexer.

**Heavy Forwarder (HF)** có thể parse, filter, route, index local và forward tiếp. HF phù hợp cho data collection phức tạp như firewall, API hoặc scripted input.

---

## 1.2. Indexers

Indexer nhận dữ liệu từ forwarder, tổ chức dữ liệu thành indexes và xử lý search request từ Search Head.

---

## 1.3. Search Heads

Search Head là nơi analyst dùng Splunk Web để chạy SPL, tạo dashboard, alert, report và knowledge objects.

---

# 2. Splunk như một SIEM

Splunk hỗ trợ SOC bằng cách:

- Search log real-time và historical.
- Correlate nhiều nguồn log.
- Tạo dashboard/visualization.
- Tạo alert.
- Hỗ trợ incident response.
- Hỗ trợ threat hunting.
- Enrich dữ liệu bằng lookup/knowledge objects.

---

# 3. SPL Basic Searching

Giả sử index chính là:

```text
index="main"
```

Tìm keyword:

```spl
index="main" "UNKNOWN"
```

Tìm wildcard:

```spl
index="main" "*UNKNOWN*"
```

Boolean operators:

```text
AND
OR
NOT
```

Ví dụ:

```spl
index="main" error OR failed
```

```spl
index="main" NOT EventCode=1
```

---

# 4. Fields và Comparison Operators

Splunk tự động nhận diện nhiều field như:

- source.
- sourcetype.
- host.
- EventCode.
- Image.
- User.
- CommandLine.

Toán tử thường dùng:

```text
=  !=  <  >  <=  >=
```

Ví dụ:

```spl
index="main" EventCode!=1
```

---

# 5. Các SPL command quan trọng

## 5.1. fields

Chọn hoặc loại field:

```spl
index="main" sourcetype="WinEventLog:Sysmon" EventCode=1
| fields - User
```

## 5.2. table

Hiển thị dạng bảng:

```spl
index="main" sourcetype="WinEventLog:Sysmon" EventCode=1
| table _time, host, Image
```

## 5.3. rename

Đổi tên field:

```spl
index="main" sourcetype="WinEventLog:Sysmon" EventCode=1
| rename Image as Process
```

## 5.4. dedup

Loại kết quả trùng:

```spl
index="main" sourcetype="WinEventLog:Sysmon" EventCode=1
| dedup Image
```

## 5.5. sort

Sắp xếp theo thời gian mới nhất:

```spl
index="main" sourcetype="WinEventLog:Sysmon" EventCode=1
| sort - _time
```

## 5.6. stats

Thống kê:

```spl
index="main" sourcetype="WinEventLog:Sysmon" EventCode=3
| stats count by _time, Image
```

## 5.7. chart

Tạo bảng phù hợp cho visualization:

```spl
index="main" sourcetype="WinEventLog:Sysmon" EventCode=3
| chart count by _time, Image
```

## 5.8. eval

Tạo hoặc biến đổi field:

```spl
index="main" sourcetype="WinEventLog:Sysmon" EventCode=1
| eval Process_Path=lower(Image)
```

## 5.9. rex

Extract field bằng regex:

```spl
index="main" EventCode=4662
| rex max_match=0 "[^%](?<guid>{.*})"
| table guid
```

## 5.10. lookup

Làm giàu dữ liệu bằng CSV lookup:

```spl
index="main" sourcetype="WinEventLog:Sysmon" EventCode=1
| rex field=Image "(?P<filename>[^\\]+)$"
| eval filename=lower(filename)
| lookup malware_lookup.csv filename OUTPUTNEW is_malware
| table filename, is_malware
```

## 5.11. inputlookup

Xem nội dung lookup:

```spl
| inputlookup malware_lookup.csv
```

## 5.12. earliest/latest

Giới hạn thời gian:

```spl
index="main" earliest=-7d EventCode!=1
```

## 5.13. transaction

Gộp event liên quan:

```spl
index="main" sourcetype="WinEventLog:Sysmon" (EventCode=1 OR EventCode=3)
| transaction Image startswith=eval(EventCode=1) endswith=eval(EventCode=3) maxspan=1m
| table Image
| dedup Image
```

## 5.14. subsearch

Loại top 100 process phổ biến để tìm process hiếm:

```spl
index="main" sourcetype="WinEventLog:Sysmon" EventCode=1
NOT [
  search index="main" sourcetype="WinEventLog:Sysmon" EventCode=1
  | top limit=100 Image
  | fields Image
]
| table _time, Image, CommandLine, User, ComputerName
```

---

# 6. Query Ideas cho SOC/Threat Hunting

## 6.1. Process creation

```spl
index="main" sourcetype="WinEventLog:Sysmon" EventCode=1
| table _time, host, User, Image, CommandLine, ParentImage
```

## 6.2. Network connection

```spl
index="main" sourcetype="WinEventLog:Sysmon" EventCode=3
| table _time, host, Image, DestinationIp, DestinationPort
```

## 6.3. Suspicious PowerShell

```spl
index="main" sourcetype="WinEventLog:Sysmon" EventCode=1 Image="*powershell.exe*"
| search CommandLine="*-enc*" OR CommandLine="*DownloadString*" OR CommandLine="*IEX*"
| table _time, host, User, Image, CommandLine, ParentImage
```

## 6.4. Rare parent process

```spl
index="main" sourcetype="WinEventLog:Sysmon" EventCode=1
| rare limit=20 ParentImage
```

## 6.5. Process creation followed by network connection

```spl
index="main" sourcetype="WinEventLog:Sysmon" (EventCode=1 OR EventCode=3)
| transaction Image startswith=eval(EventCode=1) endswith=eval(EventCode=3) maxspan=1m
| table _time, host, Image, CommandLine, DestinationIp, DestinationPort
```

## 6.6. Known suspicious filename lookup

```spl
index="main" sourcetype="WinEventLog:Sysmon" EventCode=1
| rex field=Image "(?P<filename>[^\\]+)$"
| eval filename=lower(filename)
| lookup malware_lookup.csv filename OUTPUTNEW is_malware
| search is_malware=true
| table _time, host, User, filename, Image, CommandLine
```

---

# 7. Cách xác định dữ liệu có sẵn trong Splunk bằng SPL

## 7.1. Liệt kê indexes

```spl
| eventcount summarize=false index=*
| table index
```

## 7.2. Liệt kê sourcetypes

```spl
| metadata type=sourcetypes index=*
| table sourcetype
```

## 7.3. Liệt kê sources

```spl
| metadata type=sources index=*
| table source
```

## 7.4. Xem raw data

```spl
sourcetype="WinEventLog:Security"
| table _raw
```

## 7.5. Xem toàn bộ field

```spl
sourcetype="WinEventLog:Security"
| table *
```

## 7.6. Xem field summary

```spl
sourcetype="WinEventLog:Security"
| fieldsummary
```

## 7.7. Event distribution theo ngày/index/sourcetype

```spl
index=* sourcetype=*
| bucket _time span=1d
| stats count by _time, index, sourcetype
| sort - _time
```

## 7.8. Tìm rare index/sourcetype

```spl
index=* sourcetype=*
| rare limit=10 index, sourcetype
```

## 7.9. Tìm rare field values

```spl
index="main"
| rare limit=20 useother=f ParentImage
```

## 7.10. Field xuất hiện ít

```spl
index=* sourcetype=*
| fieldsummary
| where count < 100
| table field, count, distinct_count
```

## 7.11. Event diversity

```spl
index=*
| sistats count by index, sourcetype, source, host
```

---

# 8. Cách xác định dữ liệu bằng Splunk UI

## 8.1. Data Inputs

```text
Settings → Data inputs
```

Dùng để xem nguồn dữ liệu được ingest:

- Files & directories.
- HTTP Event Collector.
- Forwarders.
- Scripts.
- TCP/UDP inputs.

## 8.2. Search & Reporting

Trong Search & Reporting:

- Fast Mode để xem nhanh.
- Verbose Mode để xem chi tiết field.
- Click event để mở raw event và extracted fields.
- Xem Selected Fields và Interesting Fields.

## 8.3. Data Models

```text
Settings → Data Models
```

Data Models giúp tổ chức dữ liệu thành object/field dễ hiểu.

## 8.4. Pivots

Pivot cho phép tạo report/visualization bằng giao diện kéo thả, không cần viết SPL quá phức tạp.

---

# 9. Bảng tóm tắt SPL command

| Command | Mục đích |
|---|---|
| search | Tìm kiếm event |
| fields | Chọn hoặc loại field |
| table | Hiển thị dạng bảng |
| rename | Đổi tên field |
| dedup | Loại kết quả trùng |
| sort | Sắp xếp kết quả |
| stats | Thống kê |
| chart | Tạo bảng visualization |
| eval | Tạo/biến đổi field |
| rex | Extract field bằng regex |
| lookup | Làm giàu dữ liệu từ lookup |
| inputlookup | Đọc lookup file |
| transaction | Gộp event liên quan |
| metadata | Xem metadata source/sourcetype |
| eventcount | Đếm event theo index |
| fieldsummary | Tóm tắt field |
| rare | Tìm giá trị hiếm |
| bucket | Gom thời gian thành bucket |
| sistats | Thống kê dạng summary indexing |

---

# 10. Câu hỏi ôn tập

## 1. Splunk là gì?

Splunk là nền tảng ingest, index, search, analyze và visualize machine data từ nhiều nguồn khác nhau.

## 2. Ba thành phần chính của kiến trúc Splunk là gì?

```text
Forwarder
Indexer
Search Head
```

## 3. Universal Forwarder khác Heavy Forwarder như thế nào?

Universal Forwarder nhẹ và chủ yếu forward log. Heavy Forwarder có thể parse, route, filter, index local và phù hợp cho collection phức tạp hơn.

## 4. SPL là gì?

SPL là Search Processing Language, ngôn ngữ dùng để search, filter, transform và visualize dữ liệu trong Splunk.

## 5. Command nào dùng để extract field bằng regex?

```text
rex
```

## 6. Command nào dùng để làm giàu dữ liệu bằng CSV?

```text
lookup
```

## 7. Command nào dùng để xem danh sách sourcetypes?

```spl
| metadata type=sourcetypes index=*
| table sourcetype
```

---

# Key Takeaway

Splunk là SIEM/log analytics platform rất mạnh khi analyst hiểu được kiến trúc và SPL.

Điểm quan trọng nhất là biết cách:

```text
Xác định data source
→ Xác định sourcetype
→ Xác định field
→ Viết SPL query
→ Transform/aggregate dữ liệu
→ Tạo dashboard/alert/hunt
```

Trong SOC hoặc threat hunting, SPL không chỉ dùng để tìm keyword, mà còn dùng để correlation, enrichment, rare event analysis, timeline building và phát hiện hành vi bất thường.

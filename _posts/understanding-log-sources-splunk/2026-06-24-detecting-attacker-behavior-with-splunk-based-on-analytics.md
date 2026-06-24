---
layout: post
title: "Detecting Attacker Behavior With Splunk Based On Analytics"
date: 2026-06-24 14:52:11 +0700
categories: [SOC, Splunk]
tags: [soc, splunk, spl, analytics, anomaly-detection, streamstats, eventstats, sysmon, cmd, dll-loading, transaction, blue-team]
description: "Tóm tắt cách phát hiện attacker behavior bằng Splunk dựa trên analytics: streamstats, anomaly detection, long commands, abnormal cmd.exe activity, high DLL loading và repeated process transactions."
toc: true
---

# Detecting Attacker Behavior With Splunk Based On Analytics

Bài này tóm tắt cách phát hiện attacker behavior bằng Splunk dựa trên **statistical analysis** và **anomaly detection**.

Khác với hướng tìm kiếm theo TTP đã biết, hướng analytics tập trung vào việc xây dựng baseline hành vi bình thường, sau đó tìm các điểm lệch khỏi baseline.

```text
Analytics-based detection = tìm hành vi bất thường dựa trên thống kê và baseline.
```

---

## Timeline cập nhật

```text
Created at: 2026-06-24 14:52:11 +0700
Timezone: Asia/Ho_Chi_Minh
Module: Understanding Log Sources & Investigating with Splunk
Section: Detecting Attacker Behavior With Splunk Based On Analytics
```

---

# 1. Ý tưởng chính

Hướng analytics-based detection dựa trên giả định:

```text
Malicious activity thường tạo ra pattern lệch khỏi hành vi bình thường.
```

Ví dụ:

- Process tạo số lượng network connection bất thường.
- Command line quá dài.
- `cmd.exe` hoạt động nhiều bất thường.
- Process load nhiều DLL trong thời gian ngắn.
- Cùng một process được tạo nhiều lần trên cùng máy.
- Parent-child process xuất hiện với tần suất bất thường.

Cách này vẫn cần hiểu attacker TTPs, nhưng không chỉ phụ thuộc vào IOC hay rule cố định.

---

# 2. Streamstats và baseline động

`streamstats` cho phép tính toán thống kê rolling theo dòng dữ liệu.

Ví dụ: theo dõi số network connections theo từng process trong mỗi giờ, rồi so sánh với rolling average và standard deviation trong 24 giờ.

## SPL

```spl
index="main" sourcetype="WinEventLog:Sysmon" EventCode=3
| bin _time span=1h
| stats count as NetworkConnections by _time, Image
| streamstats time_window=24h avg(NetworkConnections) as avg stdev(NetworkConnections) as stdev by Image
| eval isOutlier=if(NetworkConnections > (avg + (0.5*stdev)), 1, 0)
| search isOutlier=1
```

## Ý nghĩa

Query này:

1. Lấy Sysmon EventCode 3 - Network Connection.
2. Gom event theo từng giờ bằng `bin`.
3. Đếm số connection theo `Image`.
4. Dùng `streamstats` để tính rolling average và standard deviation 24h.
5. Gắn nhãn outlier nếu connection count vượt ngưỡng.
6. Chỉ giữ event bất thường.

## Dùng để phát hiện

- C2 communication.
- Data exfiltration.
- Process beacon bất thường.
- Tool tạo nhiều connection.
- Malware network activity.

---

# 3. Detection: Abnormally Long Commands

Attacker thường dùng command line rất dài để:

- Obfuscate payload.
- Chạy PowerShell encoded command.
- Chèn nhiều tham số.
- Download payload.
- Chạy one-liner.
- Né detection cơ bản.

## SPL ban đầu

```spl
index="main" sourcetype="WinEventLog:Sysmon" Image=*cmd.exe
| eval len=len(CommandLine)
| table User, len, CommandLine
| sort - len
```

## Ý nghĩa

- `len(CommandLine)` tính độ dài command.
- Sort giảm dần để command dài nhất lên đầu.
- Command quá dài cần được review.

---

## SPL đã giảm noise

```spl
index="main" sourcetype="WinEventLog:Sysmon" Image=*cmd.exe ParentImage!="*msiexec.exe" ParentImage!="*explorer.exe"
| eval len=len(CommandLine)
| table User, len, CommandLine
| sort - len
```

## Vì sao filter `msiexec.exe` và `explorer.exe`?

Một số installer hoặc user action hợp lệ có thể tạo command line dài.

Filter này giúp giảm noise, nhưng cần tune theo môi trường thật.

---

# 4. Detection: Abnormal cmd.exe Activity

Query này tìm hoạt động `cmd.exe` bất thường trong từng giờ.

## SPL

```spl
index="main" EventCode=1 (CommandLine="*cmd.exe*")
| bucket _time span=1h
| stats count as cmdCount by _time User CommandLine
| eventstats avg(cmdCount) as avg stdev(cmdCount) as stdev
| eval isOutlier=if(cmdCount > avg+1.5*stdev, 1, 0)
| search isOutlier=1
```

## Ý nghĩa

- `bucket _time span=1h`: gom theo từng giờ.
- `stats count`: đếm số lần cmd command xuất hiện.
- `eventstats`: tính average và standard deviation toàn bộ kết quả.
- `eval isOutlier`: đánh dấu outlier.
- `search isOutlier=1`: chỉ giữ hoạt động bất thường.

## Dùng để phát hiện

- Script automation đáng ngờ.
- Malware dùng `cmd.exe`.
- Reconnaissance batch commands.
- Lateral movement command execution.
- Persistence command execution.

---

# 5. Detection: Processes Loading High Number Of DLLs

Malware có thể load nhiều DLL trong thời gian ngắn.

Sysmon EventCode 7 ghi nhận image/DLL loaded.

## SPL ban đầu

```spl
index="main" EventCode=7
| bucket _time span=1h
| stats dc(ImageLoaded) as unique_dlls_loaded by _time, Image
| where unique_dlls_loaded > 3
| stats count by Image, unique_dlls_loaded
```

## Ý nghĩa

- `EventCode=7`: Image Loaded.
- `dc(ImageLoaded)`: đếm số DLL unique được load.
- `where unique_dlls_loaded > 3`: lọc process load hơn 3 DLL unique trong 1 giờ.

---

## SPL đã giảm noise

```spl
index="main" EventCode=7 NOT (Image="C:\Windows\System32*") NOT (Image="C:\Program Files (x86)*") NOT (Image="C:\Program Files*") NOT (Image="C:\ProgramData*") NOT (Image="C:\Users\waldo\AppData*")
| bucket _time span=1h
| stats dc(ImageLoaded) as unique_dlls_loaded by _time, Image
| where unique_dlls_loaded > 3
| stats count by Image, unique_dlls_loaded
| sort - unique_dlls_loaded
```

## Vì sao cần filter path?

Các path như `System32`, `Program Files`, `ProgramData`, `AppData` có thể tạo nhiều DLL load hợp lệ.

Filter giúp tập trung vào process ngoài các vị trí thường gặp.

## Lưu ý

Hành vi load nhiều DLL cũng có thể là benign. Cần kiểm tra thêm:

- Process path.
- Signature.
- Parent process.
- User.
- CommandLine.
- Hash.
- Network connection sau đó.

---

# 6. Detection: Same Process Created More Than Once On Same Computer

Query này dùng `transaction` để gom các event có cùng `ComputerName` và `Image`.

## SPL

```spl
index="main" sourcetype="WinEventLog:Sysmon" EventCode=1
| transaction ComputerName, Image
| where mvcount(ProcessGuid) > 1
| stats count by Image, ParentImage
```

## Ý nghĩa

- Lấy Sysmon EventCode 1 - Process Creation.
- Gom event cùng `ComputerName` và `Image`.
- Chỉ giữ transaction có hơn 1 `ProcessGuid`.
- Đếm theo `Image` và `ParentImage`.

## Dùng để phát hiện

- Process bị chạy lặp bất thường.
- Script hoặc malware tạo process nhiều lần.
- Tool được execute nhiều lần trên cùng host.
- Parent-child relationship đáng ngờ.

---

# 7. Drill down rundll32.exe và svchost.exe

Trong kết quả, cặp `rundll32.exe` và `svchost.exe` có count cao nên cần drill down.

## SPL

```spl
index="main" sourcetype="WinEventLog:Sysmon" EventCode=1
| transaction ComputerName, Image
| where mvcount(ProcessGuid) > 1
| search Image="C:\Windows\System32\rundll32.exe" ParentImage="C:\Windows\System32\svchost.exe"
| table CommandLine, ParentCommandLine
```

## Ý nghĩa

Query này giúp xem command line cụ thể của `rundll32.exe` khi được spawn bởi `svchost.exe`.

Cần chú ý:

- DLL path lạ.
- Export function lạ.
- CommandLine trỏ đến user-writable path.
- ParentCommandLine bất thường.
- Có liên quan persistence hoặc malicious DLL execution không.

---

# 8. Bảng tóm tắt analytics queries

| Use case | SPL idea | Command chính |
|---|---|---|
| Network connection anomaly | Rolling avg/stdev theo process | `streamstats` |
| Long command line | Tính độ dài CommandLine | `eval len()` |
| Abnormal cmd.exe activity | Outlier theo giờ | `bucket`, `eventstats` |
| High DLL loading | Đếm DLL unique theo process | `dc(ImageLoaded)` |
| Repeated process creation | Gom process theo host/image | `transaction` |
| rundll32/svchost drilldown | Xem command line cụ thể | `table` |

---

# 9. Query checklist nhanh

```spl
index="main" sourcetype="WinEventLog:Sysmon" EventCode=3
| bin _time span=1h
| stats count as NetworkConnections by _time, Image
| streamstats time_window=24h avg(NetworkConnections) as avg stdev(NetworkConnections) as stdev by Image
| eval isOutlier=if(NetworkConnections > (avg + (0.5*stdev)), 1, 0)
| search isOutlier=1
```

```spl
index="main" sourcetype="WinEventLog:Sysmon" Image=*cmd.exe
| eval len=len(CommandLine)
| table User, len, CommandLine
| sort - len
```

```spl
index="main" EventCode=1 (CommandLine="*cmd.exe*")
| bucket _time span=1h
| stats count as cmdCount by _time User CommandLine
| eventstats avg(cmdCount) as avg stdev(cmdCount) as stdev
| eval isOutlier=if(cmdCount > avg+1.5*stdev, 1, 0)
| search isOutlier=1
```

```spl
index="main" EventCode=7
| bucket _time span=1h
| stats dc(ImageLoaded) as unique_dlls_loaded by _time, Image
| where unique_dlls_loaded > 3
| stats count by Image, unique_dlls_loaded
```

```spl
index="main" sourcetype="WinEventLog:Sysmon" EventCode=1
| transaction ComputerName, Image
| where mvcount(ProcessGuid) > 1
| stats count by Image, ParentImage
```

```spl
index="main" sourcetype="WinEventLog:Sysmon" EventCode=1
| transaction ComputerName, Image
| where mvcount(ProcessGuid) > 1
| search Image="C:\Windows\System32\rundll32.exe" ParentImage="C:\Windows\System32\svchost.exe"
| table CommandLine, ParentCommandLine
```

---

# 10. Khi nào analytics-based detection hiệu quả?

Analytics-based detection hiệu quả khi:

- Có đủ dữ liệu lịch sử để xây baseline.
- Môi trường có logging ổn định.
- Field được parse đúng.
- Threshold được tune theo môi trường.
- Analyst hiểu normal activity.
- Kết quả được review định kỳ.
- Có quy trình suppress/allowlist hợp lý.

---

# 11. Hạn chế

Analytics-based detection không phải giải pháp hoàn hảo.

Hạn chế:

- Có thể nhiều false positives.
- Baseline sai thì detection sai.
- Môi trường thay đổi có thể làm query nhiễu.
- Attacker có thể mimic normal behavior.
- Threshold cứng có thể bỏ sót low-and-slow attack.
- Cần tuning liên tục.

---

# Key Takeaway

Detection dựa trên analytics giúp phát hiện hành vi bất thường mà rule theo IOC/TTP cố định có thể bỏ sót.

Tư duy quan trọng:

```text
Baseline normal behavior
→ tìm deviation
→ review context
→ giảm false positive
→ chuyển finding tốt thành alert hoặc hunting query
```

Trong Splunk, các command như `streamstats`, `eventstats`, `stats`, `bucket`, `transaction`, `eval` và `dc()` là nền tảng để xây dựng analytics-based hunting searches.

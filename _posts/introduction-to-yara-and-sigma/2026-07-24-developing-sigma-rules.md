---
layout: post
title: "Developing Sigma Rules"
date: 2026-07-24 23:33:36 +0700
categories: ["Introduction to YARA & Sigma", "Sigma"]
tags: [sigma, rule-development, sysmon, lsass, event-id-10, event-id-4776, detection-engineering, threat-hunting, false-positive, cdsa]
description: "Tóm tắt cách phát triển Sigma rule từ log thực tế, kiểm soát false positive và chuyển đổi rule sang truy vấn điều tra."
toc: true
---

# Developing Sigma Rules

## Công thức dễ nhớ

```text
LOG → FIELD → SELECTION → FILTER → TEST → TUNE
```

## 1. Quy Trình Phát Triển Rule

```text
Xác định hành vi cần phát hiện
→ chọn đúng log source
→ tìm field quan trọng
→ viết selection
→ thêm filter
→ chạy trên EVTX
→ tinh chỉnh false positive
```

## 2. Ví Dụ: Truy Cập LSASS

Nguồn log:

```text
Sysmon Event ID 10
Category: process_access
```

Field quan trọng:

```text
TargetImage
GrantedAccess
SourceImage
```

Rule cơ bản:

```yaml
logsource:
  category: process_access
  product: windows

detection:
  selection:
    TargetImage|endswith: '\lsass.exe'
    GrantedAccess|endswith: '0x1010'
  condition: selection
```

Ý nghĩa:

```text
Process lạ
→ yêu cầu quyền đọc/query LSASS
→ có khả năng credential dumping
```

## 3. Giảm False Positive

Rule tốt không chỉ nhìn `GrantedAccess`.

Nên kết hợp:

```text
TargetImage = lsass.exe
+ access flag đáng ngờ
+ SourceImage nằm trong đường dẫn bất thường
- ứng dụng hợp lệ đã biết
```

Đường dẫn đáng chú ý:

```text
\Temp\
\Users\Public\
\PerfLogs\
\AppData\
```

Condition mẫu:

```yaml
condition: selection and not 1 of filter_optional_*
```

## 4. Chuyển Sigma Thành Query

Sigma có thể được chuyển sang truy vấn điều tra.

Ví dụ PowerShell:

```text
Sigma rule
→ converter
→ Get-WinEvent query
→ chạy trên file EVTX
```

Mục đích:

- Xác thực rule.
- Điều tra nhanh khi chưa có SIEM.
- So sánh kết quả giữa các nền tảng.

## 5. Ví Dụ: Failed NTLM Logons

Nguồn log:

```text
Windows Security
Event ID 4776
```

Logic:

```text
Cùng một Workstation
→ thử nhiều TargetUserName
→ số lần vượt ngưỡng
→ cảnh báo password spraying/brute force
```

Condition:

```yaml
condition: selection | count(TargetUserName) by Workstation > 3
```

False positive thường gặp:

```text
Terminal Server
Jump Server
Citrix
Hệ thống nhiều người dùng
```

## Keyword Cần Nhớ

```text
Sysmon Event ID 10
LSASS
GrantedAccess
SourceImage
Event ID 4776
Workstation
TargetUserName
selection
filter
false positive
threshold
```

## Key Takeaway

```text
Sigma rule tốt =
đúng log source
+ đúng field
+ logic rõ
+ filter hợp lệ
+ test trên dữ liệu thật
```

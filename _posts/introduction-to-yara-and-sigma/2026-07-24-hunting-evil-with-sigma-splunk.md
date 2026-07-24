---
layout: post
title: "Hunting Evil with Sigma and Splunk"
date: 2026-07-24 23:39:03 +0700
categories: ["Introduction to YARA & Sigma", "Sigma"]
tags: [sigma, splunk, spl, sigmac, threat-hunting, lsass, rundll32, notepad, detection-engineering, cdsa]
description: "Tóm tắt cách chuyển Sigma rule sang SPL và hunting trên Splunk."
toc: true
---

# Hunting Evil with Sigma and Splunk

## Công thức dễ nhớ

```text
SIGMA RULE → CONVERT → SPL → SPLUNK → DETECTION
```

## 1. Mục Đích

Sigma giúp viết detection một lần rồi chuyển sang ngôn ngữ truy vấn của SIEM.

Trong bài này:

```text
Sigma
→ sigmac
→ SPL
→ chạy trong Splunk
```

## 2. Chuyển Sigma Sang SPL

```powershell
python sigmac `
  -t splunk `
  rule.yml `
  -c .\config\splunk-windows.yml
```

Ý nghĩa:

```text
-t splunk = chọn backend Splunk
-c         = dùng file cấu hình field/mapping
```

## 3. Ví Dụ 1: Dump LSASS Qua comsvcs.dll

Hành vi:

```text
rundll32.exe
→ gọi comsvcs.dll
→ MiniDump
→ truy cập lsass.exe
```

SPL tạo ra:

```spl
TargetImage="*\lsass.exe"
SourceImage="C:\Windows\System32\rundll32.exe"
CallTrace="*comsvcs.dll*"
```

Field cần nhớ:

```text
TargetImage
SourceImage
CallTrace
```

## 4. Ví Dụ 2: Notepad Sinh Process Đáng Ngờ

Hành vi:

```text
notepad.exe
→ powershell.exe / cmd.exe / mshta.exe
→ có thể là execution hoặc process injection
```

Logic SPL:

```spl
ParentImage="*\notepad.exe"
AND
Image IN (
  powershell.exe,
  pwsh.exe,
  cmd.exe,
  mshta.exe,
  cscript.exe,
  wscript.exe,
  regsvr32.exe,
  rundll32.exe
)
```

Field cần nhớ:

```text
ParentImage
Image
CommandLine
```

## 5. Quy Trình Hunting Trong Splunk

```text
Mở Search & Reporting
→ dán SPL
→ chọn đúng time range
→ chạy search
→ xem process, host và command line
→ xác nhận false positive
```

## 6. Điểm Quan Trọng Nhất

Sigma rule đúng chưa chắc SPL dùng được ngay.

Có thể phải sửa:

```text
Index
Sourcetype
Field names
Path format
Wildcards
Sigma config file
```

Nếu query không trả kết quả, kiểm tra:

```text
Log đã ingest chưa?
Field có đúng tên không?
Time range có đúng không?
Config mapping có đúng không?
```

## Keyword Cần Nhớ

```text
Sigma
sigmac
Splunk
SPL
splunk-windows.yml
TargetImage
SourceImage
CallTrace
ParentImage
Image
LSASS
comsvcs.dll
rundll32.exe
```

## Key Takeaway

```text
Sigma là logic phát hiện trung gian.
Splunk cần SPL đã được chuyển đổi và hiệu chỉnh đúng field.
```

```text
Rule đúng
+ config đúng
+ field đúng
+ time range đúng
= hunting hiệu quả
```

---
layout: post
title: "Hunting Evil with Sigma and Chainsaw"
date: 2026-07-24 23:33:36 +0700
categories: ["Introduction to YARA & Sigma", "Sigma"]
tags: [sigma, chainsaw, evtx, windows-event-logs, powershell, event-id-4688, event-id-4776, threat-hunting, dfir, cdsa]
description: "Tóm tắt cách dùng Chainsaw và Sigma để săn tìm hành vi đáng ngờ trong nhiều file Windows EVTX mà không cần SIEM."
toc: true
---

# Hunting Evil with Sigma and Chainsaw

## Công thức dễ nhớ

```text
EVTX → SIGMA → MAPPING → CHAINSAW → DETECTION
```

## 1. Chainsaw Là Gì?

Chainsaw dùng để phân tích nhanh Windows Event Logs.

Hỗ trợ:

```text
hunt    = chạy Sigma/Chainsaw rules
search  = tìm keyword hoặc field
lint    = kiểm tra rule
dump    = chuyển đổi artefact
analyse = phân tích artefact
```

Phù hợp khi:

- Không có SIEM.
- Cần quét nhiều file `.evtx`.
- Điều tra IR/DFIR ngoại tuyến.
- Cần kiểm thử Sigma rule.

## 2. Lệnh Hunt Cơ Bản

```powershell
chainsaw.exe hunt C:\Events\sample.evtx `
  -s C:\Rules\sigma\rule.yml `
  --mapping .\mappings\sigma-event-logs-all.yml
```

Ý nghĩa:

```text
hunt      = chạy detection
-s        = Sigma rule hoặc thư mục rule
--mapping = ánh xạ field Sigma với field EVTX
```

## 3. Ví Dụ: Failed NTLM Logons

Nguồn:

```text
Event ID 4776
```

Logic:

```text
Một Workstation
→ nhiều lần xác thực thất bại
→ nhiều tài khoản
→ vượt threshold
```

Chainsaw trả về:

```text
Timestamp
Rule name
Event ID
Computer
TargetUserName
Workstation
Count
```

## 4. Ví Dụ: PowerShell Command Quá Dài

Nguồn:

```text
Event ID 4688
```

Detection:

```yaml
selection:
  EventID: 4688
  NewProcessName|endswith:
    - '\powershell.exe'
    - '\pwsh.exe'
    - '\cmd.exe'

selection_length:
  CommandLine|re: '.{1000,}'

condition: selection and selection_length
```

Ý nghĩa:

```text
PowerShell/cmd
+ command line ≥ 1000 ký tự
→ có thể chứa Base64, compression hoặc obfuscation
```

## 5. Mapping Là Điểm Sống Còn

Rule đúng nhưng mapping sai hoặc thiếu field thì Chainsaw có thể trả về:

```text
0 detections
```

Ví dụ thiếu:

```text
NewProcessName
```

Sau khi thêm field vào mapping:

```text
Chainsaw phát hiện đúng các event 4688
```

Vì vậy phải kiểm tra đồng thời:

```text
Rule logic
Field name
Mapping file
Event source
```

## 6. Quy Trình Điều Tra

```text
Lint rule
→ kiểm tra EVTX có đúng Event ID
→ chạy hunt
→ nếu không match, kiểm tra mapping
→ dùng --full xem đầy đủ event
→ tinh chỉnh rule
```

## 7. Keyword Cần Nhớ

```text
Chainsaw
EVTX
hunt
search
lint
-s
--mapping
Event ID 4776
Event ID 4688
NewProcessName
CommandLine
threshold
```

## Key Takeaway

```text
Không có detection chưa chắc rule sai.
Mapping hoặc field có thể chưa đúng.
```

```text
Chainsaw + Sigma =
Threat Hunting nhanh trên EVTX
mà không cần SIEM.
```

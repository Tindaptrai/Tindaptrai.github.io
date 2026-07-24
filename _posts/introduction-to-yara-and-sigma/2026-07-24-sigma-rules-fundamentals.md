---
layout: post
title: "Sigma Rules Fundamentals"
date: 2026-07-24 22:53:39 +0700
categories: ["Introduction to YARA & Sigma", "Sigma"]
tags: [sigma, siem, log-analysis, splunk, elastic, detection-engineering, threat-hunting, soc, yaml, pysigma, cdsa]
description: "Tóm tắt ngắn nhưng đủ ý về Sigma: cấu trúc rule, logsource, detection, modifiers, conditions và cách chuyển đổi sang SIEM."
toc: true
---

# Sigma Rules Fundamentals

## Công thức dễ nhớ

```text
LOGSOURCE → DETECTION → CONDITION → CONVERT → SIEM
```

![Sigma conversion workflow](/assets/img/introduction-to-yara-and-sigma/sigma/sigma-conversion-workflow.png)

## 1. Sigma Là Gì?

Sigma là định dạng rule chuẩn, viết bằng YAML, dùng để mô tả hành vi đáng ngờ trong log.

```text
YARA  = File + Memory
Sigma = Log + SIEM
```

Ưu điểm chính:

```text
Viết một lần
→ chuyển đổi
→ dùng trên nhiều SIEM
```

Có thể chuyển Sigma sang Splunk, Elastic, Sentinel, QRadar, ArcSight hoặc truy vấn EDR/XDR.

> `pySigma` hiện được ưu tiên hơn `sigmac` cho việc chuyển đổi rule.

## 2. Cấu Trúc Rule

```yaml
title: Suspicious Process
id: 11111111-2222-3333-4444-555555555555
status: experimental
description: Detects suspicious process creation

logsource:
  category: process_creation
  product: windows

detection:
  selection:
    ParentImage|endswith: '\svchost.exe'
    Image|endswith: '\mshta.exe'
  condition: selection

falsepositives:
  - Unknown
level: high
```

Các phần cần nhớ:

```text
title       = phát hiện gì
id          = UUID duy nhất
status      = mức trưởng thành
logsource   = nguồn log
detection   = logic tìm kiếm
condition   = điều kiện kích hoạt
level       = mức nghiêm trọng
```

## 3. Status

```text
experimental = mới, dễ false positive
test         = đã test giới hạn
stable       = ổn định
deprecated   = đã được thay thế
unsupported  = chưa dùng được trực tiếp
```

## 4. Logsource

```yaml
logsource:
  category: process_creation
  product: windows
  service: security
```

```text
category = nhóm log
product  = nền tảng
service  = nguồn log cụ thể
```

## 5. Detection

### Map

Các field trong cùng map được nối bằng `AND`.

```yaml
selection:
  EventID: 4769
  TicketOptions: '0x40810000'
  TicketEncryption: '0x17'
```

### List

Các giá trị trong list thường nối bằng `OR`.

```yaml
Image|endswith:
  - '\cmd.exe'
  - '\powershell.exe'
```

## 6. Value Modifiers

```text
contains   = chứa chuỗi
startswith = bắt đầu bằng
endswith   = kết thúc bằng
all        = tất cả giá trị phải khớp
re         = biểu thức chính quy
```

Ví dụ:

```yaml
CommandLine|contains|all:
  - 'powershell'
  - '-enc'
```

## 7. Condition

```yaml
condition: selection
condition: selection and not filter
condition: selection1 or selection2
condition: all of selection*
condition: 1 of selection*
```

Nhớ nhanh:

```text
AND = tất cả
OR  = một trong
NOT = loại trừ
()  = ưu tiên logic
```

## 8. Ví Dụ LethalHTA

```yaml
detection:
  selection:
    ParentImage|endswith: '\svchost.exe'
    Image|endswith: '\mshta.exe'
  condition: selection
```

Ý nghĩa:

```text
svchost.exe
→ sinh mshta.exe
→ hành vi đáng ngờ
```

## 9. Vai Trò Trong SOC

Sigma hỗ trợ:

- Detection Engineering.
- Threat Hunting.
- Incident Response.
- Gap Analysis.
- Detection as Code.
- Chia sẻ rule giữa nhiều SIEM.
- Tự động hóa qua SOAR.

## Keyword Cần Nhớ

```text
Sigma
YAML
logsource
detection
selection
filter
condition
contains
startswith
endswith
all of
1 of
pySigma
SIEM
Detection as Code
```

## Key Takeaway

```text
Sigma không phải query riêng của một SIEM.
Sigma là logic phát hiện trung gian.
```

```text
Rule tốt =
logsource đúng
+ field đúng
+ condition rõ
+ filter hợp lý
+ false positive được kiểm soát
```

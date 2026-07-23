---
layout: post
title: "YARA Rules Fundamentals"
date: 2026-07-23 22:26:13 +0700
categories: ["Introduction to YARA & Sigma", "YARA"]
tags: [yara, malware-detection, memory-forensics, threat-hunting, ioc, detection-engineering, soc, cdsa]
description: "Tóm tắt ngắn và dễ nhớ về YARA: cách hoạt động, cấu trúc rule và các điều kiện quan trọng."
toc: true
---

# YARA Rules Fundamentals

## Công thức dễ nhớ

```text
PATTERN → RULE → SCAN → MATCH
```

![YARA scan workflow](/assets/img/introduction-to-yara-and-sigma/yara/yara-scan-workflow.png)

## 1. YARA Là Gì?

YARA dùng để tìm **mẫu đáng ngờ** trong:

- File.
- Thư mục.
- Memory image.
- Malware sample.

YARA phù hợp với:

```text
Malware Detection
File Classification
IOC Hunting
Memory Forensics
Incident Response
Threat Hunting
```

## 2. Cách YARA Hoạt Động

```text
Analyst viết rule
→ YARA quét file/memory
→ so sánh strings và conditions
→ file khớp thì báo match
```

## 3. Cấu Trúc Rule

```yara
rule Example_Rule
{
    meta:
        author = "Analyst"
        description = "Example detection"

    strings:
        $s1 = "suspicious.exe" ascii
        $s2 = { 4D 5A }

    condition:
        all of them
}
```

Ba phần cần nhớ:

```text
meta      = mô tả rule
strings   = mẫu cần tìm
condition = điều kiện kích hoạt
```

## 4. Strings Có Thể Là Gì?

```text
Text string
Hex bytes
Regular expression
```

Modifier thường gặp:

```text
ascii
wide
nocase
fullword
```

## 5. Condition Quan Trọng

```yara
all of them
any of them
2 of them
filesize < 100KB
uint16(0) == 0x5A4D
```

Ý nghĩa:

- `all of them`: tất cả strings phải khớp.
- `any of them`: chỉ cần một string.
- `2 of them`: ít nhất hai string.
- `filesize`: giới hạn kích thước file.
- `uint16(0)`: đọc 2 byte đầu file.

## 6. Ví Dụ Nhận Diện PE

```yara
condition:
    filesize < 100KB and
    uint16(0) == 0x5A4D
```

```text
0x5A4D = MZ
```

Điều kiện này kiểm tra file nhỏ hơn 100 KB và có PE header.

## 7. Keyword Cần Nhớ

```text
rule
meta
strings
condition
ascii
wide
nocase
fullword
all of them
any of them
filesize
uint16
MZ
```

## Key Takeaway

```text
YARA = tìm pattern trong file hoặc memory.
```

```text
Rule tốt = pattern đặc trưng + condition đủ chặt.
```

Không nên dùng một string phổ biến làm detection duy nhất vì dễ tạo false positive.

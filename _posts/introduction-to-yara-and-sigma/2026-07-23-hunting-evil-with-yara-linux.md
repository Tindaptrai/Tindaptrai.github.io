---
layout: post
title: "Hunting Evil with YARA on Linux"
date: 2026-07-23 23:41:01 +0700
categories: ["Introduction to YARA & Sigma", "YARA"]
tags: [yara, linux, memory-forensics, volatility, yarascan, threat-hunting, incident-response, wannacry, cdsa]
description: "Tóm tắt ngắn về săn tìm malware trong memory image bằng YARA và Volatility trên Linux."
toc: true
---

# Hunting Evil with YARA on Linux

## Công thức dễ nhớ

```text
MEMORY IMAGE → YARA → VOLATILITY → PROCESS
```

## 1. Khi Không Truy Cập Được Máy Nghi Nhiễm

SOC vẫn có thể phân tích nếu nhận được **memory dump**.

Công cụ thu thập RAM thường gặp:

```text
DumpIt
Belkasoft RAM Capturer
Magnet RAM Capture
FTK Imager
LiME
```

## 2. Quét Memory Image Bằng YARA

```bash
yara rule.yar compromised_system.raw --print-strings
```

YARA sẽ tìm IOC trong toàn bộ memory image và in:

```text
Rule match
Matched string
Memory offset
```

## 3. Compile Rule Với yarac

Compile là tùy chọn nhưng hữu ích khi có nhiều rule:

```bash
yarac rule.yar rule.yrc
```

Lợi ích:

```text
Tải rule nhanh hơn
Dễ triển khai
Khó đọc nội dung rule hơn
```

## 4. Volatility yarascan

Volatility giúp xác định **process nào chứa IOC**.

### Tìm một pattern trực tiếp

```bash
vol.py -f compromised_system.raw   yarascan -U "suspicious-string"
```

```text
-U = nhập trực tiếp một string/pattern
```

### Dùng cả file YARA rule

```bash
vol.py -f compromised_system.raw   yarascan -y rule.yar
```

```text
-y = đường dẫn tới file chứa một hoặc nhiều rule
```

## 5. Kết Quả Quan Trọng

Volatility có thể trả về:

```text
Rule name
Process name
PID
Memory address
Matched bytes
```

Trong ví dụ WannaCry:

```text
Process: svchost.exe
PID: 1576
IOC: tasksche.exe
IOC: mssecsvc.exe
IOC: WannaCry domain
```

## 6. Phân Biệt `-U` Và `-y`

```text
-U = tìm nhanh một pattern
-y = chạy rule file hoàn chỉnh
```

## Keyword Cần Nhớ

```text
Memory dump
YARA
yarac
Volatility
yarascan
-U
-y
PID
Memory offset
IOC
WannaCry
```

## Key Takeaway

```text
Không cần truy cập trực tiếp endpoint vẫn có thể hunting bằng memory image.
```

```text
YARA tìm pattern
Volatility xác định process chứa pattern
```

Trong CDSA:

```text
YARA       → Malware Detection
Volatility → Memory Forensics
Kết hợp    → Incident Response + Threat Hunting
```

> Chỉ phân tích memory dump trong HTB, lab hoặc môi trường được cấp phép.

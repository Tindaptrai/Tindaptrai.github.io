---
layout: post
title: "Introduction to YARA and Sigma"
date: 2026-07-21 23:14:52 +0700
categories: ["Introduction to YARA & Sigma", "Fundamentals"]
tags: [yara, sigma, soc, malware-detection, log-analysis, siem, threat-detection, ioc, chainsaw, detection-engineering, cdsa]
description: "Tóm tắt ngắn về YARA và Sigma: mục đích, sự khác nhau và vai trò trong SOC."
toc: true
---

# Introduction to YARA and Sigma

## Công thức dễ nhớ

```text
YARA  = File + Memory
Sigma = Log + SIEM
```

# 1. YARA Là Gì?

YARA dùng pattern matching để phát hiện và phân loại:

- Malware.
- File đáng ngờ.
- Memory artifact.
- IOC trong binary.

Ví dụ mục tiêu:

```text
String
Hex pattern
File structure
Condition
```

# 2. Sigma Là Gì?

Sigma là định dạng rule dùng để phát hiện hành vi trong log.

Áp dụng cho:

- Windows Event Log.
- Sysmon.
- SIEM.
- EDR/XDR.
- Cloud logs.

Sigma có thể chuyển thành query cho Splunk, ELK, Sentinel hoặc các nền tảng khác.

# 3. So Sánh Nhanh

| Công cụ | Dữ liệu chính | Mục đích |
|---|---|---|
| YARA | File, memory | Phát hiện malware và artifact |
| Sigma | Log, event | Phát hiện hành vi và TTP |

# 4. Vai Trò Trong SOC

```text
Threat detection
Log analysis
Malware classification
IOC identification
Threat hunting
Incident response
Detection engineering
```

# 5. Công Cụ Liên Quan

```text
Chainsaw  → chạy Sigma trên Windows Event Log
Uncoder   → chuyển Sigma sang query SIEM/XDR
VirusTotal YARA → kiểm tra và chia sẻ YARA rule
```

# 6. Lợi Ích Chính

- Chuẩn hóa detection rule.
- Dễ chia sẻ trong cộng đồng.
- Tùy chỉnh theo môi trường.
- Tích hợp với SIEM, EDR và IR platform.
- Giúp tự động hóa phát hiện và điều tra.

# Keyword Cần Nhớ

```text
YARA
Sigma
File
Memory
Log
SIEM
IOC
TTP
Chainsaw
Uncoder
Detection Engineering
```

# Key Takeaway

```text
YARA tìm dấu vết trong file và memory.
Sigma tìm hành vi trong log và SIEM.
```

Trong CDSA:

```text
YARA  → Malware Analysis + DFIR
Sigma → SOC Monitoring + Threat Hunting
```

---
layout: post
title: "Hunting Evil with YARA on Windows"
date: 2026-07-23 23:24:16 +0700
categories: ["Introduction to YARA & Sigma", "YARA"]
tags: [yara, windows, malware-hunting, memory-scanning, etw, silketw, powershell, dns, threat-hunting, cdsa]
description: "Tóm tắt ngắn về hunting bằng YARA trên Windows: quét file, process memory và ETW."
toc: true
---

# Hunting Evil with YARA on Windows

## Công thức dễ nhớ

```text
DISK → MEMORY → ETW
```

# 1. Quét File Trên Disk

Quy trình:

```text
Tìm string/hex đặc trưng
→ viết YARA rule
→ quét thư mục
→ xem file match
```

Ví dụ:

```powershell
yara64.exe -s C:\Rules\yara\rule.yar C:\Samples\ -r
```

Ý nghĩa:

```text
-s = in string khớp
-r = quét đệ quy
```

Rule Dharma dùng hai pattern:

```text
PDB path đặc trưng
+ chuỗi sssssbsss
```

# 2. Quét Process Memory

YARA có thể quét theo PID:

```powershell
yara64.exe C:\Rules\yara\meterpreter.yar 9084 --print-strings
```

Hoặc quét toàn bộ process:

```powershell
Get-Process | ForEach-Object {
  yara64.exe C:\Rules\yara\meterpreter.yar $_.Id
}
```

Mục tiêu:

```text
Injected shellcode
Meterpreter
In-memory malware
Process injection
```

# 3. YARA Trên ETW Với SilkETW

ETW cung cấp event từ Windows providers.

SilkETW có thể:

```text
Thu thập ETW
→ áp YARA rule
→ chỉ giữ event match
```

Ví dụ PowerShell:

```powershell
SilkETW.exe `
  -t user `
  -pn Microsoft-Windows-PowerShell `
  -ot file `
  -p .\etw_ps_logs.json `
  -y C:\Rules\yara `
  -yo Matches
```

Ví dụ DNS:

```powershell
SilkETW.exe `
  -t user `
  -pn Microsoft-Windows-DNS-Client `
  -ot file `
  -p .\etw_dns_logs.json `
  -y C:\Rules\yara `
  -yo Matches
```

# 4. ETW Provider Cần Nhớ

```text
Microsoft-Windows-PowerShell
Microsoft-Windows-DNS-Client
Microsoft-Windows-Kernel-Process
Microsoft-Windows-Kernel-File
Microsoft-Windows-Kernel-Network
Microsoft-Windows-Kernel-Registry
Microsoft-Windows-DotNETRuntime
```

# 5. Keyword Quan Trọng

```text
yara64.exe
-s
-r
PID
--print-strings
Process memory
ETW
SilkETW
Provider
PowerShell
DNS
Matches
```

# Key Takeaway

```text
YARA trên Windows có 3 hướng hunting:

DISK   = file độc
MEMORY = shellcode/process injection
ETW    = event PowerShell, DNS, process, registry
```

Trong CDSA:

```text
Disk scan   → Malware Analysis
Memory scan → DFIR
ETW scan    → SOC Monitoring + Threat Hunting
```

> Chỉ thực hành trên HTB, lab cô lập hoặc hệ thống được cấp phép.

---
layout: post
title: "Tapping Into ETW: Detection Examples"
date: 2026-06-24 15:36:56 +0700
categories: [SOC, Windows Security]
tags: [soc, etw, silk-etw, sysmon, dotnet-runtime, parent-pid-spoofing, execute-assembly, seatbelt, process-injection, blue-team]
description: "Tóm tắt cách dùng ETW để tăng visibility: phát hiện parent-child relationship bất thường, Parent PID Spoofing và malicious .NET assembly loading."
toc: true
---

# Tapping Into ETW: Detection Examples

**Event Tracing for Windows (ETW)** là nguồn telemetry sâu của Windows. ETW có thể bổ sung cho Sysmon khi attacker dùng kỹ thuật làm sai lệch log hoặc chạy payload trong memory.

---

## Timeline cập nhật

```text
Created at: 2026-06-24 15:36:56 +0700
Timezone: Asia/Ho_Chi_Minh
Topic: Tapping Into ETW
Focus: SilkETW, Sysmon, Kernel-Process provider, DotNETRuntime provider
```

---

# 1. Ý tưởng chính

Bài này tập trung vào 2 ví dụ detection:

```text
1. Detecting strange parent-child relationships
2. Detecting malicious .NET assembly loading
```

Sysmon rất hữu ích, nhưng có những tình huống cần ETW để nhìn rõ hơn:

- Parent PID Spoofing có thể làm parent process trong Sysmon bị sai lệch.
- Sysmon Event ID 7 chỉ cho biết DLL được load, chưa đủ chi tiết về .NET assembly.
- In-memory execution có thể để lại ít artifact trên disk.

---

# 2. Strange Parent-Child Relationships

Parent-child process relationship bất thường có thể là dấu hiệu attack.

Ví dụ bình thường:

```text
explorer.exe -> notepad.exe
services.exe -> svchost.exe
spoolsv.exe -> conhost.exe
```

Ví dụ đáng ngờ:

```text
calc.exe -> cmd.exe
spoolsv.exe -> whoami.exe
spoolsv.exe -> cmd.exe
notepad.exe -> powershell.exe
```

Nếu một process không nên spawn process khác nhưng lại spawn, analyst cần điều tra.

---

# 3. Parent PID Spoofing

**Parent PID Spoofing** là kỹ thuật attacker dùng để giả mạo parent process.

Ví dụ:

```text
Thực tế: powershell.exe -> cmd.exe
Log có thể hiện: spoolsv.exe -> cmd.exe
```

Khi đó Sysmon Event ID 1 có thể hiển thị parent process không đúng với process thật sự đã tạo child process.

Ví dụ mô phỏng:

```powershell
powershell -ep bypass
Import-Module .\psgetsys.ps1
[MyProcess]::CreateProcessFromParent([Process ID of spoolsv.exe],"C:\Windows\System32\cmd.exe","")
```

---

# 4. Dùng ETW Microsoft-Windows-Kernel-Process

Provider cần dùng:

```text
Microsoft-Windows-Kernel-Process
```

Tìm provider:

```cmd
logman.exe query providers | findstr "Process"
```

Thu thập bằng SilkETW:

```cmd
SilkETW.exe -t user -pn Microsoft-Windows-Kernel-Process -ot file -p C:\windows\temp\etw.json
```

Ý nghĩa:

```text
Sysmon có thể bị parent PID spoofing làm sai parent.
ETW Kernel-Process có thể giúp xác định process thật sự tạo cmd.exe.
```

---

# 5. SPL query gợi ý: parent-child bất thường

```spl
index="main" sourcetype="WinEventLog:Sysmon" EventCode=1
| stats count by ParentImage, Image, CommandLine, ComputerName, User
| sort + count
```

Tập trung vào child process nguy hiểm:

```spl
index="main" sourcetype="WinEventLog:Sysmon" EventCode=1
(Image="*\\cmd.exe" OR Image="*\\powershell.exe" OR Image="*\\whoami.exe")
| stats count by ParentImage, Image, CommandLine, ComputerName, User
| sort + count
```

Cần kiểm tra thêm:

- ParentImage.
- Image.
- CommandLine.
- User.
- Host.
- ProcessGuid / ParentProcessGuid.
- ETW Kernel-Process data.

---

# 6. Malicious .NET Assembly Loading

Attacker ngày càng dùng **Bring Your Own Land (BYOL)**:

```text
Mang theo tool .NET/C# riêng
-> load trực tiếp vào memory
-> giảm artifact trên disk
-> né detection dựa trên file
```

Ví dụ nổi bật:

```text
Cobalt Strike execute-assembly
GhostPack Seatbelt
```

---

# 7. Vì sao .NET assembly loading nguy hiểm?

.NET assembly có lợi cho attacker vì:

- Windows thường có sẵn .NET.
- C# dễ viết post-exploitation tool.
- Managed code giúp phát triển nhanh.
- Có thể execute in-memory.
- Có thư viện HTTP, crypto, IPC, named pipes.
- Ít artifact hơn nếu không ghi file xuống disk.

---

# 8. Detect .NET bằng Sysmon Event ID 7

Các DLL .NET thường thấy:

```text
clr.dll
clrjit.dll
mscoree.dll
```

SPL gợi ý:

```spl
index="main" sourcetype="WinEventLog:Sysmon" EventCode=7
(ImageLoaded="*\\clr.dll" OR ImageLoaded="*\\clrjit.dll" OR ImageLoaded="*\\mscoree.dll")
| stats count by Image, ImageLoaded, ComputerName, User
| sort - count
```

Hạn chế:

```text
Sysmon Event ID 7 cho biết DLL được load,
nhưng không nói rõ assembly/method nào đang chạy.
```

---

# 9. Dùng ETW Microsoft-Windows-DotNETRuntime

Provider cần dùng:

```text
Microsoft-Windows-DotNETRuntime
```

Thu thập bằng SilkETW:

```cmd
SilkETW.exe -t user -pn Microsoft-Windows-DotNETRuntime -uk 0x2038 -ot file -p C:\windows\temp\etw.json
```

Keyword `0x2038` tập trung vào:

| Keyword | Ý nghĩa |
|---|---|
| JitKeyword | Method được compile runtime |
| InteropKeyword | Managed code gọi unmanaged/native API |
| LoaderKeyword | Assembly loading |
| NGenKeyword | Native/precompiled assembly activity |

---

# 10. SPL query gợi ý: process load .NET runtime bất thường

```spl
index="main" sourcetype="WinEventLog:Sysmon" EventCode=7
(ImageLoaded="*\\clr.dll" OR ImageLoaded="*\\clrjit.dll" OR ImageLoaded="*\\mscoree.dll")
NOT Image="C:\\Windows\\Microsoft.NET\\*"
| stats values(ImageLoaded) as LoadedDotNetDlls count by Image, ComputerName, User
| sort - count
```

Process đáng chú ý nếu load .NET bất thường:

```text
rundll32.exe
regsvr32.exe
wmiprvse.exe
spoolsv.exe
notepad.exe
calc.exe
msbuild.exe
installutil.exe
powershell.exe
```

---

# 11. Detection Summary

| Case | Signal | Better visibility |
|---|---|---|
| Parent PID Spoofing | Parent-child process lạ trong Sysmon Event ID 1 | ETW Microsoft-Windows-Kernel-Process |
| .NET assembly loading | clr.dll / clrjit.dll / mscoree.dll loaded | ETW Microsoft-Windows-DotNETRuntime |
| Execute-Assembly | .NET DLL load trong process bất thường | DotNETRuntime Loader/JIT events |
| Seatbelt execution | .NET assembly reconnaissance | Sysmon Event ID 7 + ETW |

---

# 12. Tài liệu liên quan

```text
Samir Bousseaden parent-child relationship:
https://x.com/SBousseaden/status/1195373669930983424

SilkETW:
https://github.com/mandiant/SilkETW

AttackIQ - Hiding in Plain Sight:
https://www.attackiq.com/2023/03/16/hiding-in-plain-sight/

Mandiant:
https://cloud.google.com/security/mandiant

Detecting .NET C# Injection Execute-Assembly:
https://redhead0ntherun.medium.com/detecting-net-c-injection-execute-assembly-1894dbb04ff7

GhostPack Seatbelt:
https://github.com/GhostPack/Seatbelt
```

---

# Key Takeaway

ETW giúp SOC analyst có thêm góc nhìn sâu hơn khi Sysmon chưa đủ.

```text
Sysmon = detection nhanh, dễ ingest vào SIEM.
ETW = telemetry sâu hơn, hữu ích để xác minh spoofing và in-memory behavior.
```

Khi kết hợp Sysmon + ETW + Splunk, analyst có thể phát hiện tốt hơn các kỹ thuật như Parent PID Spoofing, malicious .NET assembly loading và execute-assembly.

---
layout: post
title: "Memory Forensics"
date: 2026-07-27 22:21:42 +0700
categories: ["Introduction to Digital Forensics", "Memory Forensics"]
tags: [memory-forensics, volatility, volatility2, ram-analysis, pslist, psscan, psxview, netscan, malfind, handles, svcscan, dlllist, hivelist, rootkit, dkom, yarascan, strings, dfir, cdsa]
description: "Tóm tắt Memory Forensics với quy trình phân tích RAM, plugin Volatility quan trọng, rootkit analysis và string analysis."
toc: true
---

# Memory Forensics

Memory Forensics là phân tích **RAM** tại một thời điểm cụ thể để tìm process, network connection, injected code, credential, encryption key và malware artifact.

## Công thức dễ nhớ

```text
PROFILE → PROCESS → NETWORK → INJECTION → HANDLES
→ SERVICES → DLLS → HIVES → ROOTKIT
```

# 1. Dữ Liệu Có Thể Tìm Trong RAM

```text
Running processes
Network connections
Open files
File handles
Registry keys
Loaded DLLs/drivers
Command history
Console sessions
Kernel structures
Credentials
Encryption keys
Malware artifacts
Process memory regions
```

Memory Forensics đặc biệt hữu ích để phát hiện:

- Fileless malware.
- Process injection và process hollowing.
- C2 connection.
- Credential hoặc key còn trong RAM.
- Hidden process và rootkit.
- Payload cần dump để reverse engineering.

# 2. Quy Trình Điều Tra

```text
1. Process Identification
2. Process Verification
3. DLL and Handle Analysis
4. Network Analysis
5. Code Injection Detection
6. Rootkit Discovery
7. Dump Suspicious Components
```

# 3. Volatility Framework

Lệnh Volatility 2 tổng quát:

```bash
vol.py -f memory.raw --profile=PROFILE plugin
```

```text
-f        = memory image
--profile = Windows profile
plugin    = tác vụ phân tích
```

> Volatility 2 cần profile; Volatility 3 tự động nhận diện tốt hơn.

# 4. Plugin Quan Trọng

| Plugin | Chức năng |
|---|---|
| `imageinfo` | Gợi ý profile |
| `pslist` | Process trong active list |
| `psscan` | Quét EPROCESS trong memory pool |
| `pstree` | Cây process |
| `psxview` | Tìm hidden process |
| `cmdline` | Command line |
| `netscan` | Network connections và ports |
| `connscan` | Connection cũ/đã terminate |
| `malfind` | Injected code và suspicious VAD |
| `handles` | File, key, process handles |
| `svcscan` | Windows services |
| `dlllist` | DLL của process |
| `ldrmodules` | DLL bị unlink |
| `hivelist` | Registry hives |
| `yarascan` | Quét memory bằng YARA |
| `procdump` | Dump process executable |
| `memdump` | Dump process memory |
| `vaddump` | Dump VAD regions |
| `cmdscan` | Command history |
| `consoles` | Console sessions |
| `timeliner` | Timeline từ memory |

# 5. Xác Định Profile

```bash
vol.py -f memory.vmem imageinfo
```

Kết quả cần xem:

```text
Suggested Profile
Image date/time
KDBG
DTB
Number of processors
```

Kiểm tra profile:

```bash
vol.py -f memory.vmem   --profile=Win7SP1x64   pslist
```

# 6. Process Analysis

## pslist

```bash
vol.py -f memory.vmem   --profile=Win7SP1x64   pslist
```

Field quan trọng:

```text
Name
PID
PPID
Threads
Handles
Session
Start Time
Exit Time
```

## pslist và psscan

```text
pslist = đọc ActiveProcessLinks
psscan = quét EPROCESS trong memory pool
```

Nếu `psscan` thấy nhưng `pslist` không thấy:

```text
→ process đã terminate
hoặc
→ process bị rootkit/DKOM che giấu
```

IOC trong ví dụ:

```text
Ransomware.wan
tasksche.exe
@WanaDecryptor
taskhsvc.exe
```

# 7. Network Analysis

## netscan

```bash
vol.py -f memory.vmem   --profile=Win7SP1x64   netscan
```

Cho biết:

```text
Protocol
Local Address
Foreign Address
State
PID
Owner
Created Time
```

Cần chú ý:

```text
External IP lạ
Port bất thường
ESTABLISHED connection
LISTENING service lạ
Process không nên dùng network
```

## connscan

`connscan` dùng pool tag scanning để tìm connection đã đóng hoặc không còn trong bảng active.

# 8. Injected Code — malfind

```bash
vol.py -f memory.vmem   --profile=Win7SP1x64   malfind --pid 608
```

Dấu hiệu đáng ngờ:

```text
PAGE_EXECUTE_READWRITE
PrivateMemory
Executable VAD
MZ header trong memory
Shellcode
Unmapped executable region
```

# 9. Handles

```bash
# Registry keys
vol.py -f memory.vmem --profile=Win7SP1x64   handles -p 1512 --object-type=Key

# Files
vol.py -f memory.vmem --profile=Win7SP1x64   handles -p 1512 --object-type=File

# Processes
vol.py -f memory.vmem --profile=Win7SP1x64   handles -p 1512 --object-type=Process
```

Handles cho biết process đang tương tác với:

```text
Files
Registry keys
Processes
Mutexes
Devices
Events
```

# 10. Services — svcscan

```bash
vol.py -f memory.vmem   --profile=Win7SP1x64   svcscan
```

Field quan trọng:

```text
Service Name
Display Name
Start Type
State
PID
Binary Path
Service Type
```

Dấu hiệu persistence:

```text
Tên service ngẫu nhiên
Binary path bất thường
Auto-start service mới
Kernel driver lạ
```

# 11. DLL Analysis — dlllist

```bash
vol.py -f memory.vmem   --profile=Win7SP1x64   dlllist -p 1512
```

Kiểm tra:

```text
DLL từ Temp/AppData/Public
DLL không có path
DLL bất thường với process
Network-related DLL
Load time bất thường
```

Plugin liên quan:

```text
ldrmodules
dlldump
apihooks
```

# 12. Registry Hives — hivelist

```bash
vol.py -f memory.vmem   --profile=Win7SP1x64   hivelist
```

Hive quan trọng:

```text
SYSTEM
SOFTWARE
SAM
SECURITY
NTUSER.DAT
UsrClass.dat
DEFAULT
HARDWARE
```

# 13. Rootkit, EPROCESS Và DKOM

`EPROCESS` là kernel structure đại diện cho process.

```text
ActiveProcessLinks = doubly-linked list
FLINK = process kế tiếp
BLINK = process trước
```

**DKOM — Direct Kernel Object Manipulation** có thể:

```text
Unlink process
Che giấu driver
Sửa kernel object
Qua mặt tool user-mode
```

Phát hiện:

```text
pslist không thấy
psscan thấy
→ nghi hidden process/rootkit
```

Nên correlate:

```text
pslist
psscan
psxview
pstree
```

# 14. Strings Analysis

Dùng:

```text
Sysinternals Strings
Binutils strings
grep
Regex
```

## IPv4

```bash
strings memory.vmem |
grep -E "\b([0-9]{1,3}\.){3}[0-9]{1,3}\b"
```

## Email

```bash
strings memory.vmem |
grep -oE "\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,4}\b"
```

## Command Artifacts

```bash
strings memory.vmem |
grep -E "(cmd|powershell|bash)[^\s]+"
```

Có thể tìm thêm:

```text
URLs
Domains
File paths
Passwords
Registry paths
Mutexes
PowerShell commands
Ransom notes
```

> String match là manh mối, không phải kết luận. Phải correlate với PID, offset và artifact khác.

# 15. Workflow Chuẩn

```text
1. Hash memory image
2. imageinfo
3. Chọn profile
4. pslist / pstree / psscan / psxview
5. cmdline và handles
6. netscan / connscan
7. dlllist / ldrmodules
8. malfind
9. svcscan
10. hivelist
11. yarascan
12. Dump process/VAD
13. strings + regex
14. Correlate PID, IOC, network, timeline
15. Báo cáo
```

# 16. Keyword Cần Nhớ

```text
RAM
Volatility
Profile
imageinfo
EPROCESS
ActiveProcessLinks
FLINK
BLINK
DKOM
pslist
psscan
psxview
pstree
cmdline
netscan
connscan
malfind
PAGE_EXECUTE_READWRITE
handles
svcscan
dlllist
ldrmodules
hivelist
yarascan
procdump
memdump
vaddump
Rootkit
Process Injection
Process Hollowing
C2
Strings
Regex
```

# Key Takeaway

```text
pslist   → active process list
psscan   → EPROCESS trong memory pool
psxview  → hidden process
netscan  → network artifacts
malfind  → injected code
handles  → object interaction
svcscan  → services
dlllist  → loaded DLLs
hivelist → registry hives
```

```text
Memory Forensics =
Process + Network + Injection + Kernel + Registry + IOC correlation
```

> Chỉ phân tích memory image từ HTB, lab hoặc hệ thống được cấp phép.

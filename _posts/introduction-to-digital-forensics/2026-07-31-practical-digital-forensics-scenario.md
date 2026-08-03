---
layout: post
title: "Practical Digital Forensics Scenario"
date: 2026-07-31 01:14:46 +0700
categories: ["Introduction to Digital Forensics", "Case Study"]
tags: [digital-forensics, dfir, volatility3, autopsy, yara, cobalt-strike, autoruns, chainsaw, sigma, pecmd, usn-journal, mft, srum, timestomping, incident-response, cdsa]
description: "Case study điều tra Windows bằng memory dump, full disk image và rapid triage artifacts để dựng lại chuỗi tấn công Cobalt Strike."
toc: true
---

# Practical Digital Forensics Scenario

## Công thức dễ nhớ

```text
MEMORY → DISK → TRIAGE → CORRELATE → TIMELINE → REPORT
```

Case study sử dụng ba nguồn evidence:

```text
Memory dump
Full disk image
Rapid triage artifacts
```

Mục tiêu:

```text
Initial Access
→ Execution
→ C2
→ Privilege Escalation
→ Persistence
→ Collection/Exfiltration
→ Anti-Forensics
```

---

# 1. Evidence Locations

```text
Memory:
C:\Users\johndoe\Desktop\memdump\PhysicalMemory.raw

Rapid triage:
C:\Users\johndoe\Desktop\kapefiles
C:\Users\johndoe\Desktop\files

Full disk image:
C:\Users\johndoe\Desktop\fulldisk.raw.001

Autopsy case:
C:\Users\johndoe\Desktop\MalwareAttack
```

Trong thực tế, không nên phân tích trực tiếp trên máy bị compromise. Cần dùng forensic workstation và bản sao evidence.

---

# 2. Tool Mapping

| Nhiệm vụ | Công cụ |
|---|---|
| Memory analysis | Volatility 3 |
| Process dump | `windows.memmap` |
| Malware matching | YARA |
| Disk/image analysis | Autopsy |
| Beacon config | CobaltStrikeParser |
| Persistence | Autoruns |
| Windows log hunting | Chainsaw + Sigma |
| Prefetch analysis | PECmd |
| USN Journal | `usn.py` |
| MFT parsing | MFTECmd / MFT Explorer |
| Network/resource usage | SRUM |
| Timeline | Autopsy / Plaso |

---

# 3. Memory Profile — `windows.info`

```cmd
python vol.py -q ^
  -f ..\memdump\PhysicalMemory.raw ^
  windows.info
```

Kết quả quan trọng:

```text
Windows 10
64-bit
Kernel/Symbols loaded
SystemTime: 2023-08-10 09:35:40
```

Volatility 3 không dùng `--profile` như Volatility 2; plugin `windows.info` cung cấp OS và kernel context.

---

# 4. Injected Code — `windows.malfind`

```cmd
python vol.py -q ^
  -f ..\memdump\PhysicalMemory.raw ^
  windows.malfind
```

Process đáng ngờ:

```text
PID 3648 → rundll32.exe
PID 6744 → powershell.exe
PID 5468 → rundll32.exe
```

Dấu hiệu:

```text
PAGE_EXECUTE_READWRITE
Private VAD
Executable + writable memory
```

Ghi nhớ:

```text
RWX không tự động đồng nghĩa malware
nhưng là dấu hiệu mạnh của injection/unpacked code
```

---

# 5. Process Analysis

## `windows.pslist`

```cmd
python vol.py -q ^
  -f ..\memdump\PhysicalMemory.raw ^
  windows.pslist
```

Dùng để xem:

```text
PID
PPID
Process name
Create time
Exit time
Session
```

## `windows.pstree`

```cmd
python vol.py -q ^
  -f ..\memdump\PhysicalMemory.raw ^
  windows.pstree
```

Quan hệ quan trọng:

```text
explorer.exe
└── rundll32.exe PID 3648
```

Process tree giúp xác định parent-child bất thường và nguồn khởi chạy.

---

# 6. Command Lines — `windows.cmdline`

```cmd
python vol.py -q ^
  -f ..\memdump\PhysicalMemory.raw ^
  windows.cmdline
```

Phát hiện nổi bật:

```text
rundll32.exe payload.dll,StartW
```

Và PowerShell:

```text
PowerShell.exe
-nop
-w hidden
-encodedcommand <Base64>
```

Keyword cần hunt:

```text
-nop
-w hidden
-encodedcommand
rundll32.exe
payload.dll
StartW
```

---

# 7. Dump Process Và Quét YARA

Dump toàn bộ memory pages của PID 3648:

```cmd
python vol.py -q ^
  -f ..\memdump\PhysicalMemory.raw ^
  windows.memmap --pid 3648 --dump
```

Output:

```text
pid.3648.dmp
```

Quét bằng nhiều rule:

```powershell
$rules = Get-ChildItem C:\Users\johndoe\Desktop\yara-4.3.2-2150-win64\rules

foreach ($rule in $rules) {
    C:\Users\johndoe\Desktop\yara-4.3.2-2150-win64\yara64.exe `
      $rule.FullName `
      C:\Users\johndoe\Desktop\pid.3648.dmp
}
```

YARA hits:

```text
HKTL_CobaltStrike_Beacon_Strings
HKTL_CobaltStrike_Beacon_4_2_Decrypt
HKTL_Win_CobaltStrike
CobaltStrike_Sleep_Decoder_Indicator
WiltedTulip_ReflectiveLoader
```

Kết luận điều tra:

```text
PID 3648 chứa dấu vết Cobalt Strike Beacon
```

---

# 8. Loaded DLLs — `windows.dlllist`

```cmd
python vol.py -q ^
  -f ..\memdump\PhysicalMemory.raw ^
  windows.dlllist --pid 3648
```

Phát hiện:

```text
E:\payload.dll
```

Đường dẫn `E:` gợi ý payload có thể đến từ:

```text
Mounted ISO
USB
External/removable media
```

Không kết luận chỉ từ drive letter; cần correlate với disk artifacts.

---

# 9. Open Handles — `windows.handles`

```cmd
python vol.py -q ^
  -f ..\memdump\PhysicalMemory.raw ^
  windows.handles --pid 3648
```

Handles có thể cho thấy process đang truy cập:

```text
Files
Registry keys
Threads
Desktop
Named pipes
Network-related objects
```

Trong case này, PID 3648 có interaction với:

```text
\Device\HarddiskVolume3\Users\johndoe\Desktop
```

Điều này hỗ trợ giả thuyết attacker truy cập hoặc thu thập dữ liệu trên Desktop.

---

# 10. Network Artifacts

## `windows.netstat`

```cmd
python vol.py -q ^
  -f ..\memdump\PhysicalMemory.raw ^
  windows.netstat
```

## `windows.netscan`

```cmd
python vol.py -q ^
  -f ..\memdump\PhysicalMemory.raw ^
  windows.netscan
```

Kết nối đáng chú ý:

```text
PID: 3648
Process: rundll32.exe
Remote IP: 44.214.212.249
Remote Port: 80
State: ESTABLISHED / LAST_ACK
```

Correlate:

```text
rundll32.exe
+ payload.dll
+ Cobalt Strike YARA hits
+ HTTP connection
= C2 activity có độ tin cậy cao
```

---

# 11. Disk Search Với Autopsy

Tìm keyword:

```text
payload.dll
```

Artifact quan trọng:

```text
Finance08062023.iso
Location: Downloads
```

Ngoài ra có Chrome cache chứa nội dung liên quan, gợi ý file ISO được tải bằng trình duyệt.

Workflow:

```text
Keyword Search
→ xác định ISO
→ Extract File
→ kiểm tra ADS/Web Downloads
→ mount ISO
```

---

# 12. Initial Access Qua ISO

`Zone.Identifier` và Web Downloads cho thấy:

```text
Finance08062023.iso
→ tải từ letsgohunt[.]site
```

Bên trong ISO:

```text
payload.dll
shortcut/LNK
```

Shortcut gọi:

```text
rundll32.exe payload.dll,StartW
```

Chuỗi initial access:

```text
Browser download
→ malicious ISO
→ user mount/open
→ LNK executes rundll32
→ payload.dll loaded
```

---

# 13. Cobalt Strike Beacon Configuration

Parser:

```cmd
python parse_beacon_config.py E:\payload.dll
```

Config nổi bật:

```text
BeaconType: HTTP
Port: 80
SleepTime: 60000
C2Server: letsgohunt.site,/load
HttpPostUri: /submit.php
Metadata: Base64 in Cookie
UsesCookies: True
```

Injection config:

```text
VirtualAllocEx
CreateThread
SetThreadContext
CreateRemoteThread
RtlCreateUserThread
StartRWX: True
UseRWX: True
```

Điểm nhớ:

```text
Config Beacon xác nhận:
C2 + URI + User-Agent + injection behavior
```

---

# 14. Persistence Với Autoruns

## Run Key

```text
HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
```

Image path:

```text
C:\ProgramData\svchost.exe
```

Timestamp được ghi nhận:

```text
2023-08-10 09:25:51 UTC
```

## Startup Folder

File đáng ngờ:

```text
photo443.exe
```

SHA-256:

```text
E986DAA66F2E8E4C47E8EAA874FCD4DCAB8045F1F727DAF7AC15843101385194
```

## Scheduled Tasks

Autoruns cũng phát hiện thêm persistence trong tab Scheduled Tasks.

Ba vùng persistence cần hunt:

```text
Run Keys
Startup Folder
Scheduled Tasks
```

---

# 15. Timestomping Qua MFT

File:

```text
C:\ProgramData\svchost.exe
```

Dấu hiệu:

```text
Timestamp hiển thị bên ngoài không khớp
với timestamp trong MFT attributes
```

Cần so sánh:

```text
$STANDARD_INFORMATION
$FILE_NAME
```

USN Journal cũng ghi:

```text
2023-08-10 09:25:48 → svchost.exe created
2023-08-10 09:26:46 → BASIC_INFO_CHANGE
```

Pattern:

```text
File Create
→ BASIC_INFO_CHANGE
→ nghi timestomping
```

---

# 16. SRUM Và Exfiltration

Artifact:

```text
SRUDB.dat
```

Autopsy Data Artifacts cho thấy `users.db` trên Desktop có network/resource activity đáng chú ý.

Con số được bài học ghi nhận:

```text
430,526,981 bytes
```

Đây là dấu hiệu có thể liên quan đến exfiltration, nhưng phải correlate thêm với:

```text
Process
Destination IP
Time window
Network logs
File handles
```

---

# 17. Windows Event Logs Với Chainsaw

```cmd
chainsaw_x86_64-pc-windows-msvc.exe hunt ^
  "..\kapefiles\auto\C%3A\Windows\System32\winevt\Logs" ^
  -s sigma/ ^
  --mapping mappings/sigma-event-logs-all.yml ^
  -r rules/ ^
  --csv ^
  --output output_csv
```

Kết quả:

```text
142 EVTX artifacts
2872 rules loaded
2212 detections
```

Output:

```text
sigma.csv
account_tampering.csv
antivirus.csv
```

Detections quan trọng:

```text
Cobalt Strike load by rundll32
Cobalt Strike named pipe
fodhelper.exe UAC bypass
Encoded PowerShell
LSASS access
New user creation
User added to Administrators
```

Account được tạo:

```text
Admin
```

---

# 18. Prefetch Với PECmd

```cmd
PECmd.exe ^
  -d "C:\Users\johndoe\Desktop\kapefiles\auto\C%3A\Windows\Prefetch" ^
  -q ^
  --csv C:\Users\johndoe\Desktop ^
  --csvf suspect_prefetch.csv
```

Output:

```text
suspect_prefetch.csv
suspect_prefetch_Timeline.csv
```

Prefetch dùng để xác nhận:

```text
Executable đã chạy
Run count
Last run time
Referenced files/directories
```

Đặc biệt kiểm tra:

```text
rundll32.exe
powershell.exe
fodhelper.exe
svchost.exe
photo443.exe
```

---

# 19. USN Journal

Parse:

```cmd
python usn.py ^
  -f "$UsnJrnl:$J" ^
  -o usn_output.csv ^
  -c
```

Lọc timeline bằng PowerShell:

```powershell
$time1 = [DateTime]::ParseExact(
  "2023-08-10 09:00:00.000000",
  "yyyy-MM-dd HH:mm:ss.ffffff",
  $null
)

$time2 = [DateTime]::ParseExact(
  "2023-08-10 10:00:00.000000",
  "yyyy-MM-dd HH:mm:ss.ffffff",
  $null
)

Import-Csv .\usn_output.csv |
Where-Object { $_.FileName -match '\.exe$|\.txt$|\.msi$|\.bat$|\.ps1$|\.iso$|\.lnk$' } |
Where-Object {
  $_.timestamp -as [DateTime] -ge $time1 -and
  $_.timestamp -as [DateTime] -lt $time2
}
```

Event nổi bật:

```text
09:24:23 → flag.txt deleted
09:25:48 → svchost.exe created
09:26:46 → svchost.exe BASIC_INFO_CHANGE
09:28:13 → photo443.exe activity
```

USN Journal rất mạnh để dựng:

```text
Create
Rename
Delete
Data Extend
Basic Info Change
Stream Change
```

---

# 20. Deleted File Recovery

`flag.txt` đã bị xóa.

Nguồn phục hồi được sử dụng:

```text
MFT record
pagefile.sys
```

Parse MFT:

```cmd
MFTECmd.exe ^
  -f C:\Users\johndoe\Desktop\files\mft_data ^
  --csv C:\Users\johndoe\Desktop ^
  --csvf mft_csv.csv
```

Tìm record:

```powershell
Select-String `
  -Path C:\Users\johndoe\Desktop\mft_csv.csv `
  -Pattern "flag.txt"
```

Trong case chính, dữ liệu file đã bị ghi đè, nhưng fragment còn trong:

```text
pagefile.sys
```

Keyword được tìm thấy:

```text
2023_Hello_you_found_our_flag
```

---

# 21. Timeline Điều Tra

Autopsy dùng Plaso để dựng timeline.

Khoảng thời gian tập trung:

```text
2023-08-10 09:13 UTC
→
2023-08-10 09:30 UTC
```

Event types:

```text
Web Activity: All
Other: All
Timezone: GMT / UTC
```

Chuỗi bằng chứng chính:

```text
Finance08062023.iso downloaded
→ ISO mounted/opened
→ LNK launches rundll32
→ payload.dll loaded
→ Cobalt Strike Beacon executes
→ C2 to letsgohunt.site / 44.214.212.249
→ PowerShell encoded execution
→ UAC bypass via fodhelper.exe
→ LSASS access
→ Admin account created
→ persistence installed
→ data potentially exfiltrated
→ timestomping and file deletion
```

---

# 22. IOC Summary

| IOC | Giá trị |
|---|---|
| Domain | `letsgohunt[.]site` |
| C2 path | `/load` |
| POST URI | `/submit.php` |
| IP | `44.214.212.249` |
| Malicious ISO | `Finance08062023.iso` |
| DLL | `payload.dll` |
| Suspicious process | `rundll32.exe` PID 3648 |
| PowerShell | `-nop -w hidden -encodedcommand` |
| Persistence binary | `C:\ProgramData\svchost.exe` |
| Startup binary | `photo443.exe` |
| SHA-256 | `E986DAA66F2E8E4C47E8EAA874FCD4DCAB8045F1F727DAF7AC15843101385194` |
| New account | `Admin` |
| Possible collected file | `users.db` |

---

# 23. ATT&CK Mapping

```text
T1204        User Execution
T1218.011    Rundll32
T1059.001    PowerShell
T1027        Obfuscated/Encoded Files or Information
T1055        Process Injection
T1548.002    Bypass User Account Control
T1003        OS Credential Dumping
T1136        Create Account
T1068/T1548  Privilege Escalation context
T1060/T1547  Registry Run Keys / Startup Folder
T1053        Scheduled Task
T1070.006    Timestomp
T1070.004    File Deletion
T1041        Exfiltration Over C2 Channel
```

ATT&CK mapping là diễn giải detection context; cần đối chiếu telemetry trước khi đưa vào báo cáo chính thức.

---

# 24. Workflow Điều Tra Chuẩn

```text
1. Hash và bảo toàn evidence
2. windows.info
3. pslist / pstree / cmdline
4. malfind
5. dlllist / handles
6. netstat / netscan
7. memmap --dump
8. YARA scan
9. Autopsy keyword and web artifacts
10. Parse Beacon configuration
11. Autoruns persistence review
12. MFT/USN timestomp analysis
13. Chainsaw + Sigma
14. PECmd execution timeline
15. SRUM exfiltration review
16. Plaso/Autopsy timeline
17. IOC and ATT&CK mapping
18. Report scope, impact and confidence
```

---

# Keyword Cần Nhớ

```text
Volatility 3
windows.info
windows.malfind
PAGE_EXECUTE_READWRITE
windows.pslist
windows.pstree
windows.cmdline
windows.memmap
windows.dlllist
windows.handles
windows.netstat
windows.netscan
YARA
Cobalt Strike
Beacon
payload.dll
rundll32.exe
Autopsy
Zone.Identifier
Finance08062023.iso
CobaltStrikeParser
Autoruns
Run Key
Scheduled Task
Startup Folder
Timestomping
SRUM
Chainsaw
Sigma
PECmd
Prefetch
USN Journal
MFTECmd
pagefile.sys
Plaso
Timeline
```

---

# Key Takeaway

```text
Memory cho thấy runtime behavior.
Disk cho thấy nguồn payload và artifact tồn tại.
Rapid triage cho thấy execution, persistence và timeline.
```

```text
Một kết luận DFIR tốt phải correlate:
Process + Command Line + DLL + Network + File System + Event Logs
```

> Chỉ thực hiện trên HTB, forensic lab hoặc hệ thống được cấp phép. Luôn làm việc trên bản sao evidence và ghi đầy đủ chain of custody.

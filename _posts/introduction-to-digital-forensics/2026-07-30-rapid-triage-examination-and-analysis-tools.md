---
layout: post
title: "Rapid Triage Examination & Analysis Tools"
date: 2026-07-30 01:27:31 +0700
categories: ["Introduction to Digital Forensics", "Rapid Triage"]
tags: [rapid-triage, eric-zimmerman-tools, mft, macb, timestomping, usn-journal, evtxecmd, eql, registry-explorer, regripper, prefetch, pecmd, shimcache, amcache, bam, api-monitor, powershell, dfir, cdsa]
description: "Tóm tắt các công cụ và artifact quan trọng trong Rapid Triage: MFT, USN Journal, EVTX, Registry, Prefetch, ShimCache, Amcache, BAM và API Monitor."
toc: true
---

# Rapid Triage Examination & Analysis Tools

## Công thức dễ nhớ

```text
MFT → USN → EVTX → REGISTRY → EXECUTION → API CALLS → TIMELINE
```

Rapid Triage nhằm nhanh chóng trích xuất và phân tích artifact có giá trị cao để dựng timeline, xác định execution, persistence và hành vi của attacker.

## 1. Eric Zimmerman Tools

| Công cụ | Artifact |
|---|---|
| `MFTECmd`, MFT Explorer | `$MFT`, `$J` |
| Timeline Explorer | CSV timeline |
| `EvtxECmd` | Windows EVTX |
| Registry Explorer | Registry hives |
| `PECmd` | Prefetch |
| `AmcacheParser` | Amcache |
| `AppCompatCacheParser` | ShimCache |
| `JLECmd` | Jump Lists |
| `LECmd` | LNK files |
| `RBCmd` | Recycle Bin |
| `SrumECmd` | SRUM |

Tải bộ công cụ:

```powershell
.\Get-ZimmermanTools.ps1 -NetVersion 6
```

## 2. MAC(b) Timestamps

```text
M = Modified
A = Accessed
C = MFT Record Changed
b = Birth/Created
```

Timestamps nằm trong:

```text
$STANDARD_INFORMATION
$FILE_NAME
```

Windows Explorer thường hiển thị thời gian từ `$STANDARD_INFORMATION`.

## 3. Timestomping

```text
MITRE ATT&CK: T1070.006
```

Phát hiện bằng cách so sánh:

```text
$STANDARD_INFORMATION
vs
$FILE_NAME
```

Nếu hai timestamp tạo file lệch nhau đáng kể, có thể nghi timestomping.

```powershell
.\MFTECmd.exe -f "D:\$MFT" --de 0x16169
```

## 4. Master File Table — `$MFT`

`$MFT` là database metadata của NTFS. Mỗi file hoặc thư mục có một record.

Có thể cung cấp:

```text
File name
Parent path
MACB timestamps
File size
Attributes
Deleted/free state
Resident/non-resident data
Zone.Identifier
```

MFT record thường gồm:

```text
File Record Header
$STANDARD_INFORMATION
$FILE_NAME
$DATA
Additional Attributes
```

### Resident và Non-Resident

```text
Resident     = dữ liệu nằm trong MFT record
Non-resident = dữ liệu nằm trong disk clusters
```

## 5. Zone.Identifier Và MotW

`Zone.Identifier` là NTFS Alternate Data Stream ghi nguồn tải file.

```powershell
Get-Item * -Stream Zone.Identifier
Get-Content * -Stream Zone.Identifier
```

Field quan trọng:

```text
ZoneId
ReferrerUrl
HostUrl
```

```text
ZoneId=3 → Internet Zone
```

Giá trị điều tra:

```text
Download source
Referrer URL
Host URL
Mark of the Web
```

## 6. Timeline Explorer

Workflow:

```text
Parse artifact thành CSV
→ mở bằng Timeline Explorer
→ lọc timestamp, entry, keyword
→ correlate nhiều nguồn
```

Dùng để dựng chuỗi:

```text
download → execution → persistence
```

## 7. USN Journal

USN Journal ghi thay đổi file/thư mục trên NTFS.

```text
$Extend\$UsnJrnl:$J
```

Sự kiện quan trọng:

```text
FileCreate
RenameOldName
RenameNewName
FileDelete
DataOverwrite
DataTruncation
SecurityChange
```

Parse:

```powershell
.\MFTECmd.exe `
  -f "D:\$Extend\$J" `
  --csv C:\Analysis `
  --csvf MFT-J.csv
```

Nhớ:

```text
MFT = trạng thái file
USN = lịch sử thay đổi file
```

## 8. Windows Event Logs Và EvtxECmd

EVTX nằm tại:

```text
Windows\System32\winevt\Logs
```

Log thường dùng:

```text
Security
System
Application
Sysmon
PowerShell
Defender
Task Scheduler
```

Cập nhật maps:

```powershell
.\EvtxECmd.exe --sync
```

Convert EVTX:

```powershell
.\EvtxECmd.exe `
  -f "Microsoft-Windows-Sysmon%4Operational.evtx" `
  --csv C:\EventLogs `
  --csvf sysmon.csv
```

Tùy chọn:

```text
-f / -d = file hoặc directory
--csv   = CSV output
--json  = JSON output
--inc   = include Event IDs
--exc   = exclude Event IDs
--sd    = start date
--ed    = end date
```

Maps chuẩn hóa field:

```text
UserName
ExecutableInfo
PayloadData1..6
RemoteHost
```

## 9. EQL — Event Query Language

```cmd
pip install eql
```

Chuyển Sysmon EVTX sang JSON:

```powershell
Import-Module .\scrape-events.ps1

Get-WinEvent -Path sysmon.evtx -Oldest |
Get-EventProps |
ConvertTo-Json |
Out-File -Encoding ASCII eql-sysmon.json
```

Ví dụ hunt user/group enumeration:

```cmd
eql query -f eql-sysmon.json ^
"EventId=1 and (Image='*net.exe' and wildcard(CommandLine, '* user*', '*localgroup *', '*group *'))"
```

Keyword:

```text
net users
net localgroup
net group
```

## 10. Registry Analysis

System hives:

```text
SYSTEM
SOFTWARE
SAM
SECURITY
DEFAULT
```

User hives:

```text
NTUSER.DAT
UsrClass.dat
```

### Registry Explorer

Dùng để browse, search, filter, xem LastWrite và bookmark key.

### RegRipper

```powershell
.\rip.exe -r HIVE -p PLUGIN
```

Plugin quan trọng:

```text
compname
timezone
nic2 / ips
installer
recentdocs
run
bam
userassist
```

Liệt kê plugin:

```powershell
.\rip.exe -l -c > rip_plugins.csv
```

## 11. Registry Persistence

Run key:

```text
HKCU\Software\Microsoft\Windows\CurrentVersion\Run
```

Ví dụ đáng ngờ:

```text
DiscordUpdate
→ C:\Windows\Tasks\update.exe
```

Kiểm tra:

```text
Value name
Executable path
LastWrite time
Hash/signature
File existence
```

## 12. Execution Artifacts

```text
Prefetch
ShimCache
Amcache
BAM
```

| Artifact | Giá trị chính |
|---|---|
| Prefetch | Program đã chạy, run count, run time |
| ShimCache | File/application trace; cần diễn giải thận trọng |
| Amcache | Path, hash, metadata, install/execution context |
| BAM | Execution theo user SID và timestamp |

## 13. Prefetch Và PECmd

Đường dẫn:

```text
C:\Windows\Prefetch
```

Thông tin:

```text
Executable name
Run count
Last run times
Volume
Referenced directories
Referenced files
```

Phân tích một file:

```powershell
.\PECmd.exe -f "DISCORD.EXE-7191FAD6.pf"
```

Parse cả thư mục:

```powershell
.\PECmd.exe `
  -d "D:\Windows\Prefetch" `
  --csv C:\PrefetchAnalysis
```

Dấu hiệu đáng ngờ:

```text
Executable chạy từ Temp/AppData/Public
Referenced install.bat
Referenced update.exe
Zone.Identifier
```

## 14. ShimCache — AppCompatCache

```text
HKLM\SYSTEM\ControlSet001\Control\Session Manager\AppCompatCache
```

Có thể chứa:

```text
Full path
Last modified time
ShimCache update time
Cache position
Execution-related flag
```

Công cụ:

```text
Registry Explorer
AppCompatCacheParser
RegRipper
```

## 15. Amcache

```text
C:\Windows\AppCompat\Programs\Amcache.hve
```

Có thể chứa:

```text
Execution path
File hash
Install information
First seen
Deletion time
Publisher/signature metadata
```

Parse:

```powershell
.\AmcacheParser.exe `
  -f "Amcache.hve" `
  --csv C:\AmcacheAnalysis
```

## 16. BAM

```text
HKLM\SYSTEM\ControlSet001\Services\bam\State\UserSettings\{USER-SID}
```

BAM hỗ trợ xác định:

```text
Executable path
User SID
Execution timestamp
```

## 17. API Monitor Và `.apmx64`

`.apmx64` chứa API call capture.

Có thể xem:

```text
Function
Parameters
Return value
Timestamp
Call stack
Process/module
```

### Registry persistence APIs

```text
RegOpenKeyExA
RegSetValueExA
```

### Process injection APIs

```text
CreateProcessA
OpenProcess
VirtualAllocEx
WriteProcessMemory
CreateRemoteThread
ResumeThread
```

Chuỗi đáng ngờ:

```text
CREATE_SUSPENDED
+ VirtualAllocEx
+ WriteProcessMemory
+ CreateRemoteThread
→ process injection
```

## 18. PowerShell Activity

Nguồn:

```text
PowerShell transcripts
Event ID 4104
Event ID 4103
Sysmon
Console history
```

Dấu hiệu hunt:

```text
Invoke-WebRequest
EncodedCommand
Base64
Unsigned scripts
Registry modification
Scheduled tasks
User creation
Privilege escalation
Network connections
Uncommon modules
```

## 19. Workflow Chuẩn

```text
1. Parse $MFT và USN Journal
2. So sánh $STANDARD_INFORMATION và $FILE_NAME
3. Kiểm tra Zone.Identifier
4. Parse EVTX bằng EvtxECmd
5. Query Sysmon bằng EQL
6. Phân tích Registry bằng Registry Explorer/RegRipper
7. Parse Prefetch bằng PECmd
8. Kiểm tra ShimCache, Amcache và BAM
9. Phân tích API Monitor
10. Kiểm tra PowerShell activity
11. Import CSV vào Timeline Explorer
12. Correlate download → execution → persistence
```

## Keyword Cần Nhớ

```text
MFTECmd
MFT Explorer
Timeline Explorer
MACB
$STANDARD_INFORMATION
$FILE_NAME
Timestomping
T1070.006
$MFT
Resident
Non-Resident
Zone.Identifier
MotW
USN Journal
EvtxECmd
Maps
EQL
Registry Explorer
RegRipper
Prefetch
PECmd
ShimCache
Amcache
BAM
API Monitor
RegSetValueExA
VirtualAllocEx
WriteProcessMemory
CreateRemoteThread
PowerShell Transcript
```

## Key Takeaway

```text
MFT/USN       → file activity
Zone.Identifier → download source
EVTX/EQL      → system and attack activity
Registry      → persistence
Prefetch      → execution
ShimCache     → application traces
Amcache       → executable metadata
BAM           → user-linked execution
API Monitor   → low-level behavior
Timeline      → reconstruct the incident
```

> Chỉ phân tích evidence từ HTB, lab hoặc hệ thống được cấp phép; luôn bảo toàn bản gốc và ghi chain of custody.

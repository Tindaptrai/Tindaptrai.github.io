---
layout: post
title: "Detecting Common User & Domain Recon"
date: 2026-08-03 23:50:37 +0700
categories: ["Detecting Windows Attacks with Splunk", "Active Directory Reconnaissance"]
tags: [active-directory, domain-recon, user-recon, bloodhound, sharphound, ldap, silketw, silkservice, sysmon, splunk, threat-hunting, detection-engineering, cdsa]
description: "Tóm tắt cách phát hiện reconnaissance trong Active Directory bằng Sysmon, LDAP ETW, SilkService và Splunk."
toc: true
---

# Detecting Common User & Domain Recon

## Công thức dễ nhớ

```text
COMMAND RECON → LDAP RECON → AGGREGATE → THRESHOLD → INVESTIGATE
```

Domain reconnaissance là giai đoạn attacker thu thập thông tin về Active Directory để tìm:

```text
Domain Controllers
Users
Groups
Domain Admins
Trust relationships
Organizational Units
Group Policies
High-value targets
Attack paths
```

# 1. Native Windows Recon

Các command thường gặp:

```cmd
whoami /all
wmic computersystem get domain
net user /domain
net group "Domain Admins" /domain
arp -a
nltest /domain_trusts
```

Tool/process cần theo dõi:

```text
arp.exe
chcp.com
ipconfig.exe
net.exe
net1.exe
nltest.exe
ping.exe
systeminfo.exe
whoami.exe
cmd.exe
powershell.exe
```

Một command riêng lẻ có thể hợp lệ. Dấu hiệu mạnh hơn là:

```text
Nhiều command recon
+ cùng parent process
+ cùng user/host
+ trong thời gian ngắn
```

# 2. Detect Native Recon Với Sysmon + Splunk

Nguồn log:

```text
Sysmon Event ID 1 = Process Creation
```

```spl
index=main
source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational"
EventID=1
earliest=1690447949 latest=1690450687
| search process_name IN (
    arp.exe,
    chcp.com,
    ipconfig.exe,
    net.exe,
    net1.exe,
    nltest.exe,
    ping.exe,
    systeminfo.exe,
    whoami.exe
  )
  OR (
    process_name IN (cmd.exe,powershell.exe)
    AND process IN (
      *arp*,
      *chcp*,
      *ipconfig*,
      *net*,
      *net1*,
      *nltest*,
      *ping*,
      *systeminfo*,
      *whoami*
    )
  )
| stats values(process) AS process,
        min(_time) AS _time
  BY parent_process,
     parent_process_id,
     dest,
     user
| where mvcount(process) > 3
```

Logic:

```text
Lọc process creation
→ tìm command recon
→ gom theo parent + host + user
→ chỉ giữ nhóm có trên 3 command khác nhau
```

Field quan trọng:

```text
process_name
process
parent_process
parent_process_id
dest
user
_time
```

Dấu hiệu đáng chú ý trong ví dụ:

```text
arp
ipconfig
net group
whoami
```

cùng được sinh từ:

```text
rundll32.exe
```

# 3. BloodHound Và SharpHound

## BloodHound

BloodHound dùng graph theory để mô hình hóa:

```text
Users
Groups
Computers
Sessions
ACLs
Trusts
Permissions
Attack paths
```

## SharpHound

SharpHound là C# collector cho BloodHound.

```cmd
SharpHound.exe -c all
```

```text
-c all = thu thập nhiều loại dữ liệu AD
```

Collector gửi nhiều LDAP query tới Domain Controller để lấy:

```text
User objects
Computer objects
Group memberships
ACLs
GPO data
Trust relationships
Sessions
```

# 4. Vì Sao LDAP Recon Khó Phát Hiện?

Windows mặc định không log đầy đủ LDAP query.

Một lựa chọn là:

```text
Event ID 1644
LDAP Performance Monitoring
```

Hạn chế:

```text
Không bật mặc định
Có thể tạo nhiều log
BloodHound không phải lúc nào cũng tạo đủ event mong đợi
```

# 5. LDAP ETW Với SilkETW/SilkService

ETW provider:

```text
Microsoft-Windows-LDAP-Client
```

Tool:

```text
SilkETW
SilkService
```

Workflow:

```text
Capture LDAP client events
→ ghi vào Windows Event Log
→ ingest vào Splunk
→ hunt bằng SearchFilter
```

Field hữu ích:

```text
ComputerName
ProcessName
ProcessId
DistinguishedName
SearchFilter
```

SilkService còn có thể áp dụng YARA rule lên LDAP query đáng ngờ.

# 6. LDAP Filter Detection

Recon tools thường dùng LDAP filter để tìm:

```text
Users
Computers
Groups
ManagedBy relationships
OUs
DFS shares
Account properties
```

Ví dụ filter:

```text
samAccountType=805306368
```

Detection tốt nên kết hợp:

```text
LDAP filter
+ process
+ query count
+ time window
+ target DN
```

# 7. Detect SharpHound Với Splunk

Nguồn:

```text
WinEventLog:SilkService-Log
```

```spl
index=main
earliest=1690195896 latest=1690285475
source="WinEventLog:SilkService-Log"
| spath input=Message
| rename XmlEventData.* AS *
| table _time,
        ComputerName,
        ProcessName,
        ProcessId,
        DistinguishedName,
        SearchFilter
| sort 0 _time
| search SearchFilter="*(samAccountType=805306368)*"
| stats min(_time) AS _time,
        max(_time) AS maxTime,
        count,
        values(SearchFilter) AS SearchFilter
  BY ComputerName,
     ProcessName,
     ProcessId
| where count > 10
| convert ctime(maxTime)
```

Logic:

```text
Đọc SilkService log
→ parse Message bằng spath
→ chuẩn hóa XmlEventData fields
→ lọc LDAP user query
→ gom theo host/process/PID
→ cảnh báo nếu query count > 10
```

Kết quả ví dụ:

```text
ProcessName: SharpHound
ProcessId: 8704
Count: 259 LDAP events
```

# 8. Ý Nghĩa Các SPL Command

| Command | Vai trò |
|---|---|
| `spath` | Parse XML/JSON trong `Message` |
| `rename` | Chuẩn hóa field |
| `table` | Chọn cột cần điều tra |
| `sort 0 _time` | Sắp xếp toàn bộ theo thời gian |
| `stats` | Gom và đếm hành vi |
| `where` | Áp threshold |
| `mvcount` | Đếm giá trị trong multivalue field |
| `convert ctime()` | Đổi Unix time sang dạng đọc được |

# 9. Tuning Và False Positives

Native recon có thể xuất hiện khi:

```text
System administrator troubleshooting
Helpdesk diagnostics
Login scripts
Inventory tools
Monitoring tools
Vulnerability scanners
```

LDAP query volume có thể đến từ:

```text
Identity management
Asset inventory
AD administration tools
Security scanners
Backup software
```

Nên tune theo:

```text
Approved admin accounts
Known management hosts
Expected parent processes
Known LDAP-heavy applications
Query count baseline
Time window
Command diversity
```

# 10. Investigation Workflow

```text
1. Xác định user và host
2. Kiểm tra parent process
3. Xem toàn bộ command line
4. Dựng process tree
5. Đếm số command recon
6. Kiểm tra LDAP filters
7. Xác định SharpHound/BloodHound artifacts
8. Correlate kết nối tới Domain Controller
9. Hunt privilege escalation/lateral movement tiếp theo
10. Tune allowlist và threshold
```

Các bước hunt tiếp theo:

```text
Kerberoasting
AS-REP Roasting
Credential dumping
PsExec
WMI
RDP
SMB lateral movement
```

# 11. Detection Summary

| Detection | Telemetry | Logic |
|---|---|---|
| Native command recon | Sysmon Event ID 1 | Nhiều command cùng parent/user/host |
| BloodHound/SharpHound | LDAP ETW qua SilkService | LDAP filter đặc trưng + query volume cao |
| Event 1644 | Directory Service log | LDAP search performance details |
| YARA over ETW | SilkETW/SilkService | Match LDAP query pattern |

# Keyword Cần Nhớ

```text
Domain Reconnaissance
Active Directory
Domain Admins
whoami /all
net user /domain
net group
nltest /domain_trusts
BloodHound
SharpHound
-c all
LDAP
Event ID 1644
Microsoft-Windows-LDAP-Client
ETW
SilkETW
SilkService
YARA
Sysmon Event ID 1
Process Creation
SearchFilter
DistinguishedName
samAccountType=805306368
spath
stats
mvcount
threshold
parent_process
```

# Key Takeaway

```text
Native recon:
nhiều command + cùng parent + cùng host/user
```

```text
BloodHound:
LDAP filter đặc trưng + query volume cao + process context
```

```text
Detection tốt =
telemetry đúng
+ aggregation
+ threshold
+ process context
+ baseline
```

> Chỉ thực hành trên HTB, lab hoặc hệ thống được cấp phép.

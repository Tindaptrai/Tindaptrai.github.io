---
layout: post
title: "Detecting Unconstrained & Constrained Delegation Attacks"
date: 2026-08-06 01:02:52 +0700
categories: ["Detecting Windows Attacks with Splunk", "Kerberos Delegation"]
tags: [unconstrained-delegation, constrained-delegation, kerberos, s4u2self, s4u2proxy, trustedfordelegation, msds-allowedtodelegateto, rubeus, event-id-4104, sysmon-event-id-3, port-88, splunk, detection-engineering, cdsa]
description: "Phát hiện lạm dụng Unconstrained và Constrained Delegation bằng PowerShell Script Block Logging, LDAP indicators, Sysmon và Splunk."
toc: true
---

# Detecting Unconstrained & Constrained Delegation Attacks

## Công thức dễ nhớ

```text
UNCONSTRAINED:
FIND TRUSTED HOST → COMPROMISE HOST → STEAL USER TGT → PASS THE TICKET

CONSTRAINED:
FIND ALLOWED SPNs → OBTAIN PRINCIPAL TGT → S4U2SELF → S4U2PROXY → IMPERSONATE USER
```

Kerberos Delegation cho phép một service xác thực tới service khác thay mặt người dùng. Khi cấu hình sai hoặc principal bị compromise, attacker có thể giả mạo user và di chuyển ngang.

---

# 1. Unconstrained Delegation Là Gì?

Unconstrained Delegation có thể được bật trên:

```text
User account
Computer account
Service account
```

Ý nghĩa:

```text
Service được tin cậy để delegate tới bất kỳ Kerberos service nào.
```

Trong AD thường liên quan đến flag:

```text
TRUSTED_FOR_DELEGATION
```

Giá trị `userAccountControl`:

```text
524288
```

---

# 2. Kerberos Flow Với Unconstrained Delegation

Trong luồng Kerberos thông thường:

```text
Client xin TGT
→ xin TGS cho service
→ gửi TGS tới service
```

Khi Unconstrained Delegation được bật:

```text
KDC nhúng TGT của user vào service ticket
→ user gửi TGS kèm delegated TGT tới service
→ service có thể dùng TGT đó để xác thực tới service khác
```

Điểm nguy hiểm:

```text
TGT của user được cache trong memory trên delegated host.
```

Nếu Domain Admin đăng nhập vào host này:

```text
Attacker có thể trích xuất Domain Admin TGT
→ reuse ticket
→ Pass-the-Ticket
```

---

# 3. Unconstrained Delegation Attack Flow

```text
1. Enumerate account có TRUSTED_FOR_DELEGATION
2. Compromise delegated host
3. Chờ privileged user authenticate
4. Extract cached TGT khỏi memory
5. Import TGT
6. Truy cập tài nguyên khác bằng identity bị đánh cắp
```

Tool thường gặp:

```text
Rubeus
Mimikatz
PowerView
ActiveDirectory PowerShell module
```

---

# 4. Enumeration Indicators

Attacker có thể dùng PowerShell hoặc LDAP để tìm account có Unconstrained Delegation.

Keyword PowerShell:

```text
TrustedForDelegation
```

LDAP bitwise filter:

```text
userAccountControl:1.2.840.113556.1.4.803:=524288
```

Giải thích:

```text
1.2.840.113556.1.4.803
= LDAP bitwise AND matching rule

524288
= TRUSTED_FOR_DELEGATION
```

---

# 5. Detect Unconstrained Delegation Enumeration Với Splunk

Nguồn:

```text
Microsoft-Windows-PowerShell/Operational
Event ID 4104
```

SPL:

```spl
index=main
earliest=1690544538 latest=1690544540
source="WinEventLog:Microsoft-Windows-PowerShell/Operational"
EventCode=4104
(
  Message="*TrustedForDelegation*"
  OR
  Message="*userAccountControl:1.2.840.113556.1.4.803:=524288*"
)
| table _time,
        ComputerName,
        EventCode,
        Message
```

Logic:

```text
PowerShell Script Block Logging
→ tìm property hoặc LDAP filter liên quan delegation
→ xác định host và script thực hiện enumeration
```

---

# 6. Event ID 4104

```text
Event ID 4104
= PowerShell Script Block Logging
```

Có thể ghi lại:

```text
PowerShell cmdlet
LDAP filters
Encoded/deobfuscated script blocks
AD enumeration commands
Rubeus/PowerView-related scripts
```

Field quan trọng:

```text
_time
ComputerName
EventCode
Message
```

---

# 7. Detection Opportunities Cho Unconstrained Delegation

Có thể phát hiện theo ba giai đoạn:

## Enumeration

```text
4104
LDAP query
TrustedForDelegation
UAC 524288
```

## Credential Theft

```text
LSASS access
Rubeus dump/monitor
Mimikatz execution
Ticket extraction
```

## Ticket Reuse

```text
Pass-the-Ticket detection
4769/4770 không có 4768 hợp lý
Remote service access
Privileged logon từ host bất thường
```

---

# 8. Constrained Delegation Là Gì?

Constrained Delegation giới hạn service chỉ được delegate tới các SPN cụ thể.

Thuộc tính quan trọng:

```text
msDS-AllowedToDelegateTo
```

Ví dụ:

```text
CIFS/dc.lab.internal.local
LDAP/dc.lab.internal.local
```

Ý nghĩa:

```text
Principal chỉ được giả mạo user tới các service đã được liệt kê.
```

Phạm vi hẹp hơn Unconstrained Delegation, nhưng nếu principal bị compromise thì attacker vẫn có thể impersonate user có đặc quyền cao.

---

# 9. Constrained Delegation Attack Flow

```text
1. Enumerate account có msDS-AllowedToDelegateTo
2. Xác định SPN được phép delegate
3. Obtain TGT của delegated principal
4. Dùng S4U2self để impersonate user
5. Dùng S4U2proxy để xin TGS tới target SPN
6. Inject TGS
7. Truy cập service với identity bị giả mạo
```

Credential material để lấy principal TGT:

```text
Cached TGT
NTLM hash
AES key
```

---

# 10. S4U2self

```text
Service for User to Self
```

Cho phép service xin TGS cho chính nó thay mặt một user.

Điểm quan trọng:

```text
Có thể yêu cầu thay mặt user bất kỳ,
ví dụ Administrator.
```

Luồng:

```text
User xác thực với Service A bằng phương thức khác
→ Service A xin TGS cho chính nó thay mặt user
```

---

# 11. S4U2proxy

```text
Service for User to Proxy
```

Cho phép service dùng ticket từ S4U2self để xin TGS tới service thứ hai.

Target bị giới hạn bởi:

```text
msDS-AllowedToDelegateTo
```

Chuỗi tấn công:

```text
S4U2self
→ impersonate privileged user
→ S4U2proxy
→ request TGS cho CIFS/LDAP/HTTP target
```

---

# 12. Detect Constrained Delegation Enumeration Với Splunk

```spl
index=main
earliest=1690544553 latest=1690562556
source="WinEventLog:Microsoft-Windows-PowerShell/Operational"
EventCode=4104
Message="*msDS-AllowedToDelegateTo*"
| table _time,
        ComputerName,
        EventCode,
        Message
```

Logic:

```text
4104
+ msDS-AllowedToDelegateTo
→ có thể là AD delegation enumeration
```

Cần kiểm tra:

```text
User chạy script
Parent process
PowerShell command line
Host role
Có phải admin tooling hợp lệ không
```

---

# 13. Kerberos Port 88 Detection

Để xin:

```text
TGT
S4U2self TGS
S4U2proxy TGS
```

Rubeus phải giao tiếp với KDC qua:

```text
TCP/UDP 88
```

Process bất thường kết nối port 88 là detection opportunity quan trọng.

---

# 14. Detect Constrained Delegation Abuse Với Sysmon + Splunk

SPL từ module:

```spl
index=main
earliest=1690562367 latest=1690562556
source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational"
| eventstats values(process) AS process BY process_id
| where EventCode=3 AND dest_port=88
| table _time,
        Computer,
        dest_ip,
        dest_port,
        Image,
        process
```

Logic:

```text
Event ID 1 cung cấp process/command line
+ Event ID 3 cung cấp network connection
→ correlate bằng process_id
→ giữ kết nối tới KDC port 88
```

---

# 15. Lưu Ý Về Query Sysmon

Query trên dùng:

```spl
| eventstats values(process) AS process BY process_id
```

Để enrich Event ID 3 bằng command line, search cần có cả:

```text
Event ID 1
Event ID 3
```

Phiên bản rõ ràng hơn:

```spl
index=main
source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational"
(EventCode=1 OR EventCode=3)
| eventstats values(process) AS process BY Computer, process_id
| where EventCode=3 AND dest_port=88
| table _time,
        Computer,
        user,
        dest_ip,
        dest_port,
        Image,
        process_id,
        process
```

Nên correlate bằng:

```text
Computer + process_id
```

để tránh PID trùng giữa các host.

---

# 16. Dấu Hiệu Command Line Đáng Ngờ

Keyword Rubeus:

```text
s4u
asktgt
ptt
impersonateuser
msdsspn
altservice
ticket
rc4
aes256
```

Ví dụ hành vi:

```text
Rubeus s4u
/user:backup
/impersonateuser:Administrator
/msdsspn:cifs/dc.corp.local
/ptt
```

Không nên chỉ dựa vào tên `Rubeus.exe` vì binary có thể bị rename.

---

# 17. Detection Summary

| Kỹ thuật | Enumeration | Abuse telemetry |
|---|---|---|
| Unconstrained Delegation | `TrustedForDelegation`, UAC `524288` | Ticket extraction + Pass-the-Ticket |
| Constrained Delegation | `msDS-AllowedToDelegateTo` | Unusual process → KDC port `88`, S4U activity |

---

# 18. False Positives

PowerShell enumeration có thể hợp lệ từ:

```text
Domain administrators
Identity engineers
AD auditing
Security scanners
Configuration management
Authorized red team
```

Port 88 từ process khác `lsass.exe` có thể hợp lệ với:

```text
Java Kerberos clients
SSO agents
Custom applications
Security products
Administrative tooling
```

Tune theo:

```text
Signed process
Known application
Parent process
User
Host role
Destination DC
Command line
Business hours
Delegation inventory
```

---

# 19. Investigation Workflow

```text
1. Xác định host và user thực hiện enumeration
2. Xem script block 4104 đầy đủ
3. Kiểm tra account có delegation thật hay không
4. Xác định SPN trong msDS-AllowedToDelegateTo
5. Hunt Rubeus/Mimikatz execution
6. Kiểm tra process kết nối DC port 88
7. Correlate 4768 và 4769 trên DC
8. Kiểm tra TGT/TGS import hoặc cache
9. Hunt access tới CIFS/LDAP/HTTP target
10. Contain host và rotate delegated principal secret
```

---

# 20. Hunt Tiếp Sau Delegation Abuse

```text
Pass-the-Ticket
Remote SMB
LDAP access
DCSync
PsExec
WMI
WinRM
Remote service creation
Scheduled task
Privileged logon
```

---

# 21. Mitigation

## Unconstrained Delegation

```text
Loại bỏ nếu không thực sự cần
Dùng Constrained Delegation
Đánh dấu privileged users là sensitive and cannot be delegated
Dùng Protected Users
Hạn chế admin logon lên delegated hosts
Monitor delegation configuration
```

## Constrained Delegation

```text
Giới hạn SPN tối thiểu cần thiết
Bảo vệ delegated principal secret
Dùng gMSA khi phù hợp
Review msDS-AllowedToDelegateTo định kỳ
Monitor S4U activity và port 88 process context
```

---

# Keyword Cần Nhớ

```text
Unconstrained Delegation
Constrained Delegation
TrustedForDelegation
TRUSTED_FOR_DELEGATION
524288
userAccountControl
1.2.840.113556.1.4.803
msDS-AllowedToDelegateTo
S4U
S4U2self
S4U2proxy
Rubeus
Mimikatz
TGT
TGS
KDC
PowerShell Event ID 4104
Sysmon Event ID 1
Sysmon Event ID 3
TCP/UDP 88
eventstats
process_id
Pass-the-Ticket
```

---

# Key Takeaway

```text
Unconstrained Delegation:
dịch vụ giữ delegated TGT của user
→ compromise host có thể dẫn tới đánh cắp TGT
```

```text
Constrained Delegation:
principal chỉ delegate tới SPN được chỉ định
→ attacker abuse S4U2self + S4U2proxy
```

```text
Detection tốt =
delegation enumeration
+ process context
+ KDC network activity
+ Kerberos ticket correlation
```

> Chỉ thực hành trên HTB, lab hoặc hệ thống được cấp phép.

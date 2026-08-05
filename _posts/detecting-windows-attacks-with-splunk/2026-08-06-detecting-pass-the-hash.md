---
layout: post
title: "Detecting Pass-the-Hash"
date: 2026-08-06 00:30:08 +0700
categories: ["Detecting Windows Attacks with Splunk", "Credential Access"]
tags: [pass-the-hash, pth, ntlm, mimikatz, lsass, access-token, logon-session, runas, netonly, event-id-4624, sysmon-event-id-10, splunk, lateral-movement, detection-engineering, cdsa]
description: "Phát hiện Pass-the-Hash bằng Logon Type 9, Logon Process seclogo, truy cập LSASS và correlation trong Splunk."
toc: true
---

# Detecting Pass-the-Hash

## Công thức nhớ nhanh

```text
NTLM HASH
→ LSASS CREDENTIAL MATERIAL MODIFICATION
→ LOGON TYPE 9 / SECLOGO
→ REMOTE AUTHENTICATION
→ LATERAL MOVEMENT
```

Pass-the-Hash cho phép attacker xác thực bằng **NTLM hash** thay vì mật khẩu plaintext.

---

# 1. Pass-the-Hash Là Gì?

Chuỗi tấn công điển hình:

```text
1. Compromise một Windows host
2. Có local administrator/SYSTEM privilege
3. Trích xuất NTLM hash từ LSASS
4. Gắn hash vào logon session mới
5. Truy cập host hoặc network resource khác
6. Di chuyển ngang trong domain
```

Tool thường gặp:

```text
Mimikatz
sekurlsa::logonpasswords
sekurlsa::pth
Impacket
CrackMapExec / NetExec
```

Ví dụ mục tiêu:

```text
\\dc01\c$
ADMIN$
SMB
WMI
Remote Service
```

---

# 2. NTLM Hash Và Điều Kiện Tấn Công

Attacker không cần biết plaintext password nếu có:

```text
Username
Domain
NTLM hash
```

Để trích xuất hoặc sửa credential material trong LSASS, attacker thường cần:

```text
Local Administrator
SeDebugPrivilege
SYSTEM
```

Điểm quan trọng:

```text
PtH không crack hash.
PtH tái sử dụng hash để xác thực.
```

---

# 3. Windows Access Token

Access token mô tả security context của process hoặc thread.

Nó chứa:

```text
User SID
Group SIDs
Privileges
Integrity level
Authentication ID
Token type
```

Khi user đăng nhập thành công:

```text
LSASS tạo logon session
→ Windows tạo access token
→ process của user nhận bản sao token
```

---

# 4. Logon Session Và Alternate Credentials

Logon session lưu context phục vụ authentication:

```text
Username
Domain
Authentication ID / LUID
Credential material
NTLM/LM-related secrets
Kerberos tickets
```

Alternate credentials cho phép process dùng credential khác khi truy cập tài nguyên.

Ví dụ:

```cmd
runas /user:CORP\Administrator cmd.exe
```

Lệnh này tạo security context cho user khác.

---

# 5. `runas /netonly`

```cmd
runas /netonly /user:CORP\Administrator cmd.exe
```

Ý nghĩa:

```text
Local identity     = vẫn là user ban đầu
Remote credentials = dùng account được chỉ định
```

Vì vậy:

```cmd
whoami
```

có thể vẫn trả về user cũ, nhưng process vẫn truy cập được tài nguyên mạng bằng alternate credential.

Điểm cần nhớ:

```text
/netonly thay đổi network logon session,
không thay đổi local access token identity.
```

---

# 6. Event ID 4624 Và Logon Types

| Trường hợp | Event | Logon Type |
|---|---:|---:|
| `runas` thông thường | `4624` | `2` — Interactive |
| `runas /netonly` | `4624` | `9` — NewCredentials |
| Pass-the-Hash kiểu Mimikatz | thường thấy `4624` | `9` — NewCredentials |

Field quan trọng:

```text
EventCode
Logon_Type
Logon_Process
user
Network_Account_Name
Network_Account_Domain
ComputerName
```

---

# 7. `Logon_Process=seclogo`

Detection cơ bản:

```text
Event ID 4624
+ Logon Type 9
+ Logon Process seclogo
```

SPL:

```spl
index=main
earliest=1690450708 latest=1690451116
source="WinEventLog:Security"
EventCode=4624
Logon_Type=9
Logon_Process=seclogo
| table _time,
        ComputerName,
        EventCode,
        user,
        Network_Account_Domain,
        Network_Account_Name,
        Logon_Type,
        Logon_Process
```

Ví dụ trong bài:

```text
Computer: ORANGE.corp.local
Local user: SYSTEM
Network account: RAUL_LYNN
Logon Type: 9
Logon Process: seclogo
```

---

# 8. Vì Sao Logon Type 9 Chưa Đủ?

`runas /netonly` hợp lệ cũng tạo:

```text
4624
Logon_Type=9
Logon_Process=seclogo
```

False positives có thể đến từ:

```text
System administrators
Helpdesk
Deployment scripts
Backup software
Management agents
Scheduled automation
```

Do đó cần thêm hành vi truy cập LSASS.

---

# 9. Sysmon Event ID 10 — Process Access

```text
Sysmon Event ID 10
= Process Access
```

Dùng để phát hiện process mở handle tới:

```text
C:\Windows\System32\lsass.exe
```

Field quan trọng:

```text
SourceImage
SourceProcessId
TargetImage
GrantedAccess
CallTrace
Computer
```

Dấu hiệu đáng chú ý:

```text
Unexpected process
→ accesses lsass.exe
→ shortly before Logon Type 9
```

---

# 10. Correlate LSASS Access Với NewCredentials Logon

```spl
index=main
earliest=1690450689 latest=1690451116
(
  source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational"
  EventCode=10
  TargetImage="C:\Windows\system32\lsass.exe"
  SourceImage!="C:\ProgramData\Microsoft\Windows Defender\platform\*\MsMpEng.exe"
)
OR
(
  source="WinEventLog:Security"
  EventCode=4624
  Logon_Type=9
  Logon_Process=seclogo
)
| sort _time, RecordNumber
| transaction host
    maxspan=1m
    startswith=(EventCode=10)
    endswith=(EventCode=4624)
| stats count
  BY _time,
     Computer,
     SourceImage,
     SourceProcessId,
     Network_Account_Domain,
     Network_Account_Name,
     Logon_Type,
     Logon_Process
| fields - count
```

---

# 11. Query Logic

```text
1. Thu Sysmon Event 10 targeting LSASS
2. Loại Microsoft Defender MsMpEng.exe
3. Thu Security Event 4624 Type 9 / seclogo
4. Sort theo time và RecordNumber
5. Correlate trên cùng host
6. Giới hạn maxspan=1 phút
7. Hiển thị source process và network account
```

Ví dụ detection:

```text
SourceImage:
C:\Windows\System32\rundll32.exe

SourceProcessId:
4596

Network account:
RAUL_LYNN
```

`rundll32.exe` truy cập LSASS rồi xuất hiện Type 9 logon là dấu hiệu có độ nghi ngờ cao.

---

# 12. Ý Nghĩa Các SPL Command

| SPL | Vai trò |
|---|---|
| `sort _time, RecordNumber` | Sắp xếp đúng thứ tự event |
| `transaction host` | Gom event theo host |
| `maxspan=1m` | Giới hạn cửa sổ correlation |
| `startswith` | Event bắt đầu là LSASS access |
| `endswith` | Event kết thúc là Type 9 logon |
| `stats ... BY` | Chuẩn hóa kết quả điều tra |
| `fields - count` | Bỏ field đếm không cần thiết |

---

# 13. Detection Confidence

## Low Confidence

```text
4624
+ Logon Type 9
+ seclogo
```

## Medium Confidence

```text
Type 9 / seclogo
+ source host bất thường
+ network account bất thường
```

## High Confidence

```text
Suspicious LSASS access
+ Type 9 / seclogo trong thời gian ngắn
+ remote SMB/WMI/service activity
```

---

# 14. Tuning Và False Positives

Allowlist có thể cần cho:

```text
MsMpEng.exe
EDR/AV processes
Credential Guard components
Backup agents
Identity/security software
Approved admin tools
```

Tune theo:

```text
SourceImage
GrantedAccess
CallTrace
Signer
Parent process
User
Host role
Known administration windows
Network account baseline
```

Không nên chỉ allowlist theo filename vì attacker có thể masquerade.

---

# 15. Detection Limitations

Logic này không bắt mọi biến thể PtH.

Các hạn chế:

```text
Không phải PtH nào cũng tạo cùng process chain
Sysmon Event 10 có thể rất noisy
Event 10 phụ thuộc Sysmon configuration
Attacker có thể dùng remote tooling khác
Logon Type 9 có nhiều hành vi hợp lệ
```

Cần correlate thêm:

```text
4624 Logon Type 3 trên máy đích
5140 / 5145 SMB share access
7045 service installation
4697 service creation
WMI activity
PsExec artifacts
Remote scheduled task
```

---

# 16. Investigation Workflow

```text
1. Xác định host tạo Type 9 logon
2. Kiểm tra Network_Account_Name/Domain
3. Xem SourceImage truy cập LSASS
4. Kiểm tra GrantedAccess và CallTrace
5. Dựng parent-child process tree
6. Correlate SMB/WMI/WinRM activity
7. Hunt 4624 Type 3 trên máy đích
8. Kiểm tra service/task được tạo từ xa
9. Hunt cùng NTLM account trên các host khác
10. Contain host và rotate credential
```

---

# 17. Hunt Tiếp Sau PtH

```text
Remote SMB access
ADMIN$ / C$
PsExec
WMI
WinRM
Remote service creation
Scheduled task
Credential dumping
Domain reconnaissance
DCSync
```

---

# Keyword Cần Nhớ

```text
Pass-the-Hash
PtH
NTLM
Mimikatz
LSASS
Access Token
Logon Session
Authentication ID
LUID
Alternate Credentials
runas
/netonly
4624
Logon Type 2
Logon Type 9
NewCredentials
seclogo
Sysmon Event ID 10
Process Access
TargetImage
SourceImage
GrantedAccess
CallTrace
transaction
maxspan
lateral movement
```

---

# Key Takeaway

```text
Logon Type 9 không đồng nghĩa Pass-the-Hash.
```

```text
Detection mạnh hơn =
LSASS access
+ 4624 Type 9 / seclogo
+ remote authentication activity
```

```text
PtH detection =
Credential Access telemetry
+ Authentication telemetry
+ Lateral Movement telemetry
```

> Chỉ thực hành trên HTB, lab hoặc hệ thống được cấp phép.

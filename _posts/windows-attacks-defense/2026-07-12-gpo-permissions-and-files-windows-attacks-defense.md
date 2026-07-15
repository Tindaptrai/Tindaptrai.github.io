---
layout: post
title: "GPO Permissions and GPO Files"
date: 2026-07-12 21:28:07 +0700
categories: ["Windows Attacks & Defense", "Windows Defense"]
tags: [active-directory, windows-attacks-defense, gpo, group-policy, sysvol, permissions, acl, event-5136, event-4725, honeypot, powershell, splunk, elk, sigma, threat-hunting, cdsa]
description: "Tóm tắt GPO Permissions and GPO Files: rủi ro quyền sửa GPO hoặc file được GPO triển khai, detection bằng Event ID 5136, honeypot GPO và phản ứng tự động với Event ID 4725."
toc: true
---

# GPO Permissions and GPO Files

Bài này thuộc module **Windows Attacks & Defense**, tập trung vào rủi ro phát sinh khi GPO hoặc file được GPO triển khai có quyền sửa đổi quá rộng.

```text
Mục tiêu: hiểu cách quyền GPO/NTFS sai có thể dẫn đến code execution trên nhiều máy, đồng thời xây detection và response dựa trên Event ID 5136 và 4725.
```

> Chỉ thực hành trong HTB, lab hoặc hệ thống có ủy quyền rõ ràng. Không thay đổi GPO hoặc file triển khai trong môi trường thật ngoài scope.

---

## Timeline cập nhật

```text
Created at: 2026-07-12 21:28:07 +0700
Timezone: Asia/Ho_Chi_Minh
Module: Windows Attacks & Defense
Section: GPO Permissions / GPO Files
Focus: GPO ACLs, SYSVOL files, Event ID 5136, honeypot GPO, automated response
```

---

# 1. GPO là gì?

**Group Policy Object (GPO)** là tập hợp các policy settings được áp dụng cho user và computer trong Active Directory.

GPO thường được liên kết với:

- Domain.
- Site.
- Organizational Unit.
- OU con.

GPO có thể được giới hạn bằng security filtering, AD groups hoặc WMI filters.

---

# 2. Vì sao quyền GPO nguy hiểm?

Rủi ro xuất hiện khi quyền sửa GPO được delegate sai cho:

```text
Authenticated Users
Domain Users
Low-privileged groups
Service accounts không cần thiết
```

Nếu attacker compromise một account có quyền sửa GPO, họ có thể:

- Thêm startup script.
- Thêm scheduled task.
- Thêm logon script.
- Thay đổi registry.
- Deploy executable.
- Tạo local admin.
- Tác động mọi computer/user trong OU liên kết.

```text
Một GPO bị sửa sai có thể biến một user compromise thành compromise hàng loạt.
```

---

# 3. GPO File Abuse

Ngay cả khi quyền GPO đúng, file được GPO tham chiếu vẫn có thể bị thay thế nếu NTFS/share permission sai.

Ví dụ:

```text
Startup script trên share writable.
Executable được triển khai từ thư mục mà nhiều user có quyền modify.
Scheduled task chạy file trên network share.
```

Attack flow:

```text
1. GPO hợp lệ trỏ tới một file.
2. File nằm trên share có quyền write quá rộng.
3. Attacker thay file bằng payload.
4. GPO tiếp tục chạy file dưới SYSTEM/user context.
```

---

# 4. Prevention

- Chỉ cho nhóm quản trị GPO chuyên trách quyền sửa.
- Không delegate `Edit settings` cho nhóm rộng.
- Review GPO ACL và owner định kỳ.
- Review NTFS/share permissions của file được GPO gọi.
- Không deploy file từ share writable bởi user thường.
- Dùng versioning và file-integrity monitoring.
- Baseline GPO permissions.
- Alert khi có permission drift.
- Tách tài khoản quản trị GPO khỏi tài khoản dùng hằng ngày.

---

# 5. Detection bằng Event ID 5136

Khi bật **Audit Directory Service Changes**, thay đổi đối tượng AD sẽ tạo:

```text
Event ID 5136 - A directory service object was modified
```

Field quan trọng:

| Field | Ý nghĩa |
|---|---|
| Account Name | User thực hiện thay đổi |
| Object DN | Distinguished Name của object |
| Object GUID | GUID của GPO/object |
| Attribute LDAP Display Name | Thuộc tính bị thay đổi |
| Operation Type | Value Added / Value Deleted |
| Correlation ID | Dùng nối chuỗi thay đổi |

![Event 5136 GPO modified](/assets/img/windows-attacks-defense/gpo-permissions-and-files/event-5136-gpo-modified.png)

---

# 6. Bật Audit Directory Service Changes

Trên Domain Controller, bật qua GPO:

```text
Computer Configuration
→ Policies
→ Windows Settings
→ Security Settings
→ Advanced Audit Policy Configuration
→ DS Access
→ Audit Directory Service Changes
```

Kiểm tra:

```powershell
auditpol /get /subcategory:"Directory Service Changes"
```

---

# 7. Splunk Detection

```spl
index=wineventlog sourcetype=WinEventLog:Security EventCode=5136
| table _time, host, SubjectUserName, ObjectDN, ObjectGUID, AttributeLDAPDisplayName, OperationType
```

Chỉ tập trung GPO:

```spl
index=wineventlog sourcetype=WinEventLog:Security EventCode=5136
ObjectDN="*CN=Policies,CN=System,*"
| stats count values(AttributeLDAPDisplayName) as attributes values(OperationType) as operations by SubjectUserName, ObjectDN, ObjectGUID
```

---

# 8. ELK/KQL Detection

```kql
event.code: "5136" and winlog.event_data.ObjectDN: *CN=Policies,CN=System*
```

---

# 9. Sigma Rule

```yaml
title: Active Directory Group Policy Object Modified
status: experimental
logsource:
  product: windows
  service: security
detection:
  selection:
    EventID: 5136
    ObjectDN|contains: 'CN=Policies,CN=System'
  condition: selection
falsepositives:
  - Approved Group Policy administration
level: medium
```

Nên thêm allowlist cho tài khoản GPO admin hợp lệ.

---

# 10. Honeypot GPO

Một honeypot GPO có thể được tạo với quyền nhìn có vẻ sai để thu hút attacker.

Điều kiện an toàn:

- Chỉ link với non-critical hosts.
- Không áp cho production quan trọng.
- Giám sát liên tục.
- Tự động unlink khi bị sửa.
- Có playbook rollback.
- Có approval rõ ràng trước khi auto-response.

```text
Honeypot GPO không nên trở thành một đường leo thang thật.
```

---

# 11. PowerShell Monitoring Script

Ví dụ kiểm tra Event ID 5136 trong 15 phút gần nhất và lọc theo GUID honeypot:

```powershell
$TimeSpan = (Get-Date) - (New-TimeSpan -Minutes 15)

$Logs = Get-WinEvent -FilterHashtable @{
    LogName   = 'Security'
    Id        = 5136
    StartTime = $TimeSpan
} -ErrorAction SilentlyContinue |
Where-Object {
    $_.Properties[8].Value -match "CN={73C66DBB-81DA-44D8-BDEF-20BA2C27056D},CN=POLICIES,CN=SYSTEM,DC=EAGLE,DC=LOCAL"
}

if ($Logs) {
    $emailBody = "Honeypot GPO modified`r`n"
    $disabledUsers = @()

    foreach ($log in $Logs) {
        $user = $log.Properties[3].Value

        if (
            (Get-ADUser -Identity $user).Enabled -eq $true -and
            $user -notin $disabledUsers
        ) {
            Disable-ADAccount -Identity $user
            $emailBody += "Disabled user $user`r`n"
            $disabledUsers += $user
        }
    }

    $emailBody
}
```

---

# 12. Event ID 4725

Khi script vô hiệu hóa account, Domain Controller ghi:

```text
Event ID 4725 - A user account was disabled
```

Field quan trọng:

- Subject Account Name.
- Target Account Name.
- Account Domain.
- Logon ID.

![Event 4725 user disabled](/assets/img/windows-attacks-defense/gpo-permissions-and-files/event-4725-user-disabled.png)

Correlation:

```text
5136: bob sửa GPO honeypot
        ↓
Automation chạy
        ↓
4725: Administrator disable bob
```

---

# 13. Splunk Correlation 5136 → 4725

```spl
index=wineventlog sourcetype=WinEventLog:Security
(EventCode=5136 ObjectDN="*CN=Policies,CN=System,*")
OR
(EventCode=4725)
| table _time, EventCode, SubjectUserName, TargetUserName, ObjectDN, ObjectGUID
```

---

# 14. File Integrity Monitoring

Nếu GPO gọi script/executable trên share, nên monitor hash:

```powershell
Get-FileHash "\\server\share\startup.ps1" -Algorithm SHA256
```

Baseline đơn giản:

```powershell
$Expected = "EXPECTED_SHA256"
$Current = (Get-FileHash "\\server\share\startup.ps1" -Algorithm SHA256).Hash

if ($Current -ne $Expected) {
    Write-Output "ALERT: GPO file changed"
}
```

Có thể gửi kết quả vào Splunk, ELK, Wazuh hoặc SOAR.

---

# 15. Blue Team Detection Strategy

Không chỉ xem `5136`, cần correlate thêm:

- GPO file changed.
- SYSVOL file write.
- Scheduled task creation.
- Service creation.
- Process execution trên target hosts.
- User/computer account changes.
- GPO `versionNumber` thay đổi.
- New startup/logon script paths.

Nguồn log hữu ích:

- Domain Controller Security logs.
- PowerShell logs.
- Sysmon Event ID 1.
- Sysmon Event ID 11.
- File Server auditing.
- GroupPolicy Operational logs.
- EDR telemetry.

---

# 16. Incident Response Checklist

```text
[ ] Xác định GPO GUID và ObjectDN.
[ ] Xác định account thực hiện thay đổi.
[ ] Xác định attribute bị thay đổi.
[ ] Export GPO hiện tại.
[ ] So sánh với backup/baseline.
[ ] Kiểm tra SYSVOL files liên quan.
[ ] Xác định OU/domain/site đang link GPO.
[ ] Kiểm tra máy nào đã nhận policy.
[ ] Disable hoặc isolate account compromise.
[ ] Unlink GPO nếu cần.
[ ] Rollback GPO.
[ ] Hunt process execution trên affected hosts.
```

---

# 17. Mapping với CDSA

## Security Operations & Monitoring

- Thu Event ID 5136 và 4725.
- Baseline GPO admins.
- Monitor GPO version/ACL drift.
- Correlate GPO changes với endpoint execution.

## Incident Response & Forensics

- Timeline thay đổi GPO.
- Xác định phạm vi OU/hosts bị ảnh hưởng.
- Thu thập SYSVOL file.
- So sánh hash và nội dung script.
- Review process execution sau policy refresh.

## Threat Hunting

- Hunt account không thuộc GPO admin sửa `CN=Policies`.
- Hunt file mới trong SYSVOL.
- Hunt startup script/scheduled task path lạ.
- Hunt GPO thay đổi ngoài change window.

## Detection Engineering

- Sigma cho Event 5136.
- Splunk/Elastic correlation.
- File integrity monitoring.
- Honeypot GPO với high-confidence alert.

---

# 18. Lab Setup

Môi trường tối thiểu:

```text
1 Domain Controller
1 Windows workstation
1 Splunk/ELK/Wazuh server
Windows Event Forwarding hoặc agent
Sysmon trên endpoint
```

Log nên thu:

```text
Security: 5136, 4725, 4688
Sysmon: 1, 11, 13
PowerShell Operational
GroupPolicy Operational
```

---

# Key Takeaway

```text
GPO có thể tác động hàng loạt máy trong domain.
Quyền sửa GPO hoặc file GPO quá rộng là escalation path nghiêm trọng.
Event ID 5136 là telemetry chính để phát hiện GPO object modification.
Event ID 4725 có thể xác nhận automated response đã disable account.
Honeypot GPO chỉ phù hợp với tổ chức có monitoring và response trưởng thành.
```

GPO abuse là chủ đề phù hợp với CDSA vì cần kết hợp AD knowledge, Windows logging, threat hunting, SIEM correlation và incident response theo phạm vi ảnh hưởng.

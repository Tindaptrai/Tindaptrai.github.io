---
layout: post
title: "GPP Passwords"
date: 2026-07-12 21:12:10 +0700
categories: ["Windows Attacks & Defense", "Windows Attacks"]
tags: [active-directory, windows-attacks-defense, gpp, sysvol, cpassword, powersploit, event-4663, event-4624, event-4625, event-4771, event-4776, splunk, elk, sigma, threat-hunting, cdsa]
description: "Tóm tắt GPP Passwords trong Windows Attacks & Defense: credential trong SYSVOL, cpassword, Get-GPPPassword, prevention, file-access auditing, logon correlation và honeypot detection."
toc: true
---

# GPP Passwords

Bài này thuộc module **Windows Attacks & Defense**, tập trung vào rủi ro credential bị lưu trong Group Policy Preferences và cách phát hiện hành vi truy cập/lạm dụng bằng Windows Security logs.

```text
Mục tiêu: hiểu vị trí GPP credential trong SYSVOL, thuộc tính cpassword, cách attacker thu thập credential và cách SOC phát hiện bằng Event ID 4663, 4624, 4625, 4771 và 4776.
```

> Chỉ thực hành trong HTB, lab hoặc hệ thống có ủy quyền rõ ràng. Không truy cập hoặc giải mã credential ngoài phạm vi được phép.

---

## Timeline cập nhật

```text
Created at: 2026-07-12 21:12:10 +0700
Timezone: Asia/Ho_Chi_Minh
Module: Windows Attacks & Defense
Section: GPP Passwords
Focus: SYSVOL, cpassword, PowerSploit, file auditing, authentication correlation, honeypot
```

---

# 1. SYSVOL là gì?

`SYSVOL` là network share tồn tại trên tất cả Domain Controllers, chứa dữ liệu cần thiết cho toàn domain:

- Group Policy Objects.
- Logon scripts.
- Startup/shutdown scripts.
- Group Policy Preferences.
- Policy-related XML files.

Đường dẫn tổng quát:

```text
\\<DOMAIN>\SYSVOL\<DOMAIN>\Policies\
```

Tất cả **Authenticated Users**, bao gồm user và computer account, thường có quyền đọc SYSVOL để nhận chính sách.

---

# 2. GPP Passwords là gì?

Group Policy Preferences từng cho phép admin lưu credential trong các XML policy files để phục vụ:

- Local users/groups.
- Scheduled tasks.
- Services.
- Drive mapping.
- Data sources.
- Printer configuration.

Credential được lưu trong thuộc tính:

```xml
cpassword="..."
```

Vấn đề là khóa AES dùng để mã hóa `cpassword` được Microsoft công khai, nên bất kỳ domain user nào đọc được XML đều có thể giải mã.

---

# 3. Ví dụ file dễ chứa cpassword

Các file thường được kiểm tra:

```text
Groups.xml
Services.xml
ScheduledTasks.xml
DataSources.xml
Drives.xml
Printers.xml
```

Ví dụ đường dẫn:

```text
\\EAGLE.LOCAL\SYSVOL\eagle.local\Policies\{GUID}\Machine\Preferences\Groups\Groups.xml
```

---

# 4. Attack Flow

```text
1. Domain user truy cập SYSVOL.
2. Tìm XML có thuộc tính cpassword.
3. Trích username và cpassword.
4. Giải mã cpassword bằng khóa AES công khai.
5. Dùng credential để xác thực tới host/service.
6. Nếu account có quyền cao, attacker có thể lateral movement hoặc privilege escalation.
```

---

# 5. Thu thập bằng Get-GPPPassword

Trong lab, PowerSploit có function:

```powershell
Import-Module .\Get-GPPPassword.ps1
Get-GPPPassword
```

Ví dụ output:

```text
UserName  : svc-iis
Password  : abcd@123
File      : \\EAGLE.LOCAL\SYSVOL\eagle.local\Policies\{GUID}\Machine\Preferences\Groups\Groups.xml
NodeName  : Groups
Cpassword : qRI/NPQtItGsMjwMkhF7ZDvK6n9KlOhBZ/XShO2IZ80
```

Điểm cần hiểu:

```text
Tool quét các XML trong SYSVOL.
Khi gặp cpassword, tool tự động giải mã.
```

---

# 6. Vì sao lỗ hổng vẫn tồn tại?

Microsoft đã phát hành bản vá `KB2962486` vào năm 2014 để ngăn tạo mới GPP credential.

Tuy nhiên:

```text
Bản vá không tự xóa credential đã tồn tại trước đó.
```

Do đó, domain cũ hoặc domain được migrate có thể vẫn còn XML chứa `cpassword`.

---

# 7. Prevention

Các biện pháp phòng thủ:

- Cài đặt đầy đủ bản vá liên quan.
- Search toàn bộ SYSVOL để tìm `cpassword`.
- Xóa XML credential cũ.
- Rotate toàn bộ password từng xuất hiện trong GPP.
- Không lưu credential trong GPO/GPP.
- Dùng gMSA cho service phù hợp.
- Dùng LAPS/Windows LAPS cho local admin.
- Audit SYSVOL định kỳ.
- Review Group Policy Preferences sau migration.
- Giới hạn quyền của service account.

---

# 8. Detection kỹ thuật 1 - Audit file access

Có thể bật SACL auditing trên file XML nhạy cảm hoặc một dummy XML honeypot.

Mục tiêu:

```text
Mọi truy cập READ vào file sẽ tạo Security Event ID 4663.
```

![Event 4663 Groups.xml access](/assets/img/windows-attacks-defense/gpp-passwords/event-4663-groups-xml-access.png)

Event ID:

```text
4663 - An attempt was made to access an object
```

Field quan trọng:

- Subject Account Name.
- Object Name.
- Process Name.
- Accesses.
- Access Mask.
- Source host/context nếu correlate thêm.

---

# 9. Bật auditing cho Groups.xml

Trên Domain Controller:

```text
Properties
→ Security
→ Advanced
→ Auditing
→ Add
→ Principal: Everyone hoặc nhóm cần giám sát
→ Type: Success
→ Read / Read & execute
```

Sau đó cần bật Audit File System qua GPO:

```text
Computer Configuration
→ Policies
→ Windows Settings
→ Security Settings
→ Advanced Audit Policy Configuration
→ Object Access
→ Audit File System
```

Khuyến nghị lab:

```text
Success: Enabled
Failure: tùy nhu cầu
```

---

# 10. Splunk detection cho Event 4663

```spl
index=wineventlog sourcetype=WinEventLog:Security EventCode=4663
Object_Name="*\\SYSVOL\\*\\Preferences\\*\\*.xml"
| table _time, host, SubjectUserName, Object_Name, Process_Name, AccessMask
```

Honeypot file:

```spl
index=wineventlog sourcetype=WinEventLog:Security EventCode=4663
Object_Name="*\\Policies\\*\\Groups.xml"
| stats count values(Process_Name) as processes values(host) as hosts by SubjectUserName, Object_Name
```

---

# 11. Detection kỹ thuật 2 - Successful logon

Nếu credential còn đúng, attacker có thể đăng nhập thành công.

Event:

```text
4624 - An account was successfully logged on
```

![Event 4624 service account logon](/assets/img/windows-attacks-defense/gpp-passwords/event-4624-service-account-logon.png)

Field cần xem:

- Account Name.
- Logon Type.
- Source Network Address.
- Workstation Name.
- Authentication Package.
- Target host.

Ví dụ đáng ngờ:

```text
Service account svc-iis đăng nhập từ workstation/user subnet lạ.
Logon Type 3 từ IP không thuộc server hợp lệ.
```

---

# 12. Detection kỹ thuật 3 - Failed logon

Nếu GPP password đã cũ, attacker thường thử đăng nhập và thất bại.

Event:

```text
4625 - An account failed to log on
```

![Event 4625 failed service account logon](/assets/img/windows-attacks-defense/gpp-passwords/event-4625-failed-service-account-logon.png)

Field quan trọng:

```text
Account Name
Logon Type
Source Network Address
Status
Sub Status
```

Ví dụ:

```text
Sub Status: 0xC000006A
```

`0xC000006A` thường nghĩa là password sai.

---

# 13. Detection kỹ thuật 4 - Kerberos pre-authentication failure

Event:

```text
4771 - Kerberos pre-authentication failed
```

Field cần xem:

- Account Name.
- Client Address.
- Failure Code.
- Pre-Authentication Type.

Ví dụ:

```text
Failure Code: 0x18
```

`0x18` thường tương ứng wrong password.

---

# 14. Detection kỹ thuật 5 - NTLM credential validation

Event:

```text
4776 - The computer attempted to validate the credentials for an account
```

Ví dụ error code:

```text
0xC000006A
```

Điều này cho thấy password không đúng khi NTLM credential validation xảy ra.

![Events 4771 and 4776 bad password](/assets/img/windows-attacks-defense/gpp-passwords/events-4771-4776-bad-password.png)

---

# 15. Correlation Logic

Một detection mạnh không chỉ dựa trên một event.

Correlation gợi ý:

```text
4663 đọc Groups.xml
        ↓
4624 đăng nhập thành công
hoặc
4625 / 4771 / 4776 đăng nhập thất bại
        ↓
Cùng account svc-iis
Cùng source IP hoặc cùng khoảng thời gian
```

Ví dụ:

```text
User bob đọc Groups.xml.
5 phút sau, source IP 172.16.18.25 thử đăng nhập svc-iis.
```

Đây là chuỗi hành vi đáng điều tra.

---

# 16. Splunk correlation

```spl
(
  index=wineventlog sourcetype=WinEventLog:Security EventCode=4663
  Object_Name="*\\SYSVOL\\*\\Preferences\\*\\*.xml"
)
OR
(
  index=wineventlog sourcetype=WinEventLog:Security
  EventCode IN (4624,4625,4771,4776)
  Account_Name="svc-iis"
)
| transaction Account_Name maxspan=15m
| table _time, EventCode, Account_Name, Source_Network_Address, Client_Address, Object_Name, Status, Sub_Status, Failure_Code
```

Trong production, nên dùng `stats`/data model thay vì `transaction` nếu volume lớn.

---

# 17. ELK/KQL

File access:

```kql
event.code: "4663" and winlog.event_data.ObjectName: *\\SYSVOL\\*\\Preferences\\*.xml
```

Failed logon:

```kql
event.code: ("4625" or "4771" or "4776") and user.name: "svc-iis"
```

Successful service-account logon:

```kql
event.code: "4624" and user.name: "svc-iis"
```

---

# 18. Sigma rule - GPP XML access

```yaml
title: Access to Group Policy Preference XML in SYSVOL
status: experimental
logsource:
  product: windows
  service: security
detection:
  selection:
    EventID: 4663
    ObjectName|contains:
      - '\\SYSVOL\\'
      - '\\Preferences\\'
    ObjectName|endswith: '.xml'
  condition: selection
falsepositives:
  - Domain clients processing legitimate Group Policy
level: medium
```

Rule này cần tune kỹ vì SYSVOL được truy cập hợp lệ thường xuyên.

---

# 19. Honeypot GPP XML

Một honeypot hiệu quả là tạo XML giả:

- Không liên kết với GPO thật.
- Chứa username/service account giả.
- Chứa password cố tình sai.
- Đặt trong path nhìn hợp lý.
- Bật auditing trên file.
- Alert mọi truy cập 4663.
- Alert mọi logon của account giả.

Ví dụ account:

```text
svc-iis
```

Expected detection:

```text
4663: đọc XML
4625: thử password sai
4771: Kerberos pre-auth failed
4776: NTLM validation failed
```

---

# 20. Thiết kế honeypot an toàn

Service account honeypot nên:

- Có tuổi đời và naming hợp lý.
- Có password thật rất mạnh.
- Password giả trong XML phải sai.
- Có dummy scheduled task để tạo baseline hợp lệ nếu cần.
- Không có quyền thực tế nguy hiểm.
- Không phải account production.
- Allowlist dummy task trong SIEM.
- Alert mọi hoạt động ngoài baseline.

---

# 21. IR Checklist

```text
[ ] Xác định ai đọc file XML.
[ ] Xác định Object Name và Process Name.
[ ] Xác định source IP của logon sau đó.
[ ] Kiểm tra 4624, 4625, 4771, 4776.
[ ] Kiểm tra PowerShell logs và Event 4688.
[ ] Tìm Get-GPPPassword/PowerSploit execution.
[ ] Kiểm tra credential đã bị dùng ở host nào.
[ ] Rotate credential exposed.
[ ] Xóa cpassword khỏi SYSVOL.
[ ] Hunt lateral movement và persistence.
```

---

# 22. Mapping với CDSA

## Security Operations & Monitoring

- Thu Event 4663 từ DC.
- Thu 4624/4625/4771/4776.
- Correlate file access với authentication.
- Build service-account baseline.

## Incident Response & Forensics

- Triage DC và source workstation.
- Phân tích PowerShell/4688/Sysmon Event 1.
- Xác định credential exposure window.
- Review lateral movement sau thời điểm truy cập XML.

## Threat Hunting

- Hunt `cpassword` trong SYSVOL.
- Hunt account service login từ workstation.
- Hunt wrong-password spike cho service account.
- Hunt read access vào GPP XML hiếm.

## Detection Engineering

- Sigma cho 4663.
- Splunk/Elastic correlation.
- Honeypot XML + fake credential.
- Tune baseline theo GPO processing hợp lệ.

---

# 23. Lab Setup

Môi trường tối thiểu:

```text
1 Domain Controller
1 Windows workstation
1 Splunk hoặc ELK server
Windows Event Forwarding hoặc Universal Forwarder/Winlogbeat
Sysmon trên workstation
```

Log cần thu:

```text
Security: 4663, 4624, 4625, 4771, 4776, 4688
PowerShell Operational
Sysmon Event ID 1
```

Kiểm tra auditing:

```powershell
auditpol /get /subcategory:"File System"
auditpol /get /subcategory:"Logon"
auditpol /get /subcategory:"Credential Validation"
auditpol /get /subcategory:"Kerberos Authentication Service"
```

---

# Key Takeaway

```text
GPP từng lưu password trong XML bằng thuộc tính cpassword.
SYSVOL cho Authenticated Users đọc, trong khi khóa giải mã là công khai.
KB2962486 ngăn tạo credential mới nhưng không xóa credential cũ.
Detection tốt nhất là kết hợp 4663 với 4624/4625/4771/4776.
Honeypot XML và service account giả tạo high-confidence alert.
```

GPP Passwords là ví dụ điển hình cho CDSA: một cấu hình legacy tạo credential exposure, còn SOC phải kết hợp file auditing, authentication logs và contextual baseline để phát hiện chính xác.

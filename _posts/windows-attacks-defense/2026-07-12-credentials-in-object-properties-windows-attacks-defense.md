---
layout: post
title: "Credentials in Object Properties"
date: 2026-07-12 22:00:53 +0700
categories: ["Windows Attacks & Defense", "Windows Attacks"]
tags: [active-directory, windows-attacks-defense, object-properties, description, info, cleartext-credentials, powershell, event-4624, event-4625, event-4738, event-4768, event-4771, event-4776, splunk, elk, sigma, threat-hunting, cdsa]
description: "Tóm tắt Credentials in Object Properties: credential bị lưu trong Description/Info của AD objects, cách tìm bằng PowerShell, prevention, behavior-based detection và honeypot account."
toc: true
---

# Credentials in Object Properties

Bài này thuộc module **Windows Attacks & Defense**, tập trung vào rủi ro credential bị lưu trực tiếp trong các thuộc tính của Active Directory objects.

```text
Mục tiêu: hiểu vì sao Description/Info có thể làm lộ password, cách tìm credential bằng PowerShell và cách phát hiện việc lạm dụng qua authentication telemetry.
```

> Chỉ thực hành trong HTB, lab hoặc hệ thống được ủy quyền. Không truy vấn hoặc sử dụng credential ngoài phạm vi cho phép.

---

## Timeline cập nhật

```text
Created at: 2026-07-12 22:00:53 +0700
Timezone: Asia/Ho_Chi_Minh
Module: Windows Attacks & Defense
Section: Credentials in Object Properties
Focus: AD object attributes, PowerShell enumeration, Event ID 4768, behavior analytics, honeypot
```

---

# 1. Vấn đề nằm ở đâu?

Active Directory objects có nhiều thuộc tính như:

- Account status.
- Expiration time.
- Password last set.
- Office.
- Phone number.
- Description.
- Info.

Một sai lầm phổ biến là admin lưu password trong:

```text
Description
Info
```

Nhiều người nghĩ chỉ Domain Admin mới đọc được các trường này, nhưng domain user thường có thể đọc phần lớn thuộc tính object.

---

# 2. Vì sao đây là misconfiguration nguy hiểm?

Nếu password nằm trong `Description` hoặc `Info`, attacker có domain account thấp quyền có thể query toàn domain và tìm:

```text
pass
password
pwd
secret
token
```

Nếu account chứa credential có quyền cao hoặc là service account, rủi ro lateral movement và privilege escalation rất lớn.

---

# 3. PowerShell Search Function

Ví dụ function tìm từ khóa trong `Description` và `Info`:

```powershell
Function SearchUserClearTextInformation {
    Param (
        [Parameter(Mandatory=$true)]
        [Array] $Terms,

        [Parameter(Mandatory=$false)]
        [String] $Domain
    )

    if ([string]::IsNullOrEmpty($Domain)) {
        $dc = (Get-ADDomain).RIDMaster
    } else {
        $dc = (Get-ADDomain $Domain).RIDMaster
    }

    $list = @()

    foreach ($t in $Terms) {
        $list += "(`$_.Description -like `"*$t*`")"
        $list += "(`$_.Info -like `"*$t*`")"
    }

    Get-ADUser -Filter * -Server $dc `
        -Properties Enabled,Description,Info,PasswordNeverExpires,PasswordLastSet |
    Where-Object { Invoke-Expression ($list -join ' -OR ') } |
    Select-Object SamAccountName,Enabled,Description,Info,PasswordNeverExpires,PasswordLastSet |
    Format-List
}
```

Chạy:

```powershell
SearchUserClearTextInformation -Terms "pass"
```

Ví dụ output:

```text
SamAccountName       : bonni
Enabled              : True
Description          : pass: Slavi123
PasswordNeverExpires : True
PasswordLastSet      : 05/12/2022 15:18:05
```

---

# 4. Attack Flow

```text
1. Attacker có domain user hợp lệ.
2. Query user objects.
3. Tìm Description/Info chứa password.
4. Lấy cleartext credential.
5. Thử xác thực.
6. Nếu thành công, tiếp tục lateral movement hoặc privilege escalation.
```

---

# 5. Prevention

Các biện pháp quan trọng:

- Không lưu password trong AD object attributes.
- Scan định kỳ Description/Info.
- Tự động hóa user/service account provisioning.
- Giáo dục admin và Service Desk.
- Dùng password vault hoặc secret manager.
- Review account có `PasswordNeverExpires`.
- Rotate credential đã từng xuất hiện trong object attributes.
- Dùng gMSA cho service account phù hợp.

---

# 6. Event ID 4738 và giới hạn detection

Khi user object bị thay đổi, Windows có thể ghi:

```text
Event ID 4738 - A user account was changed
```

Nhưng hạn chế:

```text
Event 4738 không luôn hiển thị chính xác thuộc tính nào bị sửa
và cũng không cho biết rõ giá trị mới.
```

Do đó, không thể chỉ dựa vào 4738 để xác định password vừa bị thêm vào `Description` hoặc `Info`.

---

# 7. Detection theo hành vi

Detection thực tế hơn là theo dõi việc credential bị sử dụng sau khi bị lộ.

Các event quan trọng:

```text
4624 - Successful logon
4625 - Failed logon
4768 - Kerberos TGT requested
4771 - Kerberos pre-authentication failed
4776 - NTLM credential validation
```

---

# 8. Event ID 4768

Khi credential được dùng với Kerberos:

```text
Event ID 4768 - A Kerberos authentication ticket (TGT) was requested
```

Field cần xem:

- Account Name.
- Client Address.
- Ticket Encryption Type.
- Pre-Authentication Type.
- Result Code.

![Event 4768 bonni TGT request](/assets/img/windows-attacks-defense/credentials-in-object-properties/event-4768-bonni-tgt-request.png)

Ví dụ đáng ngờ:

```text
Account Name: bonni
Client Address: 172.16.18.25
```

Nếu user `bonni` bình thường không đăng nhập từ IP này, cần điều tra.

---

# 9. Splunk Detection

TGT request cho account đáng chú ý:

```spl
index=wineventlog sourcetype=WinEventLog:Security EventCode=4768
Account_Name="bonni"
| table _time, host, Account_Name, Client_Address, Ticket_Encryption_Type, Pre_Authentication_Type, Result_Code
```

Privileged/service account từ source lạ:

```spl
index=wineventlog sourcetype=WinEventLog:Security
EventCode IN (4624,4625,4768,4771,4776)
| stats count values(Source_Network_Address) as src values(Client_Address) as kerberos_src by Account_Name, EventCode
| sort - count
```

---

# 10. ELK/KQL

Kerberos TGT request:

```kql
event.code: "4768" and user.name: "bonni"
```

Failed credential use:

```kql
event.code: ("4625" or "4771" or "4776") and user.name: "svc-iis"
```

---

# 11. Sigma Rule

```yaml
title: Suspicious Authentication With Monitored AD Account
status: experimental
logsource:
  product: windows
  service: security
detection:
  selection:
    EventID:
      - 4624
      - 4625
      - 4768
      - 4771
      - 4776
    TargetUserName:
      - bonni
      - svc-iis
  condition: selection
falsepositives:
  - Approved administrative or service activity
level: high
```

Nên thêm baseline theo source IP, workstation và logon type.

---

# 12. Honeypot Account

Có thể tạo honeypot bằng cách đặt credential giả trong `Description`.

Ví dụ:

```text
Description: pass: FakePassword123!
```

Điều kiện honeypot tốt:

- Password trong Description phải sai.
- Account thật phải dùng password khác, đủ mạnh.
- Account vẫn enabled.
- Có lịch sử logon hợp lý.
- PasswordLastSet đủ cũ để nhìn thuyết phục.
- Account không có quyền thật nguy hiểm.
- Mọi authentication attempt phải được alert.

---

# 13. Expected Honeypot Events

Nếu attacker thử credential giả:

```text
4625 - Failed logon
4771 - Kerberos pre-authentication failed
4776 - NTLM credential validation failed
```

Nếu credential honeypot vô tình đúng và login thành công:

```text
4624
4768
```

Đây phải là high-confidence alert.

---

# 14. Incident Response Checklist

```text
[ ] Xác định account có credential trong Description/Info.
[ ] Xác định credential đã bị dùng chưa.
[ ] Review 4624, 4625, 4768, 4771, 4776.
[ ] Xác định source IP và workstation.
[ ] Rotate password ngay.
[ ] Xóa credential khỏi object properties.
[ ] Search toàn domain cho từ khóa tương tự.
[ ] Hunt lateral movement sau thời điểm credential bị dùng.
[ ] Review PowerShell/4688/Sysmon Event 1.
```

---

# 15. Mapping với CDSA

## Security Operations & Monitoring

- Thu authentication events từ DC.
- Baseline user/service account behavior.
- Alert source IP bất thường.
- Monitor PasswordNeverExpires accounts.

## Incident Response & Forensics

- Timeline credential exposure.
- Xác định account compromise.
- Triage source host.
- Review subsequent logons và lateral movement.

## Threat Hunting

- Hunt `Description`/`Info` có từ khóa nhạy cảm.
- Hunt account login từ VLAN/workstation bất thường.
- Hunt service account authentication từ user endpoint.
- Hunt password failures cho honeypot account.

## Detection Engineering

- Sigma cho monitored account.
- Splunk/Elastic behavioral correlation.
- Honeypot credentials trong object properties.
- Baseline authentication geography/network segment.

---

# 16. Lab Setup

Môi trường tối thiểu:

```text
1 Domain Controller
1 Windows workstation
1 Splunk/ELK/Wazuh server
Windows Event Forwarding hoặc agent
Sysmon trên workstation
```

Log nên thu:

```text
Security: 4624, 4625, 4738, 4768, 4771, 4776, 4688
Sysmon: 1, 3
PowerShell Operational
```

---

# Key Takeaway

```text
Description và Info của AD objects có thể bị domain users đọc.
Lưu password trong object properties là credential exposure nghiêm trọng.
Event 4738 không đủ chi tiết để xác định giá trị thuộc tính bị thay đổi.
Detection hiệu quả dựa vào authentication behavior và source correlation.
Honeypot credential trong Description tạo high-confidence alert.
```

Credentials in Object Properties là ví dụ điển hình cho CDSA: misconfiguration không dễ detect ở thời điểm tạo, nhưng có thể bị phát hiện khi credential được sử dụng qua Windows authentication telemetry.

---
layout: post
title: "DCSync"
date: 2026-07-12 22:28:27 +0700
categories: ["Windows Attacks & Defense", "Windows Defense"]
tags: [active-directory, windows-attacks-defense, dcsync, mimikatz, directory-replication, event-4662, replication-rights, ntlm, kerberos, splunk, elk, sigma, threat-hunting, incident-response, cdsa]
description: "Tóm tắt DCSync trong Windows Attacks & Defense: quyền replication cần thiết, Mimikatz, Event ID 4662, prevention, detection engineering và incident response."
toc: true
---

# DCSync

Bài này thuộc module **Windows Attacks & Defense**, tập trung vào kỹ thuật **DCSync** và cách phát hiện/phản ứng theo góc nhìn SOC/CDSA.

```text
Mục tiêu: hiểu cách attacker giả lập hành vi replication của Domain Controller để lấy password hashes, đồng thời nhận diện bằng Event ID 4662 và kiểm soát quyền replication.
```

> Chỉ thực hành trong HTB, lab hoặc hệ thống có ủy quyền rõ ràng. DCSync trên domain thật ngoài phạm vi cho phép là hành vi xâm nhập nghiêm trọng.

---

## Timeline cập nhật

```text
Created at: 2026-07-12 22:28:27 +0700
Timezone: Asia/Ho_Chi_Minh
Module: Windows Attacks & Defense
Section: DCSync
Focus: Replication rights, Mimikatz, Event ID 4662, SIEM detection, IR
```

---

# 1. DCSync là gì?

**DCSync** là kỹ thuật trong đó attacker dùng một user hoặc computer account có quyền replication để giả lập hành vi của Domain Controller.

Attacker gửi yêu cầu replication đến DC và yêu cầu dữ liệu nhạy cảm như:

- NTLM hash.
- Kerberos keys.
- Password history.
- Supplemental credentials.
- Credential của account đặc quyền.

```text
DCSync không cần đăng nhập trực tiếp vào Domain Controller.
```

---

# 2. Quyền cần thiết

Account thực hiện DCSync cần các quyền replication như:

```text
Replicating Directory Changes
Replicating Directory Changes All
```

Trong một số tình huống có thể cần thêm:

```text
Replicating Directory Changes In Filtered Set
```

Các quyền này thường chỉ nên thuộc về:

- Domain Controllers.
- Domain Admins.
- Enterprise Admins.
- Một số service account đặc biệt có nhu cầu replication hợp lệ.

---

# 3. Vì sao DCSync nguy hiểm?

Nếu attacker lấy được hash của:

```text
Administrator
krbtgt
Domain Admin
Enterprise Admin
Service account đặc quyền
```

thì có thể dẫn đến:

- Pass-the-Hash.
- Pass-the-Ticket.
- Golden Ticket nếu `krbtgt` bị compromise.
- Persistence dài hạn.
- Lateral movement.
- Full domain compromise.

---

# 4. Attack Flow

```text
1. Attacker compromise account có replication rights.
2. Kết nối tới Domain Controller qua RPC/LDAP.
3. Gửi yêu cầu replication.
4. DC trả credential material.
5. Attacker thu NTLM/Kerberos keys.
6. Dùng credential để xác thực hoặc duy trì persistence.
```

---

# 5. Thực hành trong lab với Mimikatz

Ví dụ tạo shell dưới context user có replication rights:

```cmd
runas /user:eagle\rocky cmd.exe
```

Sau đó chạy Mimikatz:

```cmd
mimikatz.exe
```

Request hash của một account cụ thể:

```text
lsadump::dcsync /domain:eagle.local /user:Administrator
```

Ví dụ output quan trọng:

```text
SAM Username : Administrator
Hash NTLM    : <NTLM_HASH>
```

Có thể request toàn bộ domain bằng tham số `/all`, nhưng thao tác này rất noisy và rủi ro cao.

---

# 6. DCSync không phải password dumping cục bộ

Điểm khác biệt:

```text
LSASS dumping: lấy credential từ memory trên host.
NTDS dumping: lấy credential database từ DC.
DCSync: yêu cầu DC gửi dữ liệu replication qua network.
```

Điều này khiến DCSync đặc biệt nguy hiểm vì attacker không nhất thiết phải code execution trên DC.

---

# 7. Prevention

Không thể tắt replication vì AD cần replication để hoạt động.

Biện pháp chính:

- Chỉ cấp replication rights cho account thực sự cần.
- Audit ACL trên domain root.
- Review delegated rights định kỳ.
- Loại bỏ quyền replication khỏi user/service account không cần thiết.
- Tiering admin accounts.
- Dùng Protected Users/PAW phù hợp.
- Monitor account có replication rights.
- Hạn chế RPC từ workstation đến DC.
- Dùng RPC firewall hoặc network control để chỉ cho DC replication hợp lệ.
- Bảo vệ Domain Admin, Enterprise Admin và service accounts.
- Rotate credential ngay khi nghi compromise.

---

# 8. Event ID 4662

DCSync thường tạo:

```text
Event ID 4662 - An operation was performed on an object
```

Event này xuất hiện trên Domain Controller khi Directory Service Access auditing được bật.

Field cần xem:

- Subject Account Name.
- Subject Logon ID.
- Object Type.
- Object Name.
- Access Mask.
- Properties/Extended Rights.
- Source context khi correlate thêm.

---

# 9. Bật Audit Directory Service Access

Trên Domain Controller:

```text
Computer Configuration
→ Policies
→ Windows Settings
→ Security Settings
→ Advanced Audit Policy Configuration
→ DS Access
→ Audit Directory Service Access
```

Khuyến nghị:

```text
Success: Enabled
Failure: tùy môi trường
```

Kiểm tra:

```powershell
auditpol /get /subcategory:"Directory Service Access"
```

---

# 10. Detection Logic

DCSync detection tốt nên kiểm tra:

```text
Event ID 4662
Account không phải Domain Controller
Replication-related extended rights
Request phát sinh từ workstation/server không hợp lệ
Volume request bất thường
```

High-confidence logic:

```text
4662 + replication rights + source/account không thuộc DC
```

---

# 11. Splunk Detection

Query cơ bản:

```spl
index=wineventlog sourcetype=WinEventLog:Security EventCode=4662
| table _time, host, SubjectUserName, ObjectName, ObjectType, AccessMask, Properties
```

Tập trung account không phải computer account:

```spl
index=wineventlog sourcetype=WinEventLog:Security EventCode=4662
| where NOT like(SubjectUserName, "%$")
| stats count values(ObjectName) as objects values(Properties) as properties by SubjectUserName, host
| sort - count
```

Tập trung replication-related activity:

```spl
index=wineventlog sourcetype=WinEventLog:Security EventCode=4662
(Properties="*Replicating Directory Changes*"
 OR Properties="*Replicating Directory Changes All*")
| stats count values(ObjectName) as objects by SubjectUserName, host
```

---

# 12. ELK/KQL

```kql
event.code: "4662"
```

Tập trung replication rights:

```kql
event.code: "4662"
and winlog.event_data.Properties: (
  *Replicating\ Directory\ Changes*
  or *Replicating\ Directory\ Changes\ All*
)
```

Loại trừ computer account hợp lệ cần được tune theo môi trường.

---

# 13. Sigma Rule

```yaml
title: Possible DCSync Activity
status: experimental
logsource:
  product: windows
  service: security
detection:
  selection:
    EventID: 4662
  replication:
    Properties|contains:
      - 'Replicating Directory Changes'
      - 'Replicating Directory Changes All'
  filter_dc:
    SubjectUserName|endswith: '$'
  condition: selection and replication and not filter_dc
falsepositives:
  - Approved directory synchronization services
  - Backup or identity-management products
level: high
```

Rule cần allowlist account replication hợp lệ.

---

# 14. Threat Hunting

Hunt theo:

- User account có replication rights.
- 4662 từ non-DC account.
- 4662 ngoài maintenance window.
- User vừa được cấp ACL replication rồi phát sinh 4662.
- Source host không nằm trong DC subnet.
- Mimikatz execution trước event.
- PowerShell/cmd process bất thường.
- Pass-the-Hash sau thời điểm DCSync.

---

# 15. Correlation với các Event khác

Nên correlate thêm:

```text
4624 - Successful logon
4672 - Special privileges assigned
4688 - Process creation
4728/4732/4756 - Added to privileged group
4738 - User account changed
5136 - Directory object modified
4662 - Replication operation
```

Ví dụ chuỗi:

```text
5136: account được cấp replication right
        ↓
4624: account đăng nhập từ host lạ
        ↓
4662: replication request
        ↓
4624/4768/4769: credential mới được dùng
```

---

# 16. Incident Response Checklist

```text
[ ] Xác định account thực hiện DCSync.
[ ] Xác định source host.
[ ] Kiểm tra account có replication rights vì sao.
[ ] Review Event ID 4662.
[ ] Kiểm tra Event 5136/ACL changes trước đó.
[ ] Tìm Mimikatz hoặc tool tương tự.
[ ] Review 4688, PowerShell logs, Sysmon Event 1.
[ ] Xác định account nào bị request hash.
[ ] Rotate credential bị lộ.
[ ] Nếu krbtgt bị lộ, lập kế hoạch reset hai lần.
[ ] Hunt Pass-the-Hash/Golden Ticket/lateral movement.
[ ] Remove replication rights không hợp lệ.
```

---

# 17. Nếu krbtgt bị compromise

`krbtgt` compromise là tình huống nghiêm trọng.

Xử lý thường gồm:

```text
1. Điều tra phạm vi compromise.
2. Khóa persistence.
3. Reset krbtgt lần 1.
4. Chờ replication hoàn tất.
5. Reset krbtgt lần 2.
6. Theo dõi Kerberos anomalies.
```

Không nên reset vội trong production nếu chưa có kế hoạch, vì có thể ảnh hưởng authentication trên toàn domain.

---

# 18. Mapping với CDSA

## Security Operations & Monitoring

- Thu Event ID 4662 từ DC.
- Baseline replication accounts.
- Monitor source host và privilege assignment.
- Correlate ACL changes với replication activity.

## Incident Response & Forensics

- Triage source host.
- Tìm Mimikatz artifacts.
- Review memory/process telemetry.
- Xác định credential nào đã bị lộ.
- Đánh giá khả năng Golden Ticket.

## Threat Hunting

- Hunt non-DC account thực hiện replication.
- Hunt replication right mới được cấp.
- Hunt 4662 từ workstation.
- Hunt credential reuse sau event.

## Detection Engineering

- Sigma cho Event 4662.
- Splunk/Elastic correlation.
- Allowlist DC/service hợp lệ.
- High-confidence alert cho non-DC replication.

---

# 19. Lab Setup

Môi trường tối thiểu:

```text
1 Domain Controller
1 Windows workstation
1 Splunk/ELK/Wazuh server
Sysmon + Windows Event Forwarding/agent
```

Log nên thu:

```text
Security: 4662, 4624, 4672, 4688, 5136
PowerShell Operational
Sysmon: 1, 3, 10
```

---

# Key Takeaway

```text
DCSync giả lập replication của Domain Controller để lấy password hashes.
Quyền trọng yếu là Replicating Directory Changes và Replicating Directory Changes All.
Event ID 4662 là telemetry chính.
Detection hiệu quả phải loại trừ DC/account replication hợp lệ.
Nếu krbtgt bị lộ, cần xử lý như full domain compromise.
```

DCSync là một trong những kỹ thuật nghiêm trọng nhất trong Active Directory vì attacker có thể lấy credential cấp domain mà không cần trực tiếp dump NTDS.DIT trên Domain Controller.

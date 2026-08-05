---
layout: post
title: "Detecting DCSync & DCShadow"
date: 2026-08-06 01:05:23 +0700
categories: ["Detecting Windows Attacks with Splunk", "Active Directory Replication Abuse"]
tags: [dcsync, dcshadow, active-directory, drsgetncchanges, directory-replication, event-id-4662, event-id-4742, krbtgt, golden-ticket, mimikatz, splunk, detection-engineering, threat-hunting, cdsa]
description: "Phát hiện DCSync và DCShadow bằng Directory Service auditing, Event ID 4662, Event ID 4742 và Splunk."
toc: true
---

# Detecting DCSync & DCShadow

## Công thức dễ nhớ

```text
DCSYNC:
REPLICATION RIGHTS → DRSGetNCChanges → HASH EXTRACTION → TICKET/HASH ABUSE

DCSHADOW:
REGISTER ROGUE DC → MODIFY AD OBJECTS → REPLICATE CHANGES → UNREGISTER
```

Hai kỹ thuật đều lạm dụng cơ chế replication của Active Directory, nhưng mục tiêu khác nhau:

```text
DCSync   = đọc dữ liệu replication để lấy credential material
DCShadow = ghi thay đổi giả mạo vào AD thông qua rogue Domain Controller
```

---

# 1. DCSync Là Gì?

DCSync mô phỏng hành vi của Domain Controller để yêu cầu dữ liệu replication từ Active Directory.

Interface quan trọng:

```text
DRSGetNCChanges
```

Khi thành công, attacker có thể lấy:

```text
Current password hashes
Password history
NTLM hashes
Kerberos keys
KRBTGT secret
Administrator credentials
Service account credentials
```

---

# 2. Quyền Cần Thiết Cho DCSync

Các quyền replication quan trọng:

```text
DS-Replication-Get-Changes
DS-Replication-Get-Changes-All
DS-Replication-Get-Changes-In-Filtered-Set
```

Principal thường có khả năng này:

```text
Domain Controllers
Administrators
Domain Admins
Enterprise Admins
Delegated replication accounts
```

Attacker không nhất thiết phải chạy trên Domain Controller nếu account đã có quyền replication phù hợp.

---

# 3. DCSync Attack Flow

```text
1. Compromise domain-joined host
2. Obtain replication privileges
3. Invoke DRSGetNCChanges
4. Request target account secrets
5. Extract hashes/Kerberos keys
6. Abuse credentials for lateral movement or persistence
```

Tool thường gặp:

```text
Mimikatz
Impacket secretsdump.py
DSInternals
Custom DRSUAPI tooling
```

Hậu quả thường thấy:

```text
Golden Ticket
Silver Ticket
Pass-the-Hash
Overpass-the-Hash
Credential reuse
Domain persistence
```

---

# 4. DCSync Detection Opportunity

Event chính:

```text
Event ID 4662
= An operation was performed on an object
```

Để ghi nhận được, cần bật audit policy:

```text
Computer Configuration
→ Windows Settings
→ Security Settings
→ Advanced Audit Policy Configuration
→ DS Access
```

Điểm quan trọng:

```text
4662 không chỉ dành cho DCSync.
Phải lọc theo replication properties/GUIDs.
```

---

# 5. Replication GUID Quan Trọng

GUID thường dùng để nhận diện:

```text
1131f6aa-9c07-11d1-f79f-00c04fc2dcd2
```

Ý nghĩa:

```text
DS-Replication-Get-Changes
```

Các log có thể hiển thị property thân thiện như:

```text
Replicating Directory Changes
```

hoặc chỉ hiện GUID, tùy cách render event.

---

# 6. Detect DCSync Với Splunk

```spl
index=main
earliest=1690544278 latest=1690544280
EventCode=4662
Message="*Replicating Directory Changes*"
| rex field=Message "(?P<property>Replicating Directory Changes.*)"
| table _time,
        user,
        object_file_name,
        Object_Server,
        property
```

---

# 7. DCSync Query Breakdown

## Event Filter

```spl
EventCode=4662
```

Lấy Directory Service object access events.

## Replication Property Filter

```spl
Message="*Replicating Directory Changes*"
```

Chỉ giữ các event liên quan replication rights.

## Extract Property

```spl
| rex field=Message "(?P<property>Replicating Directory Changes.*)"
```

Tách phần replication property vào field mới:

```text
property
```

## Output Fields

```text
_time
user
object_file_name
Object_Server
property
```

Các câu hỏi điều tra:

```text
Ai thực hiện replication?
Từ host nào?
Có phải Domain Controller hợp lệ không?
Target object là gì?
Thời điểm đó có hợp lý không?
```

---

# 8. Nâng Cao DCSync Detection

Detection đáng tin cậy hơn khi loại các Domain Controller hợp lệ.

Logic:

```text
Replication request
+ source không phải Domain Controller
+ account không thuộc baseline replication
→ nghi DCSync
```

Có thể enrichment bằng lookup:

```spl
| lookup domain_controllers.csv host AS src_host OUTPUT is_dc
| where isnull(is_dc) OR is_dc!="true"
```

Nên correlate thêm:

```text
4624 privileged logon
4672 special privileges
Sysmon process creation
Mimikatz/secretsdump execution
Network traffic tới DC RPC/SMB
KRBTGT access
```

---

# 9. DCSync False Positives

Có thể đến từ:

```text
Legitimate Domain Controllers
Azure AD Connect
Identity synchronization
Backup/DR products
AD migration tools
Authorized security auditing
Delegated IAM services
```

Tune theo:

```text
Known replication accounts
Known DC IPs
Known sync servers
Approved maintenance windows
Source host role
User baseline
Object target
```

---

# 10. DCShadow Là Gì?

DCShadow là kỹ thuật đăng ký một rogue Domain Controller để đẩy thay đổi trái phép vào Active Directory thông qua replication.

Mục tiêu có thể là:

```text
Modify group membership
Change primaryGroupID
Add privileged rights
Modify SIDHistory
Change SPNs
Alter ACLs
Create persistence
```

Điểm nguy hiểm:

```text
Thay đổi được đưa vào AD qua replication
thay vì qua các API quản trị thông thường.
```

---

# 11. DCShadow Attack Flow

```text
1. Gain administrative privilege
2. Register rogue DC objects
3. Create/modify target AD object
4. Push replication to legitimate DC
5. Unregister rogue DC
```

Để giả lập Domain Controller, attacker phải tạo hoặc sửa:

```text
Server object
nTDSDSA object
Computer object attributes
ServicePrincipalName
```

---

# 12. nTDSDSA Và Global Catalog SPN

DCShadow cần:

```text
New nTDSDSA object
Global Catalog-related SPN
```

SPN thay đổi trên computer account có thể được ghi qua:

```text
Event ID 4742
= A computer account was changed
```

Đây là telemetry quan trọng để phát hiện rogue DC registration.

---

# 13. Detect DCShadow Với Splunk

```spl
index=main
earliest=1690623888 latest=1690623890
EventCode=4742
| rex field=Message "(?P<gcspn>XX\/[a-zA-Z0-9\.\-\/]+)"
| table _time,
        ComputerName,
        Security_ID,
        Account_Name,
        user,
        gcspn
| search gcspn=*
```

---

# 14. DCShadow Query Breakdown

## Event Filter

```spl
EventCode=4742
```

Lấy sự kiện thay đổi computer account.

## Extract Suspicious SPN

```spl
| rex field=Message "(?P<gcspn>XX\/[a-zA-Z0-9\.\-\/]+)"
```

Mục tiêu:

```text
Tách SPN bất thường liên quan rogue DC registration
```

## Keep Matched Results

```spl
| search gcspn=*
```

Chỉ giữ event có extracted SPN.

Field điều tra:

```text
ComputerName
Security_ID
Account_Name
user
gcspn
_time
```

---

# 15. Hạn Chế Của Regex `XX/`

Trong tài liệu, regex sử dụng:

```text
XX/
```

Đây có thể là placeholder cho pattern SPN cần phát hiện trong dataset của lab.

Trong môi trường thực tế, cần điều chỉnh để khớp các SPN liên quan DC/GC, ví dụ:

```text
GC/
LDAP/
E3514235-4B06-11D1-AB04-00C04FC2DCD2/
```

Không nên triển khai nguyên regex lab mà chưa kiểm tra format log thực tế.

---

# 16. DCShadow Detection Opportunities Bổ Sung

Nên theo dõi:

```text
Computer account SPN changes
nTDSDSA object creation
Server object creation
Unexpected directory replication
Rogue DC registration/unregistration
Privilege changes immediately after replication
```

Event/telemetry có thể liên quan:

```text
4742 — Computer account changed
4662 — Directory object access/replication
5137 — Directory service object created
5136 — Directory service object modified
5141 — Directory service object deleted
```

Việc có event nào phụ thuộc audit policy và cấu hình domain.

---

# 17. So Sánh DCSync Và DCShadow

| Thuộc tính | DCSync | DCShadow |
|---|---|---|
| Mục tiêu | Đọc credential data | Ghi thay đổi vào AD |
| Cơ chế | DRSGetNCChanges | Rogue DC replication |
| Artifact chính | Replication access | Rogue DC object/SPN changes |
| Event trọng tâm | `4662` | `4742`, có thể thêm `5136/5137` |
| Impact | Credential theft | Persistence/privilege manipulation |
| Tool điển hình | Mimikatz, secretsdump | Mimikatz DCShadow |

---

# 18. Correlation Logic

## DCSync

```text
4662 replication property
+ source host không phải DC
+ account không thuộc replication baseline
+ credential abuse sau đó
```

## DCShadow

```text
4742 SPN change
+ nTDSDSA/server object activity
+ replication event
+ privileged AD object modification
```

---

# 19. Detection Confidence

## DCSync — Low

```text
4662 có replication property
```

## DCSync — High

```text
Non-DC host
+ replication-capable account
+ DRSGetNCChanges pattern
+ KRBTGT/Admin secret access
```

## DCShadow — Low

```text
Computer account SPN changed
```

## DCShadow — High

```text
Rogue DC SPN/object creation
+ replication activity
+ privileged object modification
+ rapid unregister/delete
```

---

# 20. Investigation Workflow

```text
1. Xác định user và source host
2. Kiểm tra source có phải DC hợp lệ không
3. Xem replication property/GUID
4. Xác định object bị truy cập hoặc sửa
5. Kiểm tra SPN thay đổi
6. Kiểm tra nTDSDSA/server object creation
7. Hunt Mimikatz/secretsdump execution
8. Kiểm tra KRBTGT/Admin credential abuse
9. Hunt Golden Ticket/PtH/Overpass-the-Hash
10. Contain host và rotate credential
```

---

# 21. Response Considerations

Nếu xác nhận DCSync:

```text
Disable compromised account
Remove replication rights
Rotate privileged credentials
Reset KRBTGT theo quy trình phù hợp
Investigate Golden Ticket creation
Audit domain-wide lateral movement
```

Nếu xác nhận DCShadow:

```text
Remove rogue DC objects
Review replication metadata
Revert unauthorized AD changes
Rotate impacted secrets
Audit privileged groups and ACLs
Investigate persistence mechanisms
```

---

# Keyword Cần Nhớ

```text
DCSync
DCShadow
Active Directory Replication
DRSGetNCChanges
DS-Replication-Get-Changes
DS-Replication-Get-Changes-All
Event ID 4662
Event ID 4742
Event ID 5136
Event ID 5137
Event ID 5141
1131f6aa-9c07-11d1-f79f-00c04fc2dcd2
KRBTGT
NTDS.dit
LSASS
Mimikatz
secretsdump.py
nTDSDSA
ServicePrincipalName
Global Catalog
rogue Domain Controller
Golden Ticket
```

---

# Key Takeaway

```text
DCSync:
lạm dụng replication để đọc credential secrets.
```

```text
DCShadow:
lạm dụng replication để ghi thay đổi trái phép vào AD.
```

```text
Detection tốt =
replication auditing
+ source host validation
+ identity baseline
+ AD object change correlation
```

> Chỉ thực hành trên HTB, lab hoặc hệ thống được cấp phép.

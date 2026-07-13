---
layout: post
title: "Object ACLs"
date: 2026-07-13 23:26:30 +0700
categories: ["Windows Attacks & Defense"]
tags: [active-directory, windows-attacks-defense, object-acls, ace, dacl, sacl, bloodhound, sharphound, adaclscanner, genericall, genericwrite, writedacl, writeowner, forcechangepassword, event-4724, event-4738, event-4742, event-5136, splunk, elk, sigma, threat-hunting, incident-response, cdsa]
description: "Tóm tắt Object ACLs trong Active Directory: ACL/ACE/DACL/SACL, các quyền có thể bị lạm dụng, phân tích bằng SharpHound/BloodHound, prevention và detection bằng Windows Security logs."
toc: true
---

# Object ACLs

Bài này thuộc module **Windows Attacks & Defense**, tập trung vào cách **Access Control Lists** kiểm soát quyền trên Active Directory objects và vì sao delegated rights sai có thể tạo privilege-escalation path.

```text
Mục tiêu: hiểu ACL/ACE/DACL/SACL, nhận diện quyền nguy hiểm bằng BloodHound,
và xây detection dựa trên Event ID 4724, 4738, 4742 và 5136.
```

> Chỉ thực hành SharpHound/BloodHound và ACL abuse trong HTB, lab hoặc môi trường có ủy quyền. Thu thập `-c All` có thể tạo lượng lớn LDAP/SMB telemetry và phải nằm trong scope.

---

## Timeline cập nhật

```text
Created at: 2026-07-13 23:26:30 +0700
Timezone: Asia/Ho_Chi_Minh
Module: Windows Attacks & Defense
Section: Object ACLs
Focus: ACL/ACE, BloodHound, SharpHound, delegated rights, Windows detection
```

---

# 1. ACL, ACE, DACL và SACL

Một **ACL** là danh sách quyền gắn với securable object trong Active Directory.

Mỗi ACL gồm nhiều **ACE - Access Control Entry**. Mỗi ACE xác định:

- Trustee: user, group hoặc security principal.
- Access type: Allow hoặc Deny.
- Permission.
- Inheritance.
- Object type mà quyền áp dụng.

Hai thành phần chính:

| Thành phần | Chức năng |
|---|---|
| DACL | Quy định ai được phép hoặc bị từ chối thao tác |
| SACL | Quy định hành vi nào cần được audit |

---

# 2. Securable Object

Securable object là object có security descriptor.

Ví dụ:

- User.
- Group.
- Computer.
- Organizational Unit.
- Group Policy Object.
- Domain root.
- Service Connection Point.

Security descriptor thường chứa:

```text
Owner
DACL
SACL
Inheritance information
```

---

# 3. Delegated Rights

Trong doanh nghiệp lớn, quyền thường được delegate để Service Desk hoặc nhóm vận hành thực hiện một số công việc.

Ví dụ hợp lệ:

- Reset password.
- Unlock account.
- Sửa group membership.
- Quản lý computer object trong một OU.
- Tạo hoặc xóa user trong OU được giao.

Rủi ro xuất hiện khi:

```text
Quyền quá rộng
Inheritance sai
Nhóm được cấp quyền có quá nhiều thành viên
Quyền cũ không bị thu hồi khi nhân viên chuyển bộ phận
```

---

# 4. Những quyền ACL đáng chú ý

| Quyền | Rủi ro |
|---|---|
| GenericAll | Toàn quyền trên object |
| GenericWrite | Sửa nhiều thuộc tính của object |
| WriteDACL | Sửa DACL để tự cấp thêm quyền |
| WriteOwner | Đổi owner rồi sửa quyền |
| ForceChangePassword | Reset password không cần biết password cũ |
| AddMember | Thêm thành viên vào group |
| AllExtendedRights | Có thể bao gồm reset password hoặc quyền mở rộng khác |
| WriteSPN | Thêm SPN để tạo targeted Kerberoasting path |
| WriteAccountRestrictions | Có thể hỗ trợ delegation abuse tùy object |

---

# 5. Ví dụ GenericAll trên User

Nếu user `Bob` có `GenericAll` trên user `Anni`, Bob có thể:

- Reset password của Anni.
- Sửa thuộc tính user.
- Thêm SPN giả rồi thực hiện targeted Kerberoasting.
- Thay đổi một số group/ACL liên quan nếu quyền cho phép.
- Trực tiếp kế thừa quyền của Anni sau khi chiếm account.

```text
Nếu Anni là privileged user, GenericAll có thể trở thành đường leo thang đặc quyền trực tiếp.
```

---

# 6. Ví dụ GenericAll trên Computer

Nếu Bob có full control trên computer object `SERVER01`, các path có thể gồm:

- Sửa computer attributes.
- Abuse Resource-Based Constrained Delegation.
- Thay đổi delegation-related properties.
- Đọc LAPS password nếu quyền extended-right cho phép.
- Tạo đường local-admin hoặc lateral movement gián tiếp.

Nếu `SERVER01` còn được cấu hình Unconstrained Delegation, compromise host này có thể dẫn tới ticket theft và Tier-0 escalation.

---

# 7. Thu thập dữ liệu bằng SharpHound

Trong HTB lab:

```powershell
.\SharpHound.exe -c All
```

`All` thường thu thập:

- Groups.
- Local Admin.
- Sessions.
- Trusts.
- ACLs.
- Containers.
- RDP.
- DCOM.
- PSRemote.
- Object properties.
- GPO relationships.

Output là ZIP dùng để import vào BloodHound.

---

# 8. BloodHound Analysis

Sau khi import ZIP, nên bắt đầu từ user đã compromise.

Các hướng phân tích:

```text
Outbound Object Control
Shortest Paths to High Value Targets
Shortest Paths to Domain Admins
First Degree Object Control
Transitive Object Control
```

Với Bob, cần xem:

- Bob có quyền gì trực tiếp?
- Quyền được kế thừa từ group nào?
- Có đường qua user, group, computer hoặc GPO không?
- Object đích có high-value session hoặc delegation không?

---

# 9. ADACLScanner

Ngoài BloodHound, có thể dùng **ADACLScanner** để tạo báo cáo DACL/SACL.

Ứng dụng:

- Export quyền theo OU/object.
- So sánh baseline.
- Tìm trustee không mong đợi.
- Review inheritance.
- Phục vụ audit và báo cáo.

BloodHound tốt cho attack-path graph; ADACLScanner phù hợp cho ACL reporting chi tiết.

---

# 10. Prevention

Ba nhóm biện pháp chính:

## Continuous Assessment

- Quét ACL định kỳ.
- So sánh với baseline.
- Review high-value objects.
- Theo dõi permission drift.

## Privileged Access Hygiene

- Tách tài khoản quản trị và tài khoản dùng hằng ngày.
- Chỉ cấp quyền qua nhóm được quản lý.
- Dùng JIT/JEA/PAM khi phù hợp.
- Thu hồi quyền ngay khi đổi vai trò.

## Automation

- Tự động hóa user provisioning/deprovisioning.
- Tự động hóa access review.
- Không để admin sửa ACL thủ công nếu có thể.
- Áp approval workflow cho thay đổi quyền.

---

# 11. Event ID 4738

Khi user account bị thay đổi:

```text
Event ID 4738 - A user account was changed
```

Có thể dùng để phát hiện user thấp quyền sửa user khác.

Hạn chế:

```text
4738 không phải lúc nào cũng hiển thị đầy đủ thuộc tính đã bị sửa.
Ví dụ SPN mới có thể không xuất hiện rõ trong event.
```

Vì vậy cần correlate với 5136 hoặc directory-auditing telemetry khác.

---

# 12. Event ID 4724

Nếu ACL abuse dẫn tới password reset:

```text
Event ID 4724 - An attempt was made to reset an account's password
```

Field cần xem:

- Subject Account Name.
- Target Account Name.
- Account Domain.
- Source host qua correlation.

High-confidence condition:

```text
User không thuộc Service Desk/Admin reset password của privileged account.
```

---

# 13. Event ID 4742

Khi computer account bị thay đổi:

```text
Event ID 4742 - A computer account was changed
```

Dùng để phát hiện:

- Low-privileged user sửa computer object.
- Delegation-related modification.
- DNS hostname/SPN change.
- Computer attribute thay đổi ngoài quy trình.

Giống 4738, event này có thể thiếu chi tiết về mọi property đã đổi.

---

# 14. Event ID 5136

Nếu bật **Audit Directory Service Changes**:

```text
Event ID 5136 - A directory service object was modified
```

5136 thường hữu ích hơn để xem:

- Object DN.
- Object GUID.
- Attribute LDAP Display Name.
- Operation Type.
- Value Added/Deleted.

Các attribute nhạy cảm:

```text
servicePrincipalName
nTSecurityDescriptor
member
msDS-AllowedToDelegateTo
msDS-AllowedToActOnBehalfOfOtherIdentity
userAccountControl
```

---

# 15. Bật Directory Service Auditing

Trên Domain Controller:

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

Một số object còn cần SACL phù hợp để tạo đủ telemetry.

---

# 16. Splunk Detection

User/computer object changes:

```spl
index=wineventlog sourcetype=WinEventLog:Security
EventCode IN (4724,4738,4742,5136)
| table _time, host, EventCode, SubjectUserName, TargetUserName,
        ObjectDN, AttributeLDAPDisplayName, OperationType
```

Sensitive attributes:

```spl
index=wineventlog sourcetype=WinEventLog:Security EventCode=5136
AttributeLDAPDisplayName IN (
  "servicePrincipalName",
  "nTSecurityDescriptor",
  "member",
  "msDS-AllowedToDelegateTo",
  "msDS-AllowedToActOnBehalfOfOtherIdentity"
)
| stats count values(ObjectDN) as objects
        values(AttributeLDAPDisplayName) as attributes
  by SubjectUserName
```

Non-admin actor:

```spl
index=wineventlog sourcetype=WinEventLog:Security
EventCode IN (4724,4738,4742,5136)
| where NOT like(SubjectUserName,"admin%")
| sort - _time
```

Trong production nên dùng allowlist nhóm quản trị thật thay vì chỉ naming convention.

---

# 17. ELK/KQL

```kql
event.code: ("4724" or "4738" or "4742" or "5136")
```

Sensitive attribute modification:

```kql
event.code: "5136"
and winlog.event_data.AttributeLDAPDisplayName: (
  "servicePrincipalName"
  or "nTSecurityDescriptor"
  or "msDS-AllowedToDelegateTo"
  or "msDS-AllowedToActOnBehalfOfOtherIdentity"
)
```

---

# 18. Sigma Rule

```yaml
title: Sensitive Active Directory Object Attribute Modified
status: experimental
logsource:
  product: windows
  service: security
detection:
  selection:
    EventID: 5136
    AttributeLDAPDisplayName:
      - 'servicePrincipalName'
      - 'nTSecurityDescriptor'
      - 'msDS-AllowedToDelegateTo'
      - 'msDS-AllowedToActOnBehalfOfOtherIdentity'
  condition: selection
falsepositives:
  - Approved identity and delegation administration
level: high
```

---

# 19. Correlation Logic

Ví dụ targeted Kerberoasting path:

```text
5136: servicePrincipalName được thêm vào Anni
        ↓
4769: TGS được request cho SPN mới
        ↓
Password cracking xảy ra offline
        ↓
4624/4768: Anni được dùng từ source host mới
```

Ví dụ password-reset path:

```text
4724: Bob reset password Anni
        ↓
4624: Anni đăng nhập từ Bob's workstation
```

Ví dụ RBCD path:

```text
5136: msDS-AllowedToActOnBehalfOfOtherIdentity bị sửa
        ↓
4769: S4U/service-ticket activity
        ↓
4624: privileged logon trên target computer
```

---

# 20. Honeypot ACL

Hai mô hình honeypot:

## High-value-looking user

- Credential giả được lộ ở Description.
- Account có ACL nhìn hấp dẫn.
- Password thật mạnh.
- Mọi authentication attempt được alert.

## Modifiable honeypot user

- Nhiều user có quyền sửa account.
- Account có activity giả hợp lý.
- Không có lý do kinh doanh để ai sửa.
- Mọi 4738/4724/5136 phải alert.

Guardrails:

```text
Không cấp quyền thực tế nguy hiểm.
Không để honeypot trở thành privilege-escalation path thật.
Có playbook disable actor và điều tra source host.
```

---

# 21. Threat Hunting

Hunt theo:

- GenericAll/WriteDACL/WriteOwner từ low-privileged users.
- User thường sửa privileged user.
- SPN mới trên user không phải service account.
- Computer object bị sửa bởi user thường.
- RBCD attribute change.
- Password reset ngoài Service Desk.
- Permission drift trên high-value objects.
- SharpHound/BloodHound collection từ workstation lạ.

---

# 22. Incident Response Checklist

```text
[ ] Xác định actor và target object.
[ ] Xác định quyền ACL đã bị lạm dụng.
[ ] Review 4724, 4738, 4742, 5136.
[ ] Kiểm tra group membership và inheritance.
[ ] Review SPN/delegation changes.
[ ] Xác định credential đã bị reset hoặc account bị takeover chưa.
[ ] Rotate credential nếu cần.
[ ] Remove malicious ACE.
[ ] Reapply approved ACL baseline.
[ ] Hunt Kerberoasting, RBCD và lateral movement.
[ ] Triage source workstation.
```

---

# 23. Mapping với CDSA

## Security Operations & Monitoring

- Thu 4724, 4738, 4742, 5136.
- Baseline approved ACL managers.
- Monitor high-value object changes.
- Correlate AD changes với authentication.

## Incident Response & Forensics

- Timeline ACL/object modification.
- Xác định source account và workstation.
- So sánh ACL trước/sau.
- Hunt credential use sau thay đổi.

## Threat Hunting

- BloodHound từ góc nhìn defensive.
- Hunt low-privileged object control.
- Hunt sensitive attribute changes.
- Hunt privilege paths mới xuất hiện.

## Detection Engineering

- Sigma cho 5136.
- Splunk/Elastic correlation.
- ACL baseline comparison.
- Honeypot object detection.

---

# 24. Lab Setup

Môi trường tối thiểu:

```text
1 Domain Controller
1 Windows workstation
BloodHound + Neo4j/BloodHound database
SharpHound collector
Splunk/ELK/Wazuh nhận Security logs
Sysmon trên workstation
```

Workflow:

```text
1. Chạy SharpHound trong lab.
2. Import ZIP vào BloodHound.
3. Chọn user compromise làm starting node.
4. Review Outbound Object Control.
5. Thực hiện thay đổi ACL/object có kiểm soát.
6. Xác minh 4724/4738/4742/5136 trong SIEM.
7. Viết detection và rollback lab.
```

---

# Key Takeaway

```text
ACL quyết định ai có thể làm gì với AD object.
GenericAll, WriteDACL, WriteOwner và ForceChangePassword là các quyền rất nguy hiểm.
BloodHound giúp biến ACL phức tạp thành attack-path graph.
4738 và 4742 báo object bị sửa nhưng thường thiếu chi tiết.
5136 cung cấp visibility tốt hơn về attribute thay đổi nếu auditing đúng.
Detection hiệu quả cần correlate object change với Kerberos/logon activity.
```

Object ACLs là chủ đề cốt lõi của CDSA vì kết nối trực tiếp giữa Active Directory misconfiguration, privilege escalation, Windows logging, threat hunting và incident response.

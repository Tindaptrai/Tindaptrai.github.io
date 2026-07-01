---
layout: post
title: "Active Directory Terminology"
date: 2026-07-01 22:16:09 +0700
categories: [Active Directory, Windows Security]
tags: [active-directory, ad-ds, ldap, object, schema, guid, sid, dn, rdn, samaccountname, upn, fsmo, global-catalog, rodc, replication, spn, gpo, acl, ace, dacl, sacl, sysvol, ntds-dit, blue-team, red-team]
description: "Tóm tắt Active Directory Terminology: Object, Attributes, Schema, Domain, Forest, Tree, GUID, SID, DN/RDN, sAMAccountName, UPN, FSMO roles, Global Catalog, RODC, Replication, SPN, GPO, ACL/ACE/DACL/SACL, SYSVOL, AdminSDHolder, NTDS.DIT và các thuật ngữ quan trọng trong AD."
toc: true
---

# Active Directory Terminology

Bài này tổng hợp các thuật ngữ quan trọng khi học và làm việc với **Active Directory (AD)**.

```text
Muốn hiểu AD attack/defense thì phải hiểu object, attribute, schema, domain, forest, SID, GUID, DN, ACL, GPO, SPN và replication.
```

---

## Timeline cập nhật

```text
Created at: 2026-07-01 22:16:09 +0700
Timezone: Asia/Ho_Chi_Minh
Topic: Active Directory Terminology
Focus: AD objects, identity, naming, FSMO, GC, ACL, GPO, SYSVOL, NTDS.DIT
```

---

# 1. Object, Attributes và Schema

## Object

**Object** là bất kỳ tài nguyên nào tồn tại trong AD:

- User
- Computer
- Group
- Organizational Unit
- Printer
- Domain Controller
- GPO
- Service account

## Attributes

**Attributes** là thuộc tính mô tả object. Ví dụ:

| Object | Attributes thường gặp |
|---|---|
| User | `displayName`, `givenName`, `mail`, `sAMAccountName`, `userPrincipalName` |
| Computer | `dNSHostName`, `operatingSystem`, `lastLogonTimestamp` |
| AD object | `objectGUID`, `objectSID` |

## Schema

**Schema** là blueprint của AD, định nghĩa:

- Object class nào được tồn tại.
- Attribute nào thuộc class nào.
- Attribute nào bắt buộc hoặc optional.

Ví dụ:

```text
User object thuộc class user.
Computer object thuộc class computer.
RDS01 là instance của computer class.
```

---

# 2. Domain, Forest và Tree

## Domain

**Domain** là nhóm logic của object như user, computer, group và OU.

```text
inlanefreight.local
```

Domain có thể hoạt động độc lập hoặc kết nối với domain khác qua trust.

## Forest

**Forest** là container cấp cao nhất và là security boundary lớn nhất trong AD.

Forest có thể chứa:

- Một hoặc nhiều domain.
- Một hoặc nhiều tree.
- Shared schema.
- Global Catalog.
- Trust relationships.

## Tree

**Tree** là tập hợp domain bắt đầu từ một root domain và chia sẻ namespace.

Ví dụ:

```text
inlanefreight.local
└── corp.inlanefreight.local
```

Một forest có thể có nhiều tree, nhưng hai tree trong cùng forest không dùng chung namespace.

---

# 3. Container và Leaf

## Container

**Container** là object có thể chứa object khác.

Ví dụ:

- Users
- Computers
- Builtin
- Domain Controllers

## Leaf

**Leaf object** là object cuối nhánh, không chứa object khác.

Ví dụ:

- User account
- Computer account
- Printer object

---

# 4. GUID, Security Principal và SID

## GUID

**Global Unique Identifier (GUID)** là giá trị 128-bit duy nhất được gán cho mỗi AD object.

Attribute:

```text
objectGUID
```

Đặc điểm quan trọng:

```text
objectGUID không đổi trong suốt vòng đời object.
```

## Security Principal

**Security principal** là bất kỳ thực thể nào hệ điều hành có thể authenticate:

- User
- Computer account
- Service account
- Process/thread chạy dưới context user/computer
- Security group

## SID

**Security Identifier (SID)** là định danh duy nhất cho security principal hoặc security group.

Khi user logon, Windows tạo access token chứa:

- User SID
- Group SIDs
- Rights/privileges

Access token được dùng để kiểm tra quyền khi user truy cập tài nguyên.

---

# 5. DN, RDN, sAMAccountName và UPN

## Distinguished Name

**DN** là đường dẫn đầy đủ đến object trong AD.

Ví dụ:

```text
cn=bjones,ou=IT,ou=Employees,dc=inlanefreight,dc=local
```

## Relative Distinguished Name

**RDN** là một thành phần trong DN.

Ví dụ:

```text
cn=bjones
```

AD không cho phép hai object có cùng RDN dưới cùng parent container.

## sAMAccountName

**sAMAccountName** là logon name kiểu cũ.

```text
bjones
INLANEFREIGHT\bjones
```

Đặc điểm:

- Unique trong domain.
- Tối đa 20 ký tự.

## userPrincipalName

**UPN** có format giống email:

```text
bjones@inlanefreight.local
```

---

# 6. FSMO Roles

**FSMO** là viết tắt của:

```text
Flexible Single Master Operation
```

Có 5 FSMO roles:

| FSMO Role | Phạm vi |
|---|---|
| Schema Master | Forest-wide |
| Domain Naming Master | Forest-wide |
| RID Master | Domain-wide |
| PDC Emulator | Domain-wide |
| Infrastructure Master | Domain-wide |

FSMO giúp các tác vụ quan trọng trong AD tránh conflict và replication chạy ổn định hơn.

---

# 7. Global Catalog, RODC và Replication

## Global Catalog

**Global Catalog (GC)** là DC lưu:

```text
Full copy object trong domain hiện tại.
Partial copy object ở domain khác trong forest.
```

GC hỗ trợ:

- Object search toàn forest.
- Authentication.
- Group membership khi tạo access token.

## Read-Only Domain Controller

**RODC** là DC có AD database read-only.

Đặc điểm:

- Không push thay đổi AD database.
- Không push thay đổi SYSVOL/DNS.
- Giảm replication traffic.
- Phù hợp branch office hoặc location ít tin cậy hơn.

## Replication

**Replication** là quá trình đồng bộ thay đổi giữa các DC.

Dịch vụ hỗ trợ tạo connection replication:

```text
Knowledge Consistency Checker (KCC)
```

---

# 8. SPN và GPO

## Service Principal Name

**SPN** định danh duy nhất một service instance.

Ví dụ:

```text
HTTP/web01.inlanefreight.local
MSSQLSvc/sql01.inlanefreight.local:1433
```

SPN được Kerberos dùng để map service với logon account. SPN cũng liên quan đến attack như Kerberoasting.

## Group Policy Object

**GPO** là tập hợp policy settings, có thể áp dụng cho:

- Site
- Domain
- OU

GPO thường quản lý:

- Password policy
- Firewall
- Windows Defender
- Logon scripts
- Drive mappings
- Audit policy
- Security baseline

---

# 9. ACL, ACE, DACL và SACL

| Thuật ngữ | Ý nghĩa |
|---|---|
| ACL | Danh sách ACE áp dụng lên object |
| ACE | Entry xác định trustee và quyền allow/deny/audit |
| DACL | Xác định principal nào được grant/deny access |
| SACL | Xác định access attempt nào được audit |

Nếu object không có DACL, hệ thống có thể grant full access cho everyone. Nếu DACL tồn tại nhưng không có ACE, access sẽ bị deny.

---

# 10. FQDN, Tombstone và AD Recycle Bin

## FQDN

**FQDN** là tên đầy đủ của host:

```text
DC01.INLANEFREIGHT.LOCAL
```

## Tombstone

**Tombstone** giữ object đã bị xóa trong một khoảng thời gian trước khi xóa hoàn toàn.

Khi object bị xóa:

```text
isDeleted = TRUE
```

## AD Recycle Bin

**AD Recycle Bin** giúp khôi phục object đã xóa dễ hơn và giữ lại phần lớn attributes của object.

---

# 11. SYSVOL, AdminSDHolder, dsHeuristics và adminCount

## SYSVOL

**SYSVOL** lưu các public files trong domain:

- System policies
- Group Policy settings
- Logon/logoff scripts
- Scripts trong AD environment

## AdminSDHolder

**AdminSDHolder** quản lý ACL cho các protected/privileged groups.

SDProp process chạy trên PDC Emulator để kiểm tra ACL của protected groups, mặc định khoảng mỗi 1 giờ.

## dsHeuristics

**dsHeuristics** là forest-wide configuration attribute. Một setting có thể exclude built-in groups khỏi Protected Groups list.

## adminCount

**adminCount** cho biết user có được SDProp bảo vệ hay không:

```text
0 hoặc không set = không protected
1 = protected
```

Account có `adminCount=1` thường đáng chú ý vì có thể từng hoặc đang có đặc quyền cao.

---

# 12. ADUC, ADSI Edit, sIDHistory, NTDS.DIT và MSBROWSE

## ADUC

**Active Directory Users and Computers (ADUC)** là GUI console phổ biến để quản lý users, groups, computers, contacts và OUs.

## ADSI Edit

**ADSI Edit** là GUI tool mạnh hơn ADUC, cho phép chỉnh sâu object và attribute. Cần cẩn thận vì chỉnh sai có thể gây lỗi nghiêm trọng trong AD.

## sIDHistory

**sIDHistory** lưu SID trước đây của object, thường dùng trong migration. Nếu SID Filtering không bật hoặc cấu hình sai, attribute này có thể bị abuse.

## NTDS.DIT

**NTDS.DIT** là database trung tâm của AD trên Domain Controller.

Đường dẫn thường gặp:

```text
C:\Windows\NTDS\NTDS.DIT
```

NTDS.DIT chứa AD data và password hashes của domain users.

## MSBROWSE

**MSBROWSE** là Microsoft networking protocol cũ dùng cho browsing services trong LAN Windows đời trước. Hiện nay phần lớn đã lỗi thời, môi trường hiện đại chủ yếu dùng SMB/CIFS.

---

# 13. Bảng tóm tắt nhanh

| Thuật ngữ | Ý nghĩa ngắn |
|---|---|
| Object | Tài nguyên trong AD |
| Attribute | Thuộc tính của object |
| Schema | Blueprint của AD |
| Domain | Nhóm logic các object |
| Forest | Security boundary lớn nhất |
| Tree | Tập domain chung namespace |
| GUID | ID 128-bit của object |
| SID | ID của security principal |
| DN | Full path đến object |
| RDN | Một phần của DN |
| sAMAccountName | Logon name kiểu cũ |
| UPN | username@domain |
| FSMO | 5 role master đặc biệt |
| GC | Global Catalog |
| RODC | Read-only DC |
| SPN | ID của service instance |
| GPO | Policy settings |
| ACL/ACE | Danh sách quyền và từng entry quyền |
| DACL/SACL | Access control và audit control |
| SYSVOL | Share chứa policy/script |
| NTDS.DIT | Database AD trên DC |

---

# Key Takeaway

Active Directory terminology có thể gom thành vài nhóm chính:

```text
Identity: user, computer, security principal, SID, GUID
Structure: forest, tree, domain, OU, container, leaf
Naming: DN, RDN, sAMAccountName, UPN, FQDN
Control: GPO, ACL, ACE, DACL, SACL
Infrastructure: DC, GC, RODC, replication, FSMO, SYSVOL, NTDS.DIT
Security: SPN, AdminSDHolder, adminCount, sIDHistory
```

Nắm chắc các thuật ngữ này giúp phân tích AD rõ hơn, hiểu attack path tốt hơn và xây dựng chiến lược phòng thủ domain hiệu quả hơn.

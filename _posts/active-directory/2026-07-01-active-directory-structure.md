---
layout: post
title: "Active Directory Structure"
date: 2026-07-01 00:06:02 +0700
categories: [Active Directory, Windows Security]
tags: [active-directory, ad-ds, windows-domain, forest, domain, child-domain, organizational-unit, gpo, trust, acl, red-team, blue-team]
description: "Tóm tắt Active Directory Structure: AD DS, forest, domain, child domain, OU, GPO, trust relationship, bidirectional trust và các đối tượng có thể enumeration trong môi trường Windows domain."
toc: true
---

# Active Directory Structure

**Active Directory (AD)** là directory service cho môi trường mạng Windows. AD cho phép quản lý tập trung tài nguyên của tổ chức như user, computer, group, file share, policy, server, workstation và trust relationship.

```text
Active Directory = hệ thống quản lý định danh, tài nguyên, xác thực và phân quyền trong Windows domain environment.
```

---

## Timeline cập nhật

```text
Created at: 2026-07-01 00:06:02 +0700
Timezone: Asia/Ho_Chi_Minh
Topic: Active Directory Structure
Focus: AD DS, Forest, Domain, Child Domain, OU, GPO, Trust, ACL
```

---

# 1. Active Directory là gì?

Active Directory là một cấu trúc phân tán và phân cấp dùng trong Windows network environment.

AD hỗ trợ quản lý tập trung:

- Users.
- Computers.
- Groups.
- Network devices.
- File shares.
- Servers.
- Workstations.
- Group Policies.
- Trust relationships.
- Access control.

Trong Windows domain environment, AD cung cấp:

```text
Authentication
Authorization
```

---

# 2. Active Directory Domain Services

**Active Directory Domain Services (AD DS)** là dịch vụ lưu trữ directory data và cung cấp dữ liệu đó cho user, computer và administrator trong cùng network.

AD DS lưu trữ các thông tin như:

- Username.
- Password hash.
- Computer accounts.
- Group membership.
- Policies.
- Permissions.
- Directory objects.
- Trust information.

Nói dễ hiểu:

```text
AD DS là database trung tâm của Windows domain.
```

---

# 3. Vì sao AD quan trọng trong bảo mật?

AD thường là mục tiêu lớn trong enterprise vì nó quản lý identity và quyền truy cập.

Nếu AD bị misconfiguration hoặc bị compromise, attacker có thể:

- Lấy foothold trong internal network.
- Enumerate user/computer/group.
- Di chuyển ngang trong mạng.
- Leo thang đặc quyền.
- Truy cập file share, database, source code.
- Tấn công Domain Admin hoặc Domain Controller.
- Abuse trust relationships.

```text
Hiểu AD giúp defender biết cần bảo vệ gì.
Hiểu AD cũng giúp red team biết attack path thường bắt đầu từ đâu.
```

---

# 4. Basic user có thể enumeration gì trong AD?

Một user thường, không có đặc quyền cao, vẫn có thể enumerate nhiều object trong AD.

| Object / Information | Ý nghĩa |
|---|---|
| Domain Computers | Danh sách computer trong domain |
| Domain Users | Danh sách user |
| Domain Groups | Nhóm và membership |
| Organizational Units | OU structure |
| Default Domain Policy | Policy mặc định |
| Functional Domain Levels | Level chức năng của domain/forest |
| Password Policy | Chính sách password |
| Group Policy Objects | GPO đang áp dụng |
| Domain Trusts | Trust relationship |
| Access Control Lists | ACL/permission trên object |

---

# 5. Cấu trúc phân cấp của Active Directory

Active Directory được tổ chức theo dạng hierarchical tree.

```text
Forest
└── Domain
    └── Child Domain / Subdomain
        └── Organizational Unit
            └── Objects
```

Các object thường gặp:

- Users.
- Groups.
- Computers.
- Servers.
- Printers.
- Service accounts.
- GPOs.
- OUs.

---

# 6. Forest

**Forest** là cấp cao nhất trong AD structure.

Forest có thể chứa một hoặc nhiều domain.

```text
Forest = security boundary lớn nhất trong Active Directory.
```

Một forest có thể có:

- Root domain.
- Child domains.
- Tree root domains.
- Trusts.
- Shared schema.
- Global catalog.
- Enterprise-wide configuration.

---

# 7. Domain

**Domain** là một boundary logic chứa object như user, computer và group.

Domain cung cấp:

- Authentication boundary.
- Policy boundary.
- Administrative boundary.
- Naming boundary.

Ví dụ domain:

```text
inlanefreight.local
```

Một domain có thể chứa:

- Users.
- Computers.
- Groups.
- Domain Controllers.
- Built-in OUs.
- Custom OUs.
- Group Policies.

---

# 8. Child Domain và Subdomain

Domain có thể có child domain hoặc subdomain.

```text
inlanefreight.local
├── corp.inlanefreight.local
├── dev.inlanefreight.local
└── admin.inlanefreight.local
```

Trong ví dụ này:

```text
inlanefreight.local = root domain
corp.inlanefreight.local = child/subdomain
dev.inlanefreight.local = child/subdomain
admin.inlanefreight.local = child/subdomain
```

---

# 9. Organizational Unit

**Organizational Unit (OU)** là container trong domain dùng để tổ chức object.

OU có thể chứa:

- Users.
- Computers.
- Groups.
- Service accounts.
- Sub-OUs.

Ví dụ:

```text
OU=EMPLOYEES
├── OU=COMPUTERS
│   └── FILE01
├── OU=GROUPS
│   └── HQ Staff
└── OU=USERS
    └── barbara.jones
```

OU thường được dùng để:

- Quản lý object theo phòng ban.
- Áp dụng Group Policy.
- Delegate administrative permissions.
- Tổ chức computer/user rõ ràng hơn.

---

# 10. Group Policy Objects

**Group Policy Objects (GPOs)** là policy dùng để cấu hình user và computer trong AD.

GPO có thể áp dụng cho:

- Site.
- Domain.
- OU.

GPO thường quản lý:

- Password policy.
- Local admin settings.
- Firewall.
- Windows Defender.
- Software restrictions.
- Login scripts.
- Drive mappings.
- Security baseline.
- Audit policy.

```text
OU giúp tổ chức object.
GPO giúp áp chính sách lên object.
```

---

# 11. Ví dụ AD structure

```text
INLANEFREIGHT.LOCAL/
├── ADMIN.INLANEFREIGHT.LOCAL
│   ├── GPOs
│   └── OU
│       └── EMPLOYEES
│           ├── COMPUTERS
│           │   └── FILE01
│           ├── GROUPS
│           │   └── HQ Staff
│           └── USERS
│               └── barbara.jones
├── CORP.INLANEFREIGHT.LOCAL
└── DEV.INLANEFREIGHT.LOCAL
```

Trong cấu trúc này:

```text
INLANEFREIGHT.LOCAL = root domain
ADMIN.INLANEFREIGHT.LOCAL = child domain
CORP.INLANEFREIGHT.LOCAL = child domain
DEV.INLANEFREIGHT.LOCAL = child domain
```

---

# 12. Trust Relationship

**Trust relationship** cho phép user ở domain/forest này truy cập resource ở domain/forest khác nếu được phân quyền phù hợp.

Trust có thể là:

- One-way trust.
- Two-way trust.
- Parent-child trust.
- Tree-root trust.
- External trust.
- Forest trust.

---

## 12.1. Bidirectional Trust

Bidirectional trust là trust hai chiều.

```text
Forest A <----> Forest B
```

Ý nghĩa:

```text
User trong Forest A có thể được phép truy cập resource trong Forest B.
User trong Forest B có thể được phép truy cập resource trong Forest A.
```

Tuy nhiên, trust không có nghĩa là tự động có toàn quyền. User vẫn cần authentication hợp lệ và authorization phù hợp.

---

# 13. Trust không phải lúc nào cũng áp dụng đến mọi child domain

Ví dụ có hai forest:

```text
Forest A: inlanefreight.local
Forest B: freightlogistics.local
```

Có bidirectional trust giữa root domains:

```text
inlanefreight.local <----> freightlogistics.local
```

Nhưng điều đó không nhất thiết có nghĩa là:

```text
admin.dev.freightlogistics.local
```

có thể authenticate trực tiếp vào:

```text
wh.corp.inlanefreight.local
```

Nếu muốn direct communication giữa các child domains cụ thể, có thể cần thiết lập trust riêng hoặc cấu hình quyền phù hợp.

---

# 14. Ví dụ forests và domains

Forest A:

```text
inlanefreight.local
├── corp.inlanefreight.local
│   └── wh.corp.inlanefreight.local
└── dev.inlanefreight.local
    └── admin.dev.inlanefreight.local
```

Forest B:

```text
freightlogistics.local
├── corp.freightlogistics.local
└── dev.freightlogistics.local
    └── admin.dev.freightlogistics.local
```

Trust cấp root:

```text
inlanefreight.local <----> freightlogistics.local
```

---

# 15. Security implications

AD trust và cấu trúc domain có thể tạo rủi ro nếu quản trị không tốt.

Các vấn đề thường gặp:

- Trust quá rộng.
- Permission sai trên object.
- ACL misconfiguration.
- GPO misconfiguration.
- Weak password policy.
- User có quyền dư thừa.
- Service account có privilege cao.
- Child domain bị compromise ảnh hưởng parent/forest.
- External trust không kiểm soát tốt.
- SID History abuse.
- Kerberoasting.
- LDAP enumeration.
- Lateral movement qua SMB/RDP/WinRM.

---

# 16. Blue Team cần chú ý gì?

Khi bảo vệ AD, cần quan tâm:

- Asset inventory.
- Domain Controller security.
- Tiering model.
- Privileged access management.
- Least privilege.
- GPO review.
- ACL review.
- Trust review.
- Password policy.
- Audit policy.
- AD logs.
- Kerberos logs.
- LDAP enumeration detection.
- Lateral movement detection.
- Backup và recovery plan cho AD.

---

# 17. Red Team / Pentest mindset

Khi đánh giá AD, cần hiểu:

```text
AD là database lớn mà user thường có thể query được.
```

Các bước phổ biến:

```text
1. Enumerate domain.
2. Enumerate users/groups/computers.
3. Identify privileged groups.
4. Review GPOs and ACLs.
5. Map trust relationships.
6. Find attack paths.
7. Test misconfigurations.
8. Attempt privilege escalation if authorized.
```

---

# 18. Bảng tóm tắt thuật ngữ

| Thuật ngữ | Ý nghĩa |
|---|---|
| AD | Active Directory |
| AD DS | Active Directory Domain Services |
| Forest | Security boundary lớn nhất |
| Domain | Logical boundary chứa users/computers/groups |
| Child Domain | Domain con dưới parent/root domain |
| OU | Container tổ chức object |
| GPO | Chính sách áp lên site/domain/OU |
| Trust | Quan hệ tin cậy giữa domain/forest |
| Bidirectional Trust | Trust hai chiều |
| ACL | Access Control List |
| Domain Controller | Server xử lý authentication/authorization |
| Root Domain | Domain gốc trong forest |

---

# Key Takeaway

Active Directory là nền tảng identity của Windows enterprise.

```text
Forest chứa domain.
Domain chứa object.
OU tổ chức object.
GPO áp policy.
Trust kết nối domain/forest.
```

Vì AD thường cho phép user thường enumerate nhiều thông tin, defender cần hiểu cấu trúc AD để giảm misconfiguration, kiểm soát trust, audit quyền và phát hiện lateral movement hoặc privilege escalation path.

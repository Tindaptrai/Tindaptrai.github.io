---
layout: post
title: "Active Directory Objects"
date: 2026-07-01 22:57:06 +0700
categories: [Active Directory, Windows Security]
tags: [active-directory, ad-ds, ad-objects, users, computers, groups, organizational-units, domain-controllers, shared-folders, foreign-security-principals, sid, guid, blue-team, red-team]
description: "Tóm tắt Active Directory Objects: Users, Contacts, Printers, Computers, Shared Folders, Groups, OUs, Domains, Domain Controllers, Sites, Built-in container và Foreign Security Principals trong AD."
toc: true
---

# Active Directory Objects

Trong Active Directory, **object** là bất kỳ tài nguyên nào tồn tại trong môi trường AD.

Ví dụ:

```text
Domain
Computer
OU
Group
User
Printer
Domain Controller
Shared Folder
```

---

## Timeline cập nhật

```text
Created at: 2026-07-01 22:57:06 +0700
Timezone: Asia/Ho_Chi_Minh
Topic: Active Directory Objects
Focus: Users, Contacts, Printers, Computers, Shared Folders, Groups, OUs, Domains, DCs, Sites, FSPs
```

---

# 1. AD Object là gì?

**Object** là resource được AD lưu trữ và quản lý.

Một object có thể có:

- Attributes.
- GUID.
- SID nếu là security principal.
- Vị trí trong AD hierarchy.
- Permission hoặc relationship với object khác.

---

# 2. Leaf Object và Container Object

| Loại | Ý nghĩa | Ví dụ |
|---|---|---|
| Leaf Object | Không chứa object khác | User, Computer, Printer, Contact |
| Container Object | Có thể chứa object khác | Group, OU, Domain |

```text
Leaf = object cuối nhánh.
Container = object có thể chứa object khác.
```

---

# 3. Users

**User object** đại diện cho người dùng trong AD.

User là:

```text
Leaf object
Security principal
Có SID
Có GUID
```

User có nhiều attributes như:

- Display name.
- Last login time.
- Last password change.
- Email address.
- Description.
- Manager.
- Group membership.

Trong bảo mật AD, user là mục tiêu quan trọng vì compromise một low-privileged user vẫn có thể cho phép attacker enumerate domain.

---

# 4. Contacts

**Contact object** thường đại diện cho external user, vendor hoặc customer.

Contact là:

```text
Leaf object
Không phải security principal
Không có SID
Có GUID
```

Contact thường chứa:

- First name.
- Last name.
- Email.
- Phone number.

Contact không dùng để authenticate vào domain.

---

# 5. Printers

**Printer object** trỏ đến printer trong AD network.

Printer là:

```text
Leaf object
Không phải security principal
Không có SID
Có GUID
```

Printer attributes có thể gồm:

- Printer name.
- Driver information.
- Port number.
- Location.

---

# 6. Computers

**Computer object** đại diện cho máy tính join domain.

Computer có thể là workstation hoặc server.

Computer là:

```text
Leaf object
Security principal
Có SID
Có GUID
```

Nếu attacker có quyền admin trên computer và chạy dưới context:

```text
NT AUTHORITY\SYSTEM
```

họ có thể thực hiện nhiều tác vụ enumeration giống domain user thông thường.

---

# 7. Shared Folders

**Shared folder object** trỏ đến folder được share trên một computer cụ thể.

Shared folder là:

```text
Không phải security principal
Không có SID
Có GUID
```

Access có thể được cấu hình cho:

- Everyone.
- Authenticated Users.
- Specific users.
- Specific groups.

Shared folders cần được audit vì có thể chứa tài liệu, script, backup hoặc credential nhạy cảm.

---

# 8. Groups

**Group object** dùng để gom nhiều object nhằm quản lý permission.

Group là:

```text
Container object
Security principal
Có SID
Có GUID
```

Group có thể chứa:

- Users.
- Computers.
- Other groups.

Khi group nằm trong group khác, gọi là:

```text
Nested Groups
```

Nested group có thể tạo ra quyền ngoài ý muốn và thường là attack path quan trọng trong AD.

---

# 9. Organizational Units

**Organizational Unit (OU)** là container dùng để tổ chức object.

OU có thể chứa:

- Users.
- Groups.
- Computers.
- Child OUs.

OU thường dùng để:

- Quản lý object theo phòng ban.
- Delegate administrative tasks.
- Áp dụng Group Policy.
- Tách privileged accounts.
- Phân quyền help desk reset password.

Ví dụ:

```text
Employees
├── Marketing
├── HR
├── Finance
└── Help Desk
```

---

# 10. Domains

**Domain** là cấu trúc chính của AD network.

Domain chứa:

- Users.
- Computers.
- Groups.
- OUs.
- Domain Controllers.
- Policies.

Mỗi domain có database riêng và policy riêng, ví dụ domain password policy.

---

# 11. Domain Controllers

**Domain Controller (DC)** là bộ não của AD domain.

DC xử lý:

- Authentication requests.
- User verification.
- Authorization checks.
- Security policy enforcement.
- Lưu thông tin AD objects.
- Replication với DC khác.

```text
Domain Controller = bộ não của AD domain.
```

---

# 12. Sites

**Site** là tập hợp computer trên một hoặc nhiều subnet được kết nối bằng high-speed links.

Site dùng để:

- Tối ưu replication giữa DCs.
- Điều hướng authentication đến DC gần nhất.
- Giảm chi phí WAN replication.

---

# 13. Built-in Container

**Built-in** là container chứa default groups khi AD domain được tạo.

Ví dụ:

- Administrators.
- Users.
- Guests.
- Backup Operators.
- Account Operators.
- Server Operators.
- Remote Desktop Users.

Các group built-in cần được audit kỹ vì một số group có quyền cao.

---

# 14. Foreign Security Principals

**Foreign Security Principal (FSP)** là object đại diện cho security principal đến từ trusted external forest.

FSP được tạo khi external user/group/computer được thêm vào group trong domain hiện tại.

FSP thường nằm trong:

```text
CN=ForeignSecurityPrincipals,DC=inlanefreight,DC=local
```

Windows dùng SID trong FSP để resolve object name qua trust relationship.

---

# 15. Security Principal vs Non-Security Principal

| Object | Security Principal? | Có SID? | Có GUID? |
|---|---:|---:|---:|
| User | Có | Có | Có |
| Computer | Có | Có | Có |
| Group | Có | Có | Có |
| Contact | Không | Không | Có |
| Printer | Không | Không | Có |
| Shared Folder | Không | Không | Có |

```text
Security principal = có thể được authenticate hoặc được cấp quyền.
Non-security principal = object/resource/information, không authenticate trực tiếp.
```

---

# 16. AD Objects trong góc nhìn bảo mật

| Object | Vì sao quan trọng |
|---|---|
| Users | Login, enumerate, password attacks |
| Computers | Session, local admin, lateral movement |
| Groups | Reveal privilege path |
| OUs | Reveal delegation/GPO scope |
| Shared Folders | Có thể chứa credential/file nhạy cảm |
| Domain Controllers | Critical target |
| FSPs | Reveal trust/external access path |

---

# 17. Blue Team Checklist

Blue Team nên audit:

- User privilege.
- Computer local admin.
- Group nesting.
- Privileged groups.
- OU delegation.
- GPO scope.
- Shared folder permissions.
- Domain Controller access.
- Built-in group membership.
- Foreign Security Principals.
- Stale users/computers.
- Service accounts.

---

# 18. Bảng tóm tắt

| Object | Loại | Ý nghĩa |
|---|---|---|
| User | Leaf / Security Principal | Người dùng trong domain |
| Contact | Leaf / Non-security | External contact |
| Printer | Leaf / Non-security | Printer trong AD |
| Computer | Leaf / Security Principal | Máy tính join domain |
| Shared Folder | Non-security | Folder share trên host |
| Group | Container / Security Principal | Gom users/computers/groups |
| OU | Container | Tổ chức object, delegate, apply GPO |
| Domain | Container | Cấu trúc chính của AD |
| Domain Controller | Critical server | Xử lý auth và lưu AD data |
| Site | Logical/physical mapping | Tối ưu replication/auth |
| Built-in | Container | Chứa default groups |
| FSP | Placeholder | Đại diện object từ trusted forest |

---

# Key Takeaway

AD Objects là nền tảng để hiểu cách Active Directory vận hành.

```text
User, Computer, Group = security principals.
Contact, Printer, Shared Folder = non-security principals.
OU, Group, Domain = container/object tổ chức.
Domain Controller = bộ não của domain.
Foreign Security Principal = đại diện object từ trusted forest.
```

Khi phân tích AD, luôn xem object đó là loại gì, có SID/GUID không, thuộc container nào, bị GPO nào áp dụng và có thể tạo ra attack path hoặc rủi ro phân quyền nào không.

---
layout: post
title: "Active Directory Groups"
date: 2026-07-04 22:11:48 +0700
categories: [Active Directory, Windows Security]
tags: [active-directory, groups, security-groups, distribution-groups, group-scope, domain-local, global-group, universal-group, nested-groups, bloodhound, sid, blue-team, red-team]
description: "Tóm tắt ngắn về Active Directory Groups: security/distribution groups, group scopes, built-in/custom groups, nested group membership, BloodHound và các group attributes quan trọng."
toc: true
---

# Active Directory Groups

Sau **users**, **groups** là một object rất quan trọng trong Active Directory. Groups giúp gom user/computer/contact lại để quản lý quyền truy cập dễ hơn.

```text
Group = dùng để gom object và cấp quyền hàng loạt.
OU = dùng để tổ chức object và áp Group Policy/delegate admin task.
```

---

## Timeline cập nhật

```text
Created at: 2026-07-04 22:11:48 +0700
Timezone: Asia/Ho_Chi_Minh
Topic: Active Directory Groups
Focus: Group type, group scope, nested groups, built-in/custom groups, BloodHound, group attributes
```

---

# 1. Groups dùng để làm gì?

Groups trong AD thường dùng để:

- Gom user/computer/contact thành một đơn vị quản lý.
- Cấp quyền truy cập file share, printer, application.
- Giảm công sức cấp quyền từng user.
- Dễ audit ai có quyền gì.
- Dễ thu hồi quyền bằng cách remove user khỏi group.

Ví dụ:

```text
Thay vì cấp quyền share drive cho 50 user
→ tạo group Department-Share-Access
→ cấp quyền cho group
→ user trong group tự inherit quyền
```

---

# 2. Group khác OU như thế nào?

| Thành phần | Mục đích chính |
|---|---|
| Group | Cấp quyền truy cập resource |
| OU | Tổ chức object, apply GPO, delegate admin task |

Ví dụ:

```text
OU Help Desk dùng để tổ chức user Help Desk.
Group Help Desk Remote Access dùng để cấp quyền remote.
```

---

# 3. Group Type

Khi tạo group trong AD, có 2 loại chính:

## Security Group

**Security group** dùng để cấp quyền và rights cho collection users.

Đặc điểm:

- Dùng để assign permission.
- User trong group inherit quyền của group.
- Dùng để quản lý access đến resource.
- Là loại group quan trọng nhất trong security.

```text
Security Group = cấp quyền được.
```

## Distribution Group

**Distribution group** chủ yếu dùng cho email, ví dụ Microsoft Exchange.

Đặc điểm:

- Giống mailing list.
- Dùng để gửi email cho nhiều người.
- Không dùng để cấp quyền resource trong domain.

```text
Distribution Group = gửi mail, không cấp quyền resource.
```

---

# 4. Group Scope

Có 3 group scopes chính:

```text
Domain Local Group
Global Group
Universal Group
```

---

# 5. Domain Local Group

**Domain Local Group** dùng để quản lý permission cho resource trong chính domain nơi group được tạo.

Đặc điểm:

- Dùng cấp quyền cho resource trong cùng domain.
- Có thể chứa user từ domain khác.
- Có thể nested vào Domain Local Group khác.
- Không dùng để cấp quyền ở domain khác.

```text
Domain Local = cấp quyền local trong domain.
```

---

# 6. Global Group

**Global Group** thường dùng để gom account từ cùng domain.

Đặc điểm:

- Chỉ chứa account từ domain nơi group được tạo.
- Có thể được dùng để grant access đến resource ở domain khác.
- Có thể thêm vào Global Group khác hoặc Domain Local Group.

```text
Global Group = gom user cùng domain.
```

---

# 7. Universal Group

**Universal Group** dùng cho môi trường nhiều domain trong cùng forest.

Đặc điểm:

- Có thể cấp quyền cho object trong cùng forest.
- Có thể chứa user từ bất kỳ domain nào trong forest.
- Được lưu trong Global Catalog.
- Thay đổi membership có thể trigger forest-wide replication.

Khuyến nghị:

```text
Nên add Global Groups vào Universal Groups
thay vì add từng user trực tiếp vào Universal Groups.
```

Lý do là giảm replication overhead.

---

# 8. Built-in Groups và Custom Groups

## Built-in Groups

AD tạo sẵn nhiều group khi domain được tạo.

Ví dụ:

- Administrators.
- Users.
- Guests.
- Backup Operators.
- Print Operators.
- Remote Desktop Users.
- Domain Admins.
- Enterprise Admins.
- Schema Admins.

Các group built-in thường có quyền đặc biệt, nên phải audit kỹ.

## Custom Groups

Tổ chức thường tự tạo thêm group để quản lý quyền riêng.

Ví dụ:

```text
HR-Share-Read
IT-Server-Admins
VPN-Users
Finance-App-Access
HelpDesk-Password-Reset
```

Nếu không kiểm soát, số lượng group có thể tăng rất nhanh và gây quyền ngoài ý muốn.

---

# 9. Nested Group Membership

**Nested group** là group nằm trong group khác.

Ví dụ:

```text
User A
└── Help Desk
    └── Help Desk Level 1
        └── Tier 1 Admins
```

User có thể nhận quyền không phải vì được cấp trực tiếp, mà vì group của họ là thành viên của group khác.

Rủi ro:

```text
Nested group sai có thể tạo privilege escalation path.
```

Trong pentest hoặc audit AD, nested groups là điểm rất quan trọng.

---

# 10. BloodHound và Group Path

**BloodHound** giúp visualize các relationship trong AD, đặc biệt là:

- Group membership.
- Nested groups.
- ACL abuse path.
- Privilege escalation path.
- Attack path đến high-value targets.

Ví dụ nếu user có quyền `GenericWrite` lên một group đặc quyền, attacker có thể thêm chính mình vào group đó để nhận quyền cao hơn.

---

# 11. Group Attributes quan trọng

| Attribute | Ý nghĩa |
|---|---|
| `cn` | Common Name của group |
| `member` | Object là thành viên của group |
| `memberOf` | Group này là thành viên của group nào |
| `groupType` | Xác định type và scope của group |
| `objectSid` | SID của group |

Các attribute này rất quan trọng khi enumeration hoặc audit quyền AD.

---

# 12. Blue Team Checklist

Khi audit AD Groups, nên kiểm tra:

- Group nào có quyền cao.
- User nào nằm trong privileged groups.
- Nested group membership có bất thường không.
- Group nào không còn dùng nhưng vẫn có quyền.
- Distribution group có bị dùng nhầm mục đích không.
- Universal group có chứa quá nhiều individual users không.
- Built-in groups có member lạ không.
- Group membership có vượt quá nhu cầu công việc không.

---

# 13. Red Team Mindset

Attacker thường quan tâm:

- Domain Admins.
- Enterprise Admins.
- Schema Admins.
- Backup Operators.
- Account Operators.
- Server Operators.
- Remote Management Users.
- Remote Desktop Users.
- Group có quyền ghi lên group khác.
- Nested path dẫn đến local admin hoặc domain privilege.

```text
Trong AD, không chỉ xem user có quyền gì.
Phải xem group của user nằm trong group nào nữa.
```

---

# Key Takeaway

```text
Security Group = dùng để cấp quyền.
Distribution Group = dùng cho email.
Domain Local = cấp quyền trong domain.
Global = gom account cùng domain.
Universal = dùng across forest nhưng cần chú ý replication.
Nested Groups = có thể tạo quyền ngoài ý muốn.
```

Groups là một trong những thành phần quan trọng nhất trong AD security. Quản lý group sai có thể tạo attack path, privilege escalation và excessive access rất khó phát hiện nếu không audit định kỳ.

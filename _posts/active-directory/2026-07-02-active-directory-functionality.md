---
layout: post
title: "Active Directory Functionality"
date: 2026-07-02 23:45:50 +0700
categories: [Active Directory, Windows Security]
tags: [active-directory, ad-ds, fsmo, domain-functional-level, forest-functional-level, trusts, kerberos, gpo, rid-master, pdc-emulator, infrastructure-master, blue-team, red-team]
description: "Tóm tắt Active Directory Functionality: FSMO roles, Domain/Forest Functional Levels, AD trusts, transitive/non-transitive trust và one-way/two-way trust."
toc: true
---

# Active Directory Functionality

Bài này tóm tắt các chức năng quan trọng trong **Active Directory (AD)**: **FSMO roles**, **Domain/Forest Functional Levels** và **Trusts**.

---

## Timeline cập nhật

```text
Created at: 2026-07-02 23:45:50 +0700
Timezone: Asia/Ho_Chi_Minh
Topic: Active Directory Functionality
Focus: FSMO roles, functional levels, trusts
```

---

# 1. FSMO Roles

**FSMO** là viết tắt của:

```text
Flexible Single Master Operation
```

Trong AD có 5 FSMO roles. Các role này giúp Domain Controllers xử lý các tác vụ quan trọng mà không bị conflict trong môi trường có nhiều DC.

| FSMO Role | Phạm vi | Vai trò |
|---|---|---|
| Schema Master | Forest | Quản lý AD schema |
| Domain Naming Master | Forest | Quản lý tên domain trong forest |
| RID Master | Domain | Cấp RID blocks cho DCs |
| PDC Emulator | Domain | Auth, password changes, GPO, time sync |
| Infrastructure Master | Domain | Dịch GUID, SID, DN giữa domains |

---

# 2. Schema Master

**Schema Master** quản lý bản read/write của AD schema.

Schema định nghĩa:

- Object class nào tồn tại.
- Attribute nào có thể áp dụng cho object.
- Cấu trúc dữ liệu của AD object.

Nếu thay đổi schema, thay đổi đó được xử lý qua Schema Master.

---

# 3. Domain Naming Master

**Domain Naming Master** đảm bảo không có hai domain cùng tên trong một forest.

Nó quản lý:

- Thêm domain.
- Xóa domain.
- Naming context trong forest.

---

# 4. RID Master

**RID Master** cấp block RID cho các Domain Controllers trong domain.

SID của object thường có dạng:

```text
Domain SID + RID
```

RID Master giúp tránh việc nhiều object bị cấp cùng SID.

---

# 5. PDC Emulator

**PDC Emulator** thường là DC có vai trò authoritative trong domain.

Nó xử lý:

- Authentication requests.
- Password changes.
- Account lockout.
- Group Policy Objects.
- Time synchronization.

Nếu PDC Emulator lỗi, domain có thể gặp vấn đề về authentication, password update, GPO hoặc time skew.

---

# 6. Infrastructure Master

**Infrastructure Master** dịch và cập nhật các reference giữa domain.

Nó xử lý:

- GUID.
- SID.
- Distinguished Name.
- Cross-domain object references.

Nếu role này lỗi, ACL có thể hiện SID thay vì resolved names.

---

# 7. Domain Functional Level

**Domain Functional Level (DFL)** xác định feature khả dụng trong phạm vi domain và phiên bản Windows Server có thể chạy Domain Controller.

| Domain Functional Level | Feature đáng chú ý |
|---|---|
| Windows 2000 native | Universal groups, group nesting, SID history |
| Windows Server 2003 | lastLogonTimestamp, constrained delegation, selective authentication |
| Windows Server 2008 | DFS-R, AES Kerberos, fine-grained password policies |
| Windows Server 2008 R2 | Authentication mechanism assurance, Managed Service Accounts |
| Windows Server 2012 | Claims, compound authentication, Kerberos armoring |
| Windows Server 2012 R2 | Protected Users, Authentication Policies/Silos |
| Windows Server 2016 | Credential protection, new Kerberos features |

---

# 8. Forest Functional Level

**Forest Functional Level (FFL)** xác định feature ở cấp forest.

| Forest Functional Level | Capability |
|---|---|
| Windows Server 2003 | Forest trust, domain rename, RODC |
| Windows Server 2008 R2 | Active Directory Recycle Bin |
| Windows Server 2016 | Privileged Access Management với MIM |

Ghi nhớ:

```text
DFL = capability trong domain.
FFL = capability trong forest.
```

---

# 9. Trusts

**Trust** thiết lập authentication giữa domain hoặc forest, cho phép user ở domain/forest này truy cập resource ở domain/forest khác nếu được phân quyền.

```text
Trust = liên kết giữa authentication systems của hai domain/forest.
```

---

# 10. Trust Types

| Trust Type | Mô tả |
|---|---|
| Parent-child | Trust giữa parent domain và child domain trong cùng forest |
| Cross-link | Trust giữa child domains để tăng tốc authentication |
| External | Non-transitive trust giữa hai domain ở hai forest riêng |
| Tree-root | Two-way transitive trust giữa forest root domain và new tree root domain |
| Forest | Transitive trust giữa hai forest root domains |

---

# 11. Transitive vs Non-transitive Trust

## Transitive Trust

```text
A trusts B
B trusts C
=> A có thể trust C nếu trust transitive phù hợp
```

## Non-transitive Trust

```text
A trusts B
B trusts C
=> A không tự động trust C
```

Non-transitive trust chỉ áp dụng trực tiếp giữa hai domain được cấu hình.

---

# 12. One-way vs Two-way Trust

## Two-way Trust

Trong trust hai chiều:

```text
User từ cả hai domain có thể truy cập resource của nhau nếu được phân quyền.
```

## One-way Trust

Trong trust một chiều:

```text
Chỉ user từ trusted domain có thể truy cập resource trong trusting domain.
```

Điểm dễ nhầm:

```text
Direction of trust opposite to direction of access.
```

Nếu Domain A trusts Domain B, user ở Domain B có thể truy cập resource ở Domain A.

---

# 13. Security Implications

Trust có thể tạo attack path nếu cấu hình không tốt.

Rủi ro thường gặp:

- Trust quá rộng.
- Bidirectional trust không cần thiết.
- Trust không được review sau merger/acquisition.
- External trust cấu hình sai.
- SID filtering không phù hợp.
- Cross-forest attack path.
- Kerberoasting qua trusted domain.
- User ở domain ngoài có quyền admin trong principal domain.

---

# 14. Blue Team Checklist

Khi audit AD functionality, cần kiểm tra:

- FSMO roles đang nằm trên DC nào.
- PDC Emulator time sync có ổn không.
- Domain/Forest Functional Level hiện tại.
- DC OS version có còn support không.
- SYSVOL replication dùng DFS-R hay FRS.
- Trust nào đang tồn tại.
- Trust là one-way hay two-way.
- Trust là transitive hay non-transitive.
- SID filtering có bật không.
- Account từ trusted domain có nằm trong privileged group không.

---

# 15. Red Team / Pentest Mindset

Khi đánh giá AD, cần chú ý:

- Domain trusts.
- Forest trusts.
- External trusts.
- Privileged users across trusts.
- Kerberoastable accounts in trusted domain.
- Admin access từ domain ngoài.
- Legacy functional level.
- GPO và ACL trên cross-domain resources.

---

# Key Takeaway

Active Directory functionality xoay quanh ba nhóm chính:

```text
FSMO roles: đảm bảo tác vụ critical không conflict.
Functional levels: quyết định feature/capability của domain/forest.
Trusts: cho phép authentication và access giữa domain/forest.
```

Trong bảo mật AD, trusts và FSMO roles cần được audit kỹ vì misconfiguration có thể gây authentication issues hoặc tạo unintended attack paths.

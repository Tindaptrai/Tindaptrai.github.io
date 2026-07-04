---
layout: post
title: "Active Directory Rights and Privileges"
date: 2026-07-04 22:28:16 +0700
categories: [Active Directory, Windows Security]
tags: [active-directory, rights, privileges, user-rights-assignment, built-in-groups, domain-admins, backup-operators, server-operators, sebackupprivilege, sedebugprivilege, seimpersonateprivilege, uac, blue-team, red-team]
description: "Tóm tắt ngắn về quyền và đặc quyền trong Active Directory: rights vs privileges, built-in groups, nhóm đặc quyền cao, User Rights Assignment, whoami /priv, UAC và các rủi ro leo thang đặc quyền."
toc: true
---

# Active Directory Rights and Privileges

Bài này tóm tắt phần **quyền và đặc quyền trong Active Directory**. Đây là nền tảng rất quan trọng khi quản trị AD, audit bảo mật hoặc pentest nội bộ.

```text
Rights thường liên quan đến quyền truy cập object/resource.
Privileges thường là khả năng thực hiện hành động trên hệ thống.
```

---

## Timeline cập nhật

```text
Created at: 2026-07-04 22:28:16 +0700
Timezone: Asia/Ho_Chi_Minh
Topic: Active Directory Rights and Privileges
Focus: Rights vs privileges, built-in groups, User Rights Assignment, whoami /priv, UAC, privilege abuse
```

---

# 1. Rights và Privileges khác nhau thế nào?

Trong AD/Windows security cần phân biệt:

| Khái niệm | Ý nghĩa |
|---|---|
| Rights | Quyền truy cập vào object/resource như file, share, AD object |
| Privileges | Quyền thực hiện hành động như shutdown, debug process, backup file, reset password |

Ví dụ:

```text
Access right: đọc/ghi file share.
Privilege: SeDebugPrivilege, SeBackupPrivilege, SeImpersonatePrivilege.
```

Một user có thể nhận quyền/đặc quyền trực tiếp hoặc thông qua group membership.

---

# 2. Built-in Groups trong AD

Active Directory có nhiều nhóm tích hợp sẵn. Một số nhóm có quyền rất mạnh và nếu quản lý sai có thể dẫn đến privilege escalation.

Các nhóm cần chú ý:

| Group | Rủi ro chính |
|---|---|
| Administrators | Full control trên máy hoặc domain/DC |
| Domain Admins | Toàn quyền quản lý domain |
| Enterprise Admins | Quyền toàn forest |
| Schema Admins | Có thể sửa AD schema |
| Backup Operators | Có thể backup/restore file, rủi ro lấy SAM/NTDS.DIT |
| Server Operators | Có quyền mạnh trên Domain Controller |
| Account Operators | Tạo/sửa nhiều loại account |
| Print Operators | Có thể quản lý printer trên DC, có rủi ro abuse driver |
| DnsAdmins | Có quyền với DNS, có thể nguy hiểm nếu DNS chạy trên DC |
| Hyper-V Administrators | Nếu DC là VM, virtualization admin có thể rất nguy hiểm |
| Remote Desktop Users | Cho phép RDP |
| Remote Management Users | Cho phép remote qua WinRM |
| Event Log Readers | Đọc event log, có thể lộ thông tin nhạy cảm |
| Protected Users | Nhóm tăng cường bảo vệ credential |

---

# 3. Vì sao các built-in groups nguy hiểm?

Nhiều built-in groups không trực tiếp tên là “Admin” nhưng vẫn có thể dẫn tới quyền rất cao.

Ví dụ:

```text
Backup Operators có thể backup file nhạy cảm.
Print Operators có thể đăng nhập DC cục bộ và abuse driver.
Server Operators có thể sửa service/truy cập share trên DC.
DnsAdmins có thể bị abuse nếu cấu hình DNS nguy hiểm.
```

Vì vậy, không nên chỉ kiểm tra Domain Admins. Cần audit toàn bộ nhóm đặc quyền.

---

# 4. Domain Admins vs Server Operators

Ví dụ trong AD:

```powershell
Get-ADGroup -Identity "Server Operators" -Properties *
```

Server Operators thường là:

```text
GroupCategory: Security
GroupScope: DomainLocal
Default members: rỗng
```

Trong khi Domain Admins là:

```text
GroupCategory: Security
GroupScope: Global
Members: thường có admin users/service accounts
```

Điểm quan trọng:

```text
Server Operators mặc định rỗng nhưng nếu có member lạ thì rất đáng nghi.
Domain Admins phải được kiểm soát cực chặt vì có toàn quyền domain.
```

---

# 5. User Rights Assignment

**User Rights Assignment** là nơi GPO có thể gán đặc quyền cho user/group.

Một số privilege nguy hiểm:

| Privilege | Ý nghĩa bảo mật |
|---|---|
| SeRemoteInteractiveLogonRight | Cho phép logon qua RDP |
| SeBackupPrivilege | Có thể backup file nhạy cảm như SAM/SYSTEM/NTDS.DIT |
| SeDebugPrivilege | Có thể debug process, abuse để đọc LSASS memory |
| SeImpersonatePrivilege | Có thể impersonate token, hay bị abuse để lên SYSTEM |
| SeLoadDriverPrivilege | Có thể load/unload driver, rủi ro kernel/privilege escalation |
| SeTakeOwnershipPrivilege | Có thể take ownership object/file để chiếm quyền |

Một GPO cấu hình sai có thể cấp các privilege này cho user không phù hợp.

---

# 6. Kiểm tra privilege bằng whoami

Sau khi đăng nhập Windows host, có thể xem privilege bằng:

```powershell
whoami /priv
```

Standard domain user thường rất ít privilege:

```text
SeChangeNotifyPrivilege
SeIncreaseWorkingSetPrivilege
```

Privileged user trong non-elevated shell có thể chưa thấy đầy đủ quyền do UAC.

---

# 7. UAC và elevated shell

**UAC** giới hạn quyền trong phiên thường. Vì vậy cùng một user admin nhưng:

```text
Non-elevated PowerShell: thấy ít privilege hơn.
Elevated PowerShell: thấy đầy đủ privilege hơn.
```

Ví dụ elevated admin có thể thấy:

```text
SeDebugPrivilege
SeBackupPrivilege
SeRestorePrivilege
SeTakeOwnershipPrivilege
SeImpersonatePrivilege
SeLoadDriverPrivilege
```

Điều này giải thích vì sao nhiều thao tác cần “Run as Administrator”.

---

# 8. Backup Operators example

User thuộc **Backup Operators** có thể có một số quyền bị UAC hạn chế trong shell thường, nhưng vẫn là nhóm rất nhạy cảm.

Rủi ro:

```text
Có thể backup file hệ thống.
Có thể lấy dữ liệu nhạy cảm.
Có thể dẫn đến credential extraction nếu lấy được SAM/SYSTEM/NTDS.DIT.
```

Nhóm này nên để rỗng nếu không có nhu cầu rõ ràng.

---

# 9. Góc nhìn Red Team

Attacker thường tìm:

- User trong Domain Admins.
- User trong Backup Operators.
- User trong Server Operators.
- User trong Account Operators.
- User có SeDebugPrivilege.
- User có SeBackupPrivilege.
- User có SeImpersonatePrivilege.
- GPO có thể sửa được để cấp privilege.
- Service account có quyền quá cao.
- Built-in group có member lạ.

```text
Một privilege nhỏ bị gán sai có thể trở thành đường leo thang đặc quyền.
```

---

# 10. Blue Team Checklist

Khi phòng thủ AD, nên kiểm tra:

- Built-in privileged groups có member nào.
- Domain Admins có service account không.
- Backup Operators/Server Operators có bị dùng sai không.
- User Rights Assignment trong GPO.
- Ai có SeDebugPrivilege, SeBackupPrivilege, SeImpersonatePrivilege.
- Account đặc quyền có dùng tài khoản riêng không.
- Tài khoản admin có mật khẩu/passphrase mạnh không.
- Privileged account có bị dùng cho công việc hằng ngày không.
- Logon RDP/WinRM có giới hạn đúng không.
- Account thêm vào group đặc quyền có cảnh báo không.

---

# Key Takeaway

```text
Rights = quyền truy cập resource.
Privileges = khả năng thực hiện hành động trên hệ thống.
Built-in groups có thể rất nguy hiểm nếu có member sai.
User Rights Assignment qua GPO có thể tạo privilege escalation.
whoami /priv giúp xem privilege hiện tại.
UAC làm quyền trong shell thường và shell elevated khác nhau.
```

Trong AD security, không chỉ kiểm tra “ai là Domain Admin”. Cần kiểm tra toàn bộ built-in groups, user rights assignment và privilege được gán qua GPO để tránh excessive privilege và attack path không mong muốn.

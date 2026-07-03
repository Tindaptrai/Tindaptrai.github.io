---
layout: post
title: "User and Machine Accounts"
date: 2026-07-03 16:26:18 +0700
categories: [Active Directory, Windows Security]
tags: [active-directory, user-accounts, machine-accounts, local-accounts, domain-users, kerberos, krbtgt, system, sid, objectguid, samaccountname, upn, blue-team, red-team]
description: "Tóm tắt ngắn về User and Machine Accounts trong Active Directory: local accounts, domain users, naming attributes, domain-joined vs non-domain-joined machines, SYSTEM context và rủi ro bảo mật."
toc: true
---

# User and Machine Accounts

Bài này tóm tắt ngắn về **user accounts** và **machine accounts** trong môi trường Windows/Active Directory.

```text
User account = định danh người dùng hoặc service.
Machine account = định danh máy tính join domain.
Cả hai đều có thể ảnh hưởng trực tiếp đến bảo mật domain.
```

---

## Timeline cập nhật

```text
Created at: 2026-07-03 16:26:18 +0700
Timezone: Asia/Ho_Chi_Minh
Topic: User and Machine Accounts
Focus: Local accounts, domain users, KRBTGT, naming attributes, domain-joined machines, SYSTEM context
```

---

# 1. User Account là gì?

**User account** được tạo trên local system hoặc trong Active Directory để cho phép người dùng/chương trình đăng nhập và truy cập tài nguyên.

Khi user đăng nhập:

```text
1. Hệ thống kiểm tra password.
2. Tạo access token.
3. Token chứa identity và group membership.
4. Token được dùng khi user/process truy cập tài nguyên.
```

User account dùng để:

- Cho nhân viên/contractor đăng nhập.
- Chạy program/service dưới security context cụ thể.
- Quản lý quyền truy cập file share, file, application.
- Cấp quyền thông qua group membership.

---

# 2. Vì sao group quan trọng?

Thay vì cấp quyền cho từng user, admin thường cấp quyền cho **group**.

```text
Cấp quyền một lần cho group
→ tất cả thành viên inherit quyền đó.
```

Lợi ích:

- Dễ quản lý.
- Dễ cấp quyền.
- Dễ thu hồi quyền.
- Giảm cấu hình thủ công từng user.

Rủi ro:

```text
Group membership sai hoặc nested group sai có thể tạo quyền ngoài ý muốn.
```

---

# 3. Local Accounts

**Local accounts** được lưu trên từng server/workstation riêng lẻ.

Đặc điểm:

- Chỉ có quyền trên host đó.
- Không dùng để quản lý toàn domain.
- Là security principal ở phạm vi local host.

Các local/default accounts quan trọng:

| Account | Ý nghĩa |
|---|---|
| Administrator | Local admin mặc định, SID thường kết thúc bằng `-500` |
| Guest | Disabled mặc định, rủi ro nếu bật anonymous access |
| SYSTEM | `NT AUTHORITY\SYSTEM`, quyền cao nhất trên Windows host |
| Network Service | Account chạy service, có thể trình credentials ra remote services |
| Local Service | Account chạy service với quyền tối thiểu |

---

# 4. SYSTEM Account

**SYSTEM** hay:

```text
NT AUTHORITY\SYSTEM
```

là account có quyền cực cao trên Windows host.

Điểm cần nhớ:

- Không phải user account thông thường.
- Không xuất hiện như user bình thường trong User Manager.
- Không có profile như user thường.
- Có quyền gần như toàn bộ trên local host.

Trong pentest, nếu chiếm được SYSTEM trên máy join domain, attacker thường có thể dùng máy đó làm điểm bắt đầu để enumerate domain.

---

# 5. Domain Users

**Domain users** khác local users ở chỗ quyền được quản lý từ domain.

Domain user có thể:

- Đăng nhập vào host join domain.
- Truy cập file server.
- Truy cập printer.
- Truy cập intranet.
- Truy cập resource theo permission user/group.

```text
Local user = chủ yếu dùng trên một máy.
Domain user = dùng trong toàn domain theo quyền được cấp.
```

---

# 6. KRBTGT Account

**KRBTGT** là account đặc biệt trong AD, phục vụ Kerberos Key Distribution service.

Nếu attacker compromise KRBTGT, có thể dẫn đến:

```text
Golden Ticket attack
Persistence trong domain
Unconstrained access nếu không xử lý đúng
```

Vì vậy KRBTGT là account cần được bảo vệ và audit kỹ.

---

# 7. User Naming Attributes

Một số naming attributes quan trọng trong AD:

| Attribute | Ý nghĩa |
|---|---|
| `UserPrincipalName` / UPN | Logon name chính, thường giống email |
| `ObjectGUID` | ID duy nhất của object, không đổi |
| `SAMAccountName` | Logon name hỗ trợ Windows cũ |
| `objectSID` | SID của user, dùng trong security interactions |
| `sIDHistory` | SID cũ khi migrate domain |

Ví dụ PowerShell:

```powershell
Get-ADUser -Identity htb-student
```

Kết quả thường có:

```text
DistinguishedName
Enabled
ObjectGUID
SamAccountName
SID
UserPrincipalName
```

---

# 8. Domain-joined Machines

**Domain-joined machine** là máy đã join vào domain.

Lợi ích:

- Quản lý tập trung qua Domain Controller.
- Nhận cấu hình từ Group Policy.
- User domain có thể đăng nhập nhiều host khác nhau.
- Dễ chia sẻ tài nguyên trong enterprise.

Đây là mô hình phổ biến trong doanh nghiệp.

---

# 9. Non-domain-joined Machines

**Non-domain-joined machine** hoặc workgroup machine không được quản lý bởi domain policy.

Đặc điểm:

- User account chỉ tồn tại trên host đó.
- Profile không migrate sang máy khác.
- Chia sẻ tài nguyên khó quản lý hơn.
- Phù hợp home/small LAN hơn enterprise lớn.

---

# 10. Machine Account và SYSTEM trong AD

Điểm rất quan trọng:

```text
Machine account / SYSTEM context trên máy join domain
có nhiều quyền đọc tương tự standard domain user.
```

Nếu attacker có:

```text
Remote code execution trên domain-joined host
hoặc
Privilege escalation lên SYSTEM
```

thì có thể bắt đầu enumerate domain từ context của máy đó.

---

# 11. Góc nhìn bảo mật

## Red Team / Pentest

Cần chú ý:

- Low-privileged user vẫn enumerate được nhiều thông tin.
- SYSTEM trên domain-joined host rất có giá trị.
- Service accounts có thể bị misconfigured.
- Disabled accounts cần kiểm tra privilege còn sót.
- KRBTGT là mục tiêu cực kỳ nhạy cảm.

## Blue Team

Nên kiểm tra:

- Disabled accounts có bị giữ quyền không.
- Former employees OU có account còn group membership nguy hiểm không.
- Service accounts có privilege quá cao không.
- Local admin có bị reuse password không.
- KRBTGT rotation policy.
- Domain users có quyền ngoài ý muốn không.
- Machine accounts có hoạt động bất thường không.

---

# Key Takeaway

```text
Local account chỉ có tác dụng trên một host.
Domain user có quyền trong domain theo permission và group membership.
SYSTEM là quyền rất cao trên Windows host.
Machine account trên domain-joined host có thể đọc nhiều thông tin AD.
KRBTGT là account cực kỳ nhạy cảm vì liên quan Kerberos.
```

Trong Active Directory, user và machine accounts là một phần rất lớn của attack surface. Quản lý sai user, group, service account hoặc machine account có thể mở đường cho enumeration, privilege escalation và persistence trong domain.

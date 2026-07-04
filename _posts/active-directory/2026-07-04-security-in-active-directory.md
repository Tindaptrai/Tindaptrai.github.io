---
layout: post
title: "Security in Active Directory"
date: 2026-07-04 22:44:28 +0700
categories: [Active Directory, Windows Security]
tags: [active-directory, ad-security, hardening, defense-in-depth, laps, windows-laps, applocker, gpo, audit-policy, wsus, sccm, gmsa, mfa, rdp, restricted-groups, blue-team, red-team]
description: "Tóm tắt ngắn về bảo mật trong Active Directory: CIA Triad, defense-in-depth, LAPS/Windows LAPS, audit policy, GPO security settings, AppLocker, WSUS/SCCM, gMSA, account separation, password policy, MFA, stale objects và hardening cơ bản."
toc: true
---

# Security in Active Directory

Active Directory được thiết kế để quản lý tập trung và chia sẻ tài nguyên nhanh trong doanh nghiệp. Vì ưu tiên **Availability**, AD mặc định chưa đủ cứng nếu không triển khai hardening và monitoring.

```text
AD security = hardening + monitoring + least privilege + account hygiene + defense-in-depth.
```

---

## Timeline cập nhật

```text
Created at: 2026-07-04 22:44:28 +0700
Timezone: Asia/Ho_Chi_Minh
Topic: Security in Active Directory
Focus: AD hardening, LAPS, AppLocker, GPO, auditing, patching, gMSA, account separation, MFA, restricted groups
```

---

# 1. AD và CIA Triad

CIA Triad gồm:

| Thành phần | Ý nghĩa |
|---|---|
| Confidentiality | Bảo mật dữ liệu, giới hạn ai được xem |
| Integrity | Đảm bảo dữ liệu không bị sửa sai/trái phép |
| Availability | Đảm bảo hệ thống/tài nguyên luôn sẵn sàng |

Active Directory mặc định thiên về **Availability**, vì hệ thống cần cho phép nhiều user truy cập tài nguyên nhanh chóng.

Điều này tạo ra vấn đề:

```text
Nếu không hardening, AD rất dễ bị enumeration, lateral movement và privilege escalation.
```

---

# 2. Defense-in-depth cho AD

Không có một control nào bảo vệ AD hoàn toàn.

Cần kết hợp nhiều lớp:

- Asset inventory.
- Patch management.
- Endpoint protection.
- Network segmentation.
- Security awareness training.
- Logging and monitoring.
- Group Policy hardening.
- Least privilege.
- MFA.
- Account separation.
- Periodic access review.

```text
Defense-in-depth = nhiều lớp kiểm soát để giảm rủi ro khi một lớp bị bypass.
```

---

# 3. LAPS / Windows LAPS

**LAPS** giúp quản lý mật khẩu local administrator trên các máy domain-joined.

Mục tiêu:

- Randomize local admin password.
- Rotate password theo chu kỳ.
- Lưu password trong AD với ACL bảo vệ.
- Giảm rủi ro lateral movement do dùng chung local admin password.

```text
Không nên dùng cùng một local admin password trên nhiều máy.
```

Ghi chú quan trọng:

```text
Legacy Microsoft LAPS đã deprecated trên Windows 11 23H2 trở lên.
Nên dùng Windows LAPS trên hệ điều hành được hỗ trợ.
```

---

# 4. Audit Policy Settings

Logging và monitoring là bắt buộc trong AD security.

Cần phát hiện các hành vi như:

- Tạo user/computer bất thường.
- Sửa object trong AD.
- Đổi password.
- Đăng nhập bất thường.
- Password spraying.
- Kerberos attack.
- AD enumeration.
- Thay đổi group membership.
- Thay đổi GPO.

```text
Không có log = khó điều tra.
Có log nhưng không alert = vẫn dễ bỏ sót tấn công.
```

---

# 5. Group Policy Security Settings

GPO có thể dùng để harden AD và endpoint.

Một số nhóm cấu hình quan trọng:

| Policy area | Ví dụ |
|---|---|
| Account Policies | Password policy, lockout policy, Kerberos ticket lifetime |
| Local Policies | Audit policy, user rights assignment, security options |
| Software Restriction Policies | Kiểm soát software được chạy |
| Application Control Policies | AppLocker / App Control |
| Advanced Audit Policy | Audit logon/logoff, file access, policy changes, privilege usage |

---

# 6. AppLocker / Application Control

**AppLocker** giúp kiểm soát app/file nào user được chạy.

Có thể kiểm soát:

- Executable files.
- Scripts.
- Windows Installer files.
- DLLs.
- Packaged apps.
- Packaged app installers.

Use case phổ biến:

```text
Block CMD/PowerShell cho user không cần dùng.
Chỉ cho phép software đã approved.
Audit-only trước khi enforce.
```

Lưu ý:

```text
AppLocker là defense-in-depth control, không phải lớp bảo vệ tuyệt đối.
Cần test kỹ trước khi enforce production để tránh block nhầm app cần thiết.
```

---

# 7. Update Management: WSUS / SCCM

Patch management rất quan trọng trong môi trường Windows/AD.

Giải pháp thường gặp:

- WSUS.
- SCCM / Configuration Manager.

Mục tiêu:

- Triển khai patch đúng hạn.
- Tránh bỏ sót host.
- Giảm nguy cơ exploit lỗ hổng cũ.
- Chuẩn hóa quản lý cập nhật trong enterprise.

```text
Manual patching trong môi trường lớn dễ thiếu sót và chậm.
```

---

# 8. gMSA

**Group Managed Service Account (gMSA)** là service account do domain quản lý.

Ưu điểm:

- Password tự động quản lý.
- Password dài và mạnh.
- Không cần user biết password.
- Có thể dùng cho service/task non-interactive.
- Giảm rủi ro service account password reuse.

```text
gMSA tốt hơn service account thường trong nhiều tình huống.
```

---

# 9. Security Groups

Security groups giúp cấp quyền hàng loạt.

Nhưng cần audit vì các group built-in có thể rất mạnh:

- Account Operators.
- Administrators.
- Backup Operators.
- Domain Admins.
- Enterprise Admins.
- Schema Admins.
- Remote Desktop Users.
- Remote Management Users.

```text
Không chỉ kiểm tra Domain Admins.
Phải kiểm tra toàn bộ privileged groups và nested groups.
```

---

# 10. Account Separation

Admin nên có ít nhất 2 account:

| Account | Mục đích |
|---|---|
| Normal account | Email, web, tài liệu, công việc hằng ngày |
| Admin account | Tác vụ quản trị |

Ví dụ:

```text
sjones      = account thường
sjones_adm  = account admin
```

Lý do:

```text
Nếu máy user bị phishing/malware, credential admin không nên nằm trên máy đó.
```

---

# 11. Password Policy, Passphrase và MFA

Password complexity rules chưa đủ.

Ví dụ password có vẻ đạt complexity nhưng vẫn yếu:

```text
Welcome1
Password1
CompanyName2026!
```

Khuyến nghị:

- Dùng passphrase dài.
- Dùng enterprise password manager.
- Chặn password chứa mùa/tháng/tên công ty/common words.
- Standard users tối thiểu khoảng 12 ký tự hoặc dài hơn.
- Admin/service accounts nên dài và mạnh hơn.
- MFA cho RDP/remote access.

```text
Password dài + MFA + monitoring tốt hơn chỉ dựa vào complexity.
```

---

# 12. Hạn chế dùng Domain Admin

Domain Admin chỉ nên dùng trên Domain Controller hoặc secure admin workstation.

Không nên dùng Domain Admin để đăng nhập:

- Workstation cá nhân.
- Jump host không kiểm soát tốt.
- Web server.
- File server thường.
- Máy người dùng.

Lý do:

```text
Credential Domain Admin có thể bị lưu trong memory trên host bị compromise.
```

---

# 13. Audit stale users và objects

Cần định kỳ kiểm tra:

- Disabled accounts.
- Former employee accounts.
- Old service accounts.
- Computer accounts lâu không dùng.
- Privileged accounts không còn cần thiết.
- Weak password legacy accounts.
- Group membership còn sót.

```text
Một service account cũ 8 năm với password yếu có thể là foothold rất dễ.
```

---

# 14. Auditing Permissions and Access

Nên audit định kỳ:

- Local admin rights.
- Số lượng Domain Admins.
- Enterprise Admins.
- File share access.
- User rights assignment.
- Privileged security groups.
- RDP/WinRM access.
- GPO delegation.

```text
Least privilege = user chỉ có quyền đúng với nhu cầu công việc.
```

---

# 15. Restricted Groups

**Restricted Groups** cho phép quản lý group membership bằng GPO.

Use case:

- Kiểm soát local Administrators group trên toàn domain.
- Giới hạn Enterprise Admins.
- Giới hạn Schema Admins.
- Chuẩn hóa membership của nhóm quản trị.

```text
Restricted Groups giúp ngăn việc thêm member trái phép vào nhóm nhạy cảm.
```

---

# 16. Limiting Server Roles

Không nên cài role không cần thiết trên sensitive hosts.

Ví dụ không nên:

```text
Cài IIS trên Domain Controller.
Host web app trên Exchange server.
Gộp web server và database server nếu không cần.
```

Lý do:

```text
Càng nhiều role, attack surface càng lớn.
```

---

# 17. Limiting Local Admin và RDP Rights

Cần kiểm soát chặt:

- Ai là local admin trên máy nào.
- Ai được RDP vào máy nào.
- Group nào có quyền remote.
- Domain Users có bị cấp local admin nhầm không.

Sai lầm nguy hiểm:

```text
Cho toàn bộ Domain Users làm local admin trên một hoặc nhiều host.
```

Hậu quả:

- Low-priv user có thể lấy local admin.
- Có thể dump credential từ memory nếu có user đặc quyền logon.
- Tăng lateral movement path.

---

# 18. Blue Team Checklist

Checklist tối thiểu:

```text
[ ] Bật Windows LAPS/LAPS phù hợp môi trường.
[ ] Audit privileged groups định kỳ.
[ ] Tách admin account và daily account.
[ ] Giới hạn Domain Admin chỉ dùng trên DC/SAW.
[ ] Bật logging và alert cho AD changes.
[ ] Dùng Advanced Audit Policy.
[ ] Triển khai WSUS/SCCM hoặc patch management tương đương.
[ ] Dùng gMSA cho service phù hợp.
[ ] Review stale users/computers/service accounts.
[ ] Giới hạn local admin và RDP.
[ ] Không cài role phụ trên DC.
[ ] Test AppLocker/Application Control ở audit-only trước.
[ ] Dùng MFA cho remote access.
```

---

# Key Takeaway

```text
AD mặc định ưu tiên availability nên cần hardening thêm.
LAPS/Windows LAPS giảm rủi ro local admin password reuse.
GPO giúp enforce security baseline.
AppLocker/Application Control giúp hạn chế app/script không mong muốn.
Logging và audit giúp phát hiện tấn công.
Account separation và least privilege là bắt buộc.
Domain Admin không nên dùng trên workstation thường.
```

Bảo mật AD không phải một cấu hình đơn lẻ mà là quá trình liên tục: hardening, monitoring, patching, audit quyền và giảm attack surface.

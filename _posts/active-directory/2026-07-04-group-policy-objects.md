---
layout: post
title: "Group Policy Objects"
date: 2026-07-04 22:55:53 +0700
categories: [Active Directory, Windows Security]
tags: [active-directory, group-policy, gpo, gpmc, ou, domain-policy, link-order, enforced, block-inheritance, gpupdate, bloodhound, blue-team, red-team]
description: "Tóm tắt ngắn về Group Policy Objects trong Active Directory: GPO là gì, ví dụ cấu hình, thứ tự ưu tiên, link order, enforced, block inheritance, refresh interval, gpupdate /force và rủi ro bảo mật."
toc: true
---

# Group Policy Objects

**Group Policy** là tính năng Windows cho phép quản trị viên áp dụng các cấu hình nâng cao cho user và computer trong môi trường domain.

```text
Group Policy = công cụ quản lý tập trung.
GPO = tập hợp policy settings có thể áp dụng cho user/computer.
```

Trong bảo mật AD, GPO rất mạnh vì có thể harden toàn bộ domain. Nhưng nếu bị cấu hình sai hoặc bị attacker chiếm quyền chỉnh sửa, GPO cũng có thể trở thành đường dẫn privilege escalation hoặc persistence.

---

## Timeline cập nhật

```text
Created at: 2026-07-04 22:55:53 +0700
Timezone: Asia/Ho_Chi_Minh
Topic: Group Policy Objects
Focus: GPO, order of precedence, link order, enforced, block inheritance, gpupdate, GPO abuse
```

---

# 1. GPO là gì?

**Group Policy Object (GPO)** là tập hợp ảo các policy settings có thể áp dụng cho:

- User.
- Computer.
- OU.
- Domain.
- Site.

Mỗi GPO có:

- Tên riêng.
- GUID riêng.
- Một hoặc nhiều policy settings.
- Scope áp dụng thông qua link đến OU/domain/site.

Một GPO có thể link đến nhiều container, và một container cũng có thể có nhiều GPO.

---

# 2. GPO có thể làm gì?

Một số ví dụ cấu hình bằng GPO:

- Thiết lập password policy.
- Chặn USB/removable media.
- Bắt buộc screen saver có password.
- Chặn user chạy `cmd.exe` hoặc PowerShell.
- Enforce audit/logging policy.
- Chặn script/program không được phép.
- Deploy software trên domain.
- Chặn cài software không approved.
- Hiển thị logon banner.
- Không cho dùng LM hash.
- Chạy startup/shutdown/logon/logoff scripts.
- Cấu hình Remote Desktop settings.

```text
GPO có thể tác động cực rộng, nên cần quản lý rất chặt.
```

---

# 3. Password Complexity ví dụ mặc định

Trong Windows Server AD mặc định, password complexity thường yêu cầu:

```text
Tối thiểu 7 ký tự.
Có ký tự thuộc ít nhất 3 trong 4 nhóm:
- Uppercase A-Z
- Lowercase a-z
- Number 0-9
- Special characters
```

Tuy nhiên, complexity chưa đủ mạnh nếu password ngắn hoặc dễ đoán.

---

# 4. Thứ tự ưu tiên GPO

Group Policy được áp dụng theo thứ tự:

```text
Local Group Policy
→ Site Policy
→ Domain-wide Policy
→ Parent OU Policy
→ Child OU Policy
```

Bảng tóm tắt:

| Cấp độ | Ý nghĩa |
|---|---|
| Local Group Policy | Policy trên máy local |
| Site Policy | Policy theo AD Site |
| Domain-wide Policy | Policy toàn domain |
| OU Policy | Policy áp dụng cho OU |
| Nested OU Policy | Policy trong OU con, thường ưu tiên cao hơn |

GPO gần object hơn thường được xử lý sau, nên có thể ghi đè policy ở cấp cao hơn.

---

# 5. Computer Policy vs User Policy

Một điểm quan trọng:

```text
Nếu cùng một setting được cấu hình ở Computer Policy và User Policy,
Computer Policy thường có mức ưu tiên cao hơn.
```

Điều này đặc biệt quan trọng khi troubleshoot policy không ăn như mong muốn.

---

# 6. Link Order

Khi nhiều GPO cùng link vào một OU, chúng được xử lý theo **Link Order**.

Quy tắc nhớ nhanh:

```text
Link Order 1 = ưu tiên cao nhất.
GPO có Link Order thấp hơn được xử lý cuối cùng.
```

Ví dụ:

```text
1. Disallow LM Hash
2. Block Removable Media
3. Disable Guest Account
```

Trong trường hợp này, `Disallow LM Hash` có ưu tiên cao nhất trong OU đó.

---

# 7. Enforced

**Enforced** dùng để bắt buộc policy không bị GPO ở cấp thấp hơn ghi đè.

Nếu GPO ở domain level được set Enforced:

```text
Policy đó áp dụng xuống các OU bên dưới
và OU-level policy không thể override setting đó.
```

Trước đây Enforced còn được gọi là:

```text
No Override
```

---

# 8. Block Inheritance

**Block Inheritance** được đặt trên OU để chặn policy từ cấp trên áp xuống OU đó.

Ví dụ:

```text
OU Computers bật Block Inheritance
→ policy từ Domain/Parent OU có thể không còn áp xuống Computers OU.
```

Nhưng nếu policy cấp trên được set **Enforced**, thì Enforced sẽ thắng Block Inheritance.

```text
Enforced > Block Inheritance
```

---

# 9. Default Domain Policy và Default Domain Controllers Policy

Khi tạo domain, AD tự tạo một số GPO mặc định:

## Default Domain Policy

Thường dùng cho setting toàn domain như:

- Password policy.
- Account lockout policy.
- Kerberos policy.
- Một số security baseline chung.

## Default Domain Controllers Policy

Áp dụng cho Domain Controllers, thường chứa:

- Security settings.
- Audit settings.
- User rights assignment cho DC.

```text
Không nên chỉnh bừa GPO mặc định nếu không hiểu tác động.
Nên backup trước khi thay đổi.
```

---

# 10. Group Policy Refresh Interval

GPO không luôn apply ngay lập tức.

Mặc định:

| Đối tượng | Refresh interval |
|---|---|
| User/Computer | 90 phút +/- 30 phút |
| Domain Controller | 5 phút |

Có thể mất đến khoảng:

```text
120 phút
```

để policy mới có hiệu lực trên client.

Lệnh ép update:

```powershell
gpupdate /force
```

Lệnh này yêu cầu máy kiểm tra lại GPO từ Domain Controller và áp dụng thay đổi nếu cần.

---

# 11. GPO Security Risks

GPO có thể bị attacker abuse nếu họ có quyền sửa GPO hoặc sửa link GPO.

Một số abuse case:

- Thêm user vào local administrators.
- Tạo scheduled task chạy command độc hại.
- Cấu hình logon script độc hại.
- Cấp thêm privilege cho user attacker kiểm soát.
- Disable security controls.
- Deploy malware toàn domain.
- Tạo persistence trong môi trường AD.

```text
Có quyền write lên GPO áp vào OU quan trọng = rủi ro rất cao.
```

---

# 12. BloodHound và GPO Attack Path

BloodHound có thể phát hiện đường dẫn tấn công liên quan GPO như:

- GenericWrite lên GPO.
- WriteDacl lên GPO.
- WriteOwner lên GPO.
- GPO link đến OU chứa high-value users/computers.
- Nested group dẫn đến quyền sửa GPO.

Ví dụ logic tấn công:

```text
User thuộc group A
→ group A có quyền sửa GPO
→ GPO áp vào OU chứa server/user quan trọng
→ attacker abuse GPO để leo thang quyền
```

---

# 13. Blue Team Checklist

Khi audit GPO, nên kiểm tra:

```text
[ ] Ai có quyền edit GPO?
[ ] GPO nào link đến OU chứa admin/computer quan trọng?
[ ] Có GPO nào Enforced không?
[ ] OU nào bật Block Inheritance?
[ ] Link Order có đúng không?
[ ] GPO có startup/logon script lạ không?
[ ] GPO có scheduled task lạ không?
[ ] GPO có cấp local admin không?
[ ] GPO có disable security setting không?
[ ] GPO có thay đổi User Rights Assignment không?
[ ] Có thay đổi GPO bất thường trong log không?
```

---

# 14. Red Team Mindset

Khi pentest AD, cần chú ý:

- GPO có quyền write bởi user/group thường.
- GPO áp vào OU chứa workstation/server quan trọng.
- GPO có thể cấp local admin.
- GPO có thể chạy script/scheduled task.
- GPO có thể chỉnh User Rights Assignment.
- GPO permission path trong BloodHound.
- OU inheritance và enforced settings.

```text
GPO là một trong những attack surface quan trọng nhất trong AD.
```

---

# Key Takeaway

```text
GPO giúp quản lý tập trung user/computer trong AD.
Thứ tự áp dụng: Local → Site → Domain → OU cha → OU con.
Link Order 1 có ưu tiên cao nhất trong cùng OU.
Enforced có thể vượt qua Block Inheritance.
gpupdate /force dùng để ép cập nhật GPO.
GPO bị cấp quyền sai có thể dẫn đến lateral movement, privilege escalation hoặc domain compromise.
```

Group Policy là công cụ phòng thủ rất mạnh nếu dùng đúng, nhưng cũng là điểm tấn công nguy hiểm nếu quyền chỉnh sửa GPO bị quản lý sai.

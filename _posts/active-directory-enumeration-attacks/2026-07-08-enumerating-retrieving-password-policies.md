---
layout: post
title: "Enumerating & Retrieving Password Policies"
date: 2026-07-08 00:17:12 +0700
categories: ["Active Directory Enumeration & Attacks", "Enumeration"]
tags: [active-directory, ad-enumeration-attacks, password-policy, smb, null-session, ldap, rpcclient, enum4linux, enum4linux-ng, crackmapexec, powerview, net-accounts, password-spraying, blue-team, red-team]
description: "Tóm tắt bài Enumerating & Retrieving Password Policies: lấy password policy từ Linux/Windows bằng credentialed access, SMB NULL session, LDAP anonymous bind, rpcclient, enum4linux, enum4linux-ng, net.exe và PowerView."
toc: true
---

# Enumerating & Retrieving Password Policies

Bài này thuộc module **Active Directory Enumeration & Attacks**, tập trung vào việc lấy và phân tích **domain password policy** trước khi thực hiện các bước như user targeting hoặc password spraying.

```text
Mục tiêu: biết password length, complexity, lockout threshold, lockout duration và reset counter để tránh gây lockout hàng loạt trong quá trình kiểm thử.
```

---

## Timeline cập nhật

```text
Created at: 2026-07-08 00:17:12 +0700
Timezone: Asia/Ho_Chi_Minh
Module: Active Directory Enumeration & Attacks
Topic: Enumerating & Retrieving Password Policies
Focus: Password policy enumeration, SMB NULL session, LDAP anonymous bind, rpcclient, enum4linux, CME, PowerView, password spraying safety
```

---

# 1. Vì sao cần lấy password policy?

Trước khi password spraying hoặc thử credential, ta cần biết chính sách mật khẩu của domain.

Các thông tin quan trọng:

- Minimum password length.
- Password complexity.
- Password history length.
- Maximum password age.
- Minimum password age.
- Account lockout threshold.
- Lockout duration.
- Reset account lockout counter.
- Force logoff time.

```text
Password policy quyết định cách ta kiểm thử an toàn mà không khóa tài khoản hàng loạt.
```

---

# 2. Credentialed Password Policy Enumeration từ Linux

Nếu có domain credential hợp lệ, có thể lấy password policy từ Linux bằng CrackMapExec.

Lệnh mẫu:

```bash
crackmapexec smb 172.16.5.5 -u avazquez -p Password123 --pass-pol
```

Kết quả quan trọng có thể thấy:

```text
Minimum password length: 8
Password history length: 24
Maximum password age: Not Set
Password Complexity: Enabled
Minimum password age: 1 day 4 minutes
Reset Account Lockout Counter: 30 minutes
Locked Account Duration: 30 minutes
Account Lockout Threshold: 5
Forced Log off Time: Not Set
```

Ý nghĩa nhanh:

```text
Có credential hợp lệ thì CME có thể query password policy qua SMB.
```

---

# 3. SMB NULL Session là gì?

**SMB NULL session** là kết nối SMB không cần username/password.

Một NULL session có thể cho phép unauthenticated user lấy:

- Domain information.
- Users.
- Groups.
- Computers.
- User attributes.
- Password policy.

Điều này thường xuất hiện do:

```text
Legacy configuration
Domain Controller cũ được upgrade in-place
Cấu hình anonymous access còn sót lại
```

---

# 4. Kiểm tra SMB NULL Session bằng rpcclient

Kết nối NULL session:

```bash
rpcclient -U "" -N 172.16.5.5
```

Trong shell của `rpcclient`, chạy:

```text
querydominfo
```

Ví dụ output:

```text
Domain: INLANEFREIGHT
Total Users: 3650
Total Aliases: 37
Server Role: ROLE_DOMAIN_PDC
```

Nếu query được domain info, khả năng cao NULL session đang hoạt động.

---

# 5. Lấy Password Policy bằng rpcclient

Trong `rpcclient`, chạy:

```text
getdompwinfo
```

Ví dụ:

```text
min_password_length: 8
password_properties: 0x00000001
    DOMAIN_PASSWORD_COMPLEX
```

Ý nghĩa:

```text
Minimum password length = 8
Password complexity enabled
```

---

# 6. enum4linux

`enum4linux` là tool wrapper quanh các Samba tools như:

- `nmblookup`.
- `net`.
- `rpcclient`.
- `smbclient`.

Lệnh lấy password policy:

```bash
enum4linux -P 172.16.5.5
```

Ví dụ output quan trọng:

```text
Minimum password length: 8
Password history length: 24
Maximum password age: Not Set
Password Complexity Flags: 000001
Minimum password age: 1 day 4 minutes
Reset Account Lockout Counter: 30 minutes
Locked Account Duration: 30 minutes
Account Lockout Threshold: 5
```

---

# 7. enum4linux-ng

`enum4linux-ng` là bản viết lại bằng Python, output rõ hơn và có thể export JSON/YAML.

Lệnh mẫu:

```bash
enum4linux-ng -P 172.16.5.5 -oA ilfreight
```

Điểm hay của `-oA`:

```text
Xuất output ra nhiều format để xử lý lại.
Có thể dùng JSON/YAML cho automation hoặc báo cáo.
```

Ví dụ thông tin policy:

```yaml
domain_password_information:
  pw_history_length: 24
  min_pw_length: 8
  min_pw_age: 1 day 4 minutes
  max_pw_age: not set
  pw_properties:
  - DOMAIN_PASSWORD_COMPLEX: true

domain_lockout_information:
  lockout_observation_window: 30 minutes
  lockout_duration: 30 minutes
  lockout_threshold: 5
```

---

# 8. Một số tool và port liên quan

| Tool | Port thường dùng |
|---|---|
| nmblookup | 137/UDP |
| nbtstat | 137/UDP |
| net | 139/TCP, 135/TCP, 135 UDP/TCP, 49152-65535 |
| rpcclient | 135/TCP |
| smbclient | 445/TCP |

Ghi nhớ:

```text
SMB/RPC enumeration thường dựa vào 135, 139, 445 và dynamic RPC ports.
```

---

# 9. Enumerating NULL Session từ Windows

Từ Windows, có thể thử tạo NULL session:

```cmd
net use \\DC01\ipc$ "" /u:""
```

Nếu thành công:

```text
The command completed successfully.
```

Một số lỗi thường gặp:

## Account disabled

```cmd
net use \\DC01\ipc$ "" /u:guest
```

```text
System error 1331 has occurred.
This user can't sign in because this account is currently disabled.
```

## Password incorrect

```cmd
net use \\DC01\ipc$ "password" /u:guest
```

```text
System error 1326 has occurred.
The user name or password is incorrect.
```

## Account locked out

```text
System error 1909 has occurred.
The referenced account is currently locked out and may not be logged on to.
```

---

# 10. LDAP Anonymous Bind

**LDAP anonymous bind** cho phép unauthenticated user query LDAP.

Có thể bị dùng để lấy:

- Users.
- Groups.
- Computers.
- Attributes.
- Domain password policy.

Đây thường là legacy/misconfiguration.

Từ Windows Server 2003 trở đi, mặc định chỉ authenticated users mới được LDAP request, nhưng thực tế vẫn có thể gặp anonymous bind do ứng dụng hoặc cấu hình sai.

---

# 11. Lấy Password Policy bằng ldapsearch

Lệnh mẫu:

```bash
ldapsearch -h 172.16.5.5 -x -b "DC=INLANEFREIGHT,DC=LOCAL" -s sub "*" | grep -m 1 -B 10 pwdHistoryLength
```

Với bản mới hơn, `-h` có thể được thay bằng `-H`:

```bash
ldapsearch -H ldap://172.16.5.5 -x -b "DC=INLANEFREIGHT,DC=LOCAL" -s sub "*" | grep -m 1 -B 10 pwdHistoryLength
```

Output quan trọng:

```text
lockoutDuration: -18000000000
lockOutObservationWindow: -18000000000
lockoutThreshold: 5
maxPwdAge: -9223372036854775808
minPwdAge: -864000000000
minPwdLength: 8
pwdProperties: 1
pwdHistoryLength: 24
```

Giải thích nhanh:

```text
minPwdLength: 8        => minimum password length là 8
lockoutThreshold: 5    => sai 5 lần thì lockout
pwdProperties: 1       => password complexity enabled
pwdHistoryLength: 24   => lưu password history 24
```

---

# 12. Enumerating Password Policy từ Windows bằng net.exe

Nếu đang ở Windows host và có thể authenticate domain, dùng built-in command:

```cmd
net accounts
```

Ví dụ output:

```text
Minimum password age (days):                          1
Maximum password age (days):                          Unlimited
Minimum password length:                              8
Length of password history maintained:                24
Lockout threshold:                                    5
Lockout duration (minutes):                           30
Lockout observation window (minutes):                 30
Computer role:                                        SERVER
```

Ưu điểm:

```text
Không cần upload tool.
Hữu ích khi bị hạn chế transfer file hoặc EDR kiểm soát chặt.
```

---

# 13. Enumerating Password Policy bằng PowerView

Nếu có PowerView:

```powershell
Import-Module .\PowerView.ps1
Get-DomainPolicy
```

Output thường có:

```text
MinimumPasswordAge=1
MaximumPasswordAge=-1
MinimumPasswordLength=8
PasswordComplexity=1
PasswordHistorySize=24
LockoutBadCount=5
ResetLockoutCount=30
LockoutDuration=30
```

PowerView còn tiết lộ GPO path:

```text
Default Domain Policy
GptTmpl.inf
```

---

# 14. Phân tích Policy của INLANEFREIGHT.LOCAL

Từ các cách trên, policy chính:

| Setting | Value | Ý nghĩa |
|---|---:|---|
| Minimum password length | 8 | Có khả năng tồn tại password yếu 8 ký tự |
| Password history length | 24 | Không cho reuse 24 password cũ |
| Maximum password age | Not Set / Unlimited | Password không hết hạn |
| Password complexity | Enabled | Cần đạt complexity |
| Minimum password age | 1 day | Đổi password liên tục bị hạn chế |
| Lockout threshold | 5 | Sai 5 lần sẽ lockout |
| Lockout duration | 30 minutes | Lock 30 phút |
| Reset lockout counter | 30 minutes | Counter reset sau 30 phút |
| Force logoff | Not Set | Không ép logoff |

---

# 15. Vì sao policy này tốt cho password spraying?

Policy này khá thuận lợi cho password spraying vì:

```text
Minimum length chỉ 8.
Complexity enabled nhưng vẫn cho phép password yếu như Welcome1 hoặc Password1.
Lockout threshold là 5.
Lockout duration là 30 phút.
Account tự unlock sau 30 phút.
```

Nhưng khi test thật:

```text
Không bao giờ cố tình lockout tài khoản.
Không thử sát ngưỡng 5 lần.
Nên dùng 1-2 attempt rồi chờ đủ lâu.
```

---

# 16. Password Complexity không đồng nghĩa password mạnh

Password complexity thường yêu cầu 3/4 nhóm:

- Uppercase.
- Lowercase.
- Number.
- Special character.

Ví dụ vẫn có thể pass complexity:

```text
Password1
Welcome1
Summer22
Company1
```

Những password này vẫn yếu dù đáp ứng complexity.

---

# 17. Default Domain Password Policy

Policy mặc định khi tạo domain thường gặp:

| Policy | Default Value |
|---|---|
| Enforce password history | 24 days |
| Maximum password age | 42 days |
| Minimum password age | 1 day |
| Minimum password length | 7 |
| Password must meet complexity requirements | Enabled |
| Store passwords using reversible encryption | Disabled |
| Account lockout duration | Not set |
| Account lockout threshold | 0 |
| Reset account lockout counter after | Not set |

Một số tổ chức tạo domain xong không thay policy mặc định, nên cần kiểm tra cẩn thận.

---

# 18. Lưu ý an toàn khi Password Spraying

Nếu không lấy được password policy:

```text
Phải cực kỳ thận trọng.
Không được giả định lockout threshold là 5.
Không được giả định tự unlock sau 30 phút.
```

Cách an toàn hơn:

```text
1 attempt.
Nếu cần attempt thứ 2, chờ trên 1 giờ.
Trao đổi với client để xin policy nếu assessment cho phép.
```

Câu cần nhớ:

```text
Đừng trở thành pentester làm lockout toàn bộ công ty.
```

---

# 19. Blue Team Detection

Blue Team nên theo dõi:

- SMB NULL session attempt.
- LDAP anonymous bind.
- RPC anonymous enumeration.
- enum4linux/enum4linux-ng pattern.
- Nhiều request đến DC qua 445/135/389.
- `net accounts` hoặc domain policy query từ host lạ.
- Kerberos/SMB failed logon patterns.
- Password spray low-and-slow.
- Account lockout events.

Event/log nguồn nên xem:

- Domain Controller Security logs.
- LDAP query logs nếu có.
- Windows Event ID liên quan failed logon/account lockout.
- Firewall logs.
- EDR telemetry.
- Zeek/Suricata network logs.

---

# 20. Red Team Notes

Khi ghi chú trong pentest report, nên lưu:

- Tool đã dùng.
- Command.
- Timestamp.
- Source host.
- Target DC.
- Output policy.
- Risk đánh giá.
- Password spraying safety plan.
- Detection/mitigation recommendation.

Ví dụ note:

```text
Target: 172.16.5.5 / ACADEMY-EA-DC01
Domain: INLANEFREIGHT.LOCAL
Min password length: 8
Complexity: Enabled
Lockout threshold: 5
Lockout duration: 30 minutes
Reset counter: 30 minutes
Method: enum4linux-ng -P -oA
```

---

# 21. Mitigation

Các hướng phòng thủ:

- Disable SMB NULL session.
- Không cho LDAP anonymous bind nếu không cần.
- Tăng minimum password length lên 12-14+.
- Dùng passphrase.
- Bật password blacklist/banned password list.
- Dùng account lockout hợp lý.
- Monitor password spray low-and-slow.
- MFA cho remote access.
- Tiering admin accounts.
- Giảm password reuse.
- Dùng fine-grained password policies cho nhóm nhạy cảm.

---

# Key Takeaway

```text
Password policy là dữ liệu cực kỳ quan trọng trước khi password spraying.
Có thể lấy policy bằng CME/rpcclient/enum4linux/enum4linux-ng/ldapsearch/net.exe/PowerView.
SMB NULL session và LDAP anonymous bind là misconfiguration nguy hiểm.
Policy của INLANEFREIGHT cho thấy min length 8, complexity enabled, lockout threshold 5, lockout 30 phút.
Password complexity không đảm bảo password mạnh.
Khi không biết policy, chỉ nên thử rất ít và chờ lâu để tránh lockout.
```

Giai đoạn này giúp ta chuẩn bị user list và chiến lược password spraying an toàn hơn cho các bước tiếp theo trong AD assessment.

---
layout: post
title: "Password Spraying - Making a Target User List"
date: 2026-07-08 01:24:08 +0700
categories: ["Active Directory Enumeration & Attacks", "Credential Attacks"]
tags: [active-directory, ad-enumeration-attacks, password-spraying, user-enumeration, smb-null-session, ldap-anonymous-bind, enum4linux, rpcclient, crackmapexec, kerbrute, windapsearch, ldapsearch, badpwdcount, blue-team, red-team]
description: "Tóm tắt bài Password Spraying - Making a Target User List: xây dựng danh sách user hợp lệ bằng SMB NULL session, LDAP anonymous bind, Kerbrute, CrackMapExec và credentialed enumeration trước khi password spraying."
toc: true
---

# Password Spraying - Making a Target User List

Bài này thuộc module **Active Directory Enumeration & Attacks**, tập trung vào bước chuẩn bị trước password spraying: tạo danh sách **valid domain users** để target chính xác hơn và giảm rủi ro lockout.

```text
Mục tiêu: tạo user list sạch, hiểu password policy, ghi log hoạt động và chuẩn bị password spraying an toàn.
```


---

## Timeline cập nhật

```text
Created at: 2026-07-08 01:24:08 +0700
Timezone: Asia/Ho_Chi_Minh
Module: Active Directory Enumeration & Attacks
Topic: Password Spraying - Making a Target User List
Focus: Target user list, SMB NULL session, LDAP anonymous bind, Kerbrute, CrackMapExec, badpwdcount, password spraying safety
```


---

# 1. Vì sao cần target user list?

Muốn password spraying hiệu quả, ta cần danh sách **valid domain users**. Nếu không có user list tốt, ta sẽ tốn thời gian, tạo nhiễu log, dễ bỏ sót account giá trị và khó kiểm soát rủi ro lockout.

```text
Password spraying tốt bắt đầu từ user list tốt.
```


---

# 2. Các nguồn lấy user list

| Nguồn | Điều kiện |
|---|---|
| SMB NULL session | DC cho anonymous SMB/RPC enumeration |
| LDAP anonymous bind | LDAP cho anonymous query |
| Kerbrute | Có DC/KDC và wordlist username |
| Valid credential | Có domain user hoặc credential thu được |
| SYSTEM access | Có SYSTEM trên domain-joined host |
| OSINT | Email harvesting, LinkedIn, naming convention |


---

# 3. Luôn kiểm tra password policy trước

Trước khi spray, cần lấy password policy nếu có thể:

- Minimum password length.
- Password complexity.
- Lockout threshold.
- Bad password reset timer.
- Lockout duration.
- Account tự unlock hay cần admin unlock.

```text
Nếu không biết policy, chỉ nên thử rất ít attempt và chờ lâu giữa các lần thử.
```


---

# 4. Những thông tin bắt buộc phải ghi log

Khi password spraying, phải ghi lại:

```text
Accounts targeted
Domain Controller used
Time of spray
Date of spray
Password attempted
```

Điều này giúp tránh duplicate effort, dễ đối chiếu nếu có lockout, hỗ trợ client/SOC xác minh log và chứng minh hoạt động nằm trong scope.


---

# 5. SMB NULL Session để lấy user list

Nếu ở trong internal network nhưng chưa có credential, có thể kiểm tra SMB NULL session trên DC. Nếu misconfiguration tồn tại, ta có thể lấy user list, domain info, password policy, group/computer info.

Các tool phổ biến:

- `enum4linux`.
- `rpcclient`.
- `CrackMapExec`.


---

# 6. Lấy users bằng enum4linux

```bash
enum4linux -U 172.16.5.5 | grep "user:" | cut -f2 -d"[" | cut -f1 -d"]"
```

Ý nghĩa:

```text
-U                 enumerate users
grep "user:"       lọc dòng chứa user
cut                tách username sạch
```

Ví dụ output:

```text
administrator
guest
krbtgt
lab_adm
htb-student
avazquez
pfalcon
fanthony
wdillard
lbradford
```


---

# 7. Lấy users bằng rpcclient

Kết nối NULL session:

```bash
rpcclient -U "" -N 172.16.5.5
```

Trong shell `rpcclient`:

```text
enumdomusers
```

Ví dụ output:

```text
user:[administrator] rid:[0x1f4]
user:[guest] rid:[0x1f5]
user:[krbtgt] rid:[0x1f6]
user:[lab_adm] rid:[0x3e9]
user:[htb-student] rid:[0x457]
```

Muốn dùng để spray, cần lọc còn username mỗi dòng.


---

# 8. Lấy users bằng CrackMapExec

```bash
crackmapexec smb 172.16.5.5 --users
```

Điểm hay của CME là có thể hiển thị:

| Field | Ý nghĩa |
|---|---|
| `badpwdcount` | Số lần nhập sai hiện tại |
| `baddpwdtime` | Thời điểm nhập sai gần nhất |

Nếu account gần lockout threshold, nên loại khỏi spray list. Trong môi trường nhiều DC, `badpwdcount` có thể được duy trì riêng trên từng DC, nên muốn chính xác hơn cần query từng DC hoặc query PDC Emulator.


---

# 9. LDAP Anonymous Bind để lấy user list

Dùng `ldapsearch`:

```bash
ldapsearch -h 172.16.5.5 -x -b "DC=INLANEFREIGHT,DC=LOCAL" -s sub "(&(objectclass=user))" | grep sAMAccountName: | cut -f2 -d" "
```

Ví dụ output:

```text
guest
ACADEMY-EA-DC01$
ACADEMY-EA-MS01$
ACADEMY-EA-WEB01$
htb-student
avazquez
pfalcon
```

Computer accounts thường kết thúc bằng dấu `$`, nên cần loại khỏi human-user spray list nếu không cần.


---

# 10. Dùng windapsearch

Anonymous user enumeration:

```bash
./windapsearch.py --dc-ip 172.16.5.5 -u "" -U
```

Tool này giúp lấy userPrincipalName và thông tin user dễ đọc hơn, nhưng vẫn nên hiểu LDAP filter cơ bản để tự kiểm soát query.


---

# 11. Kerbrute Username Enumeration

Nếu không có SMB NULL session, LDAP anonymous bind hoặc credential, có thể dùng Kerbrute để validate username từ wordlist.

```bash
kerbrute userenum -d inlanefreight.local --dc 172.16.5.5 /opt/jsmith.txt
```

Kerbrute dựa vào Kerberos response:

```text
PRINCIPAL UNKNOWN      => username không tồn tại
Pre-Auth required      => username tồn tại
```

Username enumeration bằng Kerbrute không gây account lockout, nhưng khi dùng Kerbrute để password spraying thì failed pre-auth có thể tính vào failed login count.


---

# 12. Kerbrute và detection

Kerbrute username enumeration có thể tạo:

```text
Event ID 4768 - A Kerberos authentication ticket (TGT) was requested
```

Blue Team có thể hunt theo pattern nhiều 4768 bất thường từ cùng source host hoặc nhiều request tới DC/KDC trong thời gian ngắn.


---

# 13. Credentialed Enumeration để build user list

Nếu đã có domain credential, có thể dùng CME:

```bash
sudo crackmapexec smb 172.16.5.5 -u htb-student -p Academy_student_AD! --users
```

Ưu điểm:

- Nhanh.
- Output rõ.
- Có thể hiển thị `badpwdcount`.
- Hữu ích để loại account rủi ro khỏi spray list.


---

# 14. OSINT để tạo username list

Nếu không query được AD, có thể quay lại external information gathering:

- Email harvesting.
- LinkedIn.
- Company website.
- Breach data.
- Naming convention.
- Tools như `linkedin2username`.

User list từ OSINT không đầy đủ bằng AD enumeration, nhưng vẫn có thể đủ để tạo foothold.


---

# 15. Làm sạch user list

Sau khi lấy user list, nên clean:

```bash
sort -u users.txt -o users.txt
```

Loại account không nên spray:

- Disabled account nếu biết.
- Guest.
- krbtgt.
- Computer accounts kết thúc bằng `$`.
- Account có `badpwdcount` gần ngưỡng lockout.
- Admin/high-value account nếu ROE không cho phép.
- Service account nhạy cảm nếu chưa có approval.

Lọc bỏ computer accounts:

```bash
grep -v '\$$' users_raw.txt > users_clean.txt
```


---

# 16. Checklist trước khi spray

```text
[ ] Có user list sạch.
[ ] Biết password policy.
[ ] Biết lockout threshold.
[ ] Biết reset counter.
[ ] Biết lockout duration.
[ ] Biết DC sẽ target.
[ ] Đã ghi timestamp.
[ ] Đã thống nhất scope/ROE.
[ ] Có kế hoạch dừng nếu thấy lockout.
```


---

# 17. Blue Team Detection

Blue Team nên theo dõi:

- SMB NULL session.
- LDAP anonymous bind.
- RPC enumeration.
- `enumdomusers`.
- CME `--users` behavior.
- Nhiều LDAP query lấy `sAMAccountName`.
- Kerberos Event ID 4768 tăng bất thường.
- Password spray low-and-slow.
- Bad password count tăng đồng loạt.
- Logon failures phân tán theo nhiều account cùng một source.


---

# 18. Red Team Notes

Trong report nên lưu:

```text
Method: Kerbrute / enum4linux / rpcclient / CME / ldapsearch / windapsearch
Target DC: 172.16.5.5
Domain: INLANEFREIGHT.LOCAL
User count discovered:
Source host:
Command:
Timestamp:
Output file:
Policy known: yes/no
Lockout threshold:
Spray plan:
```

Ví dụ lưu output:

```bash
kerbrute userenum -d inlanefreight.local --dc 172.16.5.5 /opt/jsmith.txt -o valid_users.txt
```


---

# 19. Mitigation

Các hướng phòng thủ:

- Disable SMB NULL session.
- Disable LDAP anonymous bind.
- Monitor anonymous RPC/LDAP enumeration.
- Enable Kerberos event logging phù hợp.
- Alert khi có nhiều 4768 bất thường.
- Banned password list.
- Strong password/passphrase policy.
- MFA cho external/remote access.
- Smart lockout policy.
- Monitor password spray low-and-slow.
- Hạn chế thông tin nhân sự công khai quá chi tiết.
- Tách privilege account khỏi user thường.


---

# Key Takeaway

```text
Password spraying cần user list hợp lệ trước khi thử password.
Có thể lấy user list bằng SMB NULL session, LDAP anonymous bind, Kerbrute, valid credential hoặc OSINT.
CrackMapExec hữu ích vì hiển thị badpwdcount/baddpwdtime.
Kerbrute username enumeration không lockout account, nhưng Kerbrute password spraying thì có thể.
Luôn dựa vào password policy và ghi log đầy đủ để tránh gây lockout.
```

Bước này là nền tảng để chuyển sang password spraying một cách có kiểm soát, có bằng chứng và ít rủi ro hơn.

---
layout: post
title: "Credentialed Enumeration - from Linux"
date: 2026-07-09 00:25:53 +0700
categories: ["Active Directory Enumeration & Attacks", "Enumeration"]
tags: [active-directory, ad-enumeration-attacks, credentialed-enumeration, linux, crackmapexec, netexec, smbmap, rpcclient, impacket, psexec, wmiexec, windapsearch, bloodhound, bloodhound-python, blue-team, red-team]
description: "Tóm tắt bài Credentialed Enumeration from Linux: dùng credential thấp quyền để enumerate AD bằng CrackMapExec/NetExec, SMBMap, rpcclient, Impacket, Windapsearch và BloodHound.py."
toc: true
---

# Credentialed Enumeration - from Linux

Bài này thuộc module **Active Directory Enumeration & Attacks**, tập trung vào giai đoạn sau khi đã có foothold/credential hợp lệ và bắt đầu enumerate sâu domain từ Linux attack host.

```text
Mục tiêu: dùng low-privilege domain credentials để lấy thông tin users, groups, shares, sessions, ACLs, trusts, privileged users và attack paths trong AD.
```

---

## Timeline cập nhật

```text
Created at: 2026-07-09 00:25:53 +0700
Timezone: Asia/Ho_Chi_Minh
Module: Active Directory Enumeration & Attacks
Topic: Credentialed Enumeration - from Linux
Focus: CME/NetExec, SMBMap, rpcclient, Impacket, Windapsearch, BloodHound.py
```

---

# 1. Bối cảnh

Sau khi có credential hợp lệ, visibility trong domain tăng lên rất nhiều. Từ Linux attack host, ta có thể enumerate:

- Domain users.
- Domain computers.
- Group membership.
- Group Policy Objects.
- Permissions.
- ACLs.
- Trusts.
- Shares.
- Logged-on sessions.
- Privileged users.
- Attack paths.

```text
Tối thiểu cần cleartext password, NTLM hash, hoặc SYSTEM access trên domain-joined host.
```

---

# 2. Credential được dùng trong lab

Ví dụ lab dùng credential:

```text
Domain: INLANEFREIGHT.LOCAL
User: forend
Password: Klmcargo2
DC: 172.16.5.5
```

Credential này giúp thực hiện credentialed enumeration từ Linux attack host.

---

# 3. CrackMapExec / NetExec

**CrackMapExec** hay CME, hiện nay thường được kế thừa bằng **NetExec**, là tool mạnh để đánh giá AD qua nhiều protocol:

- SMB.
- MSSQL.
- SSH.
- WinRM.

Xem help:

```bash
crackmapexec -h
crackmapexec smb -h
```

Các flag SMB quan trọng:

| Flag | Ý nghĩa |
|---|---|
| `-u` | Username |
| `-p` | Password |
| `-H` | NTLM hash |
| `-d` | Domain |
| `--users` | Enumerate domain users |
| `--groups` | Enumerate domain groups |
| `--loggedon-users` | Enumerate logged-on users |
| `--shares` | Enumerate SMB shares |
| `--pass-pol` | Enumerate password policy |

---

# 4. CME - Domain User Enumeration

Enumerate domain users từ DC:

```bash
sudo crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 --users
```

Output có thể chứa:

```text
INLANEFREIGHT.LOCAL\administrator    badpwdcount: 0
INLANEFREIGHT.LOCAL\guest            badpwdcount: 0
INLANEFREIGHT.LOCAL\lab_adm          badpwdcount: 0
INLANEFREIGHT.LOCAL\avazquez         badpwdcount: 3
```

Điểm đáng chú ý:

```text
badPwdCount rất hữu ích khi chuẩn bị password spraying.
```

Có thể loại account có `badpwdcount > 0` để giảm rủi ro lockout.

---

# 5. CME - Domain Group Enumeration

Enumerate domain groups:

```bash
sudo crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 --groups
```

Các group cần chú ý:

- Administrators.
- Domain Admins.
- Enterprise Admins.
- Backup Operators.
- Account Operators.
- Server Operators.
- Executives.
- IT/Admin-related groups.
- Custom privileged groups.

```text
Group membership là dữ liệu rất quan trọng để xác định user/group có quyền cao hoặc có thể là đường leo thang.
```

---

# 6. CME - Logged-on Users

Kiểm tra user đang logon trên host cụ thể:

```bash
sudo crackmapexec smb 172.16.5.130 -u forend -p Klmcargo2 --loggedon-users
```

Ví dụ host file server có thể trả về:

```text
INLANEFREIGHT\clusteragent
INLANEFREIGHT\lab_adm
INLANEFREIGHT\svc_qualys
INLANEFREIGHT\wley
```

Nếu CME hiện:

```text
(Pwn3d!)
```

nghĩa là credential hiện tại có local admin trên host đó.

---

# 7. Vì sao logged-on users quan trọng?

Logged-on users giúp tìm:

- Admin đang đăng nhập trên server.
- Jump host tiềm năng.
- Host có high-value session.
- Service account đang hoạt động.
- Cơ hội credential theft hoặc token impersonation nếu nằm trong scope.

```text
Một file server có nhiều admin/session giá trị có thể là target rất đáng chú ý.
```

---

# 8. CME - Share Enumeration

Enumerate share trên DC:

```bash
sudo crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 --shares
```

Ví dụ share có thể thấy:

```text
Department Shares    READ
IPC$                 READ
NETLOGON             READ
SYSVOL               READ
User Shares          READ
ZZZ_archive          READ
```

Các share đáng kiểm tra:

- Department Shares.
- User Shares.
- ZZZ_archive.
- SYSVOL.
- NETLOGON.

```text
Share có READ access có thể chứa script, config, password, PII hoặc tài liệu nội bộ.
```

---

# 9. CME spider_plus

Dùng module `spider_plus` để liệt kê file trong share:

```bash
sudo crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 -M spider_plus --share 'Department Shares'
```

Output thường lưu ở:

```text
/tmp/cme_spider_plus/<ip>.json
```

Xem nhanh:

```bash
head -n 10 /tmp/cme_spider_plus/172.16.5.5.json
```

File đáng tìm:

- `web.config`.
- `.bat`.
- `.ps1`.
- `.vbs`.
- `.kdbx`.
- `.xlsx`.
- `.docx`.
- Backup/archive files.
- Config có hardcoded credential.

---

# 10. SMBMap

**SMBMap** dùng tốt để enumerate SMB shares, permissions và nội dung share từ Linux.

Check access:

```bash
smbmap -u forend -p Klmcargo2 -d INLANEFREIGHT.LOCAL -H 172.16.5.5
```

Ví dụ permission:

```text
ADMIN$             NO ACCESS
C$                 NO ACCESS
Department Shares  READ ONLY
IPC$               READ ONLY
NETLOGON           READ ONLY
SYSVOL             READ ONLY
User Shares        READ ONLY
ZZZ_archive        READ ONLY
```

---

# 11. SMBMap recursive listing

Liệt kê thư mục trong share:

```bash
smbmap -u forend -p Klmcargo2 -d INLANEFREIGHT.LOCAL -H 172.16.5.5 -R 'Department Shares' --dir-only
```

Có thể thấy các thư mục:

```text
Accounting
Executives
Finance
HR
IT
Legal
Marketing
Operations
R&D
Temp
Warehouse
```

```text
SMBMap phù hợp khi pillaging share để tìm file nhạy cảm hoặc dữ liệu hỗ trợ privilege escalation.
```

---

# 12. rpcclient

`rpcclient` là tool thuộc Samba, dùng MS-RPC để enumerate hoặc tương tác với AD/Windows.

Kết nối NULL session nếu được phép:

```bash
rpcclient -U "" -N 172.16.5.5
```

Hoặc dùng credential:

```bash
rpcclient -U "INLANEFREIGHT\forend%Klmcargo2" 172.16.5.5
```

Một số command hay dùng:

```text
enumdomusers
queryuser <RID>
querydominfo
getdompwinfo
enumdomgroups
querygroup <RID>
```

---

# 13. SID và RID

Trong AD, object được định danh bằng SID. RID là phần cuối của SID.

Ví dụ:

```text
Domain SID: S-1-5-21-3842939050-3880317879-2865463114
RID 0x457 = 1111
Full SID: S-1-5-21-3842939050-3880317879-2865463114-1111
```

RID nổi bật:

```text
Administrator = 500 = 0x1f4
Guest         = 501 = 0x1f5
krbtgt        = 502 = 0x1f6
```

---

# 14. rpcclient user enumeration

Liệt kê users:

```text
rpcclient $> enumdomusers
```

Query user theo RID:

```text
rpcclient $> queryuser 0x457
```

Thông tin có thể lấy:

- Username.
- Full name.
- Logon time.
- Password last set.
- Password can/must change time.
- Bad password count.
- Logon count.
- Group RID.
- User RID.

---

# 15. Impacket Toolkit

**Impacket** là toolkit Python rất mạnh để tương tác Windows protocols.

Trong section này tập trung vào:

- `psexec.py`.
- `wmiexec.py`.

Các tool Impacket thường dùng trong AD pentest vì hỗ trợ SMB, WMI, Kerberos, NTLM và nhiều protocol Windows khác.

---

# 16. psexec.py

`psexec.py` cần credential có local administrator trên target.

Ví dụ:

```bash
psexec.py inlanefreight.local/wley:'transporter@4'@172.16.5.125
```

Cơ chế tổng quát:

```text
Upload executable vào ADMIN$
Tạo remote service qua Service Control Manager
Giao tiếp qua named pipe
Thường nhận shell dưới context SYSTEM
```

Sau khi vào shell, kiểm tra:

```cmd
whoami
```

Kết quả thường là:

```text
nt authority\system
```

---

# 17. wmiexec.py

`wmiexec.py` thực thi command qua WMI và thường ít “để file” hơn psexec.

Ví dụ:

```bash
wmiexec.py inlanefreight.local/wley:'transporter@4'@172.16.5.5
```

Đặc điểm:

- Semi-interactive shell.
- Mỗi command tạo `cmd.exe` qua WMI.
- Thường chạy dưới context user local admin được dùng để kết nối.
- Ít file artifact hơn psexec, nhưng vẫn có log/process creation.

Detection đáng chú ý:

```text
Event ID 4688 - A new process has been created
```

---

# 18. Windapsearch

**Windapsearch** dùng LDAP queries để enumerate AD từ Linux.

Help:

```bash
windapsearch.py -h
```

Một số option:

| Option | Ý nghĩa |
|---|---|
| `--dc-ip` | IP Domain Controller |
| `-u` | Bind user |
| `-p` | Password |
| `-U` | Enumerate users |
| `-G` | Enumerate groups |
| `-C` | Enumerate computers |
| `--da` | Enumerate Domain Admins |
| `-PU` | Enumerate privileged users recursively |

---

# 19. Windapsearch - Domain Admins

Enumerate Domain Admins:

```bash
python3 windapsearch.py --dc-ip 172.16.5.5 -u forend@inlanefreight.local -p Klmcargo2 --da
```

Kết quả có thể liệt kê các Domain Admins như:

```text
Administrator
lab_adm
mmorgan
...
```

Cần chú ý những user đã từng thấy trong session/hash/log như:

```text
wley
svc_qualys
lab_adm
```

---

# 20. Windapsearch - Privileged Users

Enumerate privileged users recursive:

```bash
python3 windapsearch.py --dc-ip 172.16.5.5 -u forend@inlanefreight.local -p Klmcargo2 -PU
```

Điểm mạnh:

```text
Tìm nested group membership trong các nhóm quyền cao.
```

Đây là finding tốt cho report vì nested group có thể khiến user có quyền cao ngoài dự kiến.

---

# 21. BloodHound.py

**BloodHound.py** là collector/ingestor chạy từ Linux để thu thập dữ liệu AD cho BloodHound GUI.

BloodHound giúp visualize:

- Users.
- Groups.
- Computers.
- Group membership.
- GPOs.
- ACLs.
- Trusts.
- Local admin access.
- User sessions.
- RDP/WinRM access.
- Attack paths.

```text
BloodHound dùng graph theory để tìm relationship và attack path khó thấy bằng tool dòng lệnh.
```

---

# 22. Chạy BloodHound.py

Lệnh mẫu:

```bash
sudo bloodhound-python -u 'forend' -p 'Klmcargo2' -ns 172.16.5.5 -d inlanefreight.local -c all
```

Ý nghĩa:

| Flag | Ý nghĩa |
|---|---|
| `-u` | Username |
| `-p` | Password |
| `-ns` | Nameserver/DC |
| `-d` | Domain |
| `-c all` | Thu thập toàn bộ collection methods phù hợp |

Output thường là JSON:

```text
*_computers.json
*_domains.json
*_groups.json
*_users.json
```

Có thể zip lại:

```bash
zip -r ilfreight_bh.zip *.json
```

---

# 23. BloodHound GUI

Sau khi có JSON/ZIP:

```bash
sudo neo4j start
bloodhound
```

Upload data vào GUI, sau đó dùng Analysis tab.

Query phổ biến:

```text
Find Shortest Paths To Domain Admins
```

Các tab nên xem:

- Database Info.
- Node Info.
- Analysis.
- Raw Query.
- Settings.

BloodHound rất hữu ích để lập kế hoạch lateral movement và privilege escalation trong phạm vi được phép.

---

# 24. Blue Team Detection

Blue Team nên theo dõi:

- CME/NetExec enumeration pattern.
- SMB share enumeration.
- RPC enumeration bất thường.
- LDAP queries số lượng lớn.
- BloodHound/SharpHound collection behavior.
- WMI process creation.
- Remote service creation.
- Event ID 4688.
- Service Control Manager events.
- Access tới ADMIN$.
- Nhiều request LDAP/SMB từ một Linux host.

---

# 25. Red Team Notes

Trong report nên ghi:

```text
Credential used:
Source host:
Target DC:
Commands:
Timestamp:
Users enumerated:
Groups enumerated:
Shares accessible:
Logged-on users found:
Privileged users:
BloodHound collection method:
Output file paths:
Potential attack paths:
```

Nên lưu output:

```bash
sudo crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 --users | tee cme-users.txt
sudo crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 --groups | tee cme-groups.txt
```

---

# 26. Defensive Recommendations

Một số khuyến nghị:

- Monitor credentialed enumeration từ host lạ.
- Hạn chế share READ không cần thiết.
- Review sensitive files trong shares.
- Audit nested privileged groups.
- Bật SMB signing nơi phù hợp.
- Giám sát remote service creation.
- Giám sát WMI remote execution.
- Hạn chế local admin.
- Triển khai LAPS/Windows LAPS.
- Review BloodHound findings định kỳ từ góc nhìn phòng thủ.
- Segment admin workstations/jump servers.

---

# Key Takeaway

```text
Có credential hợp lệ là bước ngoặt lớn trong AD enumeration.
CME/NetExec giúp enumerate users, groups, shares, logged-on users và policy rất nhanh.
SMBMap tốt cho share permissions và recursive listing.
rpcclient hữu ích để query users/RID/domain info qua MS-RPC.
Impacket psexec/wmiexec hỗ trợ remote execution khi có local admin.
Windapsearch giúp LDAP enumeration từ Linux.
BloodHound.py giúp thu thập dữ liệu để visualize attack paths.
```

Credentialed enumeration từ Linux là giai đoạn biến thông tin rời rạc thành bản đồ AD có cấu trúc, giúp xác định privilege path và finding có giá trị cho report.

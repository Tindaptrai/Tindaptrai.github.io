---
layout: post
title: "Detecting Golden Ticket & Silver Ticket"
date: 2026-08-06 00:57:51 +0700
categories: ["Detecting Windows Attacks with Splunk", "Credential Access"]
tags: [golden-ticket, silver-ticket, kerberos, krbtgt, tgt, tgs, dcsync, ntds-dit, lsass, event-id-4720, event-id-4672, event-id-4768, event-id-4769, event-id-4770, splunk, detection-engineering, cdsa]
description: "Phát hiện Golden Ticket và Silver Ticket bằng Kerberos correlation, account baseline, privileged logon và Splunk."
toc: true
---

# Detecting Golden Ticket & Silver Ticket

## Công thức dễ nhớ

```text
GOLDEN TICKET:
KRBTGT SECRET → FORGE TGT → ANY USER/GROUP → DOMAIN-WIDE ACCESS

SILVER TICKET:
SERVICE SECRET → FORGE TGS → ONE SERVICE/HOST → LIMITED ACCESS
```

Hai kỹ thuật đều giả mạo Kerberos ticket, nhưng khác nhau về loại ticket, secret bị lộ và phạm vi truy cập.

---

# 1. Golden Ticket Là Gì?

Golden Ticket là TGT giả mạo bằng secret của tài khoản:

```text
KRBTGT
```

Khi có NTLM hash hoặc Kerberos key của `KRBTGT`, attacker có thể:

```text
Tạo TGT offline
Giả mạo username bất kỳ
Gán group/RID quyền cao
Giả mạo Domain Admin
Xin TGS cho nhiều service
Duy trì quyền truy cập lâu dài
```

Golden Ticket có phạm vi:

```text
Toàn domain
```

---

# 2. Golden Ticket Attack Flow

```text
1. Compromise Domain Controller hoặc quyền replication
2. Trích xuất KRBTGT secret
3. Forge TGT cho identity tùy ý
4. Gán quyền Domain Admin
5. Inject ticket vào logon session
6. Xin TGS và truy cập tài nguyên trong domain
```

Nguồn lấy `KRBTGT` secret:

```text
DCSync
NTDS.dit
LSASS dump trên Domain Controller
```

Tool thường gặp:

```text
Mimikatz
Rubeus
Impacket secretsdump.py
```

---

# 3. Vì Sao Golden Ticket Khó Phát Hiện?

TGT có thể được tạo:

```text
Offline
```

Điều đó có nghĩa:

```text
Không nhất thiết có process tạo ticket trên Domain Controller
Không có AS-REQ hợp lệ tương ứng
Ticket có thể mang identity không tồn tại
Ticket có thể có lifetime bất thường
```

Golden Ticket chỉ xuất hiện rõ khi attacker:

```text
Inject ticket
→ request service ticket
→ truy cập tài nguyên
```

---

# 4. Golden Ticket Detection Opportunities

Nên giám sát cách attacker lấy `KRBTGT` secret:

```text
DCSync activity
NTDS.dit access
LSASS memory access trên Domain Controller
```

Telemetry quan trọng:

```text
Sysmon Event ID 10 → Process Access tới lsass.exe
Directory replication events
Sensitive file access
Process execution trên Domain Controller
```

Golden Ticket cũng có thể được phát hiện như một biến thể Pass-the-Ticket:

```text
4769/4770
+ không có 4768 trước đó
+ cùng user/source
```

---

# 5. Detect Golden Ticket Với Splunk

```spl
index=main
earliest=1690451977 latest=1690452262
source="WinEventLog:Security"
user!=*$
EventCode IN (4768,4769,4770)
| rex field=user "(?<username>[^@]+)"
| rex field=src_ip "(\:\:ffff\:)?(?<src_ip_4>[0-9\.]+)"
| transaction username, src_ip_4
    maxspan=10h
    keepevicted=true
    startswith=(EventCode=4768)
| where closed_txn=0
| search NOT user="*$@*"
| table _time,
        ComputerName,
        username,
        src_ip_4,
        service_name,
        category
```

---

# 6. Logic Của Golden Ticket Query

```text
Thu thập 4768, 4769, 4770
→ loại computer accounts
→ chuẩn hóa username
→ chuẩn hóa IPv4-mapped IPv6
→ gom theo username + source IP
→ transaction phải bắt đầu bằng 4768
→ giữ transaction không hoàn chỉnh
```

Pattern đáng ngờ:

```text
4769 hoặc 4770
+ không có 4768 tương ứng
→ TGT có thể đã được forge/import
```

Giải thích:

```text
4768 = TGT Request
4769 = TGS Request
4770 = TGS Renewal
```

---

# 7. Hạn Chế Của Golden Ticket Detection

Một TGS không có TGT trong time window có thể hợp lệ vì:

```text
TGT được cấp trước time window
Ticket cache tồn tại lâu
User session dài
Log bị thiếu
Domain Controller coverage không đầy đủ
Forwarding delay
Ticket renewal hợp lệ
```

Do đó cần bổ sung:

```text
Privileged identity bất thường
Username không tồn tại
Group membership không hợp lý
Ticket lifetime bất thường
Source host không thường dùng account đó
DCSync/NTDS/LSASS activity trước đó
```

---

# 8. Silver Ticket Là Gì?

Silver Ticket là TGS giả mạo bằng secret của:

```text
Service account
hoặc
Computer account
```

Ví dụ service:

```text
CIFS
MSSQL
HTTP
HOST
LDAP
WSMAN
```

Phạm vi Silver Ticket:

```text
Một service cụ thể
trên một host cụ thể
```

---

# 9. Silver Ticket Attack Flow

```text
1. Trích xuất NTLM hash/AES key của service account
2. Xác định SPN và target host
3. Forge TGS cho service
4. Giả mạo user bất kỳ
5. Inject TGS vào logon session
6. Truy cập service tương ứng
```

Ví dụ:

```text
CIFS/iis.lab.internal.local
→ truy cập C$
```

Tool thường gặp:

```text
Mimikatz
Rubeus
```

---

# 10. Golden Ticket Và Silver Ticket Khác Nhau

| Thuộc tính | Golden Ticket | Silver Ticket |
|---|---|---|
| Ticket giả mạo | TGT | TGS |
| Secret cần có | `KRBTGT` | Service/computer account |
| Phạm vi | Toàn domain | Một service/host |
| KDC involvement | Dùng TGT để xin TGS | Có thể không cần liên hệ KDC |
| Impact | Rất cao | Hẹp hơn |
| Persistence | Dài hạn | Theo service secret |

---

# 11. Silver Ticket Detection Challenges

Silver Ticket có thể không tạo:

```text
4768
4769
```

trên Domain Controller vì attacker đã forge TGS offline và gửi trực tiếp tới service.

Do đó cần tìm inconsistency giữa:

```text
Identity xuất hiện trong logon
và
identity thực sự tồn tại trong AD
```

Attacker có thể giả mạo:

```text
User tồn tại
User không tồn tại
Privileged group membership
```

---

# 12. Event IDs Cho Silver Ticket

| Event ID | Ý nghĩa |
|---|---|
| `4720` | User account created |
| `4624` | Successful logon |
| `4672` | Special privileges assigned |

Hai hướng detection:

```text
1. User logon nhưng không tồn tại trong account baseline
2. Account mới/bất thường nhận special privileges
```

---

# 13. Tạo Baseline User Bằng Event 4720

```spl
index=main
latest=1690448444
EventCode=4720
| stats min(_time) AS _time,
        values(EventCode) AS EventCode
  BY user
| outputlookup users.csv
```

Ý nghĩa:

```text
4720 = User account was created
→ tạo danh sách user từng được tạo
→ lưu vào users.csv
```

Output:

```text
users.csv
```

Trong Splunk có thể upload lookup tại:

```text
Settings
→ Lookups
→ Lookup table files
→ New Lookup Table File
```

---

# 14. Detect Logon Của User Không Có Trong Baseline

```spl
index=main
latest=1690545656
EventCode=4624
| stats min(_time) AS firstTime,
        values(ComputerName) AS ComputerName,
        values(EventCode) AS EventCode
  BY user
| eval last24h=1690451977
| where firstTime > last24h
| convert ctime(firstTime)
| convert ctime(last24h)
| lookup users.csv user AS user
    OUTPUT EventCode AS Events
| where isnull(Events)
```

Có thể dùng time động:

```spl
| eval last24h=relative_time(now(),"-24h@h")
```

---

# 15. Logic Của User Correlation Query

```text
Lấy 4624 successful logon
→ tìm lần logon đầu tiên của mỗi user
→ chỉ giữ logon mới trong 24 giờ
→ lookup users.csv
→ nếu user không tồn tại trong lookup
→ nghi forged identity
```

Keyword SPL:

```text
stats min(_time)
outputlookup
lookup
isnull
relative_time
convert ctime
```

---

# 16. Hạn Chế Của `users.csv`

Chỉ dùng Event 4720 để tạo danh sách user có thể thiếu:

```text
Account có trước thời gian ingest log
Built-in accounts
Imported/migrated accounts
Service accounts cũ
Log retention không đủ
```

Baseline tốt hơn nên lấy từ:

```text
Active Directory inventory
Identity management platform
Periodic LDAP export
CMDB/IAM source
```

`users.csv` trong module chủ yếu minh họa detection logic.

---

# 17. Detect Special Privileges Trên Logon Mới

```spl
index=main
latest=1690545656
EventCode=4672
| stats min(_time) AS firstTime,
        values(ComputerName) AS ComputerName
  BY Account_Name
| eval last24h=1690451977
| where firstTime > last24h
| table firstTime,
        ComputerName,
        Account_Name
| convert ctime(firstTime)
```

Có thể thay timestamp cố định bằng:

```spl
| eval last24h=relative_time(now(),"-24h@h")
```

---

# 18. Logic Của Event 4672 Detection

```text
4672 = Special privileges assigned
```

Detection tìm:

```text
Account lần đầu xuất hiện với quyền đặc biệt
+ trong time window gần đây
```

Đặc biệt đáng ngờ khi:

```text
Account chưa từng tồn tại
Account không thuộc nhóm admin hợp lệ
Account xuất hiện trên server nhạy cảm
Không có 4720/identity record tương ứng
```

---

# 19. Detection Correlation Tốt Hơn

## Golden Ticket

```text
DCSync/NTDS/LSASS access
+ forged/imported TGT behavior
+ 4769/4770 không có 4768
+ privileged access
```

## Silver Ticket

```text
4624 của identity bất thường
+ 4672 special privileges
+ không có user trong AD baseline
+ không có Kerberos request tương ứng trên DC
```

---

# 20. False Positives

Có thể đến từ:

```text
Log retention thiếu
Lookup chưa đầy đủ
Account migration
Break-glass accounts
Service account provisioning
Long-lived Kerberos sessions
Cross-domain trusts
Scheduled administrative activity
```

Tune theo:

```text
Authoritative AD user inventory
Known privileged accounts
Known service accounts
Host role
Source IP baseline
Ticket lifetime
Trust relationships
Admin maintenance windows
```

---

# 21. Investigation Workflow

```text
1. Xác định username và source host
2. Kiểm tra user có tồn tại trong AD không
3. Kiểm tra group membership
4. Kiểm tra 4768/4769/4770 trên DC
5. Tìm DCSync/NTDS.dit/LSASS activity
6. Kiểm tra 4624 và 4672 trên service host
7. Xác định SPN/service bị truy cập
8. Hunt Mimikatz/Rubeus execution
9. Kiểm tra lateral movement tiếp theo
10. Rotate KRBTGT hoặc service secret nếu xác nhận
```

---

# 22. Response Considerations

## Golden Ticket

```text
Reset KRBTGT password hai lần
theo quy trình kiểm soát phù hợp
```

Ngoài ra:

```text
Contain compromised Domain Controller
Revoke privileged sessions
Investigate DCSync source
Rotate privileged credentials
Hunt domain-wide lateral movement
```

## Silver Ticket

```text
Rotate service/computer account secret
Purge Kerberos tickets
Restart affected service nếu cần
Review SPN ownership
Investigate target host
```

---

# Keyword Cần Nhớ

```text
Golden Ticket
Silver Ticket
KRBTGT
TGT
TGS
DCSync
NTDS.dit
LSASS
Mimikatz
Rubeus
CIFS
MSSQL
HTTP
SPN
4768
4769
4770
4720
4624
4672
Sysmon Event ID 10
transaction
closed_txn
outputlookup
lookup
users.csv
isnull
relative_time
special privileges
forged identity
```

---

# Key Takeaway

```text
Golden Ticket:
forge TGT bằng KRBTGT secret
→ phạm vi toàn domain
```

```text
Silver Ticket:
forge TGS bằng service secret
→ phạm vi một service/host
```

```text
Detection tốt =
ticket-flow anomalies
+ identity baseline
+ privilege anomalies
+ credential theft telemetry
```

> Chỉ thực hành trên HTB, lab hoặc hệ thống được cấp phép.

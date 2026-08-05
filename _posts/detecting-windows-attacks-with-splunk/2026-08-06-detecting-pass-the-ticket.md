---
layout: post
title: "Detecting Pass-the-Ticket"
date: 2026-08-06 00:36:05 +0700
categories: ["Detecting Windows Attacks with Splunk", "Credential Access"]
tags: [pass-the-ticket, ptt, kerberos, tgt, tgs, mimikatz, rubeus, event-id-4768, event-id-4769, event-id-4770, event-id-4771, splunk, lateral-movement, detection-engineering, cdsa]
description: "Phát hiện Pass-the-Ticket bằng cách correlate TGT/TGS events, source IP và hành vi Kerberos bất thường trong Splunk."
toc: true
---

# Detecting Pass-the-Ticket

## Công thức dễ nhớ

```text
STEAL TICKET
→ IMPORT TGT/TGS
→ REQUEST/USE SERVICE TICKET
→ NO PRIOR TGT REQUEST
→ LATERAL MOVEMENT
```

Pass-the-Ticket sử dụng **Kerberos ticket hợp lệ** thay cho plaintext password hoặc NTLM hash.

---

# 1. Pass-the-Ticket Là Gì?

PtT lợi dụng:

```text
TGT — Ticket Granting Ticket
TGS — Ticket Granting Service Ticket
```

Chuỗi tấn công:

```text
1. Attacker có quyền admin/SYSTEM trên host
2. Trích xuất ticket từ memory
3. Import ticket vào logon session hiện tại
4. Dùng ticket truy cập service hoặc host khác
5. Di chuyển ngang mà không cần password
```

Tool thường gặp:

```text
Mimikatz
Rubeus
klist
```

---

# 2. Kerberos Authentication Bình Thường

Luồng hợp lệ:

```text
4768 → Client request TGT
4769 → Client request TGS
4624 → Successful logon trên service host
4672 → Special privileges, nếu user có quyền cao
4648 → Explicit credentials, nếu dùng credential tường minh
```

Kerberos flow:

```text
Client
→ xin TGT từ KDC
→ dùng TGT xin TGS
→ gửi TGS tới service
→ service chấp nhận authentication
```

---

# 3. Event IDs Quan Trọng

| Event ID | Ý nghĩa |
|---|---|
| `4648` | Explicit credential logon attempt |
| `4624` | Successful logon |
| `4672` | Special privileges assigned |
| `4768` | Kerberos TGT Request |
| `4769` | Kerberos Service Ticket Request |
| `4770` | Kerberos Service Ticket Renewed |
| `4771` | Kerberos Pre-Authentication Failed |

Field cần quan sát:

```text
user
src_ip
ComputerName
service_name
Ticket_Encryption_Type
Ticket_Options
Pre_Authentication_Type
Failure_Code
category
```

---

# 4. Detection Opportunity Chính

Khi attacker import một TGT đã đánh cắp:

```text
Host attacker có thể request TGS
nhưng trước đó không request TGT trên chính host đó
```

Pattern:

```text
4769 hoặc 4770
+ không có 4768 trước đó
+ cùng user
+ cùng source IP
+ trong một time window hợp lý
```

Đây là điểm khác biệt giữa:

```text
Kerberos flow đầy đủ
vs
Kerberos flow bị thiếu bước TGT request
```

---

# 5. Splunk Query

```spl
index=main
earliest=1690392405 latest=1690451745
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

# 6. Query Breakdown

## Data Selection

```spl
EventCode IN (4768,4769,4770)
```

Thu thập:

```text
TGT requests
TGS requests
TGS renewals
```

## Exclude Computer Accounts

```spl
user!=*$
```

Loại account kết thúc bằng `$`.

## Normalize Username

```spl
| rex field=user "(?<username>[^@]+)"
```

Ví dụ:

```text
Administrator@CORP.LOCAL
→ Administrator
```

## Normalize Source IP

```spl
| rex field=src_ip "(\:\:ffff\:)?(?<src_ip_4>[0-9\.]+)"
```

Ví dụ:

```text
::ffff:10.10.0.100
→ 10.10.0.100
```

## Transaction Logic

```spl
| transaction username, src_ip_4
    maxspan=10h
    keepevicted=true
    startswith=(EventCode=4768)
```

Ý nghĩa:

```text
Gom event theo user + source IP
Transaction phải bắt đầu bằng 4768
Giới hạn tối đa 10 giờ
Giữ cả transaction không hoàn chỉnh
```

## Suspicious Open Transactions

```spl
| where closed_txn=0
```

Giữ các chuỗi Kerberos không bắt đầu bằng TGT request hợp lệ.

---

# 7. Ý Nghĩa `closed_txn=0`

```text
closed_txn=1
→ transaction hoàn chỉnh theo điều kiện

closed_txn=0
→ transaction bị thiếu event mở đầu/kết thúc
```

Trong detection này:

```text
4769/4770
+ không có 4768 tương ứng
→ nghi ticket đã được import
```

---

# 8. Detection Opportunity Thứ Hai

Có thể tìm mismatch giữa:

```text
Event 4769:
service_name / Service ID / source host

Sysmon Event ID 3:
Source IP / Destination IP / process
```

Pattern đáng ngờ:

```text
Service ticket cho một host/service
nhưng network connection thực tế không khớp
```

Tuy nhiên:

```text
Legitimate mismatch có thể xảy ra
do alias, load balancer, proxy, DNS, cluster hoặc service mapping
```

---

# 9. Event 4771 Và TGS Import

Khi attacker import TGS hoặc có Kerberos credential inconsistency, cần xem:

```text
Event ID 4771
```

Ví dụ đáng chú ý:

```text
Pre-Authentication Type = 2
Failure Code = 0x18
```

Ý nghĩa:

```text
Client gửi encrypted timestamp
nhưng KDC không giải mã được
→ pre-authentication information invalid
```

Đây là dấu hiệu cần điều tra thêm, không phải kết luận độc lập.

---

# 10. Tuning Và False Positives

Một 4769 không có 4768 có thể hợp lệ do:

```text
TGT đã được request trước time window
Ticket cache tồn tại từ trước
Log forwarding bị thiếu
DC log không đầy đủ
Client reconnect
Ticket renewal
Long-lived user session
Service account behavior
```

Nên tune theo:

```text
Ticket lifetime
User baseline
Source host baseline
Known jump servers
Known service accounts
Time window
Log completeness
Domain controller coverage
```

`maxspan=10h` nên phù hợp với ticket lifetime và môi trường thực tế.

---

# 11. Detection Confidence

## Low Confidence

```text
4769 hoặc 4770 không thấy 4768
```

## Medium Confidence

```text
Không có 4768
+ source host bất thường
+ privileged user
+ service_name nhạy cảm
```

## High Confidence

```text
Không có 4768
+ Rubeus/Mimikatz execution
+ ticket import artifact
+ remote SMB/WMI/WinRM activity
```

---

# 12. Correlation Nên Bổ Sung

```text
Sysmon Event ID 1  → process creation
Sysmon Event ID 10 → LSASS access
Sysmon Event ID 3  → network connection
4624               → successful logon
4672               → privileged logon
4768/4769/4770     → Kerberos tickets
4771               → pre-authentication failure
```

Process nên hunt:

```text
Rubeus.exe
mimikatz.exe
powershell.exe
rundll32.exe
cmd.exe
```

Command keyword:

```text
ptt
kirbi
asktgt
asktgs
dump
monitor
klist
```

---

# 13. Investigation Workflow

```text
1. Xác định username và source IP
2. Kiểm tra có 4768 trước 4769/4770 không
3. Mở rộng time window
4. Kiểm tra ticket lifetime
5. Xem service_name và target host
6. Hunt Rubeus/Mimikatz execution
7. Kiểm tra LSASS access
8. Correlate 4624/4672 trên máy đích
9. Hunt SMB/WMI/WinRM/service creation
10. Contain host và invalidate ticket
```

Response có thể gồm:

```text
Reset password
Rotate service account secret
Purge Kerberos tickets
Disable compromised account
Isolate source host
Investigate lateral movement
```

---

# 14. PtH Và PtT Khác Nhau

| Thuộc tính | Pass-the-Hash | Pass-the-Ticket |
|---|---|---|
| Credential material | NTLM hash | Kerberos ticket |
| Protocol | NTLM | Kerberos |
| Detection chính | LSASS access + Type 9 | TGS without prior TGT |
| Event nổi bật | `4624`, Sysmon `10` | `4768`, `4769`, `4770` |
| Tool | Mimikatz | Mimikatz, Rubeus |
| Mục tiêu | Remote authentication | Kerberos service access |

---

# Keyword Cần Nhớ

```text
Pass-the-Ticket
PtT
Kerberos
TGT
TGS
KDC
Mimikatz
Rubeus
klist
kirbi
4768
4769
4770
4771
4624
4672
4648
Pre_Authentication_Type
Failure_Code
closed_txn
transaction
maxspan
keepevicted
src_ip
service_name
ticket cache
lateral movement
```

---

# Key Takeaway

```text
PtT detection không tìm ticket xấu.
Nó tìm Kerberos flow bị thiếu hoặc không hợp lý.
```

```text
Detection chính:
4769/4770
+ không có 4768 trước đó
+ cùng user/source
+ behavior bất thường
```

```text
Detection tốt =
Kerberos correlation
+ process telemetry
+ network telemetry
+ user/host baseline
```

> Chỉ thực hành trên HTB, lab hoặc hệ thống được cấp phép.

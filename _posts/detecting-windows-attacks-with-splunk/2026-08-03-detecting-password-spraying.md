---
layout: post
title: "Detecting Password Spraying"
date: 2026-08-03 23:56:30 +0700
categories: ["Detecting Windows Attacks with Splunk", "Credential Access"]
tags: [password-spraying, brute-force, windows-security, event-id-4625, kerberos, ntlm, splunk, detection-engineering, threat-hunting, soc, cdsa]
description: "Tóm tắt cách phát hiện password spraying bằng Windows Security Logs và Splunk."
toc: true
---

# Detecting Password Spraying

## Công thức dễ nhớ

```text
ONE SOURCE → MANY USERS → FEW PASSWORDS → SHORT WINDOW
```

Password spraying khác brute-force truyền thống:

```text
Brute-force:
1 user + nhiều password

Password spraying:
nhiều user + ít password phổ biến
```

Mục tiêu của attacker:

```text
Tránh account lockout
→ giảm số lần thử trên từng tài khoản
→ phân tán thất bại trên nhiều user
```

# 1. Dấu Hiệu Phát Hiện

Pattern điển hình:

```text
Cùng source IP
→ nhiều failed logon
→ nhiều username khác nhau
→ trong thời gian ngắn
```

Điểm quan trọng:

```text
Không chỉ đếm tổng số thất bại.
Phải đếm số user khác nhau.
```

# 2. Windows Event IDs Quan Trọng

| Event ID | Ý nghĩa |
|---|---|
| `4625` | Failed Logon |
| `4768` + `0x6` | Kerberos invalid user |
| `4768` + `0x12` | Kerberos disabled user |
| `4776` + `0xC0000064` | NTLM invalid user |
| `4776` + `0xC000006A` | NTLM wrong password |
| `4648` | Logon using explicit credentials |
| `4771` | Kerberos pre-authentication failed |

Keyword cần nhớ:

```text
4625
4768
4776
4771
4648
Failure_Reason
Source_Network_Address
src
user
dest
```

# 3. Splunk Query

```spl
index=main
earliest=1690280680 latest=1690289489
source="WinEventLog:Security"
EventCode=4625
| bin span=15m _time
| stats values(user) AS Users,
        dc(user) AS dc_user
  BY src,
     Source_Network_Address,
     dest,
     EventCode,
     Failure_Reason
```

# 4. Query Breakdown

## Data Source

```spl
source="WinEventLog:Security"
EventCode=4625
```

Ý nghĩa:

```text
Windows Security Log
+ failed logon events
```

## Time Binning

```spl
| bin span=15m _time
```

Chia event thành các cửa sổ 15 phút để tìm burst đăng nhập thất bại.

## Aggregation

```spl
| stats values(user) AS Users,
        dc(user) AS dc_user
```

```text
values(user)
= danh sách username bị thử

dc(user)
= số username khác nhau
```

## Grouping

```spl
BY src,
   Source_Network_Address,
   dest,
   EventCode,
   Failure_Reason
```

Giúp xác định:

```text
Nguồn tấn công
IP nguồn
Máy đích
Loại sự kiện
Lý do thất bại
```

# 5. Threshold Nên Thêm

Query gốc chưa đặt ngưỡng cuối cùng.

Có thể bổ sung:

```spl
| where dc_user >= 5
```

Ví dụ hoàn chỉnh:

```spl
index=main
source="WinEventLog:Security"
EventCode=4625
| bin span=15m _time
| stats values(user) AS Users,
        dc(user) AS dc_user,
        count AS failed_attempts
  BY _time,
     src,
     Source_Network_Address,
     dest,
     Failure_Reason
| where dc_user >= 5
| sort - dc_user
```

Logic:

```text
Trong 15 phút
+ cùng source
+ ít nhất 5 user khác nhau
→ nghi password spraying
```

# 6. Field Quan Trọng

```text
_time
src
Source_Network_Address
dest
user
EventCode
Failure_Reason
Logon_Type
Workstation_Name
Authentication_Package
```

Nên bổ sung khi điều tra:

```text
Logon Type
TargetUserName
Workstation
Authentication Package
Source Port
```

# 7. False Positives

Có thể xuất hiện từ:

```text
VPN concentrator
Citrix/Terminal Server
Shared jump host
Misconfigured service
Expired cached credentials
Identity synchronization
Vulnerability scanner
Helpdesk testing
```

Nên tune theo:

```text
Known scanners
Approved admin hosts
Service accounts
VPN gateways
Expected authentication servers
User count baseline
Time window
Failure reason
```

# 8. Investigation Workflow

```text
1. Xác định source IP/host
2. Đếm số user bị thử
3. Kiểm tra thời gian và tần suất
4. Xem Failure_Reason
5. Kiểm tra có login thành công sau đó không
6. Correlate 4624 với cùng source
7. Kiểm tra account lockout 4740
8. Kiểm tra Kerberos/NTLM events
9. Hunt lateral movement tiếp theo
10. Block/reset/contain nếu xác nhận
```

Các event nên correlate:

```text
4624 = Successful Logon
4740 = Account Locked Out
4768 = Kerberos TGT Request
4771 = Kerberos Pre-Auth Failed
4776 = NTLM Authentication
4648 = Explicit Credentials
```

# 9. Detection Logic Chuẩn

```text
Same source
+ many distinct users
+ short time window
+ repeated failures
+ possible later success
```

Có thể tăng độ tin cậy bằng:

```text
Failure spike
+ one successful login
+ privileged account targeted
+ unusual source host
```

# 10. Key Takeaway

```text
Password spraying không nổi bật ở số lần thử trên một user.
Nó nổi bật ở số lượng user bị thử từ cùng một nguồn.
```

```text
Detection tốt =
same source
+ distinct user count
+ time window
+ threshold
+ success correlation
```

> Chỉ thực hành trên HTB, lab hoặc hệ thống được cấp phép.

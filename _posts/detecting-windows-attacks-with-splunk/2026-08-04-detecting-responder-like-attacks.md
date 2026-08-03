---
layout: post
title: "Detecting Responder-like Attacks"
date: 2026-08-04 00:10:44 +0700
categories: ["Detecting Windows Attacks with Splunk", "Credential Access"]
tags: [responder, llmnr, nbt-ns, mdns, netntlm, name-resolution-poisoning, sysmon, event-id-22, event-id-4648, splunk, threat-hunting, detection-engineering, cdsa]
description: "Tóm tắt cách phát hiện Responder-like attacks bằng honeypot hostname, PowerShell Event Log, Sysmon DNS Query và Windows Event 4648."
toc: true
---

# Detecting Responder-like Attacks

## Công thức dễ nhớ

```text
DNS FAILS
→ LLMNR/NBT-NS/mDNS QUERY
→ ROGUE RESPONSE
→ VICTIM AUTHENTICATES
→ NetNTLM CAPTURED
```

Responder-like attacks lợi dụng cơ chế phân giải tên cục bộ khi DNS không giải được hostname.

---

# 1. Giao Thức Bị Lợi Dụng

| Giao thức | Vai trò | Port thường gặp |
|---|---|---:|
| LLMNR | Link-local name resolution | UDP 5355 |
| NBT-NS | NetBIOS Name Service | UDP 137 |
| mDNS | Multicast DNS | UDP 5353 |

Các giao thức này được dùng khi:

```text
FQDN/DNS resolution thất bại
→ máy nạn nhân broadcast/multicast query
→ attacker giả mạo câu trả lời
```

---

# 2. Attack Flow

Ví dụ người dùng gõ sai:

```text
\\fileshrae
```

Chuỗi tấn công:

```text
1. Victim gửi DNS query
2. DNS trả về không tồn tại
3. Victim gửi LLMNR/NBT-NS/mDNS query
4. Responder trả lời bằng IP của attacker
5. Victim kết nối tới rogue host
6. NetNTLM challenge-response bị thu thập
```

Kết quả có thể là:

```text
NetNTLM hash cracking
hoặc
NTLM relay
```

---

# 3. Dấu Hiệu Phát Hiện

Pattern quan trọng:

```text
Hostname không tồn tại
→ nhưng lại được resolve thành công
```

Dấu hiệu khác:

```text
Nhiều LLMNR/NBT-NS response từ một source
Rogue IP trả lời cho nhiều hostname
Authentication tới file share lạ
DNS query cho hostname bị gõ sai
Explicit credential use tới rogue server
```

---

# 4. Honeypot Hostname Detection

Ý tưởng:

```text
Tạo hostname giả chắc chắn không tồn tại
→ định kỳ truy vấn hostname đó
→ nếu resolve thành công
→ nghi có poisoner trong mạng
```

Ví dụ hostname giả:

```text
COPY-NY-DC-02
```

Ưu điểm:

```text
Đơn giản
Chủ động
Ít phụ thuộc chữ ký
Phát hiện attacker đang spoofing
```

---

# 5. Ghi Event Bằng PowerShell

Tạo custom source:

```powershell
New-EventLog `
  -LogName Application `
  -Source LLMNRDetection
```

Ghi cảnh báo:

```powershell
Write-EventLog `
  -LogName Application `
  -Source LLMNRDetection `
  -EventId 19001 `
  -Message $msg `
  -EntryType Warning
```

Keyword cần nhớ:

```text
Application Log
LLMNRDetection
Event ID 19001
Warning
Scheduled Task
```

PowerShell detection script có thể chạy định kỳ bằng Scheduled Task.

---

# 6. Detect Custom LLMNR Alert Với Splunk

```spl
index=main
earliest=1690290078 latest=1690291207
SourceName=LLMNRDetection
| table _time,
        ComputerName,
        SourceName,
        Message
```

Ý nghĩa:

```text
SourceName=LLMNRDetection
→ lọc custom event do PowerShell script ghi lại
```

Field cần xem:

```text
_time
ComputerName
SourceName
Message
```

Trong ví dụ, message chứa các IP phản hồi cho hostname giả:

```text
::1
10.10.0.221
```

---

# 7. Sysmon Event ID 22

```text
Sysmon Event ID 22
= DNS Query
```

SPL:

```spl
index=main
earliest=1690290078 latest=1690291207
EventCode=22
| table _time,
        Computer,
        user,
        Image,
        QueryName,
        QueryResults
```

Field quan trọng:

```text
Computer
user
Image
QueryName
QueryResults
```

Ví dụ đáng chú ý:

```text
QueryName: myfileshar3
QueryResults: ::1; ::ffff:10.10.0.221;
```

Logic:

```text
Hostname có vẻ bị gõ sai
+ resolve về IP bất thường
→ nghi poisoning
```

---

# 8. Windows Event ID 4648

```text
Event ID 4648
= Logon attempted using explicit credentials
```

SPL:

```spl
index=main
earliest=1690290814 latest=1690291207
EventCode IN (4648)
| table _time,
        EventCode,
        source,
        name,
        user,
        Target_Server_Name,
        Message
| sort 0 _time
```

Event 4648 giúp phát hiện:

```text
Explicit credential use
→ tới rogue file share/server
→ sau bước poisoning
```

Field cần xem:

```text
user
Target_Server_Name
source
Message
```

---

# 9. Correlation Logic

Detection tốt nên correlate:

```text
Sysmon 22
+ custom LLMNRDetection event
+ Event 4648
```

Chuỗi:

```text
Suspicious QueryName
→ QueryResults trỏ tới rogue IP
→ explicit credential use tới target server
```

Có thể tăng độ tin cậy bằng:

```text
Same host
Same user
Same time window
Same rogue IP
```

---

# 10. False Positives

Có thể xuất hiện từ:

```text
Legitimate mDNS devices
Printers
IoT devices
Misconfigured DNS
Old NetBIOS applications
VPN adapters
Localhost/IPv6 loopback
```

Nên tune theo:

```text
Known printers
Approved multicast services
Known infrastructure IPs
Expected hostname patterns
Loopback addresses
Authorized testing hosts
```

---

# 11. Mitigation

Giảm rủi ro bằng:

```text
Disable LLMNR
Disable NBT-NS
Restrict mDNS where possible
Enforce SMB signing
Disable NTLM where feasible
Use strong DNS hygiene
Segment networks
Monitor broadcast name resolution
```

Trong CDSA/SOC:

```text
Network telemetry
+ Sysmon DNS Query
+ Windows Security Logs
+ Splunk correlation
```

---

# 12. Investigation Workflow

```text
1. Xác định hostname bị query
2. Xác định rogue IP trong QueryResults
3. Kiểm tra process tạo query
4. Kiểm tra user và host
5. Correlate Event 4648
6. Kiểm tra SMB/NTLM authentication
7. Tìm thêm victims cùng rogue IP
8. Hunt lateral movement hoặc relay
9. Block/contain rogue host
10. Tune baseline và allowlist
```

Hunt tiếp:

```text
NTLM relay
SMB authentication
Service creation
Scheduled task
PsExec
WMI
Remote logon
```

---

# Keyword Cần Nhớ

```text
Responder
LLMNR
NBT-NS
NBNS spoofing
mDNS
UDP 5355
UDP 137
UDP 5353
NetNTLM
NTLM relay
Honeypot hostname
New-EventLog
Write-EventLog
LLMNRDetection
Event ID 19001
Sysmon Event ID 22
DNS Query
QueryName
QueryResults
Event ID 4648
Explicit Credentials
Target_Server_Name
```

---

# Key Takeaway

```text
Responder detection không chỉ tìm tool.
Nó tìm chuỗi hành vi:
query sai → rogue response → credential use
```

```text
Detection tốt =
honeypot hostname
+ DNS query telemetry
+ explicit logon correlation
```

> Chỉ thực hành trên HTB, lab hoặc hệ thống được cấp phép.

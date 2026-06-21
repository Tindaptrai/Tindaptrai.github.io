---
layout: post
title: "Stuxbot Threat Hunting Query Ideas"
date: 2026-06-21 22:10:00 +0700
categories: [SOC, Threat Hunting]
tags: [soc, threat-hunting, stuxbot, elastic-stack, kibana, kql, zeek, sysmon, powershell, psexec, sharphound, blue-team]
description: "Tóm tắt các ý tưởng query khi hunt Stuxbot trong Elastic Stack: OneNote, invoice.bat, PowerShell, C2 Ngrok, default.exe, SharpHound, PsExec và logon activity."
toc: true
---

# Stuxbot Threat Hunting Query Ideas

Bài này tóm tắt các **ý tưởng query** khi hunt Stuxbot trong Elastic Stack.

Timeline được cập nhật theo giờ hiện tại:

```text
2026-06-21 22:10:00 +0700
```

Mục tiêu là đi theo attack chain:

```text
Phishing email
→ OneNote invoice.one
→ Batch file invoice.bat
→ PowerShell in memory
→ C2 / Ngrok
→ Dropped EXE default.exe
→ SharpHound / AD Discovery
→ Lateral Movement bằng PsExec / WinRM
→ Credential misuse / Logon events
```

---

## 1. Chuẩn bị Kibana

Thiết lập:

```text
Time range: Last 15 years
Timezone: Europe/Copenhagen
```

Nguồn log:

| Data source | Index pattern | Dùng để hunt |
|---|---|---|
| Windows audit logs | `windows*` | Logon, authentication, account activity |
| Sysmon logs | `windows*` | Process, file, network, DNS, hash |
| PowerShell logs | `windows*` | PowerShell execution |
| Zeek logs | `zeek*` | DNS, connection, network visibility |

---

# 2. Query Flow Tổng Quát

## Query 1: Tìm OneNote invoice.one download

```text
event.code:15 AND file.name:*invoice.one
```

Ý nghĩa:

- Sysmon Event ID 15 FileCreateStreamHash.
- Tìm browser download event.
- Phát hiện file OneNote đáng ngờ.

Field nên thêm:

```text
@timestamp
host.hostname
user.name
process.name
process.executable
file.name
file.path
event.code
```

---

## Query 2: Xác nhận file creation / Zone.Identifier

```text
event.code:11 AND file.name:invoice.one*
```

Ý nghĩa:

- Sysmon Event ID 11 File Create.
- Tìm `invoice.one:Zone.Identifier`.
- Zone.Identifier cho thấy file có nguồn gốc từ Internet.

Field nên thêm:

```text
@timestamp
host.hostname
user.name
file.name
file.path
event.code
```

---

## Query 3: Xác định IP của WS001

```text
event.code:3 AND host.hostname:WS001
```

Field nên thêm:

```text
host.hostname
source.ip
destination.ip
destination.port
process.name
event.code
```

Kết quả quan trọng trong case:

```text
WS001 = 192.168.28.130
```

---

## Query 4: Hunt DNS từ WS001 quanh thời điểm download

```text
source.ip:192.168.28.130 AND dns.question.name:*
```

Time range gợi ý:

```text
March 26, 2023 @ 22:05:00
đến
March 26, 2023 @ 22:05:48
```

Field nên thêm:

```text
@timestamp
source.ip
dns.question.name
dns.answers.data
dns.resolved_ip
zeek.dns.answers
```

Noise nên filter:

```text
google.com
google-analytics.com
gstatic.com
play.google.com
mail.google.com
```

Dấu hiệu đáng chú ý:

```text
mail.google.com
file.io
nav-edge.smartscreen.microsoft.com
```

IP file hosting:

```text
34.197.10.85
3.213.216.16
```

---

## Query 5: Xác nhận connection đến file hosting IP

```text
source.ip:192.168.28.130 AND destination.ip:(34.197.10.85 OR 3.213.216.16) AND destination.port:443
```

Ý nghĩa:

- Xác nhận WS001 kết nối đến file hosting.
- Corroborate việc tải `invoice.one`.

---

## Query 6: Tìm OneNote file được mở

```text
event.code:1 AND process.command_line:*invoice.one*
```

Ý nghĩa:

- Sysmon Event ID 1 Process Creation.
- Xác định `invoice.one` có được mở hay không.

Field nên thêm:

```text
@timestamp
host.hostname
user.name
process.name
process.command_line
process.parent.name
event.code
```

---

## Query 7: Tìm child process của OneNote

```text
event.code:1 AND process.parent.name:"ONENOTE.EXE"
```

Ý nghĩa:

- Tìm process do OneNote spawn.
- Phát hiện `cmd.exe` chạy `invoice.bat`.

Field nên thêm:

```text
@timestamp
host.hostname
user.name
process.name
process.command_line
process.parent.name
process.parent.command_line
event.code
```

---

## Query 8: Tìm child process do invoice.bat sinh ra

```text
event.code:1 AND process.parent.command_line:*invoice.bat*
```

Ý nghĩa:

- Tìm PowerShell được batch file gọi.
- Phát hiện download/execute từ Pastebin.

Field nên thêm:

```text
@timestamp
host.hostname
user.name
process.name
process.args
process.command_line
process.parent.command_line
process.pid
event.code
```

---

## Query 9: Điều tra activity của PowerShell PID

Ví dụ PID:

```text
9944
```

Query:

```text
process.pid:"9944" AND process.name:"powershell.exe"
```

Field nên thêm:

```text
@timestamp
event.code
process.name
process.args
process.command_line
file.path
dns.question.name
destination.ip
destination.port
host.hostname
user.name
```

Ý nghĩa:

- Tìm file creation.
- Tìm DNS resolution.
- Tìm network connection.
- Tìm dropped EXE.
- Tìm dấu hiệu password spraying/bruteforce.

---

## Query 10: Hunt Pastebin staging script

```text
process.command_line:*pastebin.com*
```

Hoặc:

```text
url.full:*pastebin.com* OR dns.question.name:*pastebin.com*
```

Ý nghĩa:

- Threat report nói stage PowerShell script được tải từ Pastebin.
- Query này xác nhận staging URL.

---

## Query 11: Hunt Ngrok C2 activity

```text
dns.question.name:*ngrok*
```

Theo host:

```text
source.ip:192.168.28.130 AND dns.question.name:*ngrok*
```

Field nên thêm:

```text
@timestamp
source.ip
dns.question.name
dns.answers.data
destination.ip
destination.port
```

IP C2 quan sát được:

```text
18.158.249.75
3.125.102.39
```

---

## Query 12: Hunt connection đến C2 IP

```text
destination.ip:(18.158.249.75 OR 3.125.102.39) AND destination.port:443
```

Theo WS001:

```text
source.ip:192.168.28.130 AND destination.ip:(18.158.249.75 OR 3.125.102.39) AND destination.port:443
```

Ý nghĩa:

- Xác định C2 communication.
- Kiểm tra activity có kéo dài nhiều ngày không.

---

## Query 13: Tìm default.exe execution

```text
process.name:"default.exe"
```

Field nên thêm:

```text
@timestamp
host.hostname
user.name
process.name
process.args
process.command_line
process.hash.sha256
file.path
destination.ip
dns.question.name
event.code
```

Ý nghĩa:

- `default.exe` là dropped EXE có khả năng persistence.
- Có thể thấy C2, file upload/drop, `svchost.exe`, `payload.exe`, `SharpHound.exe`.

---

## Query 14: Tìm SharpHound execution

```text
process.name:"SharpHound.exe"
```

Ý nghĩa:

- SharpHound dùng để map Active Directory.
- Đây là dấu hiệu attacker đang discovery attack path để escalation.

Field nên thêm:

```text
@timestamp
host.hostname
user.name
process.name
process.args
process.command_line
process.parent.name
event.code
```

---

## Query 15: Hunt theo SHA256 IOC

Hash quan trọng:

```text
018d37cbd3878258c29db3bc3f2988b6ae688843801b9abc28e6151141ab66d4
```

Query:

```text
process.hash.sha256:018d37cbd3878258c29db3bc3f2988b6ae688843801b9abc28e6151141ab66d4
```

Field nên thêm:

```text
@timestamp
host.hostname
user.name
process.name
process.executable
process.command_line
process.parent.name
process.parent.executable
process.hash.sha256
event.code
```

Ý nghĩa:

- Tìm cùng malware trên host khác.
- Trong case này phát hiện trên `WS001` và `PKI`.

---

## Query 16: Xác định PsExec lateral movement

```text
process.parent.name:"PSEXESVC.exe"
```

Hoặc:

```text
process.parent.executable:*PSEXESVC.exe
```

Ý nghĩa:

- `PSEXESVC.exe` là service của PsExec.
- Nếu PsExec spawn malware trên PKI, đây là bằng chứng lateral movement.

---

## Query 17: Kiểm tra account svc-sql1 bị compromise

Theo user:

```text
user.name:"svc-sql1"
```

Theo path:

```text
file.path:*svc-sql1*
```

Theo process executable:

```text
process.executable:*svc-sql1*
```

Ý nghĩa:

- Xác nhận service account bị misuse.
- Tìm file/backdoor dưới profile `svc-sql1`.

---

## Query 18: Logon activity từ WS001

```text
(event.code:4624 OR event.code:4625) AND winlog.event_data.LogonType:3 AND source.ip:192.168.28.130
```

Field nên thêm:

```text
@timestamp
event.code
host.hostname
user.name
source.ip
winlog.event_data.LogonType
winlog.event_data.TargetUserName
winlog.event_data.Status
winlog.event_data.SubStatus
```

Ý nghĩa:

- Tìm failed logon vào administrator.
- Tìm successful logon bằng `svc-sql1`.
- Chứng minh credential misuse hoặc lateral movement.

---

## Query 19: Loại noise Bob và computer account

```text
(event.code:4624 OR event.code:4625) AND winlog.event_data.LogonType:3 AND source.ip:192.168.28.130 AND NOT user.name:"bob"
```

Loại computer account:

```text
NOT user.name: *$
```

Kết hợp:

```text
(event.code:4624 OR event.code:4625) AND winlog.event_data.LogonType:3 AND source.ip:192.168.28.130 AND NOT user.name:"bob" AND NOT user.name: *$
```

---

## Query 20: Hunt WinRM lateral movement

```text
process.name:"wsmprovhost.exe"
```

```text
process.command_line:*WinRM* OR process.command_line:*winrs*
```

```text
event.code:4624 AND winlog.event_data.LogonType:3
```

Ý nghĩa:

- Threat report nói Stuxbot có thể dùng WinRM.
- Nên hunt remote management activity bất thường.

---

## Query 21: Hunt toàn bộ IOC trong report

### OneNote URLs

```text
url.full:("https://transfer.sh/get/kNxU7/invoice.one" OR "https://mega.io/dl9o1Dz/invoice.one")
```

### Pastebin URLs

```text
url.full:("https://pastebin.com/raw/AvHtdKb2" OR "https://pastebin.com/raw/gj58DKz")
```

### C2 IPs

```text
destination.ip:(91.90.213.14 OR 103.248.70.64 OR 141.98.6.59) AND destination.port:443
```

### SHA256 hashes

```text
process.hash.sha256:(226A723FFB4A91D9950A8B266167C5B354AB0DB1DC225578494917FE53867EF2 OR C346077DAD0342592DB753FE2AB36D2F9F1C76E55CF8556FE5CDA92897E99C7E OR 018D37CBD3878258C29DB3BC3F2988B6AE688843801B9ABC28E6151141AB66D4)
```

Lưu ý: Elastic có thể lưu hash dạng lowercase, nên nên thử lowercase nếu uppercase không có kết quả.

---

# 3. Timeline điều tra tóm tắt

| Thời điểm | Sự kiện |
|---|---|
| Mar 26, 2023 22:05:47 | Bob trên WS001 download `invoice.one` |
| Mar 26, 2023 22:05:53 | `invoice.one` được mở |
| Sau đó | OneNote spawn `cmd.exe` chạy `invoice.bat` |
| Sau đó | `invoice.bat` spawn PowerShell |
| Sau đó | PowerShell tải script từ Pastebin |
| Sau đó | PowerShell query Ngrok và kết nối C2 |
| Sau đó | `default.exe` xuất hiện và chạy |
| Mar 27, 2023 | SharpHound chạy để AD discovery |
| Mar 27, 2023 22:18:12 | Malware/hash xuất hiện trên PKI qua `PSEXESVC.exe` |
| Mar 28, 2023 | Successful logon bằng `svc-sql1` từ WS001 |

---

# 4. Kết luận hunting

Từ các query trên, có thể kết luận:

```text
WS001 của user Bob bị compromise qua phishing OneNote invoice.one.
OneNote kích hoạt invoice.bat, batch file chạy PowerShell để tải script từ Pastebin.
PowerShell tạo hoạt động C2 qua Ngrok và drop default.exe.
default.exe tiếp tục C2, upload/drop svchost.exe và SharpHound.exe.
SharpHound được chạy để thu thập thông tin Active Directory.
Hash IOC xuất hiện trên cả WS001 và PKI.
PKI bị compromise thông qua PsExec/PSEXESVC.exe.
Account svc-sql1 có dấu hiệu bị compromise và được dùng cho logon/lateral movement.
```

---

# 5. Query checklist nhanh

```text
event.code:15 AND file.name:*invoice.one
event.code:11 AND file.name:invoice.one*
event.code:3 AND host.hostname:WS001
source.ip:192.168.28.130 AND dns.question.name:*
source.ip:192.168.28.130 AND destination.ip:(34.197.10.85 OR 3.213.216.16)
event.code:1 AND process.command_line:*invoice.one*
event.code:1 AND process.parent.name:"ONENOTE.EXE"
event.code:1 AND process.parent.command_line:*invoice.bat*
process.pid:"9944" AND process.name:"powershell.exe"
process.command_line:*pastebin.com*
dns.question.name:*ngrok*
destination.ip:(18.158.249.75 OR 3.125.102.39) AND destination.port:443
process.name:"default.exe"
process.name:"SharpHound.exe"
process.hash.sha256:018d37cbd3878258c29db3bc3f2988b6ae688843801b9abc28e6151141ab66d4
process.parent.name:"PSEXESVC.exe"
user.name:"svc-sql1"
(event.code:4624 OR event.code:4625) AND winlog.event_data.LogonType:3 AND source.ip:192.168.28.130
```

---

# Key Takeaway

Hunt Stuxbot nên đi theo **attack chain** thay vì chỉ IOC matching.

Bắt đầu từ `invoice.one`, lần theo process tree của OneNote, batch file, PowerShell, C2 Ngrok, dropped executable, SharpHound, hash IOC, PsExec lateral movement và cuối cùng là logon activity của `svc-sql1`.

Cách này giúp analyst không chỉ biết có IOC xuất hiện, mà còn hiểu được toàn bộ chuỗi compromise và có đủ bằng chứng để kích hoạt Incident Response.

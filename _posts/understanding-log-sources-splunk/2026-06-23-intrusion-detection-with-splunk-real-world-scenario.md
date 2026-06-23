---
layout: post
title: "Intrusion Detection With Splunk: Real-world Scenario"
date: 2026-06-23 15:17:00 +0700
categories: [SOC, Splunk]
tags: [soc, splunk, intrusion-detection, spl, sysmon, windows-event-logs, dcsync, lsass-dumping, calltrace, threat-hunting, blue-team]
description: "Tóm tắt bài Intrusion Detection With Splunk: search hiệu quả, phân tích Sysmon, phát hiện parent-child process bất thường, DCSync, LSASS dumping và alert CallTrace UNKNOWN."
toc: true
---

# Intrusion Detection With Splunk: Real-world Scenario

Bài này mở rộng từ phân tích log trên một máy sang điều tra trên dataset lớn trong Splunk.

```text
Goal = tìm TTP trong nhiều máy → validate bằng log → giảm false positive → tạo hunting query/alert.
```

---

## Timeline cập nhật

```text
Created at: 2026-06-23 15:17:00 +0700
Timezone: Asia/Ho_Chi_Minh
Module: Understanding Log Sources & Investigating with Splunk
Section: Intrusion Detection With Splunk
```

---

# 1. Bối cảnh

Dataset trong lab có hơn 500.000 events. Ta không cố tìm mọi cuộc tấn công, mà học cách bắt đầu điều tra trong môi trường lớn bằng Splunk.

Nguồn log chính:

- Windows Event Logs.
- Sysmon.
- Linux syslog.
- Nhiều sourcetypes trong index `main`.

Truy vấn toàn bộ dữ liệu:

```spl
index="main" earliest=0
```

---

# 2. Xác định sourcetypes

Bước đầu trong môi trường lạ là xem có những loại log nào:

```spl
index="main"
| stats count by sourcetype
```

Sau đó tập trung vào Sysmon:

```spl
index="main" sourcetype="WinEventLog:Sysmon"
```

---

# 3. Search hiệu quả trong Splunk

Không nên search quá rộng nếu đã biết field.

Search rộng:

```spl
index="main" uniwaldo.local
```

Search wildcard rộng sẽ chậm hơn:

```spl
index="main" *uniwaldo.local*
```

Search theo field cụ thể hiệu quả hơn:

```spl
index="main" ComputerName="*uniwaldo.local"
```

Bài học:

```text
Càng chỉ rõ index, sourcetype, EventCode, field và time range thì query càng nhanh và ít noise.
```

---

# 4. Xem Sysmon EventCodes

```spl
index="main" sourcetype="WinEventLog:Sysmon"
| stats count by EventCode
```

Các EventCode quan trọng:

| EventCode | Ý nghĩa | Dùng để hunt |
|---|---|---|
| 1 | Process Creation | Parent-child process, command line |
| 3 | Network Connection | C2, outbound traffic |
| 7 | Image Loaded | DLL hijacking |
| 8 | CreateRemoteThread | Injection |
| 10 | ProcessAccess | LSASS dumping, process injection |
| 11 | FileCreate | Dropped/downloaded files |
| 12/13 | Registry events | Persistence |
| 15 | FileCreateStreamHash | Mark-of-the-Web |
| 17/18 | Pipe events | PsExec/SMB lateral movement |
| 22 | DNS Query | C2/domain beacon |
| 25 | Process Tampering | Hollowing/herpaderping |

---

# 5. Hunt parent-child process bất thường

Xem parent-child process tree:

```spl
index="main" sourcetype="WinEventLog:Sysmon" EventCode=1
| stats count by ParentImage, Image
```

Tập trung vào `cmd.exe` và `powershell.exe`:

```spl
index="main" sourcetype="WinEventLog:Sysmon" EventCode=1 (Image="*cmd.exe" OR Image="*powershell.exe")
| stats count by ParentImage, Image
```

Finding nổi bật:

```text
notepad.exe → powershell.exe
```

Notepad bình thường không nên spawn PowerShell, nên cần drill down:

```spl
index="main" sourcetype="WinEventLog:Sysmon" EventCode=1 (Image="*cmd.exe" OR Image="*powershell.exe") ParentImage="C:\Windows\System32\notepad.exe"
```

Finding:

```text
PowerShell download file.exe từ http://10.0.0.229:8080
```

---

# 6. Điều tra IP 10.0.0.229

Tìm sourcetype chứa IP:

```spl
index="main" 10.0.0.229
| stats count by sourcetype
```

Xem Linux syslog:

```spl
index="main" 10.0.0.229 sourcetype="linux:syslog"
```

Finding:

```text
10.0.0.229 = waldo-virtual-machine trên interface ens160
```

Xem command liên quan IP này:

```spl
index="main" 10.0.0.229 sourcetype="WinEventLog:sysmon"
| stats count by CommandLine
```

Xem host nào chạy command:

```spl
index="main" 10.0.0.229 sourcetype="WinEventLog:sysmon"
| stats count by CommandLine, host
```

Kết luận ban đầu:

```text
Linux host có thể bị compromise hoặc đang bị dùng làm pivot/payload server.
```

---

# 7. Hunt DCSync

DCSync có thể để lại EventCode 4662 nếu auditing được bật.

```spl
index="main" EventCode=4662 Access_Mask=0x100 Account_Name!=*$
```

Ý nghĩa:

- `EventCode=4662`: AD object accessed.
- `Access_Mask=0x100`: Control Access.
- `Account_Name!=*$`: loại machine accounts.

GUID quan trọng trong Properties:

```text
1131f6ad-9c07-11d1-f79f-00c04fc2dcd2
= DS-Replication-Get-Changes-All
```

Ý nghĩa:

```text
Cho phép replication secret domain data.
```

Finding:

```text
User waldo thực hiện DCSync thành công trên domain UNIWALDO.
```

Impact:

```text
Domain compromise, cần xem xét rotate krbtgt để phòng Golden Ticket.
```

---

# 8. Hunt LSASS Dumping

Sysmon EventCode 10 ghi nhận process access.

Tìm process mở handle đến LSASS:

```spl
index="main" EventCode=10 lsass
| stats count by SourceImage
```

Drill down Notepad access LSASS:

```spl
index="main" EventCode=10 lsass SourceImage="C:\Windows\System32\notepad.exe"
```

Finding:

```text
SourceImage: C:\Windows\System32\notepad.exe
TargetImage: C:\Windows\System32\lsass.exe
GrantedAccess: 0x1FFFFF
SourceUser: DESKTOP-EGSS5IS\waldo
```

Đây là dấu hiệu rất đáng ngờ vì Notepad không nên access LSASS với quyền cao.

---

# 9. CallTrace UNKNOWN

Trong ProcessAccess event, `CallTrace` có thể chứa:

```text
UNKNOWN
```

Ý nghĩa:

- API call đến từ memory region không map với file trên disk.
- Có thể liên quan shellcode hoặc injected code.
- Có false positive như JIT/.NET/WOW64.

Kiểm tra EventCode chứa UNKNOWN:

```spl
index="main" CallTrace="*UNKNOWN*"
| stats count by EventCode
```

Finding:

```text
UNKNOWN chủ yếu nằm trong EventCode 10.
```

---

# 10. Xây dựng alert CallTrace UNKNOWN

## Bước 1: Group theo SourceImage

```spl
index="main" CallTrace="*UNKNOWN*"
| stats count by SourceImage
```

## Bước 2: Loại process tự access chính nó

```spl
index="main" CallTrace="*UNKNOWN*"
| where SourceImage!=TargetImage
| stats count by SourceImage
```

## Bước 3: Loại .NET/JIT noise

```spl
index="main" CallTrace="*UNKNOWN*" SourceImage!="*Microsoft.NET*" CallTrace!=*ni.dll* CallTrace!=*clr.dll*
| where SourceImage!=TargetImage
| stats count by SourceImage
```

## Bước 4: Loại WOW64 noise

```spl
index="main" CallTrace="*UNKNOWN*" SourceImage!="*Microsoft.NET*" CallTrace!=*ni.dll* CallTrace!=*clr.dll* CallTrace!=*wow64*
| where SourceImage!=TargetImage
| stats count by SourceImage
```

## Bước 5: Loại Explorer.exe

```spl
index="main" CallTrace="*UNKNOWN*" SourceImage!="*Microsoft.NET*" CallTrace!=*ni.dll* CallTrace!=*clr.dll* CallTrace!=*wow64* SourceImage!="C:\Windows\Explorer.EXE"
| where SourceImage!=TargetImage
| stats count by SourceImage
```

## Query alert có context

```spl
index="main" CallTrace="*UNKNOWN*" SourceImage!="*Microsoft.NET*" CallTrace!=*ni.dll* CallTrace!=*clr.dll* CallTrace!=*wow64* SourceImage!="C:\Windows\Explorer.EXE"
| where SourceImage!=TargetImage
| stats count by SourceImage, TargetImage, CallTrace
```

Alert này có thể phát hiện:

- Shellcode/in-memory execution.
- Process access đáng ngờ.
- LSASS dumping.
- Process injection.

---

# 11. Điểm yếu và bypass

Alert này vẫn có thể bị bypass.

Ví dụ attacker có thể:

- Load DLL có tên chứa `ni.dll`.
- Inject vào process đã bị exclude.
- Lạm dụng .NET/WOW64 để trộn vào noise.
- Lạm dụng Explorer.exe.
- Dùng technique không tạo CallTrace UNKNOWN.

Bài học:

```text
Alert phải được tune liên tục, không nên phụ thuộc một field/string duy nhất.
```

---

# 12. Investigation Summary

Attack path rút ra:

```text
notepad.exe spawn powershell.exe
→ PowerShell download executable từ 10.0.0.229:8080
→ 10.0.0.229 là Linux host waldo-virtual-machine
→ Nhiều Windows hosts tương tác với Linux pivot
→ DCSync activity xuất hiện
→ EventCode 4662 xác nhận DS-Replication-Get-Changes-All
→ User waldo thực hiện DCSync
→ Notepad access LSASS với GrantedAccess 0x1FFFFF
→ CallTrace UNKNOWN gợi ý shellcode/in-memory execution
→ Có khả năng domain đã bị compromise
```

---

# 13. Query checklist nhanh

```spl
index="main" earliest=0
index="main" | stats count by sourcetype
index="main" sourcetype="WinEventLog:Sysmon"
index="main" sourcetype="WinEventLog:Sysmon" | stats count by EventCode
index="main" sourcetype="WinEventLog:Sysmon" EventCode=1 | stats count by ParentImage, Image
index="main" sourcetype="WinEventLog:Sysmon" EventCode=1 (Image="*cmd.exe" OR Image="*powershell.exe") | stats count by ParentImage, Image
index="main" sourcetype="WinEventLog:Sysmon" EventCode=1 (Image="*cmd.exe" OR Image="*powershell.exe") ParentImage="C:\Windows\System32\notepad.exe"
index="main" 10.0.0.229 | stats count by sourcetype
index="main" 10.0.0.229 sourcetype="linux:syslog"
index="main" 10.0.0.229 sourcetype="WinEventLog:sysmon" | stats count by CommandLine, host
index="main" EventCode=4662 Access_Mask=0x100 Account_Name!=*$
index="main" EventCode=10 lsass | stats count by SourceImage
index="main" EventCode=10 lsass SourceImage="C:\Windows\System32\notepad.exe"
index="main" CallTrace="*UNKNOWN*" | stats count by EventCode
index="main" CallTrace="*UNKNOWN*" | where SourceImage!=TargetImage | stats count by SourceImage
index="main" CallTrace="*UNKNOWN*" SourceImage!="*Microsoft.NET*" CallTrace!=*ni.dll* CallTrace!=*clr.dll* CallTrace!=*wow64* SourceImage!="C:\Windows\Explorer.EXE" | where SourceImage!=TargetImage | stats count by SourceImage, TargetImage, CallTrace
```

---

# 14. Bảng ôn nhanh

| Thuật ngữ | Nghĩa |
|---|---|
| SPL | Search Processing Language |
| Sourcetype | Kiểu nguồn dữ liệu |
| Sysmon EventCode 1 | Process Creation |
| Sysmon EventCode 10 | ProcessAccess |
| DCSync | Replicate credential data từ AD |
| EventCode 4662 | AD object access |
| Access_Mask 0x100 | Control Access |
| LSASS Dumping | Trích xuất credential từ LSASS |
| CallTrace UNKNOWN | API call từ memory không map file |
| False Positive | Cảnh báo sai |
| Alert Fidelity | Độ tin cậy của alert |

---

# Key Takeaway

Bài này cho thấy quy trình hunt thực tế trong Splunk:

```text
Dữ liệu rộng
→ xác định sourcetype
→ tập trung Sysmon EventCodes
→ hunt parent-child process
→ pivot sang IP/host
→ validate DCSync
→ validate LSASS dumping
→ tạo alert CallTrace UNKNOWN
```

Điểm quan trọng nhất là viết query hiệu quả, giảm noise, validate finding và hiểu rằng alert cần được tune liên tục để tránh false positive cũng như tránh bị bypass.

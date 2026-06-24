---
layout: post
title: "Detecting Attacker Behavior With Splunk Based On TTPs"
date: 2026-06-24 14:48:28 +0700
categories: [SOC, Splunk]
tags: [soc, splunk, spl, ttp, sysmon, psexec, powershell, githubusercontent, anomaly-detection, detection-engineering, blue-team]
description: "Tóm tắt cách phát hiện attacker behavior bằng Splunk dựa trên TTPs: native Windows binaries reconnaissance, githubusercontent payload hosting, PsExec, archive files, PowerShell/MS Edge downloads, suspicious locations, typo binaries và non-standard ports."
toc: true
---

# Detecting Attacker Behavior With Splunk Based On TTPs

Bài này tóm tắt cách viết **SPL searches** trong Splunk để phát hiện attacker behavior dựa trên **Tactics, Techniques and Procedures (TTPs)**.

Có hai hướng chính:

```text
1. Known TTP-based detection
2. Statistical / anomaly-based detection
```

Trong phần này, trọng tâm là cách tư duy để xây dựng search, chưa phải detection engineering hoàn chỉnh. Các query cần được tune theo môi trường thật để giảm false positive.

---

## Timeline cập nhật

```text
Created at: 2026-06-24 14:48:28 +0700
Timezone: Asia/Ho_Chi_Minh
Module: Understanding Log Sources & Investigating with Splunk
Section: Detecting Attacker Behavior With Splunk Based On TTPs
```

---

# 1. Hai hướng xây dựng SPL detection searches

## 1.1. Known TTP-based detection

Hướng thứ nhất dựa trên hiểu biết về attacker TTPs.

```text
Nếu attacker thường làm hành vi X,
ta tìm dấu hiệu X trong log.
```

Ví dụ:

- Attacker reconnaissance bằng `whoami.exe`, `net.exe`, `ipconfig.exe`.
- Attacker download payload bằng PowerShell.
- Attacker dùng PsExec để lateral movement.
- Attacker tạo archive `.zip`, `.rar`, `.7z` để exfiltrate data.
- Attacker chạy tool từ Downloads folder.
- Attacker dùng non-standard port để truyền file.

Ưu điểm:

- Dễ hiểu.
- Dễ map với MITRE ATT&CK.
- Dễ chuyển thành alert.
- Hiệu quả với hành vi đã biết.

Nhược điểm:

- Có thể bị bypass nếu attacker đổi TTP.
- Có thể tạo false positive nếu môi trường có admin activity tương tự.
- Cần tune theo baseline.

---

## 1.2. Statistical / anomaly-based detection

Hướng thứ hai dựa trên thống kê và anomaly detection.

```text
Không cần biết chính xác attacker dùng gì,
ta tìm hành vi lệch khỏi baseline bình thường.
```

Ví dụ:

- Rare process.
- Rare parent-child relationship.
- Rare destination port.
- Rare DNS query.
- Rare user logon pattern.
- Rare host-to-host connection.
- Unusual command line.

Ưu điểm:

- Có thể phát hiện unknown threat.
- Không chỉ phụ thuộc vào IOC.
- Hữu ích khi attacker dùng TTP mới.

Nhược điểm:

- Cần hiểu baseline.
- Dễ false positive nếu chưa tune.
- Cần review và cải thiện liên tục.

---

# 2. Detection 1: Reconnaissance bằng native Windows binaries

Attacker thường dùng Windows binaries có sẵn để reconnaissance.

Các binary thường gặp:

```text
ipconfig.exe
net.exe
whoami.exe
netstat.exe
nbtstat.exe
hostname.exe
tasklist.exe
```

## SPL

```spl
index="main" sourcetype="WinEventLog:Sysmon" EventCode=1 Image=*\\ipconfig.exe OR Image=*\\net.exe OR Image=*\\whoami.exe OR Image=*\\netstat.exe OR Image=*\\nbtstat.exe OR Image=*\\hostname.exe OR Image=*\\tasklist.exe
| stats count by Image,CommandLine
| sort - count
```

## Ý nghĩa

Query dùng Sysmon EventCode 1 để tìm process creation của các tool reconnaissance.

Cần chú ý:

- User nào chạy.
- Host nào chạy.
- CommandLine cụ thể.
- Có chạy liên tiếp nhiều tool không.
- Parent process là gì.
- Có xảy ra sau initial access không.

---

# 3. Detection 2: Payload hosting trên githubusercontent.com

Attacker có thể host payload/tool trên domain uy tín như:

```text
githubusercontent.com
```

Lý do:

- Dễ bị whitelist.
- Proxy/firewall thường cho phép.
- Trông giống developer traffic hợp lệ.

## SPL

```spl
index="main" sourcetype="WinEventLog:Sysmon" EventCode=22 QueryName="*github*"
| stats count by Image, QueryName
```

## Ý nghĩa

Query dùng Sysmon EventCode 22 để tìm DNS queries liên quan GitHub.

Cần kiểm tra:

- Process nào query domain.
- Có phải browser/dev tool hợp lệ không.
- Có phải PowerShell/cmd/script query không.
- Domain cụ thể là gì.
- Có connection/download sau DNS query không.

---

# 4. Detection 3: PsExec usage

**PsExec** là Sysinternals tool hợp pháp nhưng thường bị attacker lạm dụng để lateral movement và remote execution.

MITRE ATT&CK liên quan:

| Technique | ID |
|---|---|
| System Services: Service Execution | T1569.002 |
| Remote Services: SMB/Windows Admin Shares | T1021.002 |
| Lateral Tool Transfer | T1570 |

PsExec thường:

```text
Copy service executable qua ADMIN$ share
→ Tạo service trên remote host
→ Start service qua Service Control Manager
→ Giao tiếp qua named pipe
```

---

## 4.1. PsExec Case 1: Sysmon EventCode 13 - Service ImagePath registry modification

```spl
index="main" sourcetype="WinEventLog:Sysmon" EventCode=13 Image="C:\\Windows\\system32\\services.exe" TargetObject="HKLM\\System\\CurrentControlSet\\Services\\*\\ImagePath"
| rex field=Details "(?<reg_file_name>[^\\]+)$"
| eval reg_file_name=lower(reg_file_name), file_name=if(isnull(file_name),reg_file_name,lower(file_name))
| stats values(Image) AS Image, values(Details) AS RegistryDetails, values(_time) AS EventTimes, count by file_name, ComputerName
```

## Ý nghĩa

Query tìm `services.exe` sửa `ImagePath` của service. Điều này có thể liên quan đến service creation, PsExec-like execution hoặc malware persistence bằng service.

---

## 4.2. PsExec Case 2: Sysmon EventCode 11 - FileCreate bởi System

```spl
index="main" sourcetype="WinEventLog:Sysmon" EventCode=11 Image=System
| stats count by TargetFilename
```

Ý nghĩa: PsExec có thể tạo service executable trên remote host. EventCode 11 giúp thấy file được tạo.

---

## 4.3. PsExec Case 3: Sysmon EventCode 18 - Named pipe connected

```spl
index="main" sourcetype="WinEventLog:Sysmon" EventCode=18 Image=System
| stats count by PipeName
```

Ý nghĩa: PsExec dùng named pipes để giao tiếp. Pipe activity bất thường có thể gợi ý SMB lateral movement hoặc remote command execution.

---

# 5. Detection 4: Archive files dùng để transfer tools hoặc exfiltration

Attacker có thể tạo archive để nén dữ liệu trước khi exfiltrate hoặc chuyển tool sang host khác.

```spl
index="main" EventCode=11 (TargetFilename="*.zip" OR TargetFilename="*.rar" OR TargetFilename="*.7z")
| stats count by ComputerName, User, TargetFilename
| sort - count
```

Cần kiểm tra:

- User nào tạo.
- Host nào tạo.
- Archive nằm ở Downloads/Temp/Desktop không.
- Có upload/network connection sau đó không.

---

# 6. Detection 5: PowerShell download payload/tools

```spl
index="main" sourcetype="WinEventLog:Sysmon" EventCode=11 Image="*powershell.exe*"
| stats count by Image, TargetFilename
| sort + count
```

Query tìm file được tạo bởi PowerShell. Cần chú ý file `.exe`, `.dll`, `.ps1`, `.bat` trong user profile, Temp hoặc Downloads.

---

# 7. Detection 6: MS Edge download từ Internet qua Zone.Identifier

Windows dùng Alternate Data Stream `Zone.Identifier` để đánh dấu file tải từ Internet.

```spl
index="main" sourcetype="WinEventLog:Sysmon" EventCode=11 Image="*msedge.exe" TargetFilename=*"Zone.Identifier"
| stats count by TargetFilename
| sort + count
```

`Zone.Identifier` thường chỉ ra:

```text
File đến từ Internet hoặc untrusted zone.
```

---

# 8. Detection 7: Execution from Downloads folder

Attacker thường chạy tool từ thư mục user writable như:

```text
C:\Users\<user>\Downloads
```

```spl
index="main" EventCode=1
| regex Image="C:\\\\Users\\\\.*\\\\Downloads\\\\.*"
| stats count by Image
```

Dấu hiệu đáng chú ý:

- `PsExec64.exe`.
- `SharpHound.exe`.
- Random filename.
- Unsigned binary.
- Tool chạy từ user profile.

---

# 9. Detection 8: EXE/DLL created outside Windows directory

```spl
index="main" EventCode=11 (TargetFilename="*.exe" OR TargetFilename="*.dll") TargetFilename!="*\\windows\\*"
| stats count by User, TargetFilename
| sort + count
```

Query tìm EXE/DLL được tạo ngoài Windows directory. Cần chú ý các file ở Downloads, Temp, AppData hoặc Desktop.

---

# 10. Detection 9: Misspelling legitimate binaries

Attacker có thể cố tình đặt tên gần giống binary hợp lệ để masquerading.

```spl
index="main" sourcetype="WinEventLog:Sysmon" EventCode=1 (CommandLine="*psexe*.exe" NOT (CommandLine="*PSEXESVC.exe" OR CommandLine="*PsExec64.exe")) OR (ParentCommandLine="*psexe*.exe" NOT (ParentCommandLine="*PSEXESVC.exe" OR ParentCommandLine="*PsExec64.exe")) OR (ParentImage="*psexe*.exe" NOT (ParentImage="*PSEXESVC.exe" OR ParentImage="*PsExec64.exe")) OR (Image="*psexe*.exe" NOT (Image="*PSEXESVC.exe" OR Image="*PsExec64.exe"))
| table Image, CommandLine, ParentImage, ParentCommandLine
```

Query tìm binary giả mạo hoặc typo-squatting quanh PsExec.

---

# 11. Detection 10: Non-standard ports

Attacker có thể dùng non-standard port để truyền tool, C2 hoặc exfiltration.

```spl
index="main" EventCode=3 NOT (DestinationPort=80 OR DestinationPort=443 OR DestinationPort=22 OR DestinationPort=21)
| stats count by SourceIp, DestinationIp, DestinationPort
| sort - count
```

Cần kiểm tra:

- Destination IP là internal hay external.
- Port có hợp lệ với app không.
- Process nào tạo connection.
- Có liên quan payload server không.
- Có cùng thời điểm với process execution không.

---

# 12. Query checklist nhanh

```spl
index="main" sourcetype="WinEventLog:Sysmon" EventCode=1 Image=*\\ipconfig.exe OR Image=*\\net.exe OR Image=*\\whoami.exe OR Image=*\\netstat.exe OR Image=*\\nbtstat.exe OR Image=*\\hostname.exe OR Image=*\\tasklist.exe | stats count by Image,CommandLine | sort - count
index="main" sourcetype="WinEventLog:Sysmon" EventCode=22 QueryName="*github*" | stats count by Image, QueryName
index="main" sourcetype="WinEventLog:Sysmon" EventCode=13 Image="C:\\Windows\\system32\\services.exe" TargetObject="HKLM\\System\\CurrentControlSet\\Services\\*\\ImagePath" | rex field=Details "(?<reg_file_name>[^\\]+)$" | eval reg_file_name=lower(reg_file_name), file_name=if(isnull(file_name),reg_file_name,lower(file_name)) | stats values(Image) AS Image, values(Details) AS RegistryDetails, values(_time) AS EventTimes, count by file_name, ComputerName
index="main" sourcetype="WinEventLog:Sysmon" EventCode=11 Image=System | stats count by TargetFilename
index="main" sourcetype="WinEventLog:Sysmon" EventCode=18 Image=System | stats count by PipeName
index="main" EventCode=11 (TargetFilename="*.zip" OR TargetFilename="*.rar" OR TargetFilename="*.7z") | stats count by ComputerName, User, TargetFilename | sort - count
index="main" sourcetype="WinEventLog:Sysmon" EventCode=11 Image="*powershell.exe*" | stats count by Image, TargetFilename | sort + count
index="main" sourcetype="WinEventLog:Sysmon" EventCode=11 Image="*msedge.exe" TargetFilename=*"Zone.Identifier" | stats count by TargetFilename | sort + count
index="main" EventCode=1 | regex Image="C:\\\\Users\\\\.*\\\\Downloads\\\\.*" | stats count by Image
index="main" EventCode=11 (TargetFilename="*.exe" OR TargetFilename="*.dll") TargetFilename!="*\\windows\\*" | stats count by User, TargetFilename | sort + count
index="main" sourcetype="WinEventLog:Sysmon" EventCode=1 (CommandLine="*psexe*.exe" NOT (CommandLine="*PSEXESVC.exe" OR CommandLine="*PsExec64.exe")) OR (ParentCommandLine="*psexe*.exe" NOT (ParentCommandLine="*PSEXESVC.exe" OR ParentCommandLine="*PsExec64.exe")) OR (ParentImage="*psexe*.exe" NOT (ParentImage="*PSEXESVC.exe" OR ParentImage="*PsExec64.exe")) OR (Image="*psexe*.exe" NOT (Image="*PSEXESVC.exe" OR Image="*PsExec64.exe")) | table Image, CommandLine, ParentImage, ParentCommandLine
index="main" EventCode=3 NOT (DestinationPort=80 OR DestinationPort=443 OR DestinationPort=22 OR DestinationPort=21) | stats count by SourceIp, DestinationIp, DestinationPort | sort - count
```

---

# 13. Mapping nhanh theo hành vi attacker

| Hành vi | Query idea | EventCode |
|---|---|---|
| Reconnaissance | Native Windows binaries | 1 |
| Payload hosting | GitHub/GitHubusercontent DNS | 22 |
| PsExec service creation | Services ImagePath registry set | 13 |
| PsExec file creation | FileCreate by System | 11 |
| PsExec named pipe | Pipe connected | 18 |
| Archive staging/exfil | zip/rar/7z created | 11 |
| PowerShell download | FileCreate by powershell.exe | 11 |
| Browser download | Zone.Identifier by msedge.exe | 11 |
| Suspicious execution | Run from Downloads | 1 |
| Malware drop | EXE/DLL outside Windows | 11 |
| Masquerading | Misspelled PsExec-like binary | 1 |
| Non-standard C2/transfer | DestinationPort not 80/443/22/21 | 3 |

---

# Key Takeaway

Phát hiện attacker behavior bằng Splunk dựa trên TTPs yêu cầu hiểu cả attacker lẫn môi trường nội bộ.

Một query tốt không chỉ là search IOC, mà cần trả lời:

```text
Attacker đang cố làm gì?
Log source nào ghi nhận hành vi đó?
EventCode/field nào có giá trị?
False positive thường đến từ đâu?
Query này có thể chuyển thành alert không?
```

Known TTP-based detection giúp bắt các hành vi đã biết. Statistical/anomaly-based detection giúp tìm hành vi lạ chưa có IOC rõ ràng. Kết hợp cả hai hướng sẽ tạo detection coverage tốt hơn.

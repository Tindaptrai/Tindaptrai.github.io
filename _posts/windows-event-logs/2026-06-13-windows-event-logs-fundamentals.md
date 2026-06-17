---
layout: post
title: "Windows Event Logs Fundamentals"
date: 2026-06-13 00:05:00 +0700
categories: [SOC, Windows Security]
tags: [soc, windows-event-logs, event-viewer, evtx, security-logs, event-id, xml-query, blue-team, dfir]
description: "Ghi chú tổng hợp về Windows Event Logs: khái niệm, Event Viewer, cấu trúc event, XML query, logon events và các Windows Event ID hữu ích cho SOC/DFIR."
toc: true
---

# Windows Event Logs Fundamentals

**Windows Event Logs** là nguồn dữ liệu cực kỳ quan trọng trong SOC, DFIR và threat hunting trên môi trường Windows.

Log Windows giúp analyst hiểu điều gì đã xảy ra trên hệ thống, user nào liên quan, event xảy ra lúc nào, process nào tạo ra event và có dấu hiệu bất thường nào không.

Nói ngắn gọn:

```text
Windows Event Logs = nhật ký hoạt động của hệ điều hành, ứng dụng và bảo mật trên Windows.
```

---

## 1. Windows Event Logs là gì?

**Windows Event Logs** là cơ chế ghi log tích hợp sẵn trong hệ điều hành Windows.

Nó lưu trữ sự kiện từ nhiều thành phần khác nhau như:

- Operating System.
- Applications.
- Services.
- Security auditing.
- ETW providers.
- Drivers.
- System components.
- Authentication events.
- Process/service activity.

Trong an ninh mạng, Windows Event Logs được dùng để:

- Điều tra incident.
- Phát hiện xâm nhập.
- Theo dõi đăng nhập.
- Kiểm tra thay đổi hệ thống.
- Phát hiện malware.
- Phát hiện persistence.
- Phân tích lateral movement.
- Xây dựng detection rule trong SIEM.

---

## 2. Vì sao Windows Event Logs quan trọng trong SOC?

Trong môi trường Windows enterprise, rất nhiều hành vi quan trọng được ghi lại qua event logs.

Ví dụ:

- User đăng nhập thành công.
- User đăng nhập thất bại.
- Account được thay đổi.
- Tài khoản được thêm vào group.
- Scheduled task được tạo.
- Service mới được cài.
- Audit log bị xóa.
- Defender phát hiện malware.
- Network share được truy cập.
- Windows Filtering Platform block connection.

Nếu logs được thu thập vào SIEM, SOC analyst có thể tìm kiếm và correlation để phát hiện attack chain.

Ví dụ:

```text
Failed logon nhiều lần
      ↓
Successful logon
      ↓
Special privileges assigned
      ↓
Scheduled task created
      ↓
New service installed
```

Chuỗi này có thể gợi ý compromise hoặc persistence.

---

# 3. Windows Event Viewer

**Event Viewer** là công cụ giao diện đồ họa của Windows dùng để xem event logs.

Có thể mở bằng:

```text
Start Menu → Search → Event Viewer
```

Hoặc chạy:

```powershell
eventvwr.msc
```

Nếu mở với quyền administrator, ta có thể xem nhiều log hơn và truy cập các event quan trọng hơn.

---

## 3.1. Các Windows logs mặc định

Các log mặc định thường gặp:

| Log | Ý nghĩa |
|---|---|
| Application | Log của ứng dụng |
| Security | Log bảo mật/audit |
| Setup | Log cài đặt/cấu hình |
| System | Log hệ thống |
| Forwarded Events | Log được forward từ máy khác |

Trong SOC, log quan trọng nhất thường là:

```text
Security
System
Application
```

Ngoài ra, nếu có Windows Event Forwarding, log từ nhiều máy có thể tập trung trong:

```text
Forwarded Events
```

---

## 3.2. Saved Logs và file EVTX

Windows Event Viewer cũng có thể mở các file log đã lưu trước đó.

File Windows Event Log thường có định dạng:

```text
.evtx
```

Trong DFIR, analyst thường thu thập file `.evtx` từ endpoint để phân tích offline.

Ví dụ:

```text
Security.evtx
System.evtx
Application.evtx
Sysmon.evtx
```

---

# 4. Anatomy of a Windows Event Log

Mỗi event trong Windows Event Logs có nhiều thành phần.

Các field quan trọng:

| Field | Ý nghĩa |
|---|---|
| Log Name | Tên log như Security, System, Application |
| Source | Nguồn tạo event |
| Event ID | Mã định danh event |
| Task Category | Nhóm nhiệm vụ của event |
| Level | Mức độ nghiêm trọng |
| Keywords | Cờ phân loại event |
| User | User liên quan |
| OpCode | Hoạt động cụ thể |
| Logged | Thời điểm event được ghi |
| Computer | Máy xảy ra event |
| XML Data | Dữ liệu chi tiết dạng XML |

---

## 4.1. Event ID

**Event ID** là mã định danh sự kiện.

Ví dụ:

| Event ID | Ý nghĩa |
|---|---|
| 4624 | Successful logon |
| 4625 | Failed logon |
| 4672 | Special privileges assigned |
| 4698 | Scheduled task created |
| 7045 | Service installed |
| 1102 | Audit log cleared |

Event ID giúp analyst lọc nhanh loại hành vi cần điều tra.

Ví dụ trong SIEM:

```text
event.code:4625
```

hoặc trong Windows Event Viewer:

```text
Event ID: 4625
```

---

## 4.2. Level

**Level** thể hiện mức độ của event.

Các level thường gặp:

| Level | Ý nghĩa |
|---|---|
| Information | Thông tin bình thường |
| Warning | Cảnh báo |
| Error | Lỗi |
| Critical | Nghiêm trọng |
| Verbose | Chi tiết |

Ví dụ:

- Application start/stop thường là Information.
- Ứng dụng crash hoặc lỗi cấu hình có thể là Error.
- Sự cố nghiêm trọng hệ thống có thể là Critical.

---

## 4.3. Keywords

**Keywords** là cờ phân loại event.

Trong Security log, keywords thường gặp:

```text
Audit Success
Audit Failure
```

Keywords rất hữu ích khi lọc event theo loại audit.

Ví dụ:

- Failed logon thường là Audit Failure.
- Successful logon thường là Audit Success.

---

## 4.4. XML Data

Mỗi event có thể xem ở dạng XML.

XML giúp analyst thấy rõ các field chi tiết hơn, ví dụ:

- Provider.
- EventID.
- TimeCreated.
- EventRecordID.
- ProcessID.
- ThreadID.
- Computer.
- Security UserID.
- EventData.

Dạng XML rất hữu ích khi:

- Viết query XPath/XML.
- Tìm field chính xác.
- Correlate theo Logon ID.
- Phân tích event offline.
- Xây dựng SIEM parser.

---

# 5. Ví dụ Event ID 4624 - Successful Logon

**Event ID 4624** biểu thị một phiên đăng nhập thành công được tạo trên máy đích.

Event này rất quan trọng trong SOC vì nó giúp xác định:

- Ai đã đăng nhập?
- Đăng nhập vào máy nào?
- Đăng nhập lúc nào?
- Kiểu đăng nhập là gì?
- Đăng nhập từ đâu?
- Logon ID là gì?

Một số field quan trọng:

```text
TargetUserName
TargetDomainName
LogonType
LogonProcessName
AuthenticationPackageName
WorkstationName
IpAddress
LogonId
```

---

## 5.1. Logon ID

**Logon ID** là giá trị dùng để correlation các event liên quan đến cùng một session đăng nhập.

Ví dụ:

```text
Logon ID: 0x3E7
```

Analyst có thể dùng Logon ID để tìm các event khác cùng session.

Điều này hữu ích khi muốn biết:

- Sau khi login, user đã làm gì?
- Có special privileges không?
- Có tạo process/service/task không?
- Có truy cập object nào không?

---

## 5.2. Logon Type

**Logon Type** cho biết kiểu đăng nhập.

Một số Logon Type quan trọng:

| Logon Type | Ý nghĩa |
|---|---|
| 2 | Interactive - đăng nhập trực tiếp |
| 3 | Network - truy cập qua mạng |
| 4 | Batch - batch job |
| 5 | Service - service logon |
| 7 | Unlock |
| 10 | RemoteInteractive - RDP |
| 11 | CachedInteractive |

Ví dụ:

```text
Logon Type 5 = Service
```

Nếu `SYSTEM` login với Logon Type 5, có thể liên quan đến service được khởi chạy.

---

# 6. Custom XML Queries trong Event Viewer

Event Viewer hỗ trợ lọc event bằng XML query.

Đường dẫn:

```text
Filter Current Log → XML → Edit query manually
```

XML query giúp lọc chính xác hơn so với filter cơ bản.

Ví dụ muốn tìm event có:

```text
SubjectLogonId = 0x3E7
```

Có thể tạo XML query để thu hẹp các event liên quan đến Logon ID đó.

---

## 6.1. Vì sao dùng XML query?

XML query hữu ích khi cần:

- Tìm event theo field cụ thể.
- Correlate theo Logon ID.
- Lọc nhiều điều kiện.
- Phân tích event trong Security log.
- Điều tra chi tiết một session.
- Giảm nhiễu khi log quá nhiều.

Ví dụ logic:

```text
Tìm mọi event có SubjectLogonId = 0x3E7
```

Mục tiêu:

```text
Xác định chuỗi hành động liên quan đến cùng một session đăng nhập.
```

---

# 7. Ví dụ Event ID 4907 - Audit Policy / SACL Change

**Event ID 4907** thường liên quan đến thay đổi audit policy hoặc thay đổi SACL của object.

SACL là:

```text
System Access Control List
```

SACL quy định loại hành động nào trên object sẽ được audit và ghi log.

Nếu SACL của file/registry key bị thay đổi, điều này có thể ảnh hưởng đến khả năng giám sát truy cập object đó.

---

## 7.1. Vì sao Event ID 4907 đáng chú ý?

Thay đổi audit policy hoặc SACL có thể là hành vi hợp lệ.

Nhưng nó cũng có thể là dấu hiệu:

- Attacker muốn giảm khả năng bị phát hiện.
- Có thay đổi quyền audit trên file/registry.
- Có thao tác setup/cài đặt hệ thống.
- Có tiến trình cố thay đổi cơ chế logging.
- Có hành vi anti-forensics.

Cần kiểm tra:

- SubjectUserName.
- ObjectName.
- ObjectType.
- ProcessName.
- OldSd.
- NewSd.
- Logon ID.
- Thời điểm thay đổi.

---

## 7.2. ProcessName trong event

Trong ví dụ, process liên quan là:

```text
SetupHost.exe
```

Điều này có thể gợi ý một quá trình setup hợp lệ.

Tuy nhiên, cần nhớ rằng malware có thể giả mạo tên process hợp lệ.

Vì vậy, cần kiểm tra thêm:

- Đường dẫn process.
- Digital signature.
- Parent process.
- Command line.
- Hash.
- Timeline trước/sau event.
- Có event cài đặt khác đi kèm không.

---

# 8. Event ID 4672 - Special Privileges Assigned

**Event ID 4672** xuất hiện khi một tài khoản đăng nhập và được cấp quyền đặc biệt.

Event này thường xuất hiện với các tài khoản có quyền cao.

Các privilege có thể gồm:

```text
SeDebugPrivilege
SeBackupPrivilege
SeRestorePrivilege
SeTakeOwnershipPrivilege
SeAssignPrimaryTokenPrivilege
```

---

## 8.1. Vì sao SeDebugPrivilege quan trọng?

**SeDebugPrivilege** cho phép user debug hoặc thao tác với process khác.

Trong bối cảnh bảo mật, privilege này nguy hiểm vì có thể bị lạm dụng để:

- Truy cập memory của process khác.
- Dump credential.
- Tương tác với LSASS.
- Bypass một số hạn chế.
- Hỗ trợ privilege abuse.

Nếu Event ID 4672 xuất hiện bất thường, đặc biệt với user không mong đợi, cần điều tra.

---

# 9. Windows System Event Logs hữu ích

| Event ID | Ý nghĩa | Dùng để phát hiện |
|---|---|---|
| 1074 | System Shutdown/Restart | Restart/shutdown bất thường |
| 6005 | Event Log Service Started | Hệ thống/service bắt đầu ghi log |
| 6006 | Event Log Service Stopped | Shutdown hoặc anti-forensics |
| 6013 | Windows uptime | Kiểm tra reboot |
| 7040 | Service status change | Service tampering |

---

# 10. Windows Security Event Logs hữu ích

| Event ID | Ý nghĩa | Vì sao quan trọng |
|---|---|---|
| 1102 | Audit log cleared | Dấu hiệu xóa dấu vết |
| 1116 | Defender malware detection | Malware được phát hiện |
| 1118 | Defender remediation started | Bắt đầu xử lý malware |
| 1119 | Defender remediation succeeded | Xử lý thành công |
| 1120 | Defender remediation failed | Malware có thể còn tồn tại |
| 4624 | Successful logon | Theo dõi đăng nhập thành công |
| 4625 | Failed logon | Brute force/password spraying |
| 4648 | Explicit credentials | Lateral movement/credential misuse |
| 4656 | Object handle requested | Truy cập object nhạy cảm |
| 4672 | Special privileges assigned | Quyền cao được cấp |
| 4698 | Scheduled task created | Persistence |
| 4700/4701 | Scheduled task enabled/disabled | Thay đổi scheduled task |
| 4702 | Scheduled task updated | Persistence/tampering |
| 4719 | System audit policy changed | Có thể giảm logging |
| 4738 | User account changed | Account tampering |
| 4771 | Kerberos pre-auth failed | Brute force/password spraying |
| 4776 | DC credential validation | Theo dõi xác thực NTLM |
| 5001 | Defender real-time protection changed | Có thể vô hiệu hóa AV |
| 5140 | Network share accessed | Truy cập share |
| 5142 | Network share added | Share trái phép |
| 5145 | Network share checked | Share enumeration |
| 5157 | WFP blocked connection | Connection bị block |
| 7045 | Service installed | Persistence bằng service |

---

# 11. Bảng Event ID nhanh cho SOC

| Event ID | Log | Ý nghĩa | Mức chú ý |
|---|---|---|---|
| 1074 | System | Shutdown/Restart | Medium |
| 6005 | System | Event log service started | Low/Medium |
| 6006 | System | Event log service stopped | Medium |
| 6013 | System | Windows uptime | Low |
| 7040 | System | Service startup type changed | Medium |
| 1102 | Security | Audit log cleared | High |
| 1116 | Defender | Malware detected | High |
| 1120 | Defender | Remediation failed | High |
| 4624 | Security | Successful logon | Context-dependent |
| 4625 | Security | Failed logon | Medium/High |
| 4648 | Security | Explicit credentials | Medium/High |
| 4672 | Security | Special privileges assigned | Medium/High |
| 4698 | Security | Scheduled task created | Medium/High |
| 4719 | Security | Audit policy changed | High |
| 4738 | Security | User account changed | Medium |
| 4771 | Security | Kerberos pre-auth failed | Medium |
| 4776 | Security | DC credential validation | Medium |
| 5140 | Security | Network share accessed | Context-dependent |
| 5142 | Security | Network share added | Medium/High |
| 5145 | Security | Network share checked | Context-dependent |
| 5157 | Security | WFP blocked connection | Context-dependent |
| 7045 | Security/System | Service installed | High |

---

# 12. Tư duy phân tích Windows Event Logs

Khi phân tích Windows Event Logs, không nên chỉ nhìn một event đơn lẻ.

Cần correlation theo:

- User.
- Host.
- Logon ID.
- Process ID.
- Timestamp.
- Source IP.
- Destination host.
- Event sequence.
- Parent/child process.
- MITRE ATT&CK technique.

Ví dụ timeline:

```text
4625 - Failed logon
4624 - Successful logon
4672 - Special privileges assigned
4698 - Scheduled task created
7045 - Service installed
1102 - Audit log cleared
```

Timeline này đáng ngờ hơn nhiều so với từng event riêng lẻ.

---

# 13. Baseline và tuning

Một điểm quan trọng trong detection là phải hiểu điều gì là bình thường trong môi trường.

Một event có thể đáng ngờ trong môi trường này nhưng bình thường trong môi trường khác.

Ví dụ:

| Event | Có thể bình thường nếu | Có thể đáng ngờ nếu |
|---|---|---|
| 4625 | User nhập sai mật khẩu | Nhiều user bị thử từ một IP |
| 4672 | Admin login hợp lệ | User thường được cấp special privilege |
| 4698 | Admin tạo scheduled task | Task lạ chạy PowerShell |
| 7045 | Cài agent hợp lệ | Service lạ từ Temp/AppData |
| 1102 | Maintenance có kế hoạch | Xóa audit log sau compromise |

Do đó, SOC cần:

- Baseline hoạt động bình thường.
- Tuning alert.
- Whitelist hợp lý.
- Correlation nhiều nguồn log.
- Review định kỳ.
- Điều chỉnh detection theo môi trường.

---

# 14. KQL mẫu cho SIEM

## Successful logon

```text
event.code:4624
```

## Failed logon

```text
event.code:4625
```

## Special privileges

```text
event.code:4672
```

## Scheduled task created

```text
event.code:4698
```

## Service installed

```text
event.code:7045
```

## Audit log cleared

```text
event.code:1102
```

## Defender malware detection

```text
event.code:1116
```

## Explicit credentials

```text
event.code:4648
```

## Kerberos pre-auth failed

```text
event.code:4771
```

## Network share accessed

```text
event.code:5140
```

---

# 15. Bảng ôn nhanh thuật ngữ

| Thuật ngữ | Nghĩa ngắn gọn |
|---|---|
| Windows Event Logs | Nhật ký sự kiện Windows |
| Event Viewer | Công cụ xem log Windows |
| EVTX | Định dạng file Windows Event Log |
| Event ID | Mã định danh sự kiện |
| Security Log | Log bảo mật/audit |
| System Log | Log hệ thống |
| Application Log | Log ứng dụng |
| Forwarded Events | Log được forward từ máy khác |
| XML Data | Dữ liệu event dạng XML |
| Logon ID | ID dùng để correlation session đăng nhập |
| Logon Type | Kiểu đăng nhập |
| SACL | System Access Control List |
| Audit Policy | Chính sách ghi audit |
| Special Privileges | Quyền đặc biệt được cấp cho logon |
| Baseline | Hành vi bình thường của môi trường |
| Correlation | Tương quan nhiều event |

---

# 16. Câu hỏi ôn tập

## 1. Windows Event Logs là gì?

Windows Event Logs là hệ thống ghi log tích hợp trong Windows, lưu sự kiện từ hệ điều hành, ứng dụng, dịch vụ và security auditing.

## 2. Công cụ GUI nào dùng để xem Windows Event Logs?

Công cụ GUI là:

```text
Event Viewer
```

hoặc chạy:

```powershell
eventvwr.msc
```

## 3. File event log Windows thường có định dạng gì?

File event log thường có định dạng:

```text
.evtx
```

## 4. Event ID 4624 có ý nghĩa gì?

Event ID 4624 biểu thị đăng nhập thành công.

## 5. Event ID 4625 có ý nghĩa gì?

Event ID 4625 biểu thị đăng nhập thất bại.

## 6. Event ID 4672 có ý nghĩa gì?

Event ID 4672 cho biết special privileges được gán cho một logon mới.

## 7. Event ID 1102 có nguy hiểm không?

Có. Event ID 1102 cho biết audit log bị xóa, thường là dấu hiệu anti-forensics hoặc cố che dấu hoạt động.

## 8. Event ID 7045 dùng để phát hiện gì?

Event ID 7045 cho biết service mới được cài đặt, có thể liên quan đến persistence.

## 9. Vì sao cần correlation Windows Event Logs?

Vì một event đơn lẻ có thể chưa đủ kết luận, nhưng nhiều event liên quan có thể tạo thành attack chain.

---

# Key Takeaway

Windows Event Logs là nền tảng quan trọng cho SOC, DFIR và threat hunting trên Windows.

Một analyst cần hiểu Event Viewer, cấu trúc event, Event ID, Logon Type, XML Data, Logon ID và các event quan trọng như 4624, 4625, 4672, 4698, 7045, 1102.

Quan trọng nhất là không phân tích event riêng lẻ. Hãy correlation nhiều event theo user, host, timestamp, Logon ID và process để xây dựng timeline và phát hiện hành vi tấn công thực sự.

---
layout: post
title: "Windows Event Logging Basics"
date: 2026-06-24 23:44:46 +0700
categories: [SOC, Windows Event Logs]
tags: [soc, windows-event-logs, event-viewer, evtx, security-logs, system-logs, xml-query, logon-events, audit-policy, blue-team, dfir]
description: "Tóm tắt Windows Event Logs: cấu trúc log, Event Viewer, anatomy của event, XML query, Logon ID correlation và các Event ID quan trọng cho SOC/DFIR."
toc: true
---

# Windows Event Logging Basics

**Windows Event Logs** là một phần cốt lõi của Windows, dùng để ghi lại hoạt động từ hệ điều hành, ứng dụng, service, ETW providers và các thành phần bảo mật.

Trong SOC và DFIR, Windows Event Logs giúp analyst điều tra:

- Logon activity.
- Failed login.
- Privilege usage.
- Service installation.
- Scheduled task persistence.
- Audit policy change.
- Defender detection.
- Network share access.
- System shutdown/restart.
- Event log tampering.

```text
Windows Event Logs = nguồn telemetry nền tảng để điều tra hành vi trên Windows.
```

---

## Timeline cập nhật

```text
Created at: 2026-06-24 23:44:46 +0700
Timezone: Asia/Ho_Chi_Minh
Topic: Windows Event Logs
Focus: Event Viewer, event anatomy, XML filtering, useful Event IDs
```

---

# 1. Windows Event Logs là gì?

Windows Event Logs lưu lại sự kiện từ nhiều thành phần trong hệ thống.

Các nguồn ghi log có thể gồm:

- Windows OS.
- Applications.
- Services.
- Security auditing.
- ETW providers.
- Setup/install components.
- Defender/antivirus.
- Forwarded events từ máy khác.

Các log này thường được lưu dưới dạng `.evtx`.

---

# 2. Các log mặc định trong Event Viewer

Trong Event Viewer, nhóm **Windows Logs** thường gồm:

| Log | Ý nghĩa |
|---|---|
| Application | Lỗi/thông tin từ ứng dụng |
| Security | Logon, audit, privilege, object access |
| Setup | Cài đặt, update, setup activities |
| System | Service, driver, boot, shutdown, system events |
| Forwarded Events | Log được forward từ máy khác |

Trong phân tích single host, ta thường tập trung vào Application, Security, System và đôi khi Setup.

Trong môi trường enterprise, **Forwarded Events** hữu ích khi dùng Windows Event Forwarding để gom log từ nhiều máy.

---

# 3. Event Viewer và EVTX

Có thể mở Event Viewer bằng:

```text
Start Menu → Event Viewer
```

Hoặc chạy:

```powershell
eventvwr.msc
```

Event Viewer cũng có thể mở file `.evtx` đã lưu trước đó. File này sẽ nằm trong mục:

```text
Saved Logs
```

Điều này hữu ích khi làm lab hoặc điều tra offline.

---

# 4. Anatomy of an Event Log

Mỗi Windows event thường có các thành phần chính:

| Thành phần | Ý nghĩa |
|---|---|
| Log Name | Tên log, ví dụ Security/System/Application |
| Source | Nguồn tạo event |
| Event ID | Mã định danh event |
| Task Category | Loại tác vụ/sự kiện |
| Level | Information, Warning, Error, Critical, Verbose |
| Keywords | Audit Success, Audit Failure, Classic... |
| User | User liên quan |
| OpCode | Operation cụ thể |
| Logged | Thời điểm event được ghi |
| Computer | Máy tạo event |
| XML Data | Dữ liệu event ở dạng XML |

---

## 4.1. Level

Các level thường gặp:

```text
Information
Warning
Error
Critical
Verbose
```

Ví dụ:

- Information: ứng dụng/service start hoặc stop.
- Error: ứng dụng lỗi, dependency lỗi, crash.
- Critical: sự kiện nghiêm trọng cần xử lý ngay.

---

## 4.2. Keywords

`Keywords` giúp filter event theo nhóm.

Trong Security log, thường gặp:

```text
Audit Success
Audit Failure
```

Ví dụ:

- Audit Success: logon thành công, object access thành công.
- Audit Failure: failed logon, access denied.

---

# 5. Ví dụ Application Log

Application log có thể chứa lỗi như:

```text
SideBySide Event ID 35
```

Event này có thể báo lỗi activation context hoặc mismatch architecture.

Khi điều tra Application log, cần xem:

- Event ID.
- Source.
- General description.
- Process ID.
- Timestamp.
- XML details.
- Ứng dụng liên quan.

Application log hữu ích để phát hiện lỗi app, nhưng trong SOC thường cần correlation với Security/System/Sysmon để xác định có malicious activity hay không.

---

# 6. Security Log và Event ID 4624

**Event ID 4624** là successful logon.

Nó cho biết một logon session đã được tạo trên máy đích.

Các field quan trọng:

| Field | Ý nghĩa |
|---|---|
| TargetUserName | User đăng nhập |
| Logon ID | ID để correlate với event khác |
| Logon Type | Kiểu đăng nhập |
| Logon Process | Process xử lý logon |
| Authentication Package | Gói xác thực |
| Workstation Name | Máy nguồn nếu có |
| Source Network Address | IP nguồn nếu có |

---

## 6.1. Logon Type quan trọng

| Logon Type | Ý nghĩa |
|---|---|
| 2 | Interactive logon |
| 3 | Network logon |
| 4 | Batch logon |
| 5 | Service logon |
| 7 | Unlock |
| 10 | RemoteInteractive / RDP |
| 11 | CachedInteractive |

Ví dụ trong bài, Logon Type 5 thường liên quan service logon.

---

# 7. Correlation bằng Logon ID

`Logon ID` rất quan trọng vì nhiều event khác có thể dùng cùng ID này.

Ví dụ:

```text
4624 Successful Logon
      ↓
4672 Special Privileges Assigned
      ↓
Service/activity/event khác cùng Logon ID
```

Khi điều tra, không nên nhìn Event 4624 riêng lẻ. Cần correlate theo Logon ID để hiểu session đó đã làm gì.

---

# 8. Custom XML Queries trong Event Viewer

Event Viewer cho phép filter nâng cao bằng XML.

Đường dẫn:

```text
Filter Current Log → XML → Edit Query Manually
```

Ví dụ filter theo `SubjectLogonId`:

```xml
<QueryList>
  <Query Id="0" Path="Security">
    <Select Path="Security">
      *[EventData[Data[@Name='SubjectLogonId']='0x3E7']]
    </Select>
  </Query>
</QueryList>
```

Ý nghĩa:

- Tìm các event có `SubjectLogonId = 0x3E7`.
- Giúp correlate hoạt động liên quan cùng logon session.
- Hữu ích khi GUI filter thông thường chưa đủ chi tiết.

---

# 9. Event ID 4907 - Audit Policy / SACL Change

**Event ID 4907** liên quan thay đổi auditing/SACL của object.

SACL là:

```text
System Access Control List
```

SACL quyết định loại access nào sẽ được ghi vào Security Event Log.

Nếu SACL của file/registry key bị thay đổi, có thể là:

- Hoạt động setup hợp lệ.
- Cấu hình audit mới.
- Hoạt động che giấu hoặc thay đổi logging.
- Malware/tampering nếu process đáng ngờ.

Field cần xem:

- SubjectUserName.
- ObjectType.
- ObjectName.
- ProcessName.
- OldSd.
- NewSd.
- Logon ID.

---

# 10. Event ID 4672 - Special Privileges Assigned

**Event ID 4672** xuất hiện khi account logon với quyền đặc biệt.

Các privilege đáng chú ý:

```text
SeDebugPrivilege
SeBackupPrivilege
SeRestorePrivilege
SeTakeOwnershipPrivilege
SeLoadDriverPrivilege
SeImpersonatePrivilege
```

Đặc biệt:

```text
SeDebugPrivilege
```

có thể cho phép process thao tác với memory của process khác, liên quan đến credential dumping hoặc debug/tampering.

---

# 11. Useful Windows System Logs

| Event ID | Ý nghĩa | Giá trị SOC |
|---|---|---|
| 1074 | System shutdown/restart | Tìm restart bất thường |
| 6005 | Event Log Service started | Mốc boot/start log service |
| 6006 | Event Log Service stopped | Shutdown hoặc log service stop |
| 6013 | Windows uptime | Kiểm tra uptime/reboot |
| 7040 | Service startup type changed | Service tampering |

---

# 12. Useful Windows Security Logs

| Event ID | Ý nghĩa | Giá trị SOC |
|---|---|---|
| 1102 | Audit log cleared | Dấu hiệu xóa dấu vết |
| 1116 | Defender malware detected | Malware detection |
| 1118 | Defender remediation started | Bắt đầu xử lý malware |
| 1119 | Defender remediation succeeded | Xử lý thành công |
| 1120 | Defender remediation failed | Xử lý thất bại |
| 4624 | Successful logon | Baseline/logon tracking |
| 4625 | Failed logon | Bruteforce/password spray |
| 4648 | Explicit credentials used | Lateral movement/RunAs |
| 4656 | Handle to object requested | Access sensitive object |
| 4672 | Special privileges assigned | Privilege usage |
| 4698 | Scheduled task created | Persistence |
| 4700 | Scheduled task enabled | Persistence |
| 4701 | Scheduled task disabled | Tampering/persistence |
| 4702 | Scheduled task updated | Persistence modification |
| 4719 | Audit policy changed | Defense evasion |
| 4738 | User account changed | Account takeover/insider |
| 4771 | Kerberos pre-auth failed | Kerberos brute force |
| 4776 | DC credential validation | Credential validation tracking |
| 5001 | Defender real-time protection changed | AV tampering |
| 5140 | Network share accessed | Share access |
| 5142 | Network share added | Suspicious share creation |
| 5145 | Network share access checked | Share enumeration/access |
| 5157 | WFP blocked connection | Blocked network attempt |
| 7045 | Service installed | Service persistence/malware |

---

# 13. SOC hunting ideas

## 13.1. Audit log cleared

```text
Event ID: 1102
Meaning: The audit log was cleared
Why important: attacker may erase evidence
```

## 13.2. Failed logon burst

```text
Event ID: 4625
Use case: brute force/password spray
```

## 13.3. Explicit credential usage

```text
Event ID: 4648
Use case: RunAs/lateral movement
```

## 13.4. Scheduled task persistence

```text
Event IDs: 4698, 4700, 4701, 4702
Use case: persistence and task manipulation
```

## 13.5. Audit policy tampering

```text
Event ID: 4719
Use case: attacker changes audit policy to reduce visibility
```

## 13.6. Service installation

```text
Event ID: 7045
Use case: malware installed as service, PsExec-like behavior
```

---

# 14. Investigation mindset

Windows Event Logs cần được đọc theo chuỗi, không nên đọc từng event rời rạc.

Ví dụ:

```text
4624 Successful Logon
      ↓
4672 Special Privileges Assigned
      ↓
4698 Scheduled Task Created
      ↓
7045 Service Installed
      ↓
5157 Connection Blocked
```

Correlation nên dựa trên:

- Time.
- User.
- Computer.
- Logon ID.
- ProcessName.
- Source IP.
- Target object.
- Event sequence.

---

# 15. Key Takeaway

Windows Event Logs là nền tảng quan trọng của SOC/DFIR trên Windows.

Muốn điều tra tốt, analyst cần hiểu:

```text
Event anatomy
→ Event ID
→ XML details
→ Logon ID correlation
→ Useful Security/System events
→ Baseline normal behavior
→ Correlate với Sysmon/EDR/SIEM
```

Một event riêng lẻ có thể chưa đủ để kết luận malicious. Giá trị thật sự đến từ việc ghép nhiều event lại thành timeline và attack narrative.

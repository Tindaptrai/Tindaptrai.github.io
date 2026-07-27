---
layout: post
title: "Windows Forensics Overview"
date: 2026-07-27 21:01:24 +0700
categories: ["Introduction to Digital Forensics", "Windows Forensics"]
tags: [windows-forensics, ntfs, mft, usn-journal, prefetch, amcache, shimcache, userassist, lnk, jump-lists, registry, persistence, browser-forensics, srum, evtx, dfir, cdsa]
description: "Tổng quan Windows Forensics với các artifact quan trọng: NTFS, Event Logs, execution artifacts, persistence, browser data và SRUM."
toc: true
---

# Windows Forensics Overview

Bài này tập trung vào các **Windows forensic artifacts** quan trọng để dựng timeline, xác định execution, persistence và user activity.

## Công thức dễ nhớ

```text
FILE SYSTEM → EVENT LOGS → EXECUTION → PERSISTENCE → BROWSER → SRUM
```

# 1. NTFS Forensics

NTFS lưu nhiều metadata hữu ích cho điều tra.

## Artifact quan trọng

### File Metadata

```text
Creation Time
Modification Time
Access Time
File Attributes
```

Dùng để dựng timeline và xác định thay đổi trên file.

### MFT — Master File Table

MFT lưu metadata của mọi file và thư mục:

```text
File name
File size
Timestamps
Data location
File state
```

File bị xóa thường chỉ bị đánh dấu là available; dữ liệu có thể vẫn còn đến khi bị ghi đè.

### Unallocated Space và File Slack

```text
Unallocated Space = vùng chưa được gán cho file
File Slack        = phần thừa trong cluster
```

Có thể chứa dữ liệu đã xóa hoặc fragment của file cũ.

### File Signatures

Dùng magic bytes/header để xác định loại file thật dù extension bị đổi.

### USN Journal

Ghi nhận:

```text
Create
Modify
Delete
Rename
```

Rất hữu ích để dựng file activity timeline.

### Các artifact NTFS khác

```text
LNK Files            → target path, timestamp, user interaction
Registry Hives       → cấu hình, user activity, persistence
Shellbags            → thư mục từng được duyệt
Thumbnail Cache      → preview của file từng xem
Recycle Bin          → file đã xóa
Alternate Data Streams (ADS) → dữ liệu ẩn gắn với file
Volume Shadow Copies → snapshot file system
ACL/Security Descriptor → quyền truy cập
```

# 2. Windows Event Logs

Đường dẫn mặc định:

```text
C:\Windows\System32\winevt\Logs
```

Event Logs có thể ghi nhận:

```text
Process creation
Logon
Service activity
PowerShell
Application errors
Privilege escalation
Credential access
Lateral movement
```

Nguồn thường dùng:

```text
Security
System
Application
Sysmon
PowerShell
Defender
Task Scheduler
```

Công cụ phân tích:

```text
Event Viewer
Get-WinEvent
Chainsaw
Sigma
Splunk
ELK
```

# 3. Execution Artifacts

Execution artifacts giúp xác định chương trình nào đã chạy, khi nào và bởi ai.

| Artifact | Vị trí | Giá trị điều tra |
|---|---|---|
| Prefetch | `C:\Windows\Prefetch` | Execution history, run count, timestamps |
| Shimcache | `HKLM\SYSTEM\CurrentControlSet\Control\Session Manager\AppCompatCache` | File path và execution-related metadata |
| Amcache | `C:\Windows\AppCompat\Programs\Amcache.hve` | Path, size, signature, timestamp |
| UserAssist | `HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist` | App đã chạy, run count, timestamp |
| RunMRU | `HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\RunMRU` | Lệnh gần đây trong Run dialog |
| Jump Lists | `%AppData%\Microsoft\Windows\Recent` | File, folder, task gần đây |
| LNK Files | Desktop, Start Menu, Recent | Target path và user context |
| Recent Items | `%AppData%\Microsoft\Windows\Recent` | File được mở gần đây |
| Event Logs | `C:\Windows\System32\winevt\Logs` | Process creation và activity |

Nhớ nhanh:

```text
Prefetch   → chương trình đã chạy
Shimcache  → dấu vết executable
Amcache    → metadata executable
UserAssist → user đã mở ứng dụng
RunMRU     → lệnh đã nhập
Jump List  → file/task gần đây
LNK        → target path và user interaction
```

> Không nên kết luận execution chỉ từ một artifact; cần correlate nhiều nguồn.

# 4. Windows Persistence Artifacts

## Registry Run/RunOnce

```text
HKCU\Software\Microsoft\Windows\CurrentVersion\Run
HKCU\Software\Microsoft\Windows\CurrentVersion\RunOnce
HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce
```

## Winlogon

```text
HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon
HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon\Shell
```

## Startup Folder Keys

```text
HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\User Shell Folders
HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\Shell Folders
HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Shell Folders
```

## Scheduled Tasks

```text
C:\Windows\System32\Tasks
```

Mỗi task thường chứa XML với:

```text
Author
Trigger
Action
Command
Arguments
Execution context
```

## Services

```text
HKLM\System\CurrentControlSet\Services
```

Kiểm tra:

```text
ImagePath
ServiceDll
Start type
Account
Description
```

# 5. Browser Forensics

Artifact quan trọng:

```text
Browsing History
Cookies
Cache
Bookmarks
Download History
Autofill Data
Search History
Session Data
Typed URLs
Form Data
Saved Passwords
Web Storage
Favicons
Tab Recovery
Extensions
```

Ứng dụng điều tra:

- Xác định website đã truy cập.
- Tìm URL tải malware.
- Kiểm tra download history.
- Phân tích cookie/session.
- Tìm dấu vết phishing.
- Xác định extension đáng ngờ.

# 6. SRUM — System Resource Usage Monitor

Database:

```text
C:\Windows\System32\sru\SRUDB.dat
```

SRUM theo dõi:

```text
Application execution
CPU usage
Network usage
Resource consumption
User context
```

Giá trị điều tra:

```text
Application Profiling → ứng dụng/process từng hoạt động
Resource Consumption  → CPU/network usage bất thường
Timeline Reconstruction → chuỗi hoạt động theo thời gian
User Attribution      → liên kết activity với user
Malware Detection     → app lạ, network spike, resource anomaly
Incident Response     → xác định nhanh activity gần thời điểm incident
```

# 7. Mapping Artifact Theo Câu Hỏi

| Câu hỏi | Artifact nên kiểm tra |
|---|---|
| File nào đã tồn tại hoặc bị xóa? | MFT, USN Journal, Recycle Bin, Shadow Copies |
| Chương trình nào đã chạy? | Prefetch, Amcache, Shimcache, UserAssist, Event Logs |
| User đã mở file nào? | LNK, Jump Lists, Recent Items, Shellbags |
| Persistence nằm ở đâu? | Run Keys, Winlogon, Scheduled Tasks, Services |
| User đã truy cập website nào? | History, Cache, Cookies, Downloads, Favicons |
| Process nào dùng network/resource? | SRUM, Event Logs, network telemetry |
| File có bị đổi extension không? | File signatures, MFT, content inspection |

# 8. Keyword Cần Nhớ

```text
NTFS
MFT
USN Journal
File Slack
Unallocated Space
LNK
Prefetch
Shimcache
Amcache
UserAssist
RunMRU
Jump Lists
Shellbags
Thumbnail Cache
ADS
Volume Shadow Copies
EVTX
Run Keys
Scheduled Tasks
Services
Browser History
Cookies
Cache
SRUM
SRUDB.dat
Timeline
Persistence
Execution Artifact
```

# 9. Workflow SOC/DFIR

```text
1. Xác định thời gian incident
2. Thu thập EVTX và Registry hives
3. Phân tích MFT/USN Journal
4. Correlate Prefetch, Amcache, Shimcache
5. Kiểm tra persistence
6. Phân tích browser artifacts
7. Kiểm tra SRUM
8. Dựng timeline tổng hợp
9. Xác định IOC/TTP
10. Viết báo cáo
```

# Key Takeaway

```text
NTFS         → file activity
Event Logs   → system/security activity
Prefetch     → execution
Amcache      → executable metadata
UserAssist   → user execution
LNK/JumpList → file access
Registry     → configuration/persistence
Browser      → online activity
SRUM         → application + resource usage
```

Windows Forensics hiệu quả nhất khi **correlate nhiều artifact**, không kết luận chỉ từ một nguồn dữ liệu.

---
layout: post
title: "Event Tracing for Windows ETW Fundamentals"
date: 2026-06-13 00:35:00 +0700
categories: [SOC, Windows Security]
tags: [soc, etw, event-tracing-for-windows, windows-security, telemetry, logman, providers, consumers, trace-session, etl, blue-team, dfir]
description: "Ghi chú tổng hợp về Event Tracing for Windows: khái niệm ETW, kiến trúc providers/controllers/consumers, trace sessions, ETL files, logman, useful providers và restricted providers."
toc: true
---

# Event Tracing for Windows ETW Fundamentals

**Event Tracing for Windows (ETW)** là cơ chế tracing hiệu năng cao được tích hợp sâu trong Windows.

Trong SOC và DFIR, ETW là nguồn telemetry rất mạnh vì nó có thể cung cấp dữ liệu chi tiết hơn so với log truyền thống như Windows Event Logs hoặc Sysmon.

```text
ETW = cơ chế tracing tốc độ cao của Windows để thu thập telemetry từ kernel, ứng dụng và driver.
```

---

## 1. Vì sao cần ETW?

Khi điều tra incident, analyst thường dựa vào:

- Windows Event Logs.
- Sysmon logs.
- EDR telemetry.
- SIEM logs.
- Firewall, Proxy, DNS logs.

Tuy nhiên, các nguồn log này không phải lúc nào cũng đủ chi tiết. ETW giúp mở rộng visibility bằng cách thu thập event từ nhiều lớp khác nhau của hệ điều hành.

ETW có thể cung cấp thông tin về:

- Process creation và termination.
- Network activity.
- File operations.
- Registry modifications.
- Kernel activity.
- PowerShell activity.
- .NET runtime activity.
- SMB, WinRM, RDP activity.
- DNS client activity.
- Antimalware events.

---

# 2. ETW là gì?

Theo Microsoft, **Event Tracing for Windows** là một high-speed tracing facility của Windows.

ETW dùng cơ chế buffering và logging trong kernel để ghi nhận event từ:

- User-mode applications.
- Kernel-mode device drivers.
- Operating system components.
- Third-party software.
- Custom event providers.

ETW có thể dùng cho real-time monitoring, troubleshooting, forensic investigation và detection engineering.

---

# 3. ETW Architecture & Components

Kiến trúc ETW hoạt động theo mô hình publish-subscribe.

```text
Providers → ETW Sessions → Consumers / ETL Files
        ↑
   Controllers
```

Các thành phần chính:

| Thành phần | Vai trò |
|---|---|
| Providers | Tạo event |
| Controllers | Quản lý trace session |
| Trace Sessions | Nhận, buffer và điều phối event |
| Consumers | Nhận event để xử lý/phân tích |
| ETL Files | Lưu event xuống disk |

---

# 4. Controllers

**Controllers** điều khiển hoạt động ETW.

Controller có thể:

- Tạo trace session.
- Start hoặc stop trace session.
- Enable hoặc disable provider.
- Cấu hình level.
- Cấu hình keyword filter.
- Quản lý output file.

Controller phổ biến có sẵn trong Windows là:

```cmd
logman.exe
```

Ví dụ:

```cmd
logman.exe query -ets
```

Lệnh này liệt kê các Event Tracing Sessions đang chạy.

---

# 5. Providers

**Providers** là thành phần tạo event và gửi event vào ETW session.

Provider có thể đến từ:

- Windows kernel.
- Windows components.
- Applications.
- Drivers.
- Security products.
- Third-party software.
- Custom internal applications.

Ví dụ provider hữu ích:

```text
Microsoft-Windows-Kernel-Process
Microsoft-Windows-Kernel-File
Microsoft-Windows-Kernel-Network
Microsoft-Windows-PowerShell
Microsoft-Windows-DotNETRuntime
Microsoft-Windows-DNS-Client
```

---

## 5.1. Các loại ETW Providers

| Provider type | Ý nghĩa |
|---|---|
| MOF Providers | Dựa trên Managed Object Format |
| WPP Providers | Windows Software Trace Preprocessor |
| Manifest-based Providers | Dựa trên XML manifest |
| TraceLogging Providers | API tracing đơn giản, hiệu quả |

---

# 6. Consumers

**Consumers** là thành phần nhận event từ ETW session để xử lý hoặc phân tích.

Consumer có thể:

- Ghi event xuống ETL file.
- Đọc event real-time.
- Phân tích event bằng Windows API.
- Đưa event vào monitoring tool.
- Phục vụ forensic investigation.

Ví dụ consumer:

- Windows Event Viewer.
- Performance Monitor.
- EDR hoặc security product.
- DFIR tool.
- Custom ETW script/tool.

---

# 7. Channels

**Channels** là logical containers để tổ chức và lọc event.

Ví dụ:

```text
Operational
Analytic
Debug
Admin
Diagnostic
```

Lưu ý:

```text
Chỉ những ETW provider event có Channel property mới có thể được consume bởi Windows Event Log.
```

Vì vậy không phải mọi ETW event đều hiện trong Event Viewer.

---

# 8. ETL Files

ETW có thể ghi event xuống file:

```text
.etl
```

**ETL** là viết tắt của:

```text
Event Trace Log
```

ETL files hữu ích cho:

- Offline analysis.
- Long-term storage.
- Forensic investigation.
- Performance troubleshooting.
- Historical analysis.

---

# 9. Interacting With ETW bằng Logman

**Logman** là công cụ dòng lệnh có sẵn trong Windows để quản lý ETW.

## 9.1. Liệt kê trace sessions đang chạy

```cmd
logman.exe query -ets
```

Tham số `-ets` rất quan trọng. Nếu thiếu `-ets`, Logman có thể không hiển thị đúng Event Tracing Sessions đang chạy.

Ví dụ session có thể thấy:

```text
Eventlog-Security
EventLog-Application
EventLog-System
EventLog-Microsoft-Windows-Sysmon-Operational
SYSMON TRACE
SysmonDnsEtwSession
```

---

## 9.2. Query một trace session cụ thể

```cmd
logman.exe query "EventLog-System" -ets
```

Thông tin có thể thu được:

- Name.
- Status.
- Root Path.
- Max Log Size.
- Buffer Size.
- Buffers Lost.
- Providers subscribed.
- Provider GUID.
- Level.
- KeywordsAny.
- KeywordsAll.

---

## 9.3. Query tất cả providers

```cmd
logman.exe query providers
```

Kết quả gồm:

```text
Provider name
Provider GUID
```

Windows có thể có hơn 1.000 providers built-in, chưa tính provider của third-party software.

---

## 9.4. Filter provider bằng findstr

Ví dụ tìm provider Winlogon:

```cmd
logman.exe query providers | findstr "Winlogon"
```

---

## 9.5. Query chi tiết một provider

```cmd
logman.exe query providers Microsoft-Windows-Winlogon
```

Có thể xem:

- Provider GUID.
- Keywords.
- Keyword description.
- Levels.
- Process đang dùng provider.

---

# 10. GUI tools cho ETW

## 10.1. Performance Monitor

Performance Monitor có thể xem running Event Trace Sessions.

Có thể:

- Xem session đang chạy.
- Double-click để xem chi tiết.
- Xem provider đang được enable.
- Thêm hoặc xóa provider.
- Tạo session mới trong User Defined.

## 10.2. EtwExplorer

**EtwExplorer** giúp xem metadata của ETW providers.

Có thể dùng để:

- Tìm provider.
- Xem GUID.
- Xem keywords.
- Xem event metadata.
- Tìm provider liên quan PowerShell, DNS, Winlogon.

---

# 11. Useful ETW Providers cho SOC/DFIR

| Provider | Giá trị điều tra |
|---|---|
| Microsoft-Windows-Kernel-Process | Process injection, process hollowing |
| Microsoft-Windows-Kernel-File | File operations, ransomware, staging |
| Microsoft-Windows-Kernel-Network | Network activity, C2, exfiltration |
| Microsoft-Windows-SMBClient / SMBServer | SMB traffic, lateral movement |
| Microsoft-Windows-DotNETRuntime | .NET execution, assembly loading |
| OpenSSH | SSH login attempts, brute force |
| Microsoft-Windows-VPN-Client | VPN login anomalies |
| Microsoft-Windows-PowerShell | PowerShell execution, script block |
| Microsoft-Windows-Kernel-Registry | Registry persistence |
| Microsoft-Windows-CodeIntegrity | Unsigned driver/code integrity |
| Microsoft-Antimalware-Service | Antimalware service issues |
| WinRM | Remote management, lateral movement |
| Microsoft-Windows-TerminalServices-LocalSessionManager | RDP/session activity |
| Microsoft-Windows-Security-Mitigations | Security mitigation telemetry |
| Microsoft-Windows-DNS-Client | DNS tunneling, DGA, C2 lookups |
| Microsoft-Antimalware-Protection | Protection status/config changes |

---

# 12. Restricted Providers

Một số ETW providers là **restricted providers**.

Restricted providers cung cấp telemetry giá trị cao nhưng chỉ accessible bởi process có quyền phù hợp.

Ví dụ quan trọng:

```text
Microsoft-Windows-Threat-Intelligence
```

Provider này hữu ích cho:

- DFIR.
- Advanced threat detection.
- Attack investigation.
- Real-time monitoring.
- Forensic evidence.

Tuy nhiên, để truy cập provider này thường cần quyền đặc biệt như:

```text
Protected Process Light - PPL
```

---

# 13. ETW trong Detection Engineering

ETW có thể giúp phát hiện các hành vi mà Sysmon hoặc Windows Event Logs có thể không ghi nhận đầy đủ.

| Hành vi | Provider gợi ý |
|---|---|
| Process injection | Microsoft-Windows-Kernel-Process |
| File tampering | Microsoft-Windows-Kernel-File |
| C2 traffic | Microsoft-Windows-Kernel-Network / DNS-Client |
| PowerShell abuse | Microsoft-Windows-PowerShell |
| .NET execution | Microsoft-Windows-DotNETRuntime |
| Registry persistence | Microsoft-Windows-Kernel-Registry |
| RDP activity | TerminalServices-LocalSessionManager |
| WinRM lateral movement | WinRM |
| Code integrity bypass | Microsoft-Windows-CodeIntegrity |

---

# 14. Tư duy sử dụng ETW trong SOC

ETW rất mạnh nhưng cần dùng đúng cách.

SOC/DFIR nên chú ý:

- Không enable quá nhiều provider cùng lúc.
- Provider volume cao có thể gây noise.
- Cần xác định rõ mục tiêu collection.
- Cần filter bằng keyword/level nếu có thể.
- Cần lưu ETL file khi phục vụ forensic.
- Cần dùng real-time consumer cho monitoring.
- Cần correlation với Sysmon và Windows Event Logs.

Ví dụ:

```text
ETW PowerShell Provider
      +
Sysmon Process Creation
      +
Windows Security Logon Events
      =
Timeline rõ hơn về post-exploitation activity
```

---

# 15. Bảng ôn nhanh thuật ngữ

| Thuật ngữ | Nghĩa ngắn gọn |
|---|---|
| ETW | Event Tracing for Windows |
| Provider | Thành phần tạo event |
| Controller | Thành phần quản lý trace session |
| Consumer | Thành phần nhận và xử lý event |
| Trace Session | Phiên tracing nhận event từ provider |
| ETL File | Event Trace Log file |
| Channel | Logical container cho event |
| Logman | Công cụ quản lý ETW bằng CLI |
| Provider GUID | Mã định danh provider |
| KeywordsAny | Bộ lọc event theo keyword |
| Level | Mức event như Error/Warning/Information |
| PPL | Protected Process Light |
| Restricted Provider | Provider yêu cầu quyền đặc biệt |
| Microsoft-Windows-Threat-Intelligence | Restricted provider giá trị cao |

---

# 16. Câu hỏi ôn tập

## 1. ETW là gì?

ETW là cơ chế tracing tốc độ cao của Windows, dùng để thu thập event từ user-mode applications, kernel-mode drivers và Windows components.

## 2. Ba thành phần chính trong mô hình ETW là gì?

Ba thành phần quan trọng là:

```text
Providers
Controllers
Consumers
```

Ngoài ra còn có trace sessions, channels và ETL files.

## 3. Logman dùng để làm gì?

Logman dùng để quản lý ETW trace sessions và providers, ví dụ query session đang chạy, query provider, start/stop trace session.

## 4. Lệnh nào liệt kê Event Tracing Sessions đang chạy?

```cmd
logman.exe query -ets
```

## 5. Lệnh nào liệt kê tất cả providers?

```cmd
logman.exe query providers
```

## 6. ETL file dùng để làm gì?

ETL file dùng để lưu event trace xuống disk, phục vụ offline analysis, forensic investigation và long-term storage.

## 7. Vì sao không nên enable quá nhiều ETW providers cùng lúc?

Vì một số providers tạo rất nhiều event, có thể gây noise, tăng tài nguyên sử dụng và làm khó phân tích.

## 8. Microsoft-Windows-Threat-Intelligence có gì đặc biệt?

Đây là restricted provider có telemetry giá trị cao cho threat detection/DFIR, nhưng cần quyền đặc biệt như PPL để truy cập.

---

# Key Takeaway

ETW là một trong những nguồn telemetry mạnh nhất trên Windows.

Nếu Windows Event Logs và Sysmon giúp analyst nhìn thấy nhiều hoạt động quan trọng, thì ETW có thể cung cấp mức quan sát sâu hơn nữa thông qua providers, trace sessions, consumers và ETL files.

Trong SOC/DFIR, ETW rất hữu ích cho threat detection, incident response, forensic investigation và detection engineering. Tuy nhiên, ETW cần được bật và filter có chủ đích để tránh noise, giảm tải hệ thống và giữ lại đúng telemetry có giá trị.

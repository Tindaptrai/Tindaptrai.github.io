---
layout: post
title: "HTB Academy Purple Modules và Windows DFIR Toolset"
date: 2026-06-03 04:00:00 +0700
categories: [SOC, DFIR]
tags: [soc, dfir, purple-team, htb-academy, windows-forensics, velociraptor, sysmon, yara, volatility, blue-team, red-team]
description: "Ghi chú tổng hợp về HTB Academy Purple Modules, lợi ích cho Blue Team/Red Team/Purple Team và bộ công cụ Windows DFIR được cài sẵn."
toc: true
---

# HTB Academy Purple Modules và Windows DFIR Toolset

Bài này tổng hợp phần giới thiệu về **HTB Academy Purple Modules** và bộ công cụ **Windows DFIR Toolset** được cài sẵn trong các mục tiêu Windows Purple Module.

Purple Module giúp người học nhìn vấn đề từ cả hai phía:

- **Red Team**: thực hiện tấn công, mô phỏng hành vi adversary.
- **Blue Team**: điều tra, thu thập log, phân tích artifact và xây dựng detection.
- **Purple Team**: kết hợp hai bên để cải thiện năng lực phòng thủ.

---

# 1. Giới thiệu về Purple Modules

Trong HTB Academy, **Purple Modules** là các mô-đun kết hợp giữa tư duy tấn công và phòng thủ.

Các mô-đun này giúp thu hẹp khoảng cách giữa:

```text
Offensive Security
        +
Defensive Security
        =
Purple Team Learning
```

Mục tiêu là giúp người học không chỉ biết cách tấn công, mà còn hiểu:

- Cuộc tấn công để lại log gì.
- Artifact nào được tạo ra.
- Network traffic có dấu hiệu gì.
- Memory dump có thể chứa bằng chứng gì.
- Công cụ DFIR nào có thể dùng để phân tích.
- Detection rule nào có thể viết sau khi quan sát hành vi tấn công.

---

# 2. Vì sao cần thực hiện phần tấn công trước?

Phân tích pháp y cần có dữ liệu.

Vì vậy, trong Purple Module, phần tấn công thường phải xảy ra trước để tạo ra:

- Log.
- Event.
- Network traffic.
- File artifact.
- Process artifact.
- Memory artifact.
- Registry change.
- Detection telemetry.

Sau khi có dữ liệu, người học mới có thể thực hiện phần điều tra và phân tích forensic.

Ví dụ:

```text
Attack executed
      ↓
Logs generated
      ↓
Traffic captured
      ↓
Memory can be dumped
      ↓
DFIR analysis begins
```

Điều này cũng áp dụng cho **memory dump**. Bộ nhớ chỉ có ý nghĩa phân tích nếu sau khi attack xảy ra, target vẫn còn hoạt động và dữ liệu vẫn còn tồn tại trong RAM.

---

# 3. Lưu ý khi dùng Purple Module Targets

Khi spawn target trong HTB Academy, mục tiêu cần được giữ hoạt động đủ lâu để phục vụ forensic.

Một số lưu ý:

- Target phải tiếp tục chạy sau khi attack xảy ra.
- Không tắt target quá sớm.
- Có thể cần kéo dài thời gian target.
- Một số service cần vài phút để khởi động đầy đủ.
- Nếu target tắt, log/traffic/memory artifact có thể mất hoặc không còn đầy đủ.

Nói ngắn gọn:

```text
Attack xong chưa nên tắt máy.
Cần giữ target sống để điều tra.
```

---

# 4. Cấu trúc Purple Module

Mô-đun được chia thành hai phần chính:

```text
Windows Purple Module Targets
Linux Purple Module Targets
```

Mỗi phần cung cấp hướng dẫn tham khảo để người học biết cách:

- Truy cập target.
- Cấu hình cơ chế logging.
- Xác định vị trí log.
- Thu thập network traffic.
- Tạo memory dump.
- Tìm file cấu hình.
- Sử dụng công cụ DFIR được cài sẵn.
- Phân tích artifact sau khai thác.

---

# 5. Lợi ích của Purple Modules

Purple Modules có lợi cho cả Blue Team, Red Team và Purple Team.

---

## 5.1. Lợi ích cho Blue Team

Đối với Blue Team, Purple Module giúp:

- Hiểu chiến thuật và kỹ thuật của Red Team.
- Quan sát attack artifact thực tế.
- Phân tích log sau khai thác.
- Tìm IOC.
- Xây dựng detection rule.
- Kiểm tra giả thuyết threat hunting.
- Phát triển incident response playbook.
- Hiểu cách attacker di chuyển và để lại dấu vết.

Ví dụ use case:

```text
Blue Team mô phỏng tấn công
        ↓
Thu thập log và artifact
        ↓
Phân tích IOC
        ↓
Viết detection rule
        ↓
Kiểm tra lại detection
```

---

## 5.2. Lợi ích cho Red Team

Đối với Red Team, Purple Module giúp hiểu rõ hơn cuộc tấn công của mình để lại dấu vết gì.

Red Team có thể:

- Kiểm tra payload tự viết.
- Quan sát log được tạo ra.
- Quan sát process behavior.
- Xem telemetry từ endpoint.
- Hiểu tool của mình bị phát hiện ở đâu.
- Tinh chỉnh kỹ thuật để giảm dấu vết.
- Cải thiện OPSEC.

Ví dụ:

```text
Run payload
      ↓
Check logs and telemetry
      ↓
Identify detection points
      ↓
Refine technique
```

---

## 5.3. Lợi ích cho Purple Team

Đối với Purple Team, các target này là môi trường chia sẻ giữa Red Team và Blue Team.

Cả hai bên có thể cùng:

- Mô phỏng attack chain.
- Quan sát detection.
- Kiểm tra logging.
- Kiểm tra alert.
- Xây dựng detection.
- Tối ưu response playbook.
- Cải thiện khả năng phòng thủ.

Purple Team không chỉ hỏi:

```text
Có tấn công được không?
```

Mà còn hỏi:

```text
Có phát hiện được không?
Nếu không phát hiện được thì thiếu log nào?
Nếu phát hiện được thì quy trình phản ứng có tốt không?
```

---

# 6. Purple Module Targets là hạ tầng có thể tái sử dụng

Các Purple Module Targets có thể được dùng lại như một môi trường thực hành DFIR.

Blue Team có thể:

- Chuyển bằng chứng từ máy bị compromise khác sang target để phân tích.
- Cài phần mềm dễ bị tấn công.
- Mô phỏng tấn công.
- Phân tích artifact để lại.
- Xác định IOC.
- Phát triển detection rule.
- Kiểm thử threat hunting hypothesis.
- Thu thập telemetry để phân tích hành vi malware.
- Thiết kế incident response playbook.

Red Team có thể:

- Test payload.
- Quan sát log.
- Quan sát process behavior.
- Kiểm tra telemetry.
- Điều chỉnh kỹ thuật để giảm khả năng bị phát hiện.

Purple Team có thể:

- Mô phỏng đầy đủ attack-detect-response cycle.
- Dùng target như nền tảng học tập chung.
- Kiểm tra khả năng phòng thủ trong môi trường được kiểm soát.

---

# 7. Available Windows DFIR Toolset

Phần này tổng hợp các công cụ DFIR được cài sẵn trên **Windows Purple Module Targets**.

Các công cụ này phục vụ phân tích sau khai thác, bao gồm:

- System monitoring.
- Log analysis.
- Threat detection.
- Traffic capture.
- Memory dumping.
- Memory forensics.
- Telemetry.
- Malware/process/PE analysis.

---

# 8. System Monitoring

| Tool | Path | Mục đích |
|---|---|---|
| Sysmon | `C:\Windows\Sysmon64.exe` | Ghi log chi tiết phục vụ detection và forensic |

## Sysmon là gì?

**Sysmon** là công cụ của Microsoft Sysinternals giúp ghi lại nhiều sự kiện quan trọng trên Windows.

Sysmon có thể ghi:

- Process creation.
- Network connection.
- File creation.
- Registry modification.
- Driver loaded.
- Image loaded.
- Process injection indicators.
- DNS query.

Sysmon rất hữu ích cho SOC vì Windows Event Log mặc định đôi khi không đủ chi tiết để điều tra attack behavior.

---

# 9. Log Analysis

| Tool | Path | Mục đích |
|---|---|---|
| Eric Zimmerman Tools | `C:\Tools\EZ-Tools` | Bộ công cụ forensic để phân tích registry hives, event logs và artifact số |

## Eric Zimmerman Tools

**Eric Zimmerman Tools** là bộ công cụ rất phổ biến trong Windows forensic.

Một số tool nổi bật:

- EvtxECmd.
- RECmd.
- MFTECmd.
- AmcacheParser.
- AppCompatCacheParser.
- PECmd.
- Timeline Explorer.

Dùng để phân tích:

- Windows Event Logs.
- Registry.
- Prefetch.
- Amcache.
- ShimCache.
- MFT.
- Timeline artifact.

---

# 10. Threat Detection & Monitoring

| Tool | Path | Mục đích |
|---|---|---|
| Yara | `C:\Tools\yara\yara.exe` | Scan file dựa trên signature |
| Chainsaw | `C:\Tools\Sigma\chainsaw\chainsaw_x86_64-pc-windows-msvc.exe` | Hunt qua Windows Event Logs |
| Sigma | `C:\Program Files\Python312\Scripts\sigma.exe` | Định dạng rule chung cho SIEM |
| Zircolite | `C:\Tools\Sigma\zircolite\zircolite_win_x64_2.20.0.exe` | Phân tích EVTX dựa trên Sigma |
| Osquery | `C:\Program Files\osquery\osqueryi.exe` | Query endpoint bằng cú pháp giống SQL |
| Velociraptor | `C:\Program Files\Velociraptor` | Endpoint monitoring, collection và response |

---

## Yara

**Yara** là công cụ dùng để nhận diện file hoặc malware dựa trên rule/signature.

Yara thường dùng để:

- Tìm malware theo chuỗi đặc trưng.
- Scan folder nghi ngờ.
- Hunt file theo pattern.
- Tạo IOC dựa trên sample đã phân tích.

---

## Chainsaw

**Chainsaw** là công cụ command-line dùng để phân tích và hunt trong Windows Event Logs.

Chainsaw thường kết hợp với Sigma rule để phát hiện hành vi đáng ngờ trong file `.evtx`.

---

## Sigma

**Sigma** là định dạng rule chung cho detection.

Có thể hiểu Sigma giống như:

```text
YARA for logs
```

Sigma rule có thể được chuyển đổi sang nhiều SIEM khác nhau như Splunk, Elastic, Sentinel, QRadar.

---

## Zircolite

**Zircolite** dùng để phân tích Windows EVTX log dựa trên Sigma rule.

Nó hữu ích khi analyst có file event log offline và muốn nhanh chóng tìm hành vi đáng ngờ.

---

## Osquery

**Osquery** cho phép query endpoint bằng cú pháp giống SQL.

Ví dụ:

```sql
SELECT name, path, pid FROM processes;
```

Dùng để kiểm tra:

- Process.
- Service.
- User.
- Network connection.
- Scheduled task.
- Installed software.

---

## Velociraptor

**Velociraptor** là nền tảng endpoint monitoring, collection và response.

Truy cập Velociraptor:

```text
https://<Target_IP>:8889
```

Thông tin đăng nhập:

```text
Username: admin
Password: P3n#31337@LOG
```

Lưu ý: sau khi spawn Windows Purple target, nên chờ ít nhất **5 phút** để các service khởi động đầy đủ, bao gồm Velociraptor.

---

# 11. Traffic Capturing

| Tool | Path | Mục đích |
|---|---|---|
| Wireshark | `C:\Program Files\Wireshark` | Bắt và phân tích gói tin mạng |

## Wireshark

**Wireshark** dùng để phân tích traffic mạng.

Có thể dùng để kiểm tra:

- DNS query.
- HTTP request.
- TLS connection.
- SMB traffic.
- C2 traffic.
- File transfer.
- Port scanning.
- Suspicious beaconing.

---

# 12. Memory Dumping

| Tool | Path | Mục đích |
|---|---|---|
| DumpIt | `C:\Tools\Memory-Dump\DumpIt.exe` | Tạo memory dump phục vụ memory forensic |
| WinPmem | `C:\Tools\Memory-Dump\winpmem_mini_x64_rc2.exe` | Tạo memory dump phục vụ memory forensic |

## Memory Dump là gì?

**Memory dump** là bản chụp nội dung RAM tại một thời điểm.

Memory dump có thể chứa:

- Process đang chạy.
- Malware chạy trong memory.
- Network connection.
- Credential.
- Command history.
- Injected code.
- Encryption key.
- Token/session.

---

# 13. Memory Forensics

| Tool | Path | Mục đích |
|---|---|---|
| Volatility v2 | `C:\Tools\Volatility2` | Phân tích memory dump |
| Volatility v3 | `C:\Tools\volatility3` | Phân tích memory dump |

## Volatility

**Volatility** là công cụ memory forensic phổ biến.

Dùng để phân tích:

- Process list.
- Network connection.
- DLL loaded.
- Command line.
- Registry in memory.
- Malware injection.
- Hidden process.
- Credential artifact.

---

# 14. Additional Telemetry

| Tool | Path | Mục đích |
|---|---|---|
| SilkETW | `C:\Tools\SilkETW\SilkETW\SilkETW.exe` | C# wrapper cho ETW |
| SealighterTI | `C:\Tools\SealighterTI.exe` | Chạy Microsoft-Windows-Threat-Intelligence không cần driver |
| AMSI-Monitoring-Script | `C:\Tools\AMSIScript\AMSIScriptContentRetrieval.ps1` | Trích xuất nội dung script qua AMSI ETW provider |
| JonMon | `C:\Tools\JonMon` | Bộ sensor telemetry mã nguồn mở |
| Fibratus | `C:\Program Files\Fibratus` | Detection theo behavior và YARA memory scanner |

Lưu ý: một số tool như **Fibratus** có thể được thêm sau khi module phát hành và không có sẵn trong target của module này.

---

## ETW là gì?

**ETW** là viết tắt của **Event Tracing for Windows**.

ETW là cơ chế ghi nhận telemetry sâu trong Windows.

Nó có thể cung cấp thông tin về:

- Process.
- Thread.
- File.
- Registry.
- Network.
- PowerShell.
- AMSI.
- Threat intelligence events.

---

## AMSI Monitoring Script

AMSI Monitoring Script giúp trích xuất nội dung script thông qua AMSI ETW provider.

Dùng để quan sát script trước khi nó được thực thi, đặc biệt hữu ích khi phân tích:

- PowerShell.
- Script obfuscation.
- Malicious macro.
- Script-based payload.

---

# 15. Adversary Simulation

| Tool | Path | Mục đích |
|---|---|---|
| Atomic Red Team | `C:\AtomicRedTeam` | Bộ test mô phỏng kỹ thuật MITRE ATT&CK |

Lưu ý: Atomic Red Team được thêm sau khi module phát hành và có thể không có sẵn trong target của module này.

## Atomic Red Team

**Atomic Red Team** cung cấp các test nhỏ để mô phỏng kỹ thuật MITRE ATT&CK.

Ví dụ:

- Mô phỏng PowerShell execution.
- Mô phỏng credential dumping.
- Mô phỏng persistence.
- Mô phỏng lateral movement.

Dùng để kiểm tra detection rule và khả năng log/alert.

---

# 16. Malware, Process và PE Analysis

| Tool | Path | Mục đích |
|---|---|---|
| CFF Explorer | `C:\Tools\CFF-Explorer\CFF Explorer.exe` | Xem và chỉnh sửa PE file |
| Ghidra | `C:\Tools\Ghidra\ghidraRun.bat` | Reverse engineering framework |
| x64dbg | `C:\Tools\x64dbg` | Debugger x64/x32 cho Windows |
| SpeakEasy | `C:\Tools\speakeasy` | Emulator cho Windows malware |
| SysInternalsSuite | `C:\Tools\SysinternalsSuite` | Bộ công cụ troubleshooting của Sysinternals |
| Get-InjectedThread | `C:\Tools\Get-InjectedThread.ps1` | Tìm thread được tạo do code injection |
| Hollows Hunter | `C:\Tools\hollows_hunter64.exe` | Scan process để phát hiện implant |
| Moneta | `C:\Tools\Moneta64.exe` | Live usermode memory analysis |
| PE-Sieve | `C:\Tools\pe-sieve64.exe` | Phát hiện malware trong process và dump material đáng ngờ |
| API Monitor | `C:\Tools\API Monitor` | Theo dõi API call |
| PE-Bear | `C:\Tools\PE-bear` | Reversing tool cho PE file |
| Process Hacker | `C:\Tools\ProcessHacker` | Theo dõi process, debug và phát hiện malware |
| ProcMonX | `C:\Tools\ProcMonX.exe` | Process Monitor mở rộng dựa trên ETW |
| Frida | `C:\Program Files\Python312\Scripts\frida.exe` | Dynamic instrumentation toolkit |
| LitterBox | `C:\Tools\LitterBox\litterbox.py` | Malware sandbox để test payload behavior |

Lưu ý: một số tool như **Frida** và **LitterBox** có thể được thêm sau khi module phát hành và không có sẵn trong target của module này.

---

## PE file là gì?

**PE** là viết tắt của **Portable Executable**.

Đây là định dạng file thực thi trên Windows, ví dụ:

- `.exe`
- `.dll`
- `.sys`

Công cụ như CFF Explorer, PE-Bear, Ghidra, x64dbg dùng để phân tích PE file.

---

## Process Injection là gì?

**Process Injection** là kỹ thuật attacker dùng để đưa code độc hại vào process hợp lệ.

Mục tiêu:

- Né detection.
- Chạy malware dưới process tin cậy.
- Ẩn hành vi độc hại.
- Bypass một số security control.

Các tool như Get-InjectedThread, PE-Sieve, Moneta, Hollows Hunter có thể hỗ trợ phát hiện dấu hiệu injection.

---

# 17. Workflow thực hành Purple Module

Một workflow đơn giản khi thực hành:

```text
Spawn Target
      ↓
Chờ service khởi động đầy đủ
      ↓
Thực hiện phần tấn công
      ↓
Giữ target hoạt động
      ↓
Thu thập log/network/memory
      ↓
Phân tích bằng DFIR tools
      ↓
Xác định IOC/artifact
      ↓
Map MITRE ATT&CK nếu cần
      ↓
Viết detection rule
      ↓
Kiểm thử lại detection
```

---

# 18. Bảng ôn nhanh thuật ngữ

| Thuật ngữ | Nghĩa ngắn gọn |
|---|---|
| Purple Team | Kết hợp Red Team và Blue Team |
| Purple Module | Mô-đun học cả tấn công và phòng thủ |
| DFIR | Digital Forensics and Incident Response |
| Artifact | Dấu vết hoặc bằng chứng để lại trên hệ thống |
| Telemetry | Dữ liệu quan sát từ endpoint/network |
| IOC | Indicator of Compromise |
| Sysmon | Công cụ ghi log chi tiết trên Windows |
| Yara | Công cụ scan file theo rule/signature |
| Sigma | Định dạng rule phát hiện cho log/SIEM |
| Chainsaw | Công cụ hunt Windows Event Logs |
| Zircolite | Phân tích EVTX bằng Sigma |
| Osquery | Query endpoint bằng SQL-like syntax |
| Velociraptor | Endpoint monitoring, collection và response |
| Wireshark | Công cụ bắt và phân tích gói tin |
| Memory Dump | Bản chụp RAM |
| Volatility | Công cụ phân tích memory dump |
| ETW | Event Tracing for Windows |
| AMSI | Antimalware Scan Interface |
| PE | Portable Executable |
| Process Injection | Kỹ thuật inject code vào process khác |
| Atomic Red Team | Bộ test mô phỏng kỹ thuật MITRE ATT&CK |

---

# 19. Câu hỏi ôn tập

## 1. Vì sao forensic analysis cần phần tấn công xảy ra trước?

Vì tấn công tạo ra log, traffic, memory artifact và các dấu vết cần thiết để phân tích forensic.

## 2. Vì sao target cần duy trì hoạt động sau khi attack?

Vì nhiều bằng chứng như memory artifact, process, network connection hoặc service telemetry có thể mất nếu target bị tắt quá sớm.

## 3. Purple Module có lợi gì cho Blue Team?

Blue Team có thể học cách attack tạo artifact, phát triển detection rule, kiểm thử threat hunting hypothesis và xây dựng incident response playbook.

## 4. Purple Module có lợi gì cho Red Team?

Red Team có thể quan sát dấu vết mà payload hoặc kỹ thuật của mình để lại, từ đó cải thiện OPSEC và giảm khả năng bị phát hiện.

## 5. Velociraptor dùng để làm gì?

Velociraptor dùng cho endpoint monitoring, collection và response. Nó giúp thu thập artifact, điều tra endpoint và phản ứng trong quá trình DFIR.

---

# Key Takeaway

HTB Academy Purple Modules giúp người học kết nối giữa tấn công và phòng thủ.

Thay vì chỉ học cách khai thác, người học còn có thể phân tích các dấu vết mà cuộc tấn công để lại, từ đó hiểu rõ hơn về detection, forensic và incident response.

Đối với Blue Team, đây là môi trường tốt để luyện điều tra và viết detection.

Đối với Red Team, đây là môi trường tốt để hiểu tool và payload của mình bị phát hiện như thế nào.

Đối với Purple Team, đây là nền tảng giúp hai bên phối hợp để cải thiện năng lực phòng thủ thực tế.

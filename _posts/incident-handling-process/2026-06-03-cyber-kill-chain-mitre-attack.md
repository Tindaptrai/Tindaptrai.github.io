---
layout: post
title: "Cyber Kill Chain, MITRE ATT&CK và Pyramid of Pain"
date: 2026-06-03 00:00:00 +0700
categories: [SOC, Threat Analysis]
tags: [soc, mitre-attack, cyber-kill-chain, pyramid-of-pain, thehive, blue-team]
description: "Ghi chú tổng hợp về Cyber Kill Chain, MITRE ATT&CK, Pyramid of Pain và cách áp dụng trong phân tích sự cố SOC."
toc: true
---

# Cyber Kill Chain, MITRE ATT&CK và Pyramid of Pain

Bài viết này tổng hợp các khái niệm quan trọng trong phân tích tấn công mạng và vận hành SOC, bao gồm:

- **Cyber Kill Chain**
- **MITRE ATT&CK Framework**
- **Pyramid of Pain**
- **MITRE ATT&CK Mapping trong TheHive**

Mục tiêu chính là hiểu được **vòng đời tấn công**, xác định attacker đang ở giai đoạn nào, đánh giá mức độ ảnh hưởng, và lựa chọn hành động phát hiện/ngăn chặn phù hợp.

---

## 1. Cyber Kill Chain là gì?

**Cyber Kill Chain** là mô hình mô tả vòng đời của một cuộc tấn công mạng, từ lúc attacker bắt đầu thu thập thông tin cho đến khi đạt được mục tiêu cuối cùng.

Mô hình này giúp SOC analyst trả lời các câu hỏi quan trọng:

- Attacker đang ở giai đoạn nào?
- Attacker đã vào được hệ thống chưa?
- Attacker đã có persistence chưa?
- Attacker đã thiết lập Command and Control chưa?
- Attacker đã lateral movement chưa?
- Mục tiêu cuối cùng của attacker là gì?

Mục tiêu của defender là:

```text
Ngăn attacker tiến xa hơn trong kill chain,
lý tưởng nhất là phát hiện và chặn ở các giai đoạn sớm.
```

---

## 2. Các giai đoạn của Cyber Kill Chain

Cyber Kill Chain thường gồm 7 giai đoạn:

```text
Reconnaissance
   ↓
Weaponization
   ↓
Delivery
   ↓
Exploitation
   ↓
Installation
   ↓
Command and Control
   ↓
Actions on Objectives
```

---

## 3. Reconnaissance

**Reconnaissance** là giai đoạn attacker lựa chọn mục tiêu và thu thập thông tin.

Thông tin được thu thập ở giai đoạn này có thể được sử dụng cho nhiều bước sau trong kill chain, ví dụ như tạo payload phù hợp, chọn phương thức phishing, xác định công nghệ đang dùng hoặc tìm điểm yếu public-facing.

### Passive Recon

**Passive Recon** là thu thập thông tin mà không tương tác trực tiếp với hệ thống mục tiêu.

Ví dụ:

- Tìm thông tin từ LinkedIn.
- Xem Instagram, Facebook hoặc mạng xã hội.
- Đọc website công ty.
- Đọc tài liệu public.
- Xem job description.
- Tìm thông tin đối tác, vendor.
- Tìm domain, email, subdomain.
- Xác định công nghệ công ty đang dùng.

Thông tin từ job ads hoặc tài liệu public có thể tiết lộ:

- Antivirus/EDR đang sử dụng.
- Hệ điều hành.
- Cloud provider.
- Network technology.
- Framework backend/frontend.
- Công cụ quản trị nội bộ.

### Active Recon

**Active Recon** là thu thập thông tin bằng cách tương tác trực tiếp với hạ tầng mục tiêu.

Ví dụ:

- Scan IP public.
- Scan port.
- Fingerprint service.
- Enumerate subdomain.
- Kiểm tra web application.
- Map network perimeter.

Các hoạt động active recon dễ bị phát hiện hơn passive recon vì tạo ra log trên firewall, IDS/IPS, WAF hoặc web server.

---

## 4. Weaponization

**Weaponization** là giai đoạn attacker chuẩn bị payload, exploit hoặc malware để sử dụng cho initial access.

Ví dụ:

- Tạo malicious document.
- Tạo payload nhẹ để tránh antivirus.
- Đóng gói malware vào script hoặc executable.
- Chuẩn bị dropper.
- Chuẩn bị exploit cho lỗ hổng đã tìm thấy.
- Tùy chỉnh payload dựa trên thông tin về antivirus/EDR của mục tiêu.

Mục tiêu thường là tạo một **initial stager** có khả năng:

- Remote access.
- Persistence sau reboot.
- Tải thêm tool.
- Thực thi lệnh theo yêu cầu.
- Tránh detection.

---

## 5. Delivery

**Delivery** là giai đoạn attacker đưa payload đến nạn nhân.

Các phương pháp phổ biến:

- Phishing email.
- Email có file đính kèm độc hại.
- Link đến website độc hại.
- Website giả mạo login page.
- Social engineering qua điện thoại.
- USB drop.
- File script như `.bat`, `.cmd`, `.vbs`, `.js`, `.hta`.

Ví dụ luồng delivery:

```text
Victim nhận email phishing
        ↓
Victim click link hoặc mở attachment
        ↓
Payload được tải xuống hoặc thực thi
```

Ở giai đoạn này, defender có thể phát hiện qua:

- Email gateway.
- Proxy logs.
- DNS logs.
- Web filtering.
- User behavior analytics.
- Endpoint detection.

---

## 6. Exploitation

**Exploitation** là thời điểm exploit hoặc payload được kích hoạt.

Mục tiêu của attacker:

- Thực thi code trên máy nạn nhân.
- Chiếm quyền điều khiển ứng dụng.
- Khai thác lỗ hổng.
- Bypass security control.
- Mở đường cho payload chạy.

Ví dụ:

```text
Victim mở file độc hại
        ↓
Macro/script chạy
        ↓
PowerShell tải payload
        ↓
Attacker có initial access
```

Các dấu hiệu có thể phát hiện:

- Process bất thường.
- PowerShell encoded command.
- Office spawn child process.
- Script interpreter chạy bất thường.
- Exploit public-facing application.
- Suspicious parent-child process relationship.

---

## 7. Installation

**Installation** là giai đoạn attacker cài đặt malware, backdoor hoặc persistence trên hệ thống đã bị compromise.

Một số kỹ thuật phổ biến gồm:

### Droppers

Dropper là chương trình nhỏ dùng để tải hoặc cài malware vào hệ thống.

```text
Initial payload
    ↓
Dropper
    ↓
Malware / Backdoor
```

Dropper có thể được gửi qua email, website độc hại hoặc social engineering.

### Backdoors

Backdoor cho phép attacker truy cập lại hệ thống sau này.

Đặc điểm:

- Duy trì truy cập.
- Cho phép chạy lệnh từ xa.
- Có thể tải thêm module.
- Có thể kết nối đến C2 server.

### Rootkits

Rootkit được dùng để che giấu sự hiện diện của attacker.

Mục tiêu:

- Ẩn process.
- Ẩn file.
- Ẩn registry key.
- Tránh antivirus/EDR.
- Duy trì stealth.

---

## 8. Command and Control

**Command and Control**, hay **C2**, là giai đoạn attacker thiết lập kênh điều khiển từ xa đến máy bị compromise.

C2 có thể dùng:

- HTTP/HTTPS.
- DNS tunneling.
- TCP custom protocol.
- Cloud service.
- Social platform.
- Legitimate web service.

Ví dụ:

```text
Compromised Host
        ↓
Outbound HTTPS
        ↓
Attacker C2 Server
        ↓
Command Execution / Data Transfer
```

Advanced attacker thường không chỉ dùng một malware duy nhất. Họ có thể triển khai nhiều biến thể hoặc nhiều kênh truy cập để nếu một kênh bị phát hiện, họ vẫn có thể quay lại môi trường.

---

## 9. Actions on Objectives

Đây là giai đoạn attacker thực hiện mục tiêu cuối cùng.

Mục tiêu có thể là:

- Data exfiltration.
- Ransomware deployment.
- Credential theft.
- Destructive attack.
- Espionage.
- Financial fraud.
- Domain takeover.
- Lateral movement sâu hơn.
- Persistence dài hạn.

Ví dụ ransomware:

```text
Privilege Escalation
        ↓
Credential Dumping
        ↓
Lateral Movement
        ↓
Disable Security Tools
        ↓
Encrypt Data
        ↓
Demand Ransom
```

---

## 10. Kill Chain không phải lúc nào cũng tuyến tính

Cyber Kill Chain trình bày theo thứ tự tuyến tính, nhưng thực tế attacker thường lặp lại nhiều giai đoạn.

Ví dụ:

```text
Initial Compromise
        ↓
Internal Recon
        ↓
Lateral Movement
        ↓
More Recon
        ↓
Privilege Escalation
        ↓
More Lateral Movement
        ↓
Objective
```

Sau khi compromise được một máy, attacker thường quay lại giai đoạn **Reconnaissance** trong mạng nội bộ để tìm thêm mục tiêu.

---

# MITRE ATT&CK Framework

## 1. MITRE ATT&CK là gì?

**MITRE ATT&CK** là framework mô tả hành vi attacker theo dạng ma trận.

Nếu Cyber Kill Chain cho ta cái nhìn tổng quan về vòng đời tấn công, thì MITRE ATT&CK cho ta cái nhìn chi tiết hơn về:

- Tactic.
- Technique.
- Sub-technique.
- Behavior.
- Detection.
- Mitigation.

MITRE ATT&CK giúp SOC analyst map hành vi thực tế vào kỹ thuật cụ thể.

---

## 2. Tactic

**Tactic** là mục tiêu cấp cao của attacker tại một giai đoạn trong cuộc tấn công.

Ví dụ tactic:

- Initial Access.
- Execution.
- Persistence.
- Privilege Escalation.
- Defense Evasion.
- Credential Access.
- Discovery.
- Lateral Movement.
- Command and Control.
- Exfiltration.
- Impact.

Ví dụ:

```text
Tactic = Credential Access
Mục tiêu = lấy credential từ hệ thống
```

---

## 3. Technique

**Technique** là cách cụ thể attacker dùng để đạt được tactic.

Ví dụ:

| Tactic | Technique | ID |
|---|---|---|
| Execution | Command and Scripting Interpreter: PowerShell | T1059.001 |
| Credential Access | OS Credential Dumping | T1003 |
| Lateral Movement | Remote Services | T1021 |
| Command and Control | Ingress Tool Transfer | T1105 |

Ví dụ:

```text
T1059.001 = PowerShell được dùng để chạy lệnh hoặc tải payload.
T1021 = Attacker dùng SSH/RDP/SMB để lateral movement.
```

---

## 4. Sub-technique

**Sub-technique** là phiên bản chi tiết hơn của technique.

Ví dụ:

| Technique | Sub-technique | ID |
|---|---|---|
| OS Credential Dumping | LSASS Memory | T1003.001 |
| Remote Services | SMB/Windows Admin Shares | T1021.002 |
| Remote Services | Remote Desktop Protocol | T1021.001 |

Sub-technique giúp báo cáo chính xác hơn.

Thay vì nói:

```text
Credential Dumping
```

Có thể nói rõ:

```text
T1003.001 - OS Credential Dumping: LSASS Memory
```

Điều này giúp detection, reporting và incident analysis chính xác hơn.

---

# Cyber Kill Chain vs MITRE ATT&CK

| Tiêu chí | Cyber Kill Chain | MITRE ATT&CK |
|---|---|---|
| Mức độ | Tổng quan | Chi tiết |
| Mục tiêu | Hiểu vòng đời tấn công | Hiểu hành vi attacker |
| Cấu trúc | 7 giai đoạn | Matrix tactic/technique |
| Phù hợp cho | Incident overview, executive summary | Detection engineering, SOC analysis |
| Ví dụ | Delivery, Exploitation, C2 | T1059.001, T1003.001, T1021.001 |

Cách dùng tốt nhất là kết hợp cả hai:

```text
Cyber Kill Chain = attacker đang ở đâu trong vòng đời tấn công.
MITRE ATT&CK = attacker đang dùng kỹ thuật gì.
```

---

# Pyramid of Pain

## 1. Pyramid of Pain là gì?

**Pyramid of Pain** mô tả mức độ khó khăn của attacker khi defender phát hiện và chặn các loại indicator khác nhau.

Càng lên cao, attacker càng khó thay đổi hành vi.

```text
TTPs
Tools
Network/Host Artifacts
Domain Names
IP Addresses
Hash Values
```

---

## 2. Các tầng trong Pyramid of Pain

| Indicator | Mức độ gây khó cho attacker | Giải thích |
|---|---|---|
| Hash Values | Trivial | Attacker chỉ cần đổi file một chút là hash thay đổi. |
| IP Addresses | Easy | Attacker có thể đổi C2 IP nhanh. |
| Domain Names | Simple | Attacker có thể đăng ký domain mới. |
| Network/Host Artifacts | Annoying | Cần thay đổi file path, registry key, mutex, user-agent. |
| Tools | Challenging | Phải đổi hoặc viết lại tool. |
| TTPs | Tough | Phải thay đổi cách hoạt động. Đây là tầng gây đau nhất. |

---

## 3. Ví dụ

Nếu chỉ block IP C2:

```text
Block malicious IP
        ↓
Attacker đổi IP khác
        ↓
Tiếp tục hoạt động
```

Nếu detect theo TTP:

```text
Detect PowerShell abuse
Detect LSASS dumping
Detect suspicious RDP lateral movement
Detect process injection
        ↓
Attacker phải đổi cách hoạt động
        ↓
Chi phí tấn công tăng mạnh
```

Vì vậy:

```text
Hash/IP detection = dễ né.
Behavior/TTP detection = khó né hơn.
```

---

# Áp dụng MITRE ATT&CK trong SOC

SOC analyst có thể dùng MITRE ATT&CK để:

- Map alert vào tactic/technique.
- Hiểu mục tiêu của attacker.
- Dự đoán bước tiếp theo.
- Ưu tiên alert quan trọng.
- Viết detection rule.
- Viết báo cáo incident.
- Đề xuất containment và eradication.

Ví dụ:

```text
Alert: LSASS memory dump
MITRE: T1003.001
Tactic: Credential Access
Risk: Attacker có thể lấy credential và lateral movement
Response: isolate host, collect memory, reset credentials, hunt related activity
```

---

# MITRE ATT&CK Mapping trong TheHive

## 1. TheHive là gì?

**TheHive** là nền tảng case management dùng trong SOC để quản lý alert và incident.

TheHive giúp:

- Tập trung alert từ nhiều nguồn.
- Tạo case điều tra.
- Gắn nhiều alert liên quan vào cùng một case.
- Quản lý observables.
- Theo dõi trạng thái xử lý.
- Mapping alert với MITRE ATT&CK TTPs.

---

## 2. Vì sao tích hợp MITRE ATT&CK vào TheHive?

Khi alert được map với MITRE ATT&CK, analyst có thể hiểu rõ hơn:

- Alert thuộc tactic nào.
- Technique cụ thể là gì.
- Attacker đang cố đạt mục tiêu gì.
- Cần containment bước nào.
- Có cần hunt thêm hành vi liên quan không.

Ví dụ:

```text
Alert: Possible suspicious access to Windows admin shares
Tactic: Lateral Movement
Technique: Remote Services: SMB/Windows Admin Shares
ID: T1021.002
```

Điều này giúp incident analysis có cấu trúc hơn.

---

# Example MITRE ATT&CK Mapping

| Tactic | Technique | ID | Description |
|---|---|---|---|
| Initial Access | Exploit Public-Facing Application | T1190 | Confluence CVE exploited |
| Execution | Command and Scripting Interpreter: PowerShell | T1059.001 | PowerShell used for payload download |
| Persistence | Windows Service | T1543.003 | Windows Service for persistence |
| Credential Access | LSASS Memory Dumping | T1003.001 | Extracted credentials |
| Lateral Movement | Remote Desktop Protocol | T1021.001 | RDP lateral movement |
| Impact | Data Encrypted for Impact | T1486 | Ransomware encryption |

---

# Practical SOC Workflow

Một workflow phân tích SOC có thể đi như sau:

```text
Alert Received
      ↓
Validate Alert
      ↓
Identify Asset/User
      ↓
Map to MITRE ATT&CK
      ↓
Determine Kill Chain Stage
      ↓
Assess Impact
      ↓
Contain
      ↓
Hunt Related Activity
      ↓
Document Case
      ↓
Lessons Learned
```

---

## Ví dụ phân tích alert

```text
Alert:
PowerShell encoded command detected

Observed:
powershell.exe -enc SQBFAFgA...

MITRE:
T1059.001 - Command and Scripting Interpreter: PowerShell

Kill Chain:
Exploitation / Installation

Risk:
Payload download hoặc execution

Response:
- Kiểm tra parent process.
- Kiểm tra command line.
- Kiểm tra network connection.
- Kiểm tra file được tạo.
- Isolate host nếu confirmed malicious.
- Hunt hash, domain, IP, command pattern trên toàn hệ thống.
```

---

# Quick Reference

```text
Cyber Kill Chain
= Mô hình vòng đời tấn công.

MITRE ATT&CK
= Ma trận tactic, technique, sub-technique mô tả hành vi attacker.

Tactic
= Mục tiêu cấp cao của attacker.

Technique
= Cách attacker đạt được tactic.

Sub-technique
= Triển khai chi tiết của technique.

Pyramid of Pain
= Mô hình cho thấy indicator nào gây khó cho attacker khi bị phát hiện/chặn.

TheHive
= Case management platform dùng để quản lý alert và incident trong SOC.
```

---

# Key Takeaway

Cyber Kill Chain giúp analyst hiểu **attacker đang ở giai đoạn nào trong cuộc tấn công**.

MITRE ATT&CK giúp analyst hiểu **attacker đang dùng kỹ thuật gì**.

Pyramid of Pain giúp defender ưu tiên detection theo hướng **hành vi và TTP**, thay vì chỉ dựa vào hash, IP hoặc domain.

Khi kết hợp ba mô hình này trong SOC, analyst có thể điều tra incident tốt hơn, viết báo cáo rõ hơn, và đưa ra hành động containment chính xác hơn.

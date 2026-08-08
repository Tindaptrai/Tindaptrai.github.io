---
layout: post
title: "Real-world Incident Report"
date: 2026-08-08 22:02:00 +0700
categories: ["Security Incident Reporting", "Case Study"]
tags: [incident-report, case-study, soc, dfir, elastic-siem, powershell, metasploit, buffer-overflow, c2, ioc, root-cause-analysis, timeline, containment, eradication, recovery, lessons-learned, cdsa]
description: "Case study về một incident report hoàn chỉnh: executive summary, technical analysis, IoC, root cause, timeline, response, recovery và lessons learned."
toc: true
---

# Real-world Incident Report

## Công thức dễ nhớ

```text
DETECT → SCOPE → PROVE → CONTAIN → ERADICATE → RECOVER → IMPROVE
```

Case study này mô tả một incident tại **SampleCorp**, nơi attacker có mặt trong mạng nội bộ, compromise hai hệ thống và sử dụng malicious PDF, PowerShell, buffer overflow và reverse connection.

# 1. Executive Summary

```text
Incident ID:  INC2019-0422-022
Severity:     High (P2)
Status:       Resolved
```

Hệ thống bị compromise:

```text
WKST01.samplecorp.com
→ Development workstation
→ Proprietary source code
→ Third-party API keys

HR01.samplecorp.com
→ HR system
→ Employee/partner data
→ PII, payroll, performance reviews
```

Detection ban đầu dựa trên:

```text
Abnormal parent-child process relationships
Suspicious PowerShell commands
```

SOC và DFIR sau đó thực hiện:

```text
Containment
Evidence collection
Malware removal
Patching
System restoration
```

# 2. Key Findings

Các nguyên nhân/chất xúc tác chính trong source:

```text
Insufficient network access controls
Outdated Acrobat Reader
Buffer overflow in proprietary HR application
Weak network segmentation
Insufficient phishing-awareness training
```

Attacker có thể nhận internal IP chỉ bằng cách kết nối máy vào Ethernet port trong văn phòng.

Initial compromise được liên hệ với:

```text
Mozilla Thunderbird
→ cv.pdf
→ Adobe Reader 10.0
→ malicious command execution
```

Lateral movement tới `HR01`:

```text
Buffer overflow
→ TCP 31337
→ shellcode
→ callback 192.168.220.66:4444
```

# 3. Immediate Actions

```text
VLAN segmentation
Host isolation
Traffic capture collection
Host security tooling
Elastic SIEM log collection
Firewall blocking
Credential reset
API key revocation
```

Không có external incident response provider tham gia.

# 4. Stakeholder Impact

## Customers

```text
Potential data exposure
Temporary downtime
API key revocation
Possible revenue/trust impact
```

## Employees

`HR01` chứa:

```text
Personal identification information
Payroll data
Performance reviews
Social Security numbers
Bank account details
```

## Business Partners

`WKST01` chứa:

```text
Proprietary source code
Upcoming software releases
Third-party API keys
```

## Regulatory / Internal / Shareholders

```text
Compliance exposure
Possible fines
Security budget reallocation
Reputational damage
Potential short-term shareholder impact
```

# 5. Technical Analysis — WKST01

Các artifact/process quan trọng:

```text
Mozilla Thunderbird
cv.pdf
Adobe Reader 10.0
cmd.exe
powershell.exe
wmiprvse.exe
```

PowerShell tải nội dung từ:

```text
192.168.220.66
```

Process relationship đáng chú ý:

```text
wmiprvse.exe
→ cmd.exe
→ powershell.exe
```

Command artifacts:

```text
cmd.exe /Q /c cd
cmd.exe /Q /c dir
whoami
powershell.exe -nop -w hidden
IEX
New-Object Net.WebClient
DownloadString(...)
Get-ModifiableService
ADMIN$
```

# 6. Network Context

| IP | Hostname |
|---|---|
| `192.168.220.20` | `DC01.samplecorp.com` |
| `192.168.220.200` | `WKST01.samplecorp.com` |
| `192.168.220.101` | `HR01.samplecorp.com` |
| `192.168.220.202` | `ENG01.samplecorp.com` |

Attacker/C2 IP:

```text
192.168.220.66
```

# 7. Technical Analysis — HR01

Network traffic cho thấy:

```text
Target: HR01.samplecorp.com
Service port: 31337
Technique: buffer overflow
```

Packet bytes chứa shellcode được export thành raw binary và phân tích bằng:

```text
scdbg
```

Shellcode cố kết nối tới:

```text
192.168.220.66:4444
```

Chuỗi:

```text
Traffic to TCP 31337
→ buffer overflow payload
→ shellcode
→ reverse connection to 192.168.220.66:4444
```

# 8. Indicators of Compromise

```text
C2 IP:
192.168.220.66

Malicious file:
cv.pdf

SHA-256:
ef59d7038cfd565fd65bae12588810d5361df938244ebad33b71882dcf683011
```

# 9. Root Cause Analysis

```text
Insufficient network access controls
Outdated Acrobat Reader
Buffer overflow in proprietary application
Inadequate network segregation
Insufficient phishing-awareness training
```

# 10. Technical Timeline

```text
2019-04-22 00:27:27
cv.pdf opened on WKST01
→ outdated Acrobat Reader exploited
→ malicious payload executed

2019-04-22 00:35:09
Attacker accessed WKST01 directories
→ source code
→ API keys

2019-04-22 00:50:18
Buffer overflow discovered/exploited on HR01

2019-04-22 01:30:12
Source states attacker accessed unencrypted HR database,
compressed data and exfiltrated it through an SSH tunnel

2019-04-22 02:30:11
WKST01 and HR01 isolated with VLAN segmentation

2019-04-22 03:10:14
Host security solution connected for evidence collection

2019-04-22 03:43:34
Firewall blocked known C2 IP

2019-04-22 04:11:00
Malware removed

2019-04-22 04:30:00
Acrobat Reader updated

2019-04-22 05:01:08
API keys revoked

2019-04-22 05:05:08
Affected credentials reset

2019-04-22 05:21:20
WKST01 restored from verified backup

2019-04-22 05:58:50
HR01 restored from verified backup

2019-04-22 06:33:44
Emergency patch deployed for HR buffer overflow
```

# 11. Nature of the Attack

SOC phân tích PowerShell và nhận thấy:

```text
Double encoding
In-memory PowerShell execution
Metasploit-related code
```

Shellcode được export thành:

```text
a.bin
```

Sau đó được kiểm tra qua VirusTotal.

Các keyword được source liên hệ với Metasploit:

```text
metacoder
shikata
```

Tool attribution nên dựa trên nhiều artifact, không chỉ một indicator.

# 12. Impact Analysis

Cần đánh giá:

```text
Confidentiality
Integrity
Availability
Customer impact
Employee-data exposure
IP exposure
Downtime
Financial loss
Regulatory exposure
Reputation
```

# 13. Response & Recovery

## Access Revocation

```text
Firewall block
Forced logoff
Credential resets
API key revocation
```

## Short-Term Containment

```text
VLAN segmentation
Host isolation
Firewall blocking
```

## Long-Term Containment

```text
Stronger network segmentation
Network Access Control
Authorized-device enforcement
```

## Eradication

```text
Malware removal
Secondary scan
Heuristic analysis
Acrobat Reader update
Emergency application patch
```

## Recovery

```text
Backup checksum validation
Restore from validated backups
SHA-256 integrity checks
Firewall/IDS update
Operational and load testing
```

# 14. Post-Incident Actions

Enhanced monitoring:

```text
Behavioral analytics
Baseline deviation detection
Asset inventory
Network access control preparation
Elastic SIEM correlation rules
```

Lessons learned:

```text
Network access control gap
Email filtering gap
Network segmentation gap
Security awareness gap
Patch-management gap
```

Recommendations:

```text
Inventory and asset management
Improved email filtering
Security awareness training
Granular network controls
Network segmentation
Zero Trust
Improved SIEM correlation
```

# 15. Source Consistency Checks

Case study có một số điểm cần reconcile trước khi phát hành report thực tế.

## Exfiltration

Executive Summary nói:

```text
No widespread data exfiltration was detected.
```

Nhưng Technical Timeline ghi:

```text
Data was compressed and exfiltrated via SSH tunnel.
```

Report thực tế phải làm rõ liệu đây là:

```text
No exfiltration
Confirmed limited exfiltration
No widespread exfiltration but specific HR data exfiltration
```

## Detection Time

Source có cả:

```text
01:05:00
→ suspicious activity detected/identified

02:30:11
→ timeline says unauthorized activities detected and hosts isolated
```

Nên tách rõ:

```text
Initial detection time
Incident declaration time
Containment start time
```

# 16. SOC/CDSA Mapping

## Security Operations & Monitoring

```text
Elastic SIEM
Process tree
PowerShell telemetry
Network traffic
IoC correlation
```

## Incident Response & Forensics

```text
Packet capture
Evidence preservation
Timeline reconstruction
Containment
Eradication
Recovery
```

## Malware Analysis

```text
Shellcode
scdbg
PowerShell decoding
Metasploit indicators
VirusTotal enrichment
```

## Threat Hunting

```text
192.168.220.66
cv.pdf hash
wmiprvse.exe → cmd.exe → powershell.exe
IEX / Net.WebClient
TCP 31337
TCP 4444
```

## Detection Engineering

```text
Parent-child process anomalies
Encoded PowerShell
Internal C2
Network callback
Behavioral baselining
Elastic correlation rules
```

# Keyword Cần Nhớ

```text
INC2019-0422-022
P2 High
SampleCorp
WKST01.samplecorp.com
HR01.samplecorp.com
192.168.220.66
cv.pdf
Elastic SIEM
Mozilla Thunderbird
Adobe Reader 10.0
cmd.exe
powershell.exe
wmiprvse.exe
IEX
Net.WebClient
DownloadString
ADMIN$
buffer overflow
31337
4444
scdbg
Metasploit
metacoder
shikata
VLAN segmentation
API key revocation
SHA-256
Root Cause Analysis
Technical Timeline
Containment
Eradication
Recovery
Lessons Learned
```

# Key Takeaway

```text
Một incident report tốt không chỉ kể lại incident.
Nó phải chứng minh từng claim bằng evidence.
```

```text
Case này:
physical/internal access
→ malicious PDF
→ PowerShell
→ lateral movement
→ buffer overflow
→ reverse connection
→ containment
→ eradication
→ recovery
```

```text
Report chất lượng =
evidence rõ
+ timeline nhất quán
+ impact có context
+ response có timestamp
+ contradiction được reconcile
+ lessons learned có action
```

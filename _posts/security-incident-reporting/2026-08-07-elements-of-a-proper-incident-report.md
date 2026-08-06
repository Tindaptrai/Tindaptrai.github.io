---
layout: post
title: "Elements of a Proper Incident Report"
date: 2026-08-07 01:10:26 +0700
categories: ["Security Incident Reporting", "Report Structure"]
tags: [incident-report, executive-summary, technical-analysis, timeline, evidence, ioc, root-cause-analysis, impact-analysis, response, recovery, lessons-learned, soc, incident-response, cdsa]
description: "Cấu trúc một Security Incident Report hoàn chỉnh: Executive Summary, Technical Analysis, Impact, Response, Recovery, Appendices và Lessons Learned."
toc: true
---

# Elements of a Proper Incident Report

## Công thức dễ nhớ

```text
SUMMARY
→ EVIDENCE
→ TIMELINE
→ IMPACT
→ RESPONSE
→ RECOVERY
→ LESSONS
```

Một incident report tốt phải trả lời được:

```text
What happened?
When did it happen?
How did it happen?
What was affected?
What evidence proves it?
What did we do?
What remains to be done?
How do we prevent recurrence?
```

---

# 1. Executive Summary

Executive Summary là phần mở đầu dành cho:

```text
Management
Legal
Compliance
Finance
Technical leadership
Non-technical stakeholders
```

Nhiều stakeholder có thể chỉ đọc phần này, nên nội dung phải:

```text
Concise
Accurate
Business-oriented
Evidence-based
Actionable
```

## Nội dung bắt buộc

| Thành phần | Nội dung |
|---|---|
| Incident ID | Mã định danh duy nhất |
| Incident Overview | Loại incident, thời gian, duration, affected systems/data, status |
| Key Findings | Root cause, exploited vulnerability, compromised/exfiltrated data |
| Immediate Actions Taken | Isolation, access revocation, containment, external assistance |
| Stakeholder Impact | Downtime, financial impact, customer/employee data, reputation |

---

# 2. Incident Overview

Incident Overview cần mô tả ngắn gọn:

```text
Initial detection
Incident type
Estimated start time
Detection time
Incident duration
Affected systems
Affected data
Current status
```

Ví dụ status:

```text
Ongoing
Contained
Eradicated
Recovered
Resolved
Escalated
```

Ví dụ:

```text
A credential phishing campaign resulted in unauthorized access
to a Microsoft 365 account. The incident began at 09:14 UTC,
was detected at 10:02 UTC, and was contained at 10:31 UTC.
```

---

# 3. Key Findings

Key Findings phải trả lời:

```text
What was the root cause?
Which control failed?
Was a CVE exploited?
Was data accessed or exfiltrated?
Were credentials compromised?
Was persistence established?
```

Không nên viết:

```text
“The attacker hacked the system.”
```

Nên viết:

```text
“The attacker exploited CVE-XXXX-YYYY on the public web server,
then used the web service account to execute PowerShell.”
```

---

# 4. Immediate Actions Taken

Phần này ghi lại hành động khẩn cấp:

```text
Disabled compromised accounts
Reset credentials
Revoked active sessions
Isolated endpoints
Blocked malicious IP/domain/hash
Updated firewall rules
Removed malicious inbox rules
Engaged third-party IR support
```

Cần ghi rõ:

```text
Action
Time
Owner
Reason
Result
```

---

# 5. Stakeholder Impact

Phân tích tác động tới:

```text
Customers
Employees
Partners
Management
Regulators
Shareholders
Business operations
```

Các loại impact:

```text
Service downtime
Financial loss
Data exposure
Regulatory liability
Operational disruption
Reputational damage
Intellectual property risk
```

---

# 6. Technical Analysis

Đây thường là phần dài nhất của report.

Công thức:

```text
SCOPE
→ EVIDENCE
→ IOC
→ ROOT CAUSE
→ TIMELINE
→ TTP
```

---

# 7. Affected Systems & Data

Liệt kê tất cả tài sản:

```text
Confirmed compromised
Potentially affected
Investigated but clean
Not yet assessed
```

Thông tin nên có:

```text
Hostname
IP address
User
Operating system
Asset owner
Business function
Compromise status
Evidence reference
```

Nếu có data exfiltration:

```text
Data type
Estimated volume
Records affected
Storage location
Sensitivity
Encryption status
```

---

# 8. Evidence Sources & Analysis

Nguồn evidence có thể gồm:

```text
SIEM logs
EDR/XDR telemetry
Windows Event Logs
Sysmon
Firewall/proxy logs
DNS logs
Email headers
Cloud audit logs
Memory dump
Disk image
Packet capture
Malware sample
Identity logs
```

Trong CDSA có thể sử dụng:

```text
Splunk/ELK
Volatility
Autopsy
KAPE
Chainsaw
YARA
Sigma
Wireshark
Zeek
Suricata
```

## Evidence Integrity

Best practice:

```text
Hash evidence before analysis
Preserve original copy
Record acquisition time
Record analyst/custodian
Maintain chain of custody
```

Các hash thường dùng:

```text
SHA-256
SHA-1
MD5
```

SHA-256 nên là giá trị chính để kiểm tra integrity.

---

# 9. Indicators of Compromise

IoC hỗ trợ:

```text
Scoping
Threat hunting
Blocking
Partner notification
Attribution support
Detection engineering
```

Ví dụ IoC:

```text
IP address
Domain
URL
File hash
Filename
Registry key
Scheduled task
Service name
Process command line
Email sender
User agent
Mutex
```

Nên ghi:

```text
IOC value
IOC type
First seen
Last seen
Source
Confidence
Context
Disposition
```

Không nên chỉ đưa một danh sách hash/IP không có context.

---

# 10. Root Cause Analysis

Root Cause Analysis phải xác định:

```text
Underlying cause
Initial access vector
Exploited weakness
Control failure
Contributing factors
```

Ví dụ:

```text
Root cause:
Internet-facing VPN appliance was not patched for CVE-X.

Contributing factors:
No external attack-surface monitoring.
MFA was not enforced for legacy authentication.
```

Phải phân biệt:

```text
Root cause
Trigger
Symptom
Impact
```

---

# 11. Technical Timeline

Timeline là thành phần cốt lõi để hiểu chuỗi sự kiện.

## Các giai đoạn cần xem xét

```text
Reconnaissance
Initial Compromise
Execution
Persistence
C2 Communications
Enumeration
Privilege Escalation
Lateral Movement
Data Access
Exfiltration
Malware Deployment
Containment
Eradication
Recovery
```

## Field đề xuất

| Time | Host/User | Event | Evidence | ATT&CK | Analyst Note |
|---|---|---|---|---|---|

Thời gian phải thống nhất:

```text
UTC preferred
hoặc
ghi timezone rõ ràng
```

Ví dụ:

```text
2026-08-07 01:08:00 +0700
```

Không trộn nhiều timezone mà không ghi chú.

---

# 12. Nature of the Attack

Phần này mô tả:

```text
Attack type
Tactics
Techniques
Procedures
Tooling
Infrastructure
Adversary behavior
```

Có thể ánh xạ:

```text
MITRE ATT&CK tactic
MITRE ATT&CK technique
Observed evidence
Detection source
```

Ví dụ:

```text
T1059.001 — PowerShell
T1003.001 — LSASS Memory
T1021.002 — SMB/Windows Admin Shares
```

---

# 13. Impact Analysis

Impact Analysis đánh giá:

```text
Confidentiality
Integrity
Availability
```

Ngoài CIA triad, cần đánh giá:

```text
Financial impact
Operational impact
Legal impact
Regulatory impact
Reputational impact
Customer impact
Employee impact
```

Nên phân biệt:

```text
Confirmed impact
Potential impact
Worst-case impact
Unknown impact
```

---

# 14. Response & Recovery Analysis

Công thức:

```text
REVOKE
→ CONTAIN
→ ERADICATE
→ PATCH
→ RESTORE
→ VALIDATE
→ MONITOR
```

Phần này phải ghi theo trình tự thời gian và giải thích:

```text
What was done
Why it was done
Who performed it
When it was performed
Whether it succeeded
```

---

# 15. Immediate Response — Access Revocation

## Identification

Mô tả cách xác định:

```text
Compromised accounts
Compromised hosts
Active sessions
Malicious tokens
Rogue API keys
```

## Timeframe

Ghi chính xác:

```text
Detection time
Revocation time
Elapsed response time
```

## Method

Ví dụ:

```text
Disable account
Reset password
Revoke refresh tokens
Terminate VPN session
Remove permissions
Block source IP
Update firewall rule
```

## Impact

Đánh giá:

```text
Stopped further access
Prevented exfiltration
Interrupted C2
Reduced lateral movement
```

---

# 16. Containment Strategy

## Short-Term Containment

```text
Endpoint isolation
Account disablement
Network block
Domain sinkhole
Service shutdown
Temporary segmentation
```

## Long-Term Containment

```text
Network segmentation
Zero Trust controls
Privileged access redesign
Application isolation
Permanent policy changes
```

## Effectiveness

Cần đánh giá:

```text
Was lateral movement stopped?
Did the attacker maintain access?
Did containment affect business operations?
Were any assets missed?
```

---

# 17. Eradication — Malware Removal

## Identification

```text
EDR alert
YARA match
Memory analysis
Disk forensics
Process tree
Persistence artifact
```

## Removal

```text
Delete/quarantine file
Remove scheduled task
Delete malicious service
Remove registry persistence
Reimage host
Revoke credential
```

## Verification

```text
Rescan endpoint
Validate hashes
Re-run YARA
Review autoruns
Confirm no C2 traffic
Check persistence locations
```

---

# 18. Eradication — System Patching

Phải ghi:

```text
Vulnerability identifier
Affected version
Patch version
Testing process
Deployment time
Verification result
```

## Fallback Procedure

```text
Rollback plan
Backup configuration
Maintenance window
Service recovery procedure
```

---

# 19. Recovery — Data Restoration

## Backup Validation

```text
Backup date
Backup source
Malware scan
Hash verification
Recovery point
```

## Restoration Process

```text
Restore data
Decrypt if required
Rebuild system
Reapply secure configuration
```

## Data Integrity Checks

```text
Checksum comparison
Database consistency
File count comparison
Application validation
```

---

# 20. Recovery — System Validation

## Security Checks

```text
Firewall configuration
EDR health
IDS rules
Patch status
Credential reset
Persistence scan
```

## Operational Checks

```text
Application startup
Service availability
User acceptance testing
Performance validation
Business owner approval
```

Không đưa hệ thống trở lại production chỉ vì “máy đã bật lên được”.

---

# 21. Post-Incident Monitoring

Enhanced Monitoring Plan cần mô tả:

```text
What will be monitored
Which telemetry is required
How long monitoring will continue
Who owns the monitoring
What triggers re-escalation
```

Ví dụ:

```text
Monitor affected accounts for 30 days
Hunt IOC across all endpoints
Increase authentication logging
Deploy new Sigma/Splunk detection
Track recurrence of the exploited TTP
```

---

# 22. Lessons Learned

## Gap Analysis

```text
Which control failed?
Why was the alert missed?
Was telemetry available?
Was escalation delayed?
Was the runbook adequate?
```

## Recommendations

Mỗi recommendation cần:

```text
Action
Priority
Owner
Due date
Expected risk reduction
Status
```

## Future Strategy

```text
Policy changes
Architecture changes
Detection improvements
Training
Tabletop exercises
Third-party review
```

---

# 23. Diagrams

Visual aids giúp đơn giản hóa incident phức tạp.

## Incident Flowchart

```text
Initial Access
→ Execution
→ Persistence
→ C2
→ Lateral Movement
→ Exfiltration
```

## Affected Systems Map

Nên thể hiện:

```text
Network zones
Compromised hosts
Suspected hosts
Clean hosts
Critical assets
Severity
```

## Attack Vector Diagram

Thể hiện:

```text
Entry point
Defensive control bypass
Post-exploitation path
Target systems
Data flow
```

---

# 24. Appendices

Appendices lưu tài liệu bổ trợ và raw artifacts.

Có thể bao gồm:

```text
Log files
Pre/post-incident network diagrams
Disk images
Memory dumps
Packet captures
Code snippets
Incident response checklist
Communication records
Legal/regulatory documents
Glossary and acronyms
IOC table
Hash manifests
```

Appendices tăng:

```text
Traceability
Verifiability
Credibility
Technical depth
```

---

# 25. Best Practices

## Root Cause Analysis

```text
Không chỉ xử lý symptom.
Phải tìm nguyên nhân nền.
```

## Community Sharing

Chia sẻ thông tin không nhạy cảm với cộng đồng phòng thủ:

```text
TTP
Detection logic
Lessons learned
Sanitized IoCs
```

## Regular Updates

Trong incident đang diễn ra:

```text
Maintain update cadence
Record major decisions
Notify relevant stakeholders
Avoid silence during uncertainty
```

## External Review

Có thể dùng chuyên gia độc lập để:

```text
Validate findings
Challenge assumptions
Review containment
Assess legal/regulatory exposure
```

---

# 26. Report Quality Checklist

```text
[ ] Incident ID rõ ràng
[ ] Executive Summary dễ hiểu
[ ] Timeline có timezone
[ ] Scope đầy đủ
[ ] Evidence có nguồn và hash
[ ] IoC có context
[ ] Root cause được phân biệt với symptom
[ ] Impact được định lượng/định tính
[ ] Containment, eradication, recovery có thời gian
[ ] Fact và inference được tách riêng
[ ] Recommendation có owner và deadline
[ ] Appendices có thể kiểm chứng
```

---

# 27. SOC/CDSA Mapping

## Security Operations & Monitoring

```text
Alert evidence
SIEM timeline
Detection gaps
Rule tuning
```

## Incident Response & Forensics

```text
Containment
Evidence integrity
Chain of custody
Timeline reconstruction
Recovery
```

## Malware Analysis

```text
Behavior
Persistence
IoCs
YARA
Process injection
```

## Threat Hunting

```text
Scope expansion
Environment-wide IOC search
TTP hunting
Unknown affected assets
```

## Detection Engineering

```text
New Splunk/Sigma rules
False-positive analysis
Coverage validation
Post-incident monitoring
```

---

# Keyword Cần Nhớ

```text
Executive Summary
Incident ID
Incident Overview
Key Findings
Immediate Actions
Stakeholder Impact
Affected Systems
Evidence Sources
Evidence Integrity
Indicators of Compromise
Root Cause Analysis
Technical Timeline
TTP
Impact Analysis
Containment
Eradication
Recovery
Access Revocation
Malware Removal
System Patching
Data Restoration
System Validation
Enhanced Monitoring
Lessons Learned
Gap Analysis
Appendices
Chain of Custody
```

---

# Key Takeaway

```text
Executive Summary
= giúp lãnh đạo hiểu incident.
```

```text
Technical Analysis
= chứng minh incident bằng evidence.
```

```text
Response & Recovery
= ghi lại cách tổ chức kiểm soát và phục hồi.
```

```text
Lessons Learned
= biến incident thành cải tiến lâu dài.
```

```text
Incident report tốt =
clear narrative
+ verifiable evidence
+ precise timeline
+ business impact
+ actionable remediation
```

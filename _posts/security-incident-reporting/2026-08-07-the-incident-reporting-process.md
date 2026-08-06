---
layout: post
title: "The Incident Reporting Process"
date: 2026-08-07 01:13:11 +0700
categories: ["Security Incident Reporting", "Process"]
tags: [incident-reporting-process, incident-detection, preliminary-analysis, incident-logging, stakeholder-notification, investigation, final-report, lessons-learned, jira, thehive, soc, incident-response, cdsa]
description: "Quy trình báo cáo sự cố từ phát hiện, phân tích sơ bộ, ghi nhận, thông báo, điều tra, lập báo cáo đến feedback loop."
toc: true
---

# The Incident Reporting Process

## Công thức dễ nhớ

```text
DETECT
→ TRIAGE
→ LOG
→ NOTIFY
→ INVESTIGATE
→ REPORT
→ IMPROVE
```

Incident reporting process là framework liên kết toàn bộ hoạt động:

```text
Detection
Analysis
Documentation
Communication
Investigation
Reporting
Lessons Learned
```

Mục tiêu không chỉ là tạo một tài liệu cuối cùng, mà còn biến dữ liệu của incident thành quyết định và cải tiến có thể hành động.

---

# 1. Initial Detection & Acknowledgement

Incident phải được phát hiện và xác nhận trước khi được đưa vào quy trình báo cáo chính thức.

Nguồn phát hiện có thể gồm:

```text
SIEM alert
EDR/XDR detection
IDS/IPS alert
Antivirus
NetFlow anomaly
User report
Third-party notification
Threat actor activity
```

Trong ransomware, attacker có thể tự tạo dấu hiệu phát hiện thông qua:

```text
Ransom note
Mass encryption
Service disruption
Extortion message
```

## Mục tiêu của bước này

```text
Xác nhận signal đã được tiếp nhận
Ghi nhận thời gian phát hiện
Xác định người hoặc hệ thống báo cáo
Tạo initial incident reference
Chuyển sang preliminary analysis
```

Thông tin tối thiểu:

```text
Detection time
Alert/report source
Affected asset or user
Short description
Initial evidence
Acknowledging analyst
```

---

# 2. Preliminary Analysis

Preliminary analysis tương đương bước triage ban đầu.

Cần xác định:

```text
Incident có thực sự xảy ra không?
Scope ban đầu là gì?
Tác động tiềm năng ra sao?
Có cần containment ngay không?
Category và severity nào phù hợp?
```

## Hoạt động chính

```text
Validate alert
Collect initial context
Identify affected assets
Identify affected users
Estimate business impact
Assign category
Assign severity
Escalate when required
```

Ví dụ:

```text
Alert:
Multiple failed O365 logins.

Preliminary analysis:
Cùng source IP, nhiều user, có một successful login.

Classification:
Potential password spraying with account compromise.

Severity:
P2 High.
```

---

# 3. Incident Logging

Mọi hành động, evidence và observation phải được ghi lại trong một hệ thống thống nhất.

Nền tảng phổ biến:

```text
JIRA
TheHive
ServiceNow
Dedicated IR platform
Case management system
```

Nếu không có hệ thống chuyên dụng:

```text
Spreadsheet
Shared document
Notebook
Pen and paper
```

Công cụ đơn giản vẫn hữu ích nếu tổ chức bảo đảm:

```text
Timestamps
Ownership
Access control
Version history
Evidence references
Regular backups
```

---

# 4. Nội Dung Cần Ghi Trong Incident Log

```text
Incident ID
Timestamp
Analyst
Action performed
Observation
Evidence source
Decision
Reason for decision
Status change
Assigned owner
Next action
```

Ví dụ:

| Time | Analyst | Action | Result | Evidence |
|---|---|---|---|---|
| 09:15 UTC | SOC L1 | Validated alert | Suspicious login confirmed | Splunk search |
| 09:22 UTC | SOC L2 | Disabled account | Sessions blocked | IAM audit log |
| 09:31 UTC | IR | Isolated endpoint | Host disconnected | EDR console |

Nguyên tắc:

```text
If it was not documented,
it may be treated as if it did not happen.
```

---

# 5. Notification of Relevant Parties

Stakeholder phải được xác định và thông báo đúng thời điểm.

Communication chia thành:

```text
Internal Communications
External Communications
```

Mỗi notification phải phù hợp với:

```text
Audience
Incident severity
Business impact
Legal obligations
Need-to-know principle
```

---

# 6. Internal Communications

Các bên nội bộ có thể gồm:

```text
SOC
Incident Response
IT Operations
Network Team
Legal
Compliance
Public Relations
Executive Leadership
Human Resources
Finance
Business Owners
```

Thông báo ban đầu nên có:

```text
Incident ID
Current severity
Known facts
Affected systems
Immediate actions
Current business impact
Incident owner
Next update time
```

Với incident nghiêm trọng, có thể cần:

```text
Organization-wide notification
```

Nhưng thông điệp phải được Legal/PR phê duyệt và không tiết lộ chi tiết nhạy cảm.

---

# 7. External Communications

Tùy incident, có thể phải thông báo cho:

```text
Customers
Partners
Vendors
Cyber insurers
Regulatory bodies
Law enforcement
General public
```

Trước khi gửi external communication cần xác nhận:

```text
Legal requirement
Reporting deadline
Approved content
Authorized spokesperson
Evidence supporting the statement
```

Không nên công bố:

```text
Unverified conclusions
Internal credentials
Detailed defensive gaps
Sensitive personal information
Information that harms the investigation
```

---

# 8. Detailed Investigation & Reporting

Đây thường là giai đoạn dài nhất.

Thời lượng có thể là:

```text
Hours
Days
Months
Years
```

Phụ thuộc vào:

```text
Incident complexity
Data volume
Number of affected systems
Regulatory requirements
Availability of evidence
Third-party involvement
Legal proceedings
```

## Nội dung điều tra

```text
Scope
Initial access
Execution
Persistence
Privilege escalation
Credential access
Lateral movement
Command and control
Data access
Exfiltration
Containment
Eradication
Recovery
```

---

# 9. Evidence Và Technical Analysis

Nguồn evidence có thể gồm:

```text
SIEM logs
EDR telemetry
Windows Event Logs
Firewall/proxy logs
DNS logs
Cloud audit logs
Disk image
Memory dump
Packet capture
Email artifacts
Malware samples
```

Công cụ CDSA thường gặp:

```text
Splunk
ELK
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

Cần duy trì:

```text
Evidence integrity
Hash values
Chain of custody
Acquisition timestamps
Analyst notes
Reproducible findings
```

---

# 10. Investigation Notes Và Final Report Khác Nhau

## Investigation Notes

```text
Raw observations
Working hypotheses
Search queries
Interim findings
Unconfirmed leads
Analyst-to-analyst detail
```

## Final Report

```text
Validated facts
Confirmed timeline
Business impact
Root cause
Response actions
Recovery status
Recommendations
```

Không nên đưa toàn bộ suy đoán chưa xác minh vào final report.

---

# 11. Final Report Creation

Final report là sản phẩm cuối của analyst hoặc incident responder.

Đối tượng sử dụng có thể gồm:

```text
Executive leadership
Regulators
Cyber insurers
Legal counsel
Auditors
Technical teams
```

Final report phải trả lời:

```text
What happened?
How was it detected?
What caused it?
What was affected?
What evidence confirms it?
What actions were taken?
Was the incident contained?
What is the residual risk?
What must change next?
```

---

# 12. Cấu Trúc Final Report Đề Xuất

```text
1. Executive Summary
2. Incident Overview
3. Scope and Affected Assets
4. Evidence Sources
5. Technical Analysis
6. Incident Timeline
7. Indicators of Compromise
8. Root Cause Analysis
9. Impact Analysis
10. Containment
11. Eradication
12. Recovery
13. Lessons Learned
14. Recommendations
15. Appendices
```

---

# 13. Feedback Loop

Feedback loop biến incident thành cơ hội cải tiến.

Công thức:

```text
WHAT FAILED?
→ WHY?
→ WHAT WORKED?
→ WHAT MUST CHANGE?
→ WHO OWNS IT?
→ WHEN IS IT DUE?
```

Các câu hỏi cần trả lời:

```text
Alert có đủ sớm không?
Telemetry có đầy đủ không?
Escalation có bị chậm không?
Runbook có phù hợp không?
Containment có hiệu quả không?
Có communication gap không?
Có recurring control failure không?
```

---

# 14. Lessons Learned

Lessons Learned nên tạo ra:

```text
Detection improvements
Logging improvements
Policy changes
Architecture changes
Training requirements
Process updates
New playbooks
Control ownership
```

Mỗi recommendation cần có:

```text
Priority
Owner
Due date
Expected outcome
Tracking status
```

Ví dụ:

| Recommendation | Priority | Owner | Due |
|---|---|---|---|
| Enable PowerShell 4104 logging | High | Endpoint Team | 14 days |
| Deploy password spraying rule | High | Detection Engineering | 7 days |
| Update escalation matrix | Medium | SOC Manager | 30 days |

---

# 15. Reporting Process Và NIST IR

Có thể ánh xạ quy trình này vào NIST Incident Response:

| Reporting Process | NIST IR Stage |
|---|---|
| Detection & Acknowledgement | Detection & Analysis |
| Preliminary Analysis | Detection & Analysis |
| Incident Logging | Tất cả giai đoạn |
| Notification | Tất cả giai đoạn |
| Detailed Investigation | Detection, Containment, Eradication |
| Final Report | Post-Incident Activity |
| Feedback Loop | Post-Incident Activity |

---

# 16. SOC/CDSA Mapping

## Security Operations & Monitoring

```text
Detection
Triage
Severity assignment
Incident creation
```

## Incident Response & Forensics

```text
Evidence collection
Timeline reconstruction
Containment
Eradication
Recovery
```

## Malware Analysis

```text
Malware behavior
Persistence
IoCs
Scope expansion
```

## Threat Hunting

```text
Hypothesis-driven investigation
Environment-wide search
Unknown affected systems
```

## Detection Engineering

```text
Rule tuning
Coverage gaps
New Splunk/Sigma detections
Validation after remediation
```

---

# 17. Common Process Failures

```text
Không acknowledge alert
Triage quá chậm
Severity không có căn cứ
Incident log thiếu timestamp
Không xác định owner
Stakeholder được thông báo quá muộn
Investigation notes bị thất lạc
Final report trộn fact với assumption
Không tạo recommendation
Không theo dõi feedback action
```

---

# 18. Process Quality Checklist

```text
[ ] Detection được acknowledge
[ ] Category và severity được gán
[ ] Incident ID được tạo
[ ] Mọi action có timestamp
[ ] Evidence source được ghi rõ
[ ] Stakeholder phù hợp được thông báo
[ ] Scope được cập nhật liên tục
[ ] Containment/eradication/recovery được ghi nhận
[ ] Final report đã được review
[ ] Lessons learned có owner và deadline
```

---

# Keyword Cần Nhớ

```text
Initial Detection
Acknowledgement
Preliminary Analysis
Triage
Incident Logging
JIRA
TheHive
Stakeholder Notification
Internal Communication
External Communication
Detailed Investigation
Technical Analysis
Final Report
Feedback Loop
Lessons Learned
Evidence Integrity
Chain of Custody
Severity
Scope
Root Cause
Remediation
```

---

# Key Takeaway

```text
Detect mà không log
→ không có audit trail.
```

```text
Điều tra mà không notify
→ response bị phân mảnh.
```

```text
Report mà không feedback
→ tổ chức sẽ lặp lại cùng sai sót.
```

```text
Incident reporting process tốt =
detect nhanh
+ document đầy đủ
+ communicate đúng
+ investigate sâu
+ improve liên tục
```

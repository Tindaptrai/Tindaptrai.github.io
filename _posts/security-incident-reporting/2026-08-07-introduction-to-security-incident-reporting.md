---
layout: post
title: "Introduction to Security Incident Reporting"
date: 2026-08-07 01:01:22 +0700
categories: ["Security Incident Reporting", "Fundamentals"]
tags: [security-incident-reporting, incident-response, soc, incident-identification, incident-categorization, severity, p1, p2, p3, p4, siem, edr, xdr, ids, ips, cdsa]
description: "Vai trò của Security Incident Reporting, nguồn nhận diện sự cố, phân loại incident và mức độ nghiêm trọng trong SOC."
toc: true
---

# Introduction to Security Incident Reporting

## Công thức dễ nhớ

```text
IDENTIFY → CATEGORIZE → PRIORITIZE → REPORT → LEARN
```

Security Incident Reporting là cầu nối giữa:

```text
Threat Detection
→ Incident Response
→ Remediation
→ Lessons Learned
→ Future Prevention
```

Mục tiêu không chỉ là ghi lại sự cố, mà còn giúp tổ chức phân bổ nguồn lực, đáp ứng pháp lý, đánh giá rủi ro, ước tính tác động tài chính và cải thiện khả năng phòng thủ.

---

# 1. Vì Sao Incident Reporting Quan Trọng?

Trong môi trường hiện đại, câu hỏi không còn là:

```text
“Liệu sự cố có xảy ra hay không?”
```

mà là:

```text
“Khi nào sự cố sẽ xảy ra?”
```

Công nghệ giúp tăng hiệu quả vận hành nhưng đồng thời mở rộng:

```text
Attack Surface
Operational Risk
Data Exposure
Dependency Risk
```

Một quy trình báo cáo tốt giúp tổ chức phản ứng nhất quán khi threat landscape liên tục thay đổi.

---

# 2. Vai Trò Của Incident Report

| Stakeholder | Nhu cầu |
|---|---|
| SOC/IR Team | Timeline, evidence, containment, remediation |
| Technical Teams | Root cause, affected assets, corrective actions |
| Legal | Regulatory and legal obligations |
| Management | Business impact and risk |
| Finance | Financial loss and recovery cost |
| Compliance | Audit trail and reporting requirements |
| Lessons Learned | Process and control improvements |

Một báo cáo tốt phải cân bằng:

```text
Technical Granularity
+
Business Accessibility
```

Nó phải đủ chi tiết cho analyst nhưng vẫn dễ hiểu với stakeholder không chuyên kỹ thuật.

---

# 3. Incident Identification

Incident có thể xuất hiện dưới dạng:

```text
Detection
Anomaly
Alert
Baseline Deviation
User Report
External Notification
```

Ba nguồn chính:

```text
Security Tooling
Human Observation
Third-Party Notification
```

---

# 4. Security Systems & Tooling

Nguồn telemetry phổ biến:

```text
IDS/IPS
EDR/XDR
SIEM
Antivirus
Firewall
NetFlow
Email Security
Cloud Security
Authentication Logs
```

Trong CDSA/SOC thường gặp:

```text
Splunk/ELK
Sysmon
Wazuh
Suricata
Zeek
Microsoft Defender
```

Tooling giúp trả lời:

```text
What happened?
When?
Which host/user?
How broad is the activity?
Does it match known TTPs?
```

---

# 5. Human Observations

Người dùng có thể phát hiện:

```text
Suspicious email
Unexpected MFA prompt
Unusual system behavior
Unknown application
Strange login notification
Missing or altered files
```

Human reporting quan trọng vì không phải mọi threat đều tạo alert tự động.

Ví dụ:

```text
User báo email lạ
→ SOC kiểm tra header/URL
→ xác định phishing
→ hunt thêm mailbox và endpoint
```

---

# 6. Third-Party Notifications

Nguồn bên ngoài có thể gồm:

```text
Partners
Vendors
Customers
MSSP
Cloud providers
Threat intelligence providers
Law enforcement
Researchers
```

Third party có thể thông báo về:

```text
Compromised credentials
Exposed data
Vulnerability
Supply-chain incident
Suspicious traffic
Breach affecting shared systems
```

---

# 7. Incident Categorization

Phân loại incident giúp:

```text
Prioritize response
Assign the right team
Allocate resources
Define escalation path
Brief stakeholders
Track incident trends
```

Một incident có thể thuộc nhiều category:

```text
Phishing
→ credential theft
→ unauthorized access
→ data leakage
```

---

# 8. Common Incident Types

## Malware

```text
Virus
Worm
Trojan
Ransomware
Infostealer
Backdoor
Rootkit
```

## Phishing

```text
Credential phishing
Business Email Compromise
Malicious attachment
Malicious link
OAuth phishing
Device code phishing
```

## DDoS

```text
Volumetric attack
Protocol attack
Application-layer attack
```

## Unauthorized Access

```text
Stolen account
Privilege abuse
Compromised VPN
Remote access misuse
```

## Data Leakage

```text
Accidental exposure
Unauthorized sharing
Cloud bucket exposure
Exfiltration
Misconfigured access control
```

## Physical Breach

```text
Unauthorized facility access
Device theft
Tailgating
Tampering
```

---

# 9. Incident Severity Levels

## Critical — P1

```text
Immediate threat
Core business impact
Sensitive data at risk
Major outage
Active ransomware
Domain compromise
```

Yêu cầu:

```text
Immediate escalation
24/7 response
Executive notification
Rapid containment
```

## High — P2

```text
Serious threat
Elevated business risk
Potential spread
Privileged account compromise
Confirmed malware without major outage
```

## Medium — P3

```text
No immediate critical impact
Requires timely investigation
Limited scope
Suspicious but not fully confirmed
```

## Low — P4

```text
Routine anomaly
Low-risk event
Minor policy violation
Informational finding
```

---

# 10. Severity Có Thể Thay Đổi

Severity phải:

```text
Dynamic
Evidence-driven
Business-context aware
```

Ví dụ nâng cấp:

```text
P3 suspicious login
→ phát hiện privileged account compromise
→ escalate P1/P2
```

Ví dụ hạ cấp:

```text
P1 ransomware alert
→ xác minh là test file
→ downgrade
```

---

# 11. Categorization Challenges

Một incident có thể đồng thời là:

```text
Malware
Unauthorized Access
Credential Theft
Data Leakage
Persistence
```

Ví dụ:

```text
Phishing email
→ user mở attachment
→ malware executes
→ credentials stolen
→ attacker logs in remotely
→ data exfiltrated
```

Không nên ép incident vào một category duy nhất nếu điều đó làm mất context.

---

# 12. Incident Identification Workflow

```text
1. Receive alert/report
2. Validate signal
3. Identify affected asset/user
4. Determine incident category
5. Assign severity
6. Open incident record
7. Escalate when required
8. Begin containment/investigation
9. Update category and severity as evidence changes
10. Produce final report and lessons learned
```

---

# 13. Minimum Information Khi Mở Incident

```text
Incident ID
Detection time
Reporter/source
Affected user
Affected host
Initial category
Initial severity
Alert/evidence reference
Short description
Assigned analyst/team
Current status
```

Tránh tạo ticket chỉ có:

```text
“Suspicious activity detected”
```

mà không có context.

---

# 14. Good Incident Report Principles

```text
Accurate
Consistent
Timely
Evidence-based
Clear
Audience-aware
Traceable
Actionable
```

Phải phân biệt:

```text
Fact
Inference
Assumption
Unconfirmed hypothesis
```

Ví dụ:

```text
Fact:
User logged in from IP X at time Y.

Inference:
The login may be related to credential theft.

Unknown:
The initial access vector has not been confirmed.
```

---

# 15. SOC/CDSA Mapping

## Security Operations & Monitoring

```text
Alert triage
SIEM detection
Baseline deviation
Incident creation
```

## Incident Response & Forensics

```text
Evidence handling
Timeline
Containment
Root cause
Impact assessment
```

## Malware Analysis

```text
Sample findings
Behavior summary
IOC extraction
Scope determination
```

## Threat Hunting

```text
Hypothesis
Hunt results
Affected hosts
Detection gaps
```

## Detection Engineering

```text
Rule performance
False positives
Coverage gaps
Tuning recommendations
```

---

# 16. Common Reporting Mistakes

```text
Thiếu timeline
Không ghi source evidence
Trộn fact với assumption
Severity không có lý do
Không cập nhật scope
Quá kỹ thuật với executive audience
Quá chung chung với technical audience
Không ghi containment/remediation
Không lưu lessons learned
```

---

# Keyword Cần Nhớ

```text
Security Incident Reporting
Incident Identification
Incident Categorization
Incident Severity
P1 Critical
P2 High
P3 Medium
P4 Low
Security Tooling
IDS/IPS
EDR/XDR
SIEM
NetFlow
Human Observation
Third-Party Notification
Malware
Phishing
DDoS
Unauthorized Access
Data Leakage
Physical Breach
Stakeholder
Lessons Learned
Business Impact
Evidence
Escalation
```

---

# Key Takeaway

```text
Incident identification
quyết định ta đang xử lý sự kiện gì.
```

```text
Incident categorization
quyết định đội nào và quy trình nào được kích hoạt.
```

```text
Incident severity
quyết định tốc độ, nguồn lực và mức escalation.
```

```text
Một SOC hiệu quả cần:
identify nhanh
+ categorize đúng
+ prioritize hợp lý
+ report nhất quán
```

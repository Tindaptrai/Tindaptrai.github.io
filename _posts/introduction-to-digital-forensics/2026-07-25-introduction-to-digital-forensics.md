---
layout: post
title: "Introduction to Digital Forensics"
date: 2026-07-25 00:05:53 +0700
categories: ["Introduction to Digital Forensics", "Fundamentals"]
tags: [digital-forensics, dfir, incident-response, evidence, chain-of-custody, forensic-image, timeline-analysis, ioc, soc, cdsa]
description: "Tóm tắt nền tảng Digital Forensics dành cho SOC Analyst: bằng chứng số, quy trình điều tra, forensic image, timeline, IOC và báo cáo."
toc: true
---

# Introduction to Digital Forensics

Digital Forensics là quá trình **thu thập, bảo toàn, phân tích và trình bày bằng chứng số** để điều tra sự cố an ninh.

## Công thức dễ nhớ

```text
IDENTIFY → COLLECT → PRESERVE → ANALYZE → PRESENT
```

## 1. Mục Tiêu Chính

Digital Forensics giúp:

- Xác định điều gì đã xảy ra.
- Xác định thời điểm xâm nhập.
- Tìm hệ thống và dữ liệu bị ảnh hưởng.
- Khôi phục timeline của attacker.
- Thu thập IOC và TTP.
- Hỗ trợ Incident Response và pháp lý.

## 2. Electronic Evidence

Bằng chứng số có thể đến từ:

```text
Files
Emails
Windows Event Logs
Registry
Memory
Network traffic
Databases
Cloud services
Mobile devices
Storage media
```

## 3. Preservation of Evidence

Bằng chứng phải được giữ nguyên tính toàn vẹn.

Cần thực hiện:

```text
Hash
Timestamp
Write blocker
Read-only analysis
Chain of custody
Document every action
```

Không phân tích trực tiếp trên bản gốc nếu có thể tránh.

## 4. Forensic Process

```text
Identification
→ xác định nguồn bằng chứng

Collection
→ thu thập bằng phương pháp forensically sound

Examination
→ trích xuất artifact liên quan

Analysis
→ diễn giải dữ liệu và dựng lại sự cố

Presentation
→ báo cáo kết quả rõ ràng
```

## 5. Các Bước Điều Tra Cơ Bản

```text
1. Tạo forensic image
2. Ghi nhận trạng thái hệ thống
3. Xác định và bảo toàn bằng chứng
4. Phân tích artifact
5. Dựng timeline
6. Xác định IOC
7. Viết báo cáo
```

## 6. Các Loại Vụ Việc

Digital Forensics thường áp dụng cho:

- Cybercrime.
- Data breach.
- Malware infection.
- Insider threat.
- Gian lận và đánh cắp dữ liệu.
- Vi phạm chính sách nội bộ.
- Hỗ trợ tố tụng.

## 7. Vai Trò Với SOC Analyst

### Incident Triage

```text
Alert
→ xác định host bị ảnh hưởng
→ thu thập artifact
→ đánh giá mức độ nghiêm trọng
```

### Scope Incident

Xác định:

```text
Initial access
Affected systems
Persistence
Lateral movement
Credential access
Exfiltration
```

### Threat Hunting

IOC và TTP từ điều tra có thể dùng để:

- Viết Sigma rule.
- Viết YARA rule.
- Tạo Splunk/ELK query.
- Hunt trên endpoint khác.
- Bổ sung detection coverage.

### Incident Response

Forensics hỗ trợ:

```text
Containment
Eradication
Recovery
Lessons learned
```

## 8. Artifact Quan Trọng Trên Windows

```text
Event Logs
Registry
Prefetch
Amcache
Shimcache
LNK files
Jump Lists
Browser history
MFT
USN Journal
Memory dump
Process tree
```

## 9. Keyword Cần Nhớ

```text
Digital Evidence
Forensic Image
Integrity
Hash
Chain of Custody
Timeline
IOC
TTP
Artifact
Memory Forensics
Disk Forensics
Incident Response
```

## 10. Checklist SOC/DFIR

```text
[ ] Đã ghi nhận thời gian và timezone?
[ ] Đã hash evidence?
[ ] Đã tạo forensic copy?
[ ] Đã bảo quản bản gốc?
[ ] Đã ghi chain of custody?
[ ] Đã dựng timeline?
[ ] Đã xác định IOC/TTP?
[ ] Đã đánh giá scope?
[ ] Đã ghi rõ kết luận và giới hạn?
```

## Key Takeaway

```text
Digital Forensics không chỉ trả lời:
"Có gì đáng ngờ?"

Nó phải trả lời:
Điều gì xảy ra?
Khi nào?
Trên hệ thống nào?
Bằng cách nào?
Ảnh hưởng đến đâu?
```

Trong CDSA:

```text
Digital Forensics
→ Incident Response
→ Threat Hunting
→ Detection Engineering
→ SOC Improvement
```

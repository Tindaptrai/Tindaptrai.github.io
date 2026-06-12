---
layout: post
title: "Alert Triaging Process trong SOC"
date: 2026-06-12 22:55:00 +0700
categories: [SOC, Incident Response]
tags: [soc, alert-triage, escalation, de-escalation, incident-response, risk-assessment, threat-intelligence, blue-team]
description: "Ghi chú tổng hợp về quy trình phân loại cảnh báo trong SOC: alert triaging, escalation, correlation, enrichment, risk assessment, response execution, continuous monitoring và de-escalation."
toc: true
---

# Alert Triaging Process trong SOC

**Alert Triaging** là quá trình SOC analyst đánh giá, phân loại và ưu tiên các cảnh báo bảo mật để xác định mức độ nguy hiểm, tác động tiềm ẩn và hướng xử lý phù hợp.

Nói ngắn gọn:

```text
Alert Triaging = xem alert có đáng lo không, ưu tiên mức nào và cần xử lý ra sao.
```

Trong SOC, không phải alert nào cũng là incident thật. Vì vậy, triage giúp SOC tránh bị ngập trong alert, giảm false positive và tập trung vào các sự kiện có rủi ro cao.

---

## 1. Alert Triaging là gì?

**Alert triaging** là quá trình xem xét các cảnh báo được tạo bởi nhiều hệ thống giám sát và phát hiện như:

- SIEM.
- EDR/XDR.
- IDS/IPS.
- Firewall.
- Email Security Gateway.
- Cloud Security.
- Antivirus.
- Network Monitoring.
- Threat Intelligence platform.

Mục tiêu của triage là xác định:

- Alert có đáng ngờ không?
- Alert là false positive hay true positive?
- Mức độ nghiêm trọng là gì?
- Tài sản nào bị ảnh hưởng?
- Có cần escalation không?
- Có cần kích hoạt incident response không?

---

## 2. Vì sao alert triage quan trọng?

Trong môi trường doanh nghiệp, SOC có thể nhận hàng trăm hoặc hàng nghìn alert mỗi ngày.

Nếu không có quy trình triage rõ ràng, SOC dễ gặp các vấn đề:

- Alert fatigue.
- Bỏ sót incident thật.
- Phản ứng chậm với sự cố nghiêm trọng.
- Xử lý alert không nhất quán.
- Escalation sai hoặc chậm.
- Lãng phí nguồn lực vào false positive.

Triage giúp SOC trả lời nhanh:

```text
Alert này có nguy hiểm không?
Có cần xử lý ngay không?
Có cần chuyển lên Tier 2/Tier 3/IR không?
```

---

# 3. Escalation là gì?

**Escalation** là quá trình chuyển một alert hoặc incident lên cấp xử lý cao hơn.

Trong SOC, escalation có thể bao gồm:

- Thông báo SOC Manager.
- Chuyển case cho Tier 2/Tier 3.
- Kích hoạt Incident Response team.
- Thông báo IT Operations.
- Thông báo system owner.
- Tham gia Legal/Compliance nếu cần.
- Liên hệ CERT, law enforcement hoặc vendor bên ngoài trong trường hợp nghiêm trọng.

Escalation đảm bảo các alert quan trọng được chú ý kịp thời và có đủ người có thẩm quyền để ra quyết định.

---

## 3.1. Khi nào cần escalation?

Nên escalation nếu alert có các dấu hiệu:

- Ảnh hưởng đến critical asset.
- Có dấu hiệu compromise thật.
- Có attacker activity đang diễn ra.
- Có lateral movement.
- Có data exfiltration.
- Có ransomware hoặc destructive activity.
- Có privileged account bị ảnh hưởng.
- Có threat actor hoặc malware nghiêm trọng.
- Có tác động rộng đến nhiều hệ thống.
- Có nghĩa vụ pháp lý hoặc compliance.

---

# 4. Quy trình Alert Triaging lý tưởng

Một quy trình triage lý tưởng có thể gồm các bước:

```text
Initial Alert Review
      ↓
Alert Classification
      ↓
Alert Correlation
      ↓
Enrichment of Alert Data
      ↓
Risk Assessment
      ↓
Contextual Analysis
      ↓
Incident Response Planning
      ↓
Consultation with IT Operations
      ↓
Response Execution
      ↓
Escalation
      ↓
Continuous Monitoring
      ↓
De-escalation
```

---

# 5. Initial Alert Review

**Initial Alert Review** là bước xem xét cảnh báo ban đầu.

SOC analyst cần kiểm tra:

- Alert name.
- Alert source.
- Severity.
- Timestamp.
- Source IP.
- Destination IP.
- Source host.
- Destination host.
- User liên quan.
- Process liên quan.
- Rule/signature kích hoạt.
- Log source.
- MITRE ATT&CK mapping nếu có.
- Event raw data.

Ví dụ:

```text
Alert: Suspicious PowerShell Execution
Time: 2026-06-12 10:15
Host: WS001
User: john
Process: powershell.exe
Command line: powershell.exe -enc ...
```

Mục tiêu của bước này là hiểu alert đang nói về điều gì.

---

# 6. Alert Classification

**Alert Classification** là phân loại alert theo mức độ nghiêm trọng, tác động và độ khẩn cấp.

Một hệ thống phân loại phổ biến:

| Severity | Ý nghĩa |
|---|---|
| Informational | Chỉ để ghi nhận |
| Low | Ít rủi ro, cần theo dõi |
| Medium | Có dấu hiệu đáng ngờ, cần điều tra |
| High | Khả năng incident cao, cần xử lý nhanh |
| Critical | Tác động nghiêm trọng, cần phản ứng ngay |

Khi phân loại alert, cần xét:

- Alert có liên quan đến tài sản quan trọng không?
- User có phải privileged account không?
- Có dấu hiệu exploit/malware không?
- Có nhiều event tương tự không?
- Có threat intelligence match không?
- Có hành vi bất thường so với baseline không?

---

# 7. Alert Correlation

**Alert Correlation** là tham chiếu alert với các event, alert hoặc incident liên quan.

Mục tiêu là tìm pattern hoặc attack chain.

Ví dụ một alert đơn lẻ:

```text
PowerShell encoded command
```

Có thể nguy hiểm hơn nếu liên quan với:

```text
Email attachment opened
      ↓
Word spawned PowerShell
      ↓
PowerShell downloaded payload
      ↓
Endpoint connected to external IP
```

Nguồn dữ liệu dùng để correlation:

- SIEM logs.
- EDR telemetry.
- Firewall logs.
- Proxy logs.
- DNS logs.
- VPN logs.
- Windows Event Logs.
- Authentication logs.
- Threat intelligence feeds.

---

## 7.1. IOC trong correlation

Khi correlation, analyst nên tìm các IOC như:

- IP address.
- Domain.
- URL.
- File hash.
- File path.
- Process name.
- Command line.
- Registry key.
- Username.
- Hostname.
- Email sender.
- Mutex.
- Scheduled task.

IOC giúp liên kết nhiều event thành một bức tranh incident rõ hơn.

---

# 8. Enrichment of Alert Data

**Enrichment** là làm giàu dữ liệu alert bằng thông tin bổ sung.

Mục tiêu là tăng context để analyst quyết định chính xác hơn.

Các nguồn enrichment:

- Threat Intelligence.
- VirusTotal.
- AbuseIPDB.
- WHOIS.
- Passive DNS.
- Sandbox.
- EDR process tree.
- Asset inventory.
- CMDB.
- GeoIP.
- User department.
- Vulnerability data.
- Packet capture.
- Memory dump.
- File sample.

Ví dụ:

```text
Alert có destination IP 185.x.x.x
      ↓
Tra threat intelligence
      ↓
IP được đánh dấu là C2 server
      ↓
Tăng severity và escalation
```

---

## 8.1. Enrichment cần thu thập gì?

Tùy loại alert, analyst có thể thu thập:

- Network packet capture.
- Memory dump.
- File sample.
- Process tree.
- Parent/child process.
- Command line.
- Network connection.
- DNS query.
- Proxy request.
- Registry modification.
- File creation/modification.
- User login history.
- Host vulnerability status.

---

# 9. Risk Assessment

**Risk Assessment** là đánh giá rủi ro và tác động tiềm ẩn của alert.

Cần xem xét:

- Tài sản bị ảnh hưởng có quan trọng không?
- Dữ liệu trên hệ thống có nhạy cảm không?
- User có quyền cao không?
- Có ảnh hưởng đến compliance không?
- Có khả năng attacker di chuyển ngang không?
- Có khả năng privilege escalation không?
- Có dấu hiệu data exfiltration không?
- Có khả năng ransomware không?

Ví dụ:

```text
Cùng một alert PowerShell,
nếu xảy ra trên máy test có thể là Medium,
nhưng nếu xảy ra trên Domain Controller thì có thể là Critical.
```

---

# 10. Contextual Analysis

**Contextual Analysis** là phân tích alert trong bối cảnh cụ thể của tổ chức.

SOC analyst cần xem:

- Host có thuộc production không?
- Host có phải server quan trọng không?
- User có phải admin không?
- Thời điểm xảy ra có bất thường không?
- Hành vi có khớp baseline không?
- Có thay đổi hệ thống gần đây không?
- Có maintenance window không?
- Có rule nào đang tạo nhiều false positive không?
- Control bảo mật nào đã chặn hoặc bỏ sót hành vi này?

---

## 10.1. Kiểm tra control bảo mật

Analyst nên đánh giá các control liên quan:

- Firewall có block không?
- IDS/IPS có alert không?
- EDR có detect không?
- Antivirus có quarantine không?
- Proxy có ghi nhận download không?
- Email gateway có phát hiện phishing không?
- SIEM có alert correlation không?

Nếu alert cho thấy attacker bypass được control, đây có thể là dấu hiệu cần escalation.

---

# 11. Incident Response Planning

Nếu alert có dấu hiệu nghiêm trọng, SOC cần bắt đầu kế hoạch Incident Response.

Cần ghi lại:

- Chi tiết alert.
- Hệ thống bị ảnh hưởng.
- User liên quan.
- Hành vi quan sát được.
- IOC tiềm năng.
- Dữ liệu enrichment.
- Timeline ban đầu.
- Severity.
- Risk assessment.
- Bằng chứng đã thu thập.

Sau đó phân công vai trò:

- Incident Commander.
- SOC Analyst.
- Incident Responder.
- Forensic Analyst.
- IT Operations.
- System Owner.
- Communications/Legal nếu cần.

---

# 12. Consultation with IT Operations

Không phải alert nào cũng là tấn công.

SOC nên tham khảo IT Operations hoặc team liên quan khi cần thêm context.

Cần hỏi:

- Hệ thống có đang maintenance không?
- Có thay đổi cấu hình gần đây không?
- Có deployment mới không?
- Có script hoặc automation hợp lệ không?
- Có service account nào đang lỗi không?
- Có cấu hình sai gây alert không?
- Có sự cố nội bộ đang diễn ra không?

Ví dụ false positive:

```text
Nhiều failed login từ service account
      ↓
IT xác nhận password service vừa được đổi
      ↓
Scheduled task vẫn dùng password cũ
```

Thông tin này giúp tránh escalation không cần thiết.

---

# 13. Response Execution

Sau khi review, correlation, enrichment và risk assessment, SOC xác định hành động phản ứng.

Có hai hướng chính:

## 13.1. Alert không độc hại

Nếu alert được xác định là benign hoặc false positive:

- Đóng alert.
- Ghi rõ lý do.
- Cập nhật tuning nếu cần.
- Thêm whitelist nếu hợp lý.
- Thông báo team liên quan nếu cần.

## 13.2. Alert đáng ngờ hoặc true positive

Nếu alert vẫn có dấu hiệu bảo mật:

- Tiếp tục điều tra.
- Escalate lên Tier 2/Tier 3.
- Tạo incident case.
- Kích hoạt IRP.
- Cô lập host nếu cần.
- Thu thập bằng chứng.
- Block IOC.
- Disable account nếu cần.
- Reset credential nếu cần.
- Phối hợp IT để containment.

---

# 14. Escalation Process

Escalation cần dựa trên chính sách tổ chức và severity của alert.

Các trigger escalation có thể gồm:

- Critical asset bị ảnh hưởng.
- Attack đang diễn ra.
- Technique phức tạp hoặc chưa quen thuộc.
- Tác động rộng.
- Threat nội bộ.
- Ransomware.
- Data exfiltration.
- Privileged account compromise.
- Domain Controller compromise.
- Compliance/legal impact.

---

## 14.1. Nội dung cần cung cấp khi escalation

Khi escalation, SOC analyst nên cung cấp bản tóm tắt rõ ràng:

```text
Alert Summary:
Severity:
Affected Systems:
Affected Users:
Timeline:
Observed Behavior:
Potential Impact:
Evidence:
IOCs:
MITRE Mapping:
Risk Assessment:
Actions Taken:
Recommended Next Steps:
```

Thông tin càng rõ, team cấp cao hoặc Incident Response càng xử lý nhanh.

---

## 14.2. Ghi lại escalation

Mọi communication liên quan đến escalation cần được ghi lại:

- Ai được thông báo?
- Thông báo lúc nào?
- Nội dung thông báo là gì?
- Ai nhận trách nhiệm?
- Hành động tiếp theo là gì?
- Quyết định nào đã được đưa ra?
- Có liên hệ bên ngoài không?

Ghi log đầy đủ giúp phục vụ audit, post-incident review và legal/compliance.

---

# 15. External Escalation

Trong một số trường hợp, tổ chức có thể cần escalation ra bên ngoài.

Ví dụ:

- Law enforcement.
- CERT/CSIRT quốc gia.
- Incident response vendor.
- Cloud provider.
- ISP.
- Legal counsel.
- Cyber insurance provider.
- Regulator.

External escalation thường xảy ra khi có:

- Data breach.
- Ransomware nghiêm trọng.
- Attack diện rộng.
- Yêu cầu pháp lý.
- Ảnh hưởng khách hàng.
- Cần hỗ trợ forensic chuyên sâu.

---

# 16. Continuous Monitoring

Sau escalation hoặc response, SOC vẫn phải tiếp tục theo dõi.

Cần monitor:

- Alert có tái diễn không?
- Attacker có đổi kỹ thuật không?
- IOC có xuất hiện trên host khác không?
- Có lateral movement không?
- Có data exfiltration không?
- Containment có hiệu quả không?
- Severity có thay đổi không?
- Có cần mở rộng scope không?

SOC cần duy trì giao tiếp với các team liên quan và cập nhật tình hình liên tục.

---

# 17. De-escalation

**De-escalation** là giảm mức độ leo thang khi tình hình đã được kiểm soát.

Có thể de-escalate khi:

- Rủi ro đã được giảm thiểu.
- Incident đã được containment.
- Không còn attacker activity.
- Không cần thêm nguồn lực cấp cao.
- Alert được xác định là false positive.
- Hệ thống đã ổn định.
- Các action quan trọng đã hoàn tất.

Khi de-escalate, cần thông báo cho các bên liên quan:

- Tình trạng hiện tại.
- Hành động đã thực hiện.
- Kết quả xử lý.
- Rủi ro còn lại.
- Bài học kinh nghiệm.
- Đề xuất cải thiện.

---

# 18. Quy trình triage dạng checklist

SOC analyst có thể dùng checklist sau:

```text
[ ] Xem alert name, severity, timestamp.
[ ] Kiểm tra source/destination IP.
[ ] Kiểm tra affected host và affected user.
[ ] Kiểm tra raw event.
[ ] Xác định rule/signature kích hoạt.
[ ] Kiểm tra alert có trùng/lặp không.
[ ] Correlate với log khác.
[ ] Tra threat intelligence.
[ ] Kiểm tra asset criticality.
[ ] Kiểm tra user privilege.
[ ] Kiểm tra baseline và maintenance window.
[ ] Tham khảo IT Operations nếu cần.
[ ] Xác định false positive hay true positive.
[ ] Đánh giá severity/risk.
[ ] Ghi lại kết quả triage.
[ ] Escalate nếu vượt thẩm quyền hoặc rủi ro cao.
[ ] Theo dõi liên tục.
[ ] De-escalate/close case nếu đã kiểm soát.
```

---

# 19. Bảng ôn nhanh thuật ngữ

| Thuật ngữ | Nghĩa ngắn gọn |
|---|---|
| Alert Triaging | Phân loại và ưu tiên cảnh báo |
| Alert | Cảnh báo bảo mật |
| False Positive | Cảnh báo sai |
| True Positive | Cảnh báo đúng |
| Escalation | Chuyển lên cấp xử lý cao hơn |
| De-escalation | Giảm mức leo thang |
| Correlation | Tương quan nhiều event |
| Enrichment | Làm giàu dữ liệu cảnh báo |
| IOC | Indicator of Compromise |
| Risk Assessment | Đánh giá rủi ro |
| Contextual Analysis | Phân tích theo bối cảnh |
| Incident Response Plan | Kế hoạch ứng phó sự cố |
| IT Operations | Đội vận hành CNTT |
| Continuous Monitoring | Theo dõi liên tục |
| Severity | Mức độ nghiêm trọng |
| Critical Asset | Tài sản quan trọng |
| SLA | Cam kết thời gian xử lý |
| SOC Analyst | Người phân tích cảnh báo trong SOC |

---

# 20. Câu hỏi ôn tập

## 1. Alert triaging là gì?

Alert triaging là quá trình đánh giá, phân loại và ưu tiên cảnh báo bảo mật để xác định mức độ nguy hiểm và hướng xử lý phù hợp.

## 2. Escalation trong SOC là gì?

Escalation là quá trình chuyển alert hoặc incident lên cấp xử lý cao hơn như Tier 2, Tier 3, Incident Response team hoặc quản lý.

## 3. Vì sao cần correlation?

Correlation giúp kết nối nhiều event liên quan để xác định pattern, attack chain hoặc IOC, thay vì chỉ nhìn alert đơn lẻ.

## 4. Enrichment dùng để làm gì?

Enrichment bổ sung context cho alert, ví dụ threat intelligence, asset criticality, user privilege, process tree hoặc IP reputation.

## 5. Khi nào cần escalation?

Cần escalation khi alert ảnh hưởng critical asset, có dấu hiệu compromise thật, attack đang diễn ra, có lateral movement, data exfiltration hoặc tác động compliance/legal.

## 6. De-escalation là gì?

De-escalation là giảm mức độ leo thang khi rủi ro đã được kiểm soát, incident đã được containment hoặc alert được xác định là false positive.

---

# Key Takeaway

Alert triaging là kỹ năng cốt lõi của SOC analyst.

Một quy trình triage tốt giúp SOC xác định alert nào quan trọng, alert nào là false positive, khi nào cần escalation và khi nào có thể de-escalation.

Triage hiệu quả cần kết hợp review alert, classification, correlation, enrichment, risk assessment, contextual analysis, consultation với IT Operations, response execution và continuous monitoring.

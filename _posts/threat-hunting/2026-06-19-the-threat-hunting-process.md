---
layout: post
title: "The Threat Hunting Process"
date: 2026-06-19 22:56:00 +0700
categories: [SOC, Threat Hunting]
tags: [soc, threat-hunting, threat-intelligence, hypothesis, emotet, incident-response, ioc, ttp, blue-team, dfir]
description: "Ghi chú tổng hợp về quy trình threat hunting: chuẩn bị, xây dựng hypothesis, thiết kế hunt, thu thập dữ liệu, đánh giá phát hiện, mitigations, after-action và ví dụ hunt Emotet."
toc: true
---

# The Threat Hunting Process

**Threat Hunting Process** là quy trình có hệ thống để chủ động tìm kiếm threat trong môi trường trước khi attacker gây thiệt hại lớn hoặc trước khi hệ thống tạo alert rõ ràng.

Một hunt tốt không phải là tìm kiếm ngẫu nhiên. Nó cần có mục tiêu, hypothesis, data source, phương pháp phân tích, cách xác minh và hành động sau khi có kết quả.

Nói ngắn gọn:

```text
Threat Hunting Process = chuẩn bị → đặt giả thuyết → tìm kiếm → xác minh → xử lý → cải thiện.
```

---

## 1. Tổng quan quy trình Threat Hunting

Một quy trình threat hunting cơ bản gồm các bước:

```text
Setting the Stage
      ↓
Formulating Hypotheses
      ↓
Designing the Hunt
      ↓
Data Gathering and Examination
      ↓
Evaluating Findings and Testing Hypotheses
      ↓
Mitigating Threats
      ↓
After the Hunt
      ↓
Continuous Learning and Enhancement
```

Mỗi bước đều có vai trò riêng trong việc biến một hoạt động hunt từ ý tưởng thành kết quả có thể hành động được.

---

# 2. Setting the Stage

**Setting the Stage** là giai đoạn chuẩn bị và lập kế hoạch.

Ở giai đoạn này, threat hunting team xác định:

- Mục tiêu hunt là gì.
- Tài sản nào quan trọng.
- Rủi ro nào cần ưu tiên.
- Threat intelligence nào đang liên quan.
- Threat actor nào có khả năng nhắm đến tổ chức.
- TTP nào cần quan tâm.
- Data source nào cần thu thập.
- Tool nào cần dùng.
- Môi trường đã đủ logging chưa.

---

## 2.1. Việc cần làm trong giai đoạn chuẩn bị

Các việc quan trọng:

- Xác định critical assets.
- Xác định business requirements.
- Review threat intelligence.
- Review threat actor profiles.
- Bật logging cần thiết.
- Đảm bảo SIEM/EDR/IDS hoạt động đúng.
- Kiểm tra data ingestion.
- Xác định baseline hoạt động bình thường.
- Chuẩn bị query, dashboard hoặc notebook.
- Xác định escalation path nếu phát hiện incident.

Ví dụ:

```text
Threat hunting team nghiên cứu threat intel mới,
xác định ngành của tổ chức đang bị nhắm bởi phishing campaign,
sau đó chuẩn bị log từ email gateway, endpoint, DNS và proxy để hunt.
```

---

# 3. Formulating Hypotheses

**Formulating Hypotheses** là bước tạo giả thuyết hunting.

Hypothesis là dự đoán có cơ sở về hành vi attacker có thể đang xảy ra trong môi trường.

Một hypothesis tốt cần:

- Cụ thể.
- Có thể kiểm chứng.
- Dựa trên threat intelligence hoặc observation.
- Gắn với data source rõ ràng.
- Có thể chuyển thành query hoặc analytic.

---

## 3.1. Nguồn hình thành hypothesis

Hypothesis có thể đến từ:

- Threat intelligence report.
- Industry advisory.
- CVE mới.
- Alert từ SIEM/EDR.
- Network anomaly.
- Incident trước đó.
- Professional intuition.
- MITRE ATT&CK technique.
- Red Team/Purple Team finding.

Ví dụ hypothesis:

```text
APT group có thể đang khai thác lỗ hổng web server để tạo C2 channel.
```

Một hypothesis tốt hơn:

```text
APT group đang khai thác lỗ hổng trên web server public-facing để upload web shell và tạo outbound HTTPS connection đến C2 infrastructure.
```

---

# 4. Designing the Hunt

Sau khi có hypothesis, team cần thiết kế chiến lược hunt.

Giai đoạn này xác định:

- Data source nào cần phân tích.
- Query nào cần chạy.
- IOC/IOA nào cần tìm.
- Pattern nào là bất thường.
- Tool nào sẽ dùng.
- Time range là bao lâu.
- Cách validate finding.
- Khi nào cần escalation.

---

## 4.1. Data source thường dùng

Các data source phổ biến:

| Data source | Dùng để tìm gì |
|---|---|
| SIEM logs | Correlation nhiều nguồn log |
| EDR telemetry | Process, file, registry, network |
| DNS logs | C2 domain, DGA, tunneling |
| Proxy logs | URL, user-agent, outbound web |
| Firewall logs | Network connection |
| Email gateway logs | Phishing, malicious attachment |
| Windows Event Logs | Logon, privilege, account activity |
| Sysmon | Process, image load, LSASS access |
| NetFlow/PCAP | Network pattern, exfiltration |
| Cloud audit logs | IAM abuse, API activity |

---

## 4.2. Thiết kế query

Ví dụ hypothesis:

```text
Attacker dùng PowerShell encoded command sau phishing.
```

Query ý tưởng:

```text
process.name:"powershell.exe" AND process.command_line:*"-enc"*
```

Data source cần:

- Sysmon Event ID 1.
- EDR process telemetry.
- Windows PowerShell logs.
- Email gateway logs.
- Proxy/DNS logs.

---

# 5. Data Gathering and Examination

Đây là giai đoạn hunt thực sự.

Team thu thập và phân tích dữ liệu để tìm bằng chứng ủng hộ hoặc bác bỏ hypothesis.

Các kỹ thuật phân tích có thể gồm:

- Keyword search.
- Statistical analysis.
- Behavioral analysis.
- Frequency analysis.
- Rare event analysis.
- Baseline comparison.
- Timeline analysis.
- IOC matching.
- Graph/correlation analysis.
- Signature-based detection.
- Anomaly detection.

---

## 5.1. Tính lặp của quá trình hunt

Threat hunting thường không đi theo đường thẳng.

Trong quá trình phân tích, team có thể:

- Tinh chỉnh hypothesis.
- Mở rộng data source.
- Thu hẹp time range.
- Thêm IOC mới.
- Thay đổi query.
- Tìm thêm host liên quan.
- Chuyển từ hypothesis này sang hypothesis khác.

Ví dụ:

```text
Ban đầu hunt C2 domain.
Sau đó phát hiện process tạo kết nối là powershell.exe.
Team mở rộng hunt sang process creation và parent process.
```

---

# 6. Evaluating Findings and Testing Hypotheses

Sau khi có kết quả, team cần đánh giá phát hiện.

Cần trả lời:

- Finding có xác nhận hypothesis không?
- Finding có phải false positive không?
- Có bao nhiêu host bị ảnh hưởng?
- Có user nào liên quan?
- Có IOC nào mới không?
- Có dấu hiệu persistence không?
- Có lateral movement không?
- Có data exfiltration không?
- Impact tiềm năng là gì?
- Có cần incident response không?

---

## 6.1. Ví dụ đánh giá finding

Nếu hypothesis là:

```text
Attacker đang brute force credential.
```

Finding có thể là:

```text
Nhiều failed login từ một IP liên quan threat actor.
```

Nếu thấy thêm:

```text
Failed logon nhiều lần
      ↓
Successful logon
      ↓
Special privileges assigned
      ↓
RDP session
```

thì finding mạnh hơn và có thể cần escalation.

---

# 7. Mitigating Threats

Nếu hunt xác nhận có threat thật, team cần thực hiện remediation hoặc kích hoạt Incident Response.

Các hành động có thể gồm:

- Isolate affected systems.
- Block IOC.
- Disable compromised accounts.
- Reset password.
- Revoke tokens/sessions.
- Remove malware.
- Patch vulnerability.
- Update EDR/SIEM rule.
- Adjust firewall/proxy policy.
- Collect forensic evidence.
- Start incident response process.

---

## 7.1. Ví dụ mitigation

Nếu phát hiện host đang kết nối C2:

```text
1. Isolate host khỏi network.
2. Thu thập triage package.
3. Kiểm tra process tree.
4. Block C2 domain/IP.
5. Hunt các host khác có cùng IOC.
6. Reset credential user liên quan.
7. Điều tra initial access.
```

---

# 8. After the Hunt

Sau khi hunt kết thúc, cần document và chia sẻ kết quả.

Đây là bước rất quan trọng vì hunt không chỉ để tìm threat, mà còn để cải thiện năng lực phòng thủ.

Cần thực hiện:

- Document findings.
- Document methodology.
- Document queries used.
- Update threat intelligence platform.
- Add new IOC.
- Improve detection rules.
- Tune SIEM/EDR analytics.
- Refine IR playbooks.
- Update security policies.
- Share learning với SOC/IR/IT.
- Tạo follow-up tasks.

---

## 8.1. Hunt report nên có gì?

Một hunt report nên gồm:

```text
Hunt name:
Date/time:
Hunter/team:
Hypothesis:
Threat intelligence source:
MITRE ATT&CK mapping:
Data sources:
Queries:
Time range:
Findings:
False positives:
True positives:
Affected assets:
Impact:
Actions taken:
Recommended detections:
Lessons learned:
Follow-up tasks:
```

---

# 9. Continuous Learning and Enhancement

Threat hunting không phải hoạt động một lần.

Mỗi hunt nên giúp team học và cải thiện.

Sau mỗi chu kỳ hunt, team cần review:

- Hypothesis có tốt không?
- Data source có đủ không?
- Query có hiệu quả không?
- Tool có hạn chế gì không?
- False positive đến từ đâu?
- Có blind spot nào không?
- Detection nào cần thêm?
- IR playbook nào cần sửa?
- Có cần thêm logging không?

Thông điệp quan trọng:

```text
Mỗi hunt nên làm cho hunt tiếp theo tốt hơn.
```

---

# 10. Threat Hunting là Art và Science

Threat hunting vừa là khoa học, vừa là nghệ thuật.

Nó cần khoa học vì phải dựa trên:

- Data.
- Telemetry.
- Query.
- Statistics.
- Baseline.
- Detection engineering.

Nó cũng cần nghệ thuật vì hunter cần:

- Sáng tạo.
- Tư duy attacker.
- Hiểu môi trường.
- Đặt câu hỏi tốt.
- Nhìn ra pattern ẩn.
- Biết khi nào cần đào sâu.

---

# 11. Threat Hunting Process VS Emotet

Phần này mô phỏng cách áp dụng quy trình threat hunting để hunt **Emotet malware** trong tổ chức.

Emotet thường liên quan đến:

- Phishing emails.
- Malicious Word documents.
- Macro payloads.
- Compromised email accounts.
- C2 communication.
- Malware loader behavior.
- Lateral movement hoặc payload delivery tiếp theo.

---

# 12. Emotet - Setting the Stage

Trong giai đoạn chuẩn bị, team cần nghiên cứu:

- Emotet TTPs.
- Infection vectors.
- Previous Emotet campaigns.
- Threat intelligence reports.
- Malware samples nếu có.
- Email subject patterns.
- Attachment types.
- Known C2 infrastructure.
- Critical assets dễ bị ảnh hưởng.

Data source cần chuẩn bị:

- Email gateway logs.
- Mailbox audit logs.
- Endpoint telemetry.
- DNS logs.
- Proxy logs.
- Firewall logs.
- Sandbox results.
- SIEM correlation data.

---

# 13. Emotet - Formulating Hypotheses

Một hypothesis có thể là:

```text
Emotet đang sử dụng compromised email accounts để gửi phishing email có malicious Word documents chứa macro.
```

Hypothesis này cụ thể và có thể kiểm chứng vì ta có thể tìm:

- Email gửi từ compromised account.
- Attachment là Word document.
- Macro execution.
- Office spawning child process.
- DNS/HTTP connection đến C2.
- Payload download.

---

# 14. Emotet - Designing the Hunt

Team cần xác định data source và query.

Data source:

| Data source | Mục tiêu |
|---|---|
| Email logs | Tìm phishing email |
| Endpoint logs | Tìm Office spawn process |
| DNS logs | Tìm C2 domain |
| Proxy logs | Tìm payload download |
| EDR telemetry | Tìm process/file/registry |
| Sandbox | Phân tích attachment |

IOC/Pattern cần tìm:

- Suspicious email subject.
- Word document attachment.
- Office macro execution.
- `winword.exe` spawn `powershell.exe`.
- External network connection sau khi mở Office.
- Known Emotet C2 IP/domain.
- File hash liên quan Emotet.

---

# 15. Emotet - Data Gathering and Examination

Trong giai đoạn hunt, team phân tích dữ liệu.

Ví dụ:

- Xem email server logs để tìm attachment đáng ngờ.
- Phân tích email header.
- Kiểm tra mailbox gửi hàng loạt email lạ.
- Tìm endpoint mở attachment.
- Tìm Office process tạo child process.
- Tìm DNS query đến domain lạ.
- Tìm network connection đến C2.
- Tìm file payload được tạo trên endpoint.
- Chạy sandbox để xác định behavior.

Ví dụ pattern:

```text
outlook.exe
      ↓
winword.exe
      ↓
powershell.exe
      ↓
HTTP request to unknown domain
      ↓
payload.exe created
```

---

# 16. Emotet - Evaluating Findings and Testing Hypotheses

Sau khi phân tích, team cần xác định hypothesis đúng hay sai.

Finding có thể xác nhận hypothesis nếu thấy:

- Email có subject/attachment giống campaign Emotet.
- Attachment chứa macro.
- Endpoint mở file và sinh process đáng ngờ.
- Có C2 connection đến known Emotet infrastructure.
- Có payload hoặc hash liên quan Emotet.
- Nhiều user nhận cùng mẫu email.

Cần đánh giá:

- Bao nhiêu mailbox bị ảnh hưởng?
- Bao nhiêu endpoint đã mở file?
- Host nào có execution?
- Có lateral movement không?
- Có credential theft không?
- Có payload thứ hai không?
- Có data exfiltration không?

---

# 17. Emotet - Mitigating Threats

Nếu xác nhận Emotet infection, hành động có thể gồm:

- Isolate affected endpoints.
- Remove malware.
- Block C2 IP/domain.
- Quarantine malicious emails.
- Remove emails from mailboxes.
- Disable compromised email accounts.
- Reset passwords.
- Revoke sessions.
- Patch exploited vulnerability nếu có.
- Update EDR/SIEM detections.
- Hunt toàn bộ môi trường với IOC mới.

---

# 18. Emotet - After the Hunt

Sau hunt, team cần:

- Document finding.
- Update threat intelligence platform.
- Share Emotet IOC.
- Update SIEM detection rules.
- Update EDR block rule.
- Refine phishing response playbook.
- Improve email gateway detection.
- Train users về phishing.
- Add hunting query vào playbook.
- Tạo report cho management nếu cần.

---

# 19. Emotet - Continuous Learning

Emotet thay đổi TTP theo thời gian.

Team cần:

- Theo dõi threat intelligence mới.
- Cập nhật IOC.
- Cập nhật detection logic.
- Điều chỉnh sandbox analysis.
- Cải thiện phishing detection.
- Tập luyện tabletop hoặc purple team.
- Review hiệu quả của hunt.

---

# 20. Bảng quy trình nhanh

| Giai đoạn | Mục tiêu | Output |
|---|---|---|
| Setting the Stage | Chuẩn bị mục tiêu, tool, data | Scope, data sources |
| Formulating Hypotheses | Đặt giả thuyết có thể test | Hypothesis |
| Designing the Hunt | Thiết kế query/phương pháp | Hunt plan |
| Data Gathering | Thu thập và phân tích dữ liệu | Findings |
| Evaluating Findings | Xác minh hypothesis | True/false positive |
| Mitigating Threats | Xử lý threat thật | Containment/remediation |
| After the Hunt | Document và cải thiện | Report, rules, playbooks |
| Continuous Learning | Cải tiến liên tục | Better hunts |

---

# 21. Bảng ôn nhanh thuật ngữ

| Thuật ngữ | Nghĩa ngắn gọn |
|---|---|
| Threat Hunting Process | Quy trình chủ động tìm threat |
| Setting the Stage | Chuẩn bị hunt |
| Hypothesis | Giả thuyết cần kiểm chứng |
| Hunt Design | Thiết kế chiến lược hunt |
| Data Gathering | Thu thập và phân tích dữ liệu |
| Evaluation | Đánh giá finding |
| Mitigation | Hành động xử lý threat |
| After the Hunt | Document và cải thiện sau hunt |
| Emotet | Malware/phishing campaign nổi tiếng |
| IOC | Indicator of Compromise |
| TTP | Tactics, Techniques and Procedures |
| C2 | Command and Control |
| Baseline | Hành vi bình thường |
| Hunt Report | Báo cáo kết quả threat hunt |

---

# 22. Câu hỏi ôn tập

## 1. Threat hunting process gồm những bước chính nào?

Các bước chính gồm setting the stage, formulating hypotheses, designing the hunt, data gathering, evaluating findings, mitigating threats, after the hunt và continuous learning.

## 2. Hypothesis trong threat hunting là gì?

Hypothesis là giả thuyết có cơ sở và có thể kiểm chứng về hành vi attacker có thể đang xảy ra trong môi trường.

## 3. Vì sao data gathering là bước lặp?

Vì trong quá trình phân tích, team có thể phát hiện thông tin mới và phải tinh chỉnh hypothesis, query hoặc mở rộng data source.

## 4. After the Hunt cần làm gì?

Cần document findings, update IOC, cải thiện detection rules, refine IR playbooks và chia sẻ bài học.

## 5. Trong ví dụ Emotet, data source nào quan trọng?

Các data source quan trọng gồm email logs, endpoint telemetry, DNS logs, proxy logs, EDR logs và sandbox results.

## 6. Emotet hypothesis có thể là gì?

Một hypothesis là Emotet sử dụng compromised email accounts để gửi phishing email có malicious Word documents chứa macro.

## 7. Vì sao continuous learning quan trọng?

Vì threat landscape thay đổi liên tục, hunter cần cập nhật hypothesis, methodology, tool và detection logic sau mỗi hunt.

---

# Key Takeaway

Threat hunting là một vòng lặp cải tiến liên tục.

Một hunt hiệu quả cần bắt đầu bằng preparation tốt, hypothesis rõ ràng, data source phù hợp, phân tích có phương pháp và kết thúc bằng hành động cải thiện detection/response.

Ví dụ Emotet cho thấy threat hunting không chỉ là tìm IOC, mà là hiểu toàn bộ attack chain từ phishing email, macro execution, endpoint behavior, C2 communication đến remediation và lessons learned.

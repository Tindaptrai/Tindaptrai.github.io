---
layout: post
title: "Threat Intelligence Fundamentals"
date: 2026-06-20 22:46:00 +0700
categories: [SOC, Threat Intelligence]
tags: [soc, threat-intelligence, cti, threat-hunting, strategic-intelligence, operational-intelligence, tactical-intelligence, ioc, ttp, emotet, blue-team]
description: "Ghi chú tổng hợp về Cyber Threat Intelligence: định nghĩa CTI, bốn tiêu chí Relevance/Timeliness/Actionability/Accuracy, strategic/operational/tactical intelligence và cách đọc tactical report."
toc: true
---

# Threat Intelligence Fundamentals

**Cyber Threat Intelligence (CTI)** là thông tin tình báo về mối đe dọa mạng đã được thu thập, phân tích, đánh giá và chuyển thành insight có thể dùng để phòng thủ.

CTI giúp SOC chuyển từ tư duy chỉ phản ứng sau sự cố sang tư duy **chủ động dự đoán, chuẩn bị và ngăn chặn threat** trước khi thiệt hại xảy ra.

```text
Cyber Threat Intelligence = thông tin có ngữ cảnh giúp phòng thủ trước adversary.
```

---

## 1. CTI là gì?

CTI cung cấp thông tin đã được xử lý về:

- Adversary.
- Threat actor.
- Campaign.
- Malware.
- IOC.
- TTP.
- Infrastructure.
- Vulnerability.
- Targeted industry.
- Attack pattern.
- Business impact.

CTI giúp trả lời:

```text
Ai có thể tấn công mình?
Họ tấn công bằng cách nào?
Họ nhắm vào tài sản nào?
Họ có khả năng gì?
Mình cần phòng thủ ra sao?
```

---

## 2. Vai trò của CTI trong SOC

SOC có thể dùng CTI để:

- Làm giàu alert.
- Ưu tiên incident.
- Xây dựng detection rule.
- Tạo threat hunting hypothesis.
- Cập nhật IOC vào SIEM, EDR, Firewall, Proxy.
- Hiểu attacker TTP.
- Dự đoán campaign có thể ảnh hưởng đến tổ chức.
- Cải thiện incident response playbook.
- Hỗ trợ lãnh đạo ra quyết định về risk.

Ví dụ:

```text
CTI báo ransomware group đang nhắm vào ngành tài chính
      ↓
SOC kiểm tra exposure, log source và detection coverage
      ↓
Threat hunter hunt dấu hiệu initial access/lateral movement
      ↓
Detection engineer tạo rule mới
```

---

# 3. Bốn tiêu chí của CTI có giá trị

Một CTI tốt cần có bốn yếu tố:

```text
Relevant
Timely
Actionable
Accurate
```

---

## 3.1. Relevance

**Relevance** nghĩa là intelligence phải liên quan đến tổ chức.

Không phải threat intel nào cũng quan trọng với mọi doanh nghiệp. Nếu có CVE nghiêm trọng trong phần mềm mà tổ chức không dùng, mức ưu tiên xử lý sẽ thấp hơn.

Cần hỏi:

- Threat này có liên quan đến ngành của mình không?
- Công nghệ bị ảnh hưởng có trong môi trường không?
- Threat actor này có từng nhắm vào tổ chức tương tự không?
- IOC/TTP này có khả năng xuất hiện trong hệ thống của mình không?

---

## 3.2. Timeliness

**Timeliness** nghĩa là thông tin phải kịp thời.

Threat intelligence cũ có thể giảm giá trị vì attacker có thể đã đổi:

- IP.
- Domain.
- Malware hash.
- C2 infrastructure.
- TTP.
- Campaign.

Thông tin càng mới càng có giá trị cho phòng thủ.

---

## 3.3. Actionability

**Actionability** nghĩa là intelligence phải chuyển thành hành động được.

Ví dụ actionable intelligence:

```text
Block domain A.
Hunt process chain B.
Tạo SIEM rule cho technique C.
Kiểm tra endpoint có hash D.
Update EDR detection cho behavior E.
```

Nếu intelligence không dẫn đến hành động rõ ràng, giá trị của nó thấp.

---

## 3.4. Accuracy

**Accuracy** nghĩa là intelligence phải chính xác hoặc có confidence rõ ràng.

Thông tin sai có thể gây:

- Block nhầm dịch vụ hợp lệ.
- False positive.
- Tốn thời gian điều tra.
- Misattribution.
- Sai hướng incident response.
- Alert fatigue.

Nếu chưa chắc chắn, nên ghi confidence:

```text
Confidence: High / Medium / Low
```

---

# 4. CTI tạo ra giá trị gì?

Khi đủ relevance, timeliness, actionability và accuracy, CTI giúp:

- Hiểu adversary operations.
- Hiểu campaign có thể nhắm vào tổ chức.
- Làm giàu dữ liệu hiện có.
- Phát hiện TTP của attacker.
- Tạo mitigation hiệu quả.
- Hỗ trợ quyết định kinh doanh.
- Giảm risk.
- Chuẩn bị crisis action plan.
- Cải thiện detection và response.

---

# 5. Threat Intelligence vs Threat Hunting

| Tiêu chí | Threat Intelligence | Threat Hunting |
|---|---|---|
| Mục tiêu | Dự đoán adversary | Tìm adversary trong môi trường |
| Tính chất | Predictive | Proactive và reactive |
| Output | Report, IOC, TTP, assessment | Findings, detections, incidents |
| Câu hỏi chính | Ai sẽ tấn công, bằng gì, khi nào? | Có dấu hiệu attacker trong hệ thống không? |
| Người dùng | SOC, IR, Leadership, Risk team | SOC, IR, Hunter, Detection engineer |

Threat Intelligence hỗ trợ Threat Hunting bằng cách cung cấp IOC, TTP, campaign detail và adversary profile. Ngược lại, Threat Hunting cung cấp findings thực tế để CTI refine lại intelligence.

---

# 6. Ba loại Cyber Threat Intelligence

CTI thường được chia thành ba loại:

```text
Strategic Intelligence
Operational Intelligence
Tactical Intelligence
```

Ba loại này khác nhau về đối tượng sử dụng, độ chi tiết và mục tiêu.

---

## 6.1. Strategic Intelligence

**Strategic Intelligence** là intelligence cấp chiến lược.

Đối tượng sử dụng:

- C-level executives.
- VPs.
- Board members.
- Company leaders.
- Risk management.

Mục tiêu:

- Gắn intelligence với business risk.
- Hỗ trợ quyết định cấp cao.
- Hiểu adversary operations theo thời gian.
- Trả lời câu hỏi **Who?** và **Why?**

Ví dụ:

```text
Báo cáo về APT28/Fancy Bear mô tả động cơ, campaign trước đây,
nhóm mục tiêu, chiến lược dài hạn và sự thay đổi TTP theo thời gian.
```

---

## 6.2. Operational Intelligence

**Operational Intelligence** nằm giữa strategic và tactical.

Đối tượng sử dụng:

- Mid-level management.
- SOC manager.
- IR manager.
- Security operations leadership.
- Threat hunting lead.

Mục tiêu:

- Mô tả campaign cụ thể.
- Mô tả cách adversary vận hành.
- Cung cấp nhiều chi tiết hơn strategic report.
- Trả lời câu hỏi **How?** và **Where?**

Ví dụ:

```text
Báo cáo về REvil ransomware campaign mô tả initial access,
lateral movement, credential dumping và thời điểm triển khai ransomware.
```

---

## 6.3. Tactical Intelligence

**Tactical Intelligence** là intelligence cấp kỹ thuật, có thể hành động ngay.

Đối tượng sử dụng:

- SOC analyst.
- Incident responder.
- Threat hunter.
- Detection engineer.
- Network defender.

Ví dụ tactical intelligence:

- IP address.
- Domain.
- URL.
- File hash.
- Registry key.
- Mutex.
- File path.
- User-agent.
- Email subject.
- YARA/Sigma/Snort rule.

---

# 7. Strategic vs Operational vs Tactical Intelligence

| Loại intelligence | Đối tượng | Mục tiêu | Ví dụ |
|---|---|---|---|
| Strategic | Lãnh đạo | Business risk, who/why | APT report cấp cao |
| Operational | Quản lý/SOC lead | Campaign, how/where | Ransomware campaign analysis |
| Tactical | Analyst/Responder | IOC, rule, action | IP/hash/domain/YARA/Sigma |

Các loại intelligence này có overlap. Tactical intelligence có thể góp phần tạo operational picture, và operational picture có thể hỗ trợ strategic overview.

---

# 8. Cách đọc Tactical Threat Intelligence Report

Một tactical threat intelligence report thường chứa nhiều IOC và technical details.

Quy trình nên dùng:

```text
1. Understand report scope and narrative.
2. Identify and classify IOCs.
3. Understand attack lifecycle.
4. Analyze and validate IOCs.
5. Incorporate IOCs into security infrastructure.
6. Proactively threat hunt.
7. Monitor continuously and learn.
```

---

## 8.1. Understand Report Scope and Narrative

Cần hiểu bối cảnh tổng thể:

- Report nói về threat nào?
- Campaign đang diễn ra hay đã kết thúc?
- Ngành nào bị nhắm đến?
- Có liên quan đến tổ chức mình không?
- Attacker có mục tiêu gì?
- Report có confidence ra sao?

Ví dụ:

```text
Report mô tả Emotet campaign đang nhắm vào doanh nghiệp cùng ngành.
```

---

## 8.2. Identify and Classify IOCs

Nên phân loại IOC để xử lý dễ hơn.

| Loại IOC | Ví dụ |
|---|---|
| Network-based IOC | IP, domain, URL, C2 server |
| Host-based IOC | File hash, registry key, file path, mutex |
| Email-based IOC | Sender, subject, attachment name, message ID |
| Behavior-based IOC | User-Agent, HTTP header, DNS pattern, API call |

Sau đó có thể enrichment IOC bằng geolocation, WHOIS, passive DNS, VirusTotal, OTX hoặc threat intelligence platform.

---

## 8.3. Understand Attack Lifecycle

Không nên chỉ nhìn IOC rời rạc. Cần hiểu lifecycle và mapping với MITRE ATT&CK.

Ví dụ Emotet lifecycle:

```text
Initial Access: Phishing email
Execution: Macro/script execution
Persistence: Registry key/scheduled task
Defense Evasion: Obfuscation/process injection
Command and Control: C2 communication
Impact or Payload Delivery: Secondary malware/ransomware
```

---

## 8.4. Analysis and Validation of IOCs

Không phải IOC nào cũng chính xác hoặc còn hiệu lực.

Cần kiểm tra:

- IOC có mới không?
- Source có đáng tin không?
- IOC có false positive cao không?
- IOC có từng được whitelist không?
- IP/domain có dùng shared hosting không?
- Hash có đúng malware sample không?
- Confidence là High/Medium/Low?

---

## 8.5. Incorporate IOCs into Security Infrastructure

Sau khi xác thực, có thể đưa IOC vào hệ thống phòng thủ:

- Firewall block IP.
- Proxy block domain/URL.
- EDR block hash.
- IDS/IPS signature.
- SIEM correlation rule.
- Email gateway rule.
- DNS sinkhole.
- YARA/Sigma rule.

Cần đánh giá business impact trước khi block và document theo change management.

---

## 8.6. Proactive Threat Hunting

Sau khi có IOC/TTP từ report, SOC nên hunt trong môi trường.

Ví dụ Emotet:

- Tìm email subject/sender.
- Tìm attachment name.
- Tìm endpoint mở file Word.
- Tìm `winword.exe` spawn `powershell.exe`.
- Tìm DNS query đến C2 domain.
- Tìm proxy request đến URL lạ.
- Tìm payload hash.
- Tìm persistence artifact.

Query ý tưởng:

```text
process.parent.name:"winword.exe" AND process.name:"powershell.exe"
```

---

## 8.7. Continuous Monitoring and Learning

Sau khi đưa IOC vào hệ thống, cần tiếp tục monitor.

Nếu có hit:

- Tạo incident.
- Escalate theo playbook.
- Xác định affected host.
- Thu thập evidence.
- Hunt thêm IOC/TTP liên quan.
- Cập nhật detection rule.

Nếu phát hiện IOC/TTP mới, có thể chia sẻ lại cho cộng đồng threat intelligence hoặc ISAC/ISAO nếu phù hợp.

---

# 9. Ví dụ: Tactical Report về Emotet

Một report về Emotet campaign có thể chứa:

- C2 IP.
- Domain.
- File hash.
- Email subject.
- Attachment name.
- Macro behavior.
- Registry key.
- Mutex.
- PowerShell command pattern.
- User-Agent.
- MITRE ATT&CK mapping.

Quy trình xử lý:

```text
1. Xác định report có liên quan tổ chức không.
2. Phân loại IOC thành network, host, email.
3. Validate IOC với nguồn khác.
4. Đưa IOC đáng tin vào SIEM/EDR/Firewall/Email gateway.
5. Hunt theo IOC và behavior.
6. Kiểm tra endpoint, DNS, proxy, email logs.
7. Nếu có hit, kích hoạt incident response.
8. Cập nhật detection và awareness training.
```

---

# 10. Bảng ôn nhanh thuật ngữ

| Thuật ngữ | Nghĩa ngắn gọn |
|---|---|
| CTI | Cyber Threat Intelligence |
| Relevance | Mức độ liên quan với tổ chức |
| Timeliness | Tính kịp thời |
| Actionability | Khả năng chuyển thành hành động |
| Accuracy | Độ chính xác |
| Strategic Intelligence | Intelligence cho lãnh đạo/risk |
| Operational Intelligence | Intelligence về campaign/how/where |
| Tactical Intelligence | IOC/rule/action kỹ thuật |
| IOC | Indicator of Compromise |
| TTP | Tactics, Techniques and Procedures |
| C2 | Command and Control |
| Emotet | Malware/campaign thường dùng phishing |
| Enrichment | Làm giàu dữ liệu IOC |
| ISAC/ISAO | Cộng đồng chia sẻ threat intelligence |

---

# 11. Câu hỏi ôn tập

## 1. CTI là gì?

CTI là intelligence đã được phân tích và có ngữ cảnh để giúp tổ chức hiểu, dự đoán và phòng thủ trước cyber threat.

## 2. Bốn yếu tố làm CTI có giá trị là gì?

```text
Relevance
Timeliness
Actionability
Accuracy
```

## 3. Threat Intelligence khác Threat Hunting như thế nào?

Threat Intelligence thiên về dự đoán adversary và cung cấp insight, còn Threat Hunting chủ động tìm dấu hiệu adversary trong môi trường.

## 4. Strategic Intelligence dành cho ai?

Strategic Intelligence thường dành cho C-level executives, VPs và lãnh đạo để hỗ trợ quyết định business/risk.

## 5. Tactical Intelligence gồm những gì?

Tactical Intelligence gồm IOC, technical details, rule/query và thông tin có thể dùng ngay để detect, block hoặc hunt.

## 6. Khi đọc tactical report, vì sao cần validate IOC?

Vì IOC có thể sai, cũ, false positive cao hoặc ảnh hưởng business nếu block trực tiếp.

## 7. Với Emotet report, có nên chỉ hunt IOC không?

Không. Nên hunt cả IOC và behavior/TTP như Office spawn PowerShell, DNS query lạ, proxy request đến C2 hoặc payload execution.

---

# Key Takeaway

Cyber Threat Intelligence giúp SOC chuyển từ phòng thủ phản ứng sang phòng thủ chủ động và dự đoán.

Một CTI tốt phải relevant, timely, actionable và accurate. Ngoài IOC, CTI còn phải giúp hiểu adversary, campaign, TTP, lifecycle và business risk.

Khi đọc tactical threat intelligence report, SOC analyst nên phân loại IOC, validate, enrichment, đưa vào security controls, threat hunt chủ động và tiếp tục monitor để cải thiện detection/response.

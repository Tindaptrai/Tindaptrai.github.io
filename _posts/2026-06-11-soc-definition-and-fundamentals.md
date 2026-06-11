---
layout: post
title: "SOC Definition và Fundamentals"
date: 2026-06-11 01:00:00 +0700
categories: [SOC, Fundamentals]
tags: [soc, security-operations-center, blue-team, incident-response, siem, edr, threat-hunting, soc-analyst]
description: "Ghi chú tổng hợp về SOC: định nghĩa, cách hoạt động, vai trò trong SOC, mô hình tier analyst và sự phát triển từ SOC 1.0 đến SOC 2.0, Cognitive SOC và NextGen SOC."
toc: true
---

# SOC Definition và Fundamentals

**SOC** là viết tắt của:

```text
Security Operations Center
```

SOC là trung tâm vận hành an ninh mạng, nơi đội ngũ bảo mật liên tục giám sát, phân tích và phản ứng với các sự kiện an toàn thông tin trong tổ chức.

Nói ngắn gọn:

```text
SOC = nơi phát hiện, phân tích và xử lý sự cố an ninh mạng.
```

---

## 1. SOC là gì?

**Security Operations Center (SOC)** là một bộ phận hoặc trung tâm chuyên trách về an ninh mạng.

SOC thường bao gồm các chuyên gia bảo mật có nhiệm vụ:

- Giám sát trạng thái bảo mật của tổ chức.
- Phân tích cảnh báo bảo mật.
- Phát hiện hành vi đáng ngờ.
- Điều tra sự cố.
- Phối hợp xử lý incident.
- Cải thiện khả năng phát hiện và phản ứng.
- Báo cáo tình hình an ninh cho tổ chức.

Mục tiêu chính của SOC là giúp tổ chức phát hiện và phản ứng với các mối đe dọa càng sớm càng tốt, từ đó giảm thiểu thiệt hại nếu có security breach xảy ra.

---

## 2. Vì sao SOC quan trọng?

Trong môi trường doanh nghiệp, mỗi ngày có thể có rất nhiều sự kiện xảy ra:

- Người dùng đăng nhập.
- Máy chủ nhận request.
- Firewall ghi nhận traffic.
- Endpoint chạy process.
- Email gateway xử lý email.
- IDS/IPS phát hiện traffic đáng ngờ.
- EDR phát hiện hành vi bất thường.

Nếu không có SOC, tổ chức rất dễ bỏ sót dấu hiệu tấn công.

SOC giúp:

- Giám sát hệ thống liên tục.
- Phát hiện sớm incident.
- Phân tích alert có hệ thống.
- Phối hợp incident response.
- Giảm thời gian phát hiện và xử lý.
- Giảm tác động của sự cố.
- Cải thiện security posture của tổ chức.

---

# 3. SOC sử dụng những công nghệ nào?

SOC thường dùng nhiều công nghệ để giám sát và phát hiện threat.

Một số công nghệ quan trọng:

| Công nghệ | Vai trò |
|---|---|
| SIEM | Thu thập log, correlation, alerting |
| IDS/IPS | Phát hiện hoặc ngăn chặn xâm nhập mạng |
| EDR/XDR | Giám sát và phản ứng trên endpoint |
| Firewall | Kiểm soát traffic |
| Threat Intelligence | Cung cấp thông tin về mối đe dọa |
| SOAR | Tự động hóa response workflow |
| Case Management | Quản lý case điều tra |
| Vulnerability Scanner | Phát hiện lỗ hổng |
| Network Monitoring | Giám sát lưu lượng mạng |

Ví dụ:

```text
Firewall ghi nhận traffic lạ
      ↓
SIEM nhận log và tạo alert
      ↓
SOC analyst điều tra
      ↓
EDR kiểm tra endpoint liên quan
      ↓
Incident responder cô lập máy nếu cần
```

---

## 4. SOC không chỉ là công cụ

Một SOC hiệu quả không chỉ cần tool, mà còn cần:

- Con người có kỹ năng.
- Quy trình rõ ràng.
- Playbook xử lý incident.
- Chính sách phân quyền.
- Khả năng phối hợp với IT/IR/Legal.
- Logging đầy đủ.
- Detection rule tốt.
- Threat hunting liên tục.
- Báo cáo và cải tiến sau incident.

Nói cách khác:

```text
SOC = People + Process + Technology
```

---

# 5. SOC hoạt động như thế nào?

Chức năng chính của SOC là quản lý phần vận hành liên tục của an ninh thông tin doanh nghiệp.

SOC thường không tập trung chính vào việc thiết kế kiến trúc bảo mật hoặc xây dựng chiến lược dài hạn. Thay vào đó, SOC tập trung vào hoạt động hằng ngày:

- Detect.
- Assess.
- Respond.
- Report.
- Improve.
- Prevent.

Một workflow SOC cơ bản:

```text
Log/Event Generated
      ↓
SIEM/EDR/IDS creates alert
      ↓
Tier 1 triage
      ↓
Escalate if suspicious
      ↓
Tier 2/Tier 3 investigation
      ↓
Incident response
      ↓
Containment / Eradication / Recovery
      ↓
Reporting and lessons learned
```

---

## 6. Các năng lực nâng cao của SOC

Ngoài việc giám sát alert, một số SOC có năng lực nâng cao như:

- Digital Forensics.
- Malware Analysis.
- Threat Hunting.
- Detection Engineering.
- Threat Intelligence.
- Purple Teaming.
- Incident Response.
- Cloud Security Monitoring.
- Identity Threat Detection.
- Automation and SOAR.

Những năng lực này giúp SOC không chỉ phản ứng với alert, mà còn chủ động tìm kiếm dấu hiệu tấn công chưa được phát hiện.

---

# 7. Các vai trò trong SOC

Một SOC có thể có nhiều vai trò khác nhau tùy quy mô tổ chức.

Các vai trò phổ biến:

- SOC Director.
- SOC Manager.
- Tier 1 Analyst.
- Tier 2 Analyst.
- Tier 3 Analyst.
- Detection Engineer.
- Incident Responder.
- Threat Intelligence Analyst.
- Security Engineer.
- Compliance and Governance Specialist.
- Security Awareness and Training Coordinator.

---

## 7.1. SOC Director

**SOC Director** chịu trách nhiệm quản lý tổng thể và định hướng chiến lược cho SOC.

Nhiệm vụ:

- Quản lý ngân sách.
- Lập kế hoạch nhân sự.
- Định hướng chiến lược SOC.
- Đảm bảo SOC phù hợp với mục tiêu bảo mật của tổ chức.
- Báo cáo với lãnh đạo cấp cao.

---

## 7.2. SOC Manager

**SOC Manager** quản lý hoạt động hằng ngày của SOC.

Nhiệm vụ:

- Điều phối team analyst.
- Giám sát quy trình vận hành.
- Theo dõi hiệu suất SOC.
- Điều phối incident response.
- Phối hợp với các phòng ban khác.
- Đảm bảo alert/case được xử lý đúng SLA.

---

## 7.3. Tier 1 Analyst

**Tier 1 Analyst** còn gọi là first responder.

Nhiệm vụ chính:

- Theo dõi alert.
- Triage alert ban đầu.
- Kiểm tra alert là false positive hay suspicious.
- Thu thập thông tin ban đầu.
- Escalate incident lên Tier 2 nếu cần.

Ví dụ công việc:

```text
SIEM báo failed login bất thường
      ↓
Tier 1 kiểm tra user, source IP, thời gian, tần suất
      ↓
Nếu đáng ngờ thì escalate
```

---

## 7.4. Tier 2 Analyst

**Tier 2 Analyst** xử lý các alert đã được escalate.

Nhiệm vụ:

- Phân tích sâu incident.
- Tìm pattern và trend.
- Kiểm tra log liên quan.
- Xác định scope.
- Đề xuất mitigation.
- Hỗ trợ incident response.
- Tuning rule để giảm false positive.

Tier 2 cần hiểu rõ hơn về log, endpoint, network, attack technique và MITRE ATT&CK.

---

## 7.5. Tier 3 Analyst

**Tier 3 Analyst** là analyst có kinh nghiệm cao, xử lý các incident phức tạp.

Nhiệm vụ:

- Điều tra incident nghiêm trọng.
- Threat hunting.
- Malware analysis cơ bản hoặc nâng cao.
- Phát triển detection nâng cao.
- Hỗ trợ forensic.
- Phân tích attack chain.
- Cải thiện security posture.

Tier 3 thường xử lý các case có độ khó cao hoặc ảnh hưởng lớn đến tổ chức.

---

## 7.6. Detection Engineer

**Detection Engineer** chịu trách nhiệm xây dựng và duy trì các detection rule.

Nhiệm vụ:

- Viết rule cho SIEM.
- Viết rule cho IDS/IPS.
- Viết detection logic cho EDR/XDR.
- Phân tích detection gap.
- Tuning rule.
- Mapping rule với MITRE ATT&CK.
- Chuyển threat intelligence thành detection use case.

Ví dụ:

```text
Threat intel báo attacker dùng PowerShell encoded command
      ↓
Detection Engineer viết rule phát hiện powershell.exe -enc
      ↓
SOC dùng rule để alert
```

---

## 7.7. Incident Responder

**Incident Responder** chịu trách nhiệm xử lý incident đang diễn ra.

Nhiệm vụ:

- Điều tra chuyên sâu.
- Cô lập hệ thống.
- Thu thập bằng chứng.
- Containment.
- Eradication.
- Recovery.
- Phối hợp với IT để khôi phục dịch vụ.
- Ngăn incident tái diễn.

---

## 7.8. Threat Intelligence Analyst

**Threat Intelligence Analyst** thu thập và phân tích thông tin về mối đe dọa.

Nhiệm vụ:

- Theo dõi threat actor.
- Phân tích IOC.
- Phân tích TTP.
- Cập nhật threat landscape.
- Cung cấp intelligence cho SOC.
- Hỗ trợ threat hunting và detection engineering.

---

## 7.9. Security Engineer

**Security Engineer** triển khai, vận hành và bảo trì hạ tầng bảo mật.

Nhiệm vụ:

- Cấu hình SIEM.
- Cấu hình EDR.
- Quản lý firewall.
- Duy trì IDS/IPS.
- Tích hợp log source.
- Tối ưu pipeline log.
- Hỗ trợ SOC về mặt kỹ thuật.

---

## 7.10. Compliance and Governance Specialist

Vai trò này đảm bảo hoạt động bảo mật tuân thủ tiêu chuẩn, quy định và best practice.

Nhiệm vụ:

- Hỗ trợ audit.
- Quản lý compliance requirement.
- Kiểm tra policy.
- Chuẩn bị report.
- Đảm bảo log retention và monitoring đáp ứng yêu cầu.

---

## 7.11. Security Awareness and Training Coordinator

Vai trò này phụ trách đào tạo nhận thức bảo mật cho nhân viên.

Nhiệm vụ:

- Tổ chức training.
- Tạo chương trình security awareness.
- Mô phỏng phishing.
- Hướng dẫn nhân viên báo cáo sự cố.
- Xây dựng văn hóa bảo mật trong tổ chức.

---

# 8. Cấu trúc Tier trong SOC

SOC thường được tổ chức theo mô hình nhiều tier.

## Tier 1: Initial Triage

Tier 1 tập trung vào:

- Monitor alert.
- Triage ban đầu.
- Xác định mức độ ưu tiên.
- Loại bỏ false positive rõ ràng.
- Escalate case nghiêm trọng.

## Tier 2: Deeper Analysis

Tier 2 tập trung vào:

- Điều tra sâu.
- Phân tích log.
- Xác định scope.
- Tìm nguyên nhân.
- Đề xuất response action.
- Tuning security tools.

## Tier 3: Advanced Investigation

Tier 3 tập trung vào:

- Incident phức tạp.
- Threat hunting.
- Advanced detection.
- Forensics.
- Malware analysis.
- Cải thiện chiến lược phòng thủ.

Bảng tóm tắt:

| Tier | Vai trò chính | Mức độ |
|---|---|---|
| Tier 1 | Monitor và triage alert | Cơ bản |
| Tier 2 | Phân tích sâu và xác định scope | Trung cấp |
| Tier 3 | Điều tra nâng cao và threat hunting | Nâng cao |

---

# 9. SOC Stages

SOC đã phát triển qua nhiều giai đoạn, từ mô hình giám sát cơ bản đến mô hình chủ động, dự đoán và có hỗ trợ nhận thức.

Có thể chia thành:

```text
SOC 1.0
SOC 2.0
Cognitive SOC / NextGen SOC
```

---

## 9.1. SOC 1.0

**SOC 1.0** là giai đoạn đầu của SOC.

Đặc điểm:

- Tập trung nhiều vào network security.
- Tập trung vào perimeter.
- Các security layer chưa tích hợp tốt.
- Alert bị phân mảnh trên nhiều nền tảng.
- Nhiều task tồn đọng.
- Thiếu correlation.
- Phản ứng chủ yếu sau khi sự cố xảy ra.

Mô hình này thường mang tính:

```text
Post-facto response
```

Tức là phản ứng sau khi vấn đề đã xảy ra.

Vấn đề của SOC 1.0:

- Alert rời rạc.
- Thiếu visibility toàn diện.
- Dễ bỏ sót attack vector ngoài network perimeter.
- Không phù hợp với threat hiện đại.
- Phụ thuộc nhiều vào xử lý thủ công.

---

## 9.2. SOC 2.0

**SOC 2.0** xuất hiện khi threat trở nên phức tạp hơn.

Các threat hiện đại thường:

- Multi-vector.
- Persistent.
- Asynchronous.
- Có IOC bị che giấu.
- Dùng malware.
- Dùng botnet.
- Dùng nhiều giai đoạn tấn công.

SOC 2.0 được xây dựng dựa trên intelligence.

Đặc điểm:

- Tích hợp security telemetry.
- Dùng threat intelligence.
- Phân tích network flow.
- Dùng anomaly detection.
- Phân tích layer 7.
- Chuẩn bị trước incident.
- Học từ post-event analysis.
- Có vulnerability management.
- Có configuration management.
- Có dynamic risk management.
- Có in-depth forensics.
- Có refinement detection rule.

SOC 2.0 hướng đến:

```text
Designed Proactive
```

Tức là được thiết kế để chủ động hơn, nhưng không phải lúc nào cũng được thực hiện đầy đủ trong thực tế.

---

## 9.3. Cognitive SOC / NextGen SOC

**Cognitive SOC** hoặc **NextGen SOC** là mô hình SOC thế hệ mới.

Mục tiêu là khắc phục các hạn chế còn lại của SOC 2.0.

SOC 2.0 có thể đã có đầy đủ subsystem, nhưng vẫn gặp vấn đề:

- Thiếu kinh nghiệm vận hành.
- Thiếu phối hợp giữa business và security.
- Rule chưa phản ánh đúng business process.
- Thiếu incident response procedure chuẩn hóa.
- Thiếu recovery procedure chuẩn hóa.
- Analyst bị quá tải alert.

Cognitive SOC dùng các hệ thống học hỏi để hỗ trợ quyết định bảo mật.

Đặc điểm:

- Học từ dữ liệu lịch sử.
- Hỗ trợ ra quyết định.
- Phân tích pattern.
- Hỗ trợ threat research.
- Tự động hóa một phần triage.
- Cải thiện detection theo thời gian.
- Kết hợp AI/ML và analytics.
- Hướng tới proactive, assistive và predictive.

Có thể hiểu:

```text
NextGen SOC = Proactive + Assistive + Predictive
```

---

# 10. SOC 1.0 vs SOC 2.0 vs NextGen SOC

| Tiêu chí | SOC 1.0 | SOC 2.0 | NextGen/Cognitive SOC |
|---|---|---|---|
| Cách tiếp cận | Phản ứng sau sự cố | Chủ động theo intelligence | Chủ động, hỗ trợ và dự đoán |
| Trọng tâm | Network/perimeter | Threat intelligence và telemetry | AI/ML, automation, business context |
| Dữ liệu | Rời rạc | Tích hợp hơn | Hợp nhất và có ngữ cảnh |
| Alert | Thiếu correlation | Có correlation tốt hơn | Triage/correlation thông minh hơn |
| IR procedure | Chưa chuẩn hóa | Bắt đầu chuẩn hóa | Chuẩn hóa và cải tiến liên tục |
| Threat hunting | Ít | Có | Mạnh hơn và có hỗ trợ analytics |
| Mục tiêu | Phát hiện cơ bản | Nhận thức tình huống | Dự đoán và phản ứng nhanh |

---

# 11. Cognitive Support trong SOC

Cognitive SOC có thể dùng hệ thống hỗ trợ nhận thức để phân tích dữ liệu bảo mật.

Mục tiêu:

- Gom thông tin bảo mật tương quan.
- Hiểu context của offense.
- Phân tích observables.
- Tra cứu knowledge graph.
- Hỗ trợ threat research.
- Đề xuất kết quả điều tra.
- Áp dụng intelligence để đánh giá incident.

Ví dụ workflow nhận thức:

```text
Correlated enterprise security information
      ↓
Offenses
      ↓
Local context and research strategy
      ↓
Observables
      ↓
Threat research and knowledge graph
      ↓
Research results
      ↓
Apply intelligence and qualify incident
```

Nói dễ hiểu: hệ thống hỗ trợ analyst hiểu sự kiện nhanh hơn bằng cách kết hợp alert, context, observables và threat intelligence.

---

# 12. SOC phối hợp với Incident Response như thế nào?

SOC thường là nơi phát hiện alert và điều tra ban đầu.

Khi sự kiện được xác nhận là incident, SOC phối hợp với Incident Response team.

Ví dụ:

```text
SOC phát hiện alert
      ↓
Tier 1 triage
      ↓
Tier 2 xác minh suspicious activity
      ↓
Tạo case incident
      ↓
Incident Responder thực hiện containment
      ↓
SOC tiếp tục monitoring
      ↓
Post-incident report và lessons learned
```

SOC và IR cần phối hợp chặt để đảm bảo sự cố được phát hiện, xử lý và khôi phục đúng quy trình.

---

# 13. SOC và Threat Hunting

SOC không chỉ chờ alert.

Một SOC trưởng thành còn thực hiện **Threat Hunting**.

Threat hunting là quá trình chủ động tìm kiếm dấu hiệu attacker trong môi trường, ngay cả khi chưa có alert rõ ràng.

Ví dụ hunt:

- Tìm PowerShell encoded command.
- Tìm login từ quốc gia lạ.
- Tìm lateral movement qua SMB/RDP.
- Tìm process bất thường.
- Tìm beaconing C2.
- Tìm service mới được tạo.
- Tìm scheduled task đáng ngờ.

Threat hunting giúp phát hiện các attack bị rule truyền thống bỏ sót.

---

# 14. Bảng ôn nhanh thuật ngữ

| Thuật ngữ | Nghĩa ngắn gọn |
|---|---|
| SOC | Security Operations Center |
| Security Monitoring | Giám sát an ninh |
| SOC Analyst | Người phân tích cảnh báo bảo mật |
| SOC Director | Người quản lý chiến lược SOC |
| SOC Manager | Người quản lý vận hành SOC |
| Tier 1 Analyst | Analyst triage alert ban đầu |
| Tier 2 Analyst | Analyst điều tra sâu hơn |
| Tier 3 Analyst | Analyst xử lý case phức tạp/threat hunting |
| Detection Engineer | Người viết và duy trì detection rule |
| Incident Responder | Người xử lý incident |
| Threat Intelligence Analyst | Người phân tích threat intelligence |
| Security Engineer | Người triển khai và vận hành công cụ bảo mật |
| SIEM | Công cụ thu thập, correlation và alert log |
| IDS/IPS | Hệ thống phát hiện/ngăn chặn xâm nhập |
| EDR | Giám sát và phản ứng trên endpoint |
| Threat Hunting | Chủ động tìm dấu hiệu tấn công |
| SOC 1.0 | SOC phản ứng cơ bản, tập trung network/perimeter |
| SOC 2.0 | SOC dựa trên intelligence và telemetry |
| Cognitive SOC | SOC có hỗ trợ nhận thức/AI/analytics |
| NextGen SOC | SOC chủ động, hỗ trợ và dự đoán |

---

# 15. Câu hỏi ôn tập

## 1. SOC là gì?

SOC là trung tâm vận hành an ninh mạng, nơi đội ngũ bảo mật giám sát, phân tích và phản ứng với các sự kiện an toàn thông tin.

## 2. Mục tiêu chính của SOC là gì?

Mục tiêu chính là phát hiện, phân tích và xử lý cybersecurity incidents càng sớm càng tốt để giảm thiểu thiệt hại.

## 3. Tier 1 Analyst làm gì?

Tier 1 Analyst theo dõi alert, triage ban đầu và escalate các sự kiện đáng ngờ lên tier cao hơn.

## 4. Tier 2 Analyst khác Tier 1 như thế nào?

Tier 2 phân tích sâu hơn, xác định scope, tìm pattern, đề xuất mitigation và hỗ trợ incident response.

## 5. Tier 3 Analyst làm gì?

Tier 3 xử lý incident phức tạp, threat hunting, forensic, malware analysis và phát triển detection nâng cao.

## 6. Detection Engineer có vai trò gì?

Detection Engineer viết, triển khai, tuning và duy trì detection rule cho SIEM, IDS/IPS, EDR và các công cụ monitoring.

## 7. SOC 1.0 có hạn chế gì?

SOC 1.0 thường tập trung vào network/perimeter, thiếu integration, thiếu correlation và phản ứng chủ yếu sau sự cố.

## 8. SOC 2.0 cải thiện điều gì?

SOC 2.0 tích hợp threat intelligence, telemetry, network flow analysis, anomaly detection và post-event learning.

## 9. NextGen SOC khác gì?

NextGen SOC hướng đến proactive, assistive và predictive bằng cách dùng analytics, automation, AI/ML và business context.

---

# Key Takeaway

SOC là thành phần cốt lõi trong chiến lược an ninh mạng của tổ chức.

Một SOC hiệu quả cần kết hợp con người, quy trình và công nghệ để phát hiện, phân tích và phản ứng với incident.

SOC hiện đại không chỉ phản ứng với alert, mà còn phải chủ động threat hunting, cải thiện detection, học từ incident và phát triển từ mô hình phản ứng sang mô hình chủ động, hỗ trợ và dự đoán.

---

# References

- LinkedIn - Evolution of Security Operations Center 2.0 and Beyond: <https://www.linkedin.com/pulse/evolution-security-operations-center-20-beyond-krishnan-jagannathan/>

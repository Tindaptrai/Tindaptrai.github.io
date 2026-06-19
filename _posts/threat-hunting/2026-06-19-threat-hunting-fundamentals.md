---
layout: post
title: "Threat Hunting Fundamentals"
date: 2026-06-19 22:46:00 +0700
categories: [SOC, Threat Hunting]
tags: [soc, threat-hunting, threat-intelligence, incident-response, dwell-time, ttp, risk-assessment, blue-team, dfir]
description: "Ghi chú tổng hợp về Threat Hunting Fundamentals: định nghĩa threat hunting, dwell time, mối quan hệ với incident handling, cấu trúc team, thời điểm nên hunt và liên hệ với risk assessment."
toc: true
---

# Threat Hunting Fundamentals

**Threat Hunting** là hoạt động chủ động tìm kiếm dấu hiệu attacker trong môi trường, ngay cả khi hệ thống chưa tạo alert rõ ràng.

Nếu SIEM/EDR truyền thống thường thiên về phản ứng với cảnh báo, thì threat hunting yêu cầu analyst chủ động đặt giả thuyết, tìm kiếm dấu hiệu bất thường và xác minh xem attacker có đang tồn tại trong hệ thống hay không.

```text
Threat Hunting = chủ động đi tìm threat trước khi alert rõ ràng xuất hiện.
```

---

## 1. Dwell Time là gì?

**Dwell time** là khoảng thời gian từ lúc attacker xâm nhập thành công đến khi tổ chức phát hiện được sự cố.

Dwell time có thể kéo dài nhiều ngày, nhiều tuần hoặc nhiều tháng. Dwell time càng dài, attacker càng có nhiều thời gian để duy trì persistence, leo thang đặc quyền, di chuyển ngang, thu thập credential, tìm dữ liệu quan trọng và exfiltrate dữ liệu.

Mục tiêu lớn của threat hunting là:

```text
Reduce dwell time.
```

---

## 2. Threat Hunting là gì?

**Threat Hunting** là một hoạt động chủ động, do con người dẫn dắt, thường dựa trên giả thuyết để tìm kiếm threat đã vượt qua các cơ chế phòng thủ hiện có.

Threat hunting thường dựa trên:

- Hypothesis.
- Threat Intelligence.
- Attacker TTPs.
- Baseline hoạt động bình thường.
- High-fidelity telemetry.
- SIEM/EDR logs.
- Network data.
- Endpoint data.
- Knowledge về môi trường nội bộ.

Threat hunting không chỉ tìm IOC như IP, hash hoặc domain, mà tập trung nhiều hơn vào hành vi attacker.

---

## 3. Threat Hunting khác Alert Triage như thế nào?

| Tiêu chí | Alert Triage | Threat Hunting |
|---|---|---|
| Cách tiếp cận | Phản ứng với alert | Chủ động tìm threat |
| Điểm bắt đầu | Alert đã được tạo | Giả thuyết hoặc intelligence |
| Mục tiêu | Xác minh alert | Tìm attacker chưa bị phát hiện |
| Tính chất | Reactive | Proactive |
| Dữ liệu | Alert/context liên quan | Toàn bộ telemetry có thể dùng |
| Kết quả | Close/escalate incident | Phát hiện mới, rule mới, insight mới |

Ví dụ:

```text
Alert triage: SIEM báo failed logon nhiều lần.
Threat hunting: Chủ động tìm dấu hiệu password spraying dù chưa có alert.
```

---

# 4. Mục tiêu của Threat Hunting

Threat hunting hướng đến các mục tiêu chính:

- Giảm dwell time.
- Tìm attacker đang ẩn trong môi trường.
- Phát hiện kỹ thuật chưa có rule.
- Xác minh threat intelligence.
- Tìm IOC/IOA mới.
- Tăng detection coverage.
- Cải thiện incident response.
- Cải thiện security posture.
- Xây dựng detection rule mới.
- Tuning SIEM/EDR để giảm blind spot.

---

# 5. Threat Hunting dựa trên giả thuyết

Một hunt thường bắt đầu bằng một giả thuyết.

Ví dụ giả thuyết:

```text
Attacker có thể đang dùng PowerShell encoded command để chạy payload trong môi trường Windows.
```

Từ giả thuyết đó, hunter sẽ xác định:

- Technique liên quan là gì?
- MITRE ATT&CK ID nào?
- Log source nào cần dùng?
- Field nào cần truy vấn?
- Dữ liệu baseline như thế nào?
- Điều gì là bất thường?
- Nếu phát hiện thì xử lý ra sao?

---

## 5.1. Ví dụ hypothesis

```text
Hypothesis:
Nếu attacker đã có credential hợp lệ, họ có thể sử dụng RDP để lateral movement đến server quan trọng.

Data sources:
- Windows Event ID 4624
- Logon Type 10
- Source IP
- Destination host
- User privilege
- PAW/non-PAW context

Hunt objective:
Tìm RDP login bất thường từ host không phải PAW hoặc ngoài giờ làm việc.
```

---

# 6. Các thành phần quan trọng của Threat Hunting

## 6.1. Threat Intelligence

Threat Intelligence giúp hunter biết attacker có thể dùng TTP nào.

Ví dụ:

- Threat actor đang dùng PowerShell.
- Malware mới dùng DNS tunneling.
- Campaign mới khai thác VPN.
- IOC mới liên quan đến C2 infrastructure.
- TTP mới được map vào MITRE ATT&CK.

---

## 6.2. Hiểu attacker mindset

Hunter cần có khả năng suy nghĩ như attacker:

```text
Nếu tôi là attacker, tôi sẽ làm gì tiếp theo?
Tôi sẽ ẩn ở đâu?
Tôi sẽ dùng tool hợp pháp nào?
Tôi sẽ né detection bằng cách nào?
```

Tư duy này giúp hunter tìm các dấu hiệu mà rule truyền thống có thể bỏ sót.

---

## 6.3. Hiểu môi trường nội bộ

Threat hunting không thể hiệu quả nếu không hiểu môi trường.

Hunter cần biết:

- Network topology.
- Critical assets.
- Crown jewels.
- Domain controllers.
- Admin workstations.
- Normal user behavior.
- Normal service account behavior.
- Baseline network traffic.
- Normal PowerShell usage.
- Normal RDP/SSH/SMB usage.

Một hành vi bất thường trong môi trường này có thể là bình thường trong môi trường khác.

---

## 6.4. High-fidelity data

Threat hunting cần dữ liệu chất lượng cao.

Nguồn dữ liệu thường dùng:

- Windows Event Logs.
- Sysmon.
- EDR telemetry.
- DNS logs.
- Proxy logs.
- Firewall logs.
- VPN logs.
- Authentication logs.
- Cloud audit logs.
- NetFlow.
- Packet capture.
- Threat intelligence feeds.

---

# 7. Threat Hunting và Incident Handling

Threat hunting có quan hệ rất chặt với Incident Handling.

Các giai đoạn Incident Handling có thể liên hệ với threat hunting như sau:

```text
Preparation
Detection & Analysis
Containment, Eradication & Recovery
Post-Incident Activity
```

---

## 7.1. Preparation

Trong giai đoạn **Preparation**, tổ chức cần chuẩn bị quy tắc vận hành cho threat hunting.

Cần xác định:

- Hunting team được quyền làm gì?
- Khi nào được can thiệp?
- Khi phát hiện incident thì escalation ra sao?
- Hunter phối hợp với SOC/IR như thế nào?
- Evidence được lưu thế nào?
- Hunting hypothesis được document thế nào?
- Hunt findings được chuyển thành detection rule ra sao?

Một số tổ chức tích hợp threat hunting vào policy và procedure của Incident Handling, thay vì tạo bộ policy riêng.

---

## 7.2. Detection & Analysis

Trong giai đoạn **Detection & Analysis**, threat hunter có thể hỗ trợ điều tra.

Hunter giúp:

- Xác minh IOC có thật sự là incident không.
- Tìm thêm artifact bị bỏ sót.
- Mở rộng scope điều tra.
- Tìm host liên quan.
- Tìm dấu hiệu lateral movement.
- Tìm persistence.
- Tìm các TTP attacker đang dùng.
- Xác định attack chain.

Ví dụ:

```text
SOC phát hiện một endpoint có C2 beacon.
Threat hunter tìm thêm host khác có cùng DNS pattern hoặc cùng parent-child process chain.
```

---

## 7.3. Containment, Eradication and Recovery

Ở giai đoạn **Containment, Eradication and Recovery**, vai trò của hunter tùy theo tổ chức.

Một số nơi yêu cầu hunter hỗ trợ:

- Xác định full scope.
- Tìm host còn bị compromise.
- Kiểm tra persistence còn tồn tại không.
- Xác minh IOC đã bị loại bỏ chưa.
- Hunt lại sau containment để đảm bảo attacker không còn foothold.

Tuy nhiên, hunter không phải lúc nào cũng trực tiếp thực hiện containment hoặc recovery. Vai trò này cần được quy định trong procedure.

---

## 7.4. Post-Incident Activity

Trong giai đoạn **Post-Incident Activity**, hunter có thể đóng góp nhiều giá trị.

Hunter có thể:

- Đề xuất detection rule mới.
- Cập nhật hunting playbook.
- Đề xuất hardening.
- Cải thiện log source.
- Cập nhật threat model.
- Tạo use case mới cho SIEM.
- Rút bài học từ incident.
- Đề xuất cải thiện security posture.

---

# 8. Cấu trúc Threat Hunting Team

Một threat hunting team hiệu quả cần nhiều kỹ năng khác nhau.

| Vai trò | Nhiệm vụ chính |
|---|---|
| Threat Hunter | Chủ động tìm IOC/IOA/TTP trong môi trường |
| Threat Intelligence Analyst | Thu thập và phân tích threat intelligence |
| Incident Responder | Xử lý containment, eradication, recovery |
| Forensics Expert | Phân tích forensic, malware, timeline |
| Data Analyst / Data Scientist | Phân tích dữ liệu lớn, pattern, anomaly |
| Security Engineer / Architect | Thiết kế công cụ và security infrastructure |
| Network Security Analyst | Phân tích traffic và network behavior |
| SOC Manager | Điều phối team, quy trình và communication |

---

# 9. Khi nào nên Threat Hunt?

Threat hunting không nên chỉ diễn ra khi có sự cố. Nó nên là hoạt động liên tục, chủ động và có lịch định kỳ.

## 9.1. Khi có thông tin mới về adversary hoặc vulnerability

Nếu có thông tin mới về threat actor hoặc lỗ hổng ảnh hưởng đến hệ thống đang dùng, nên hunt ngay.

Ví dụ:

```text
Có CVE mới ảnh hưởng VPN đang dùng trong tổ chức.
```

Hunt objective:

```text
Tìm dấu hiệu exploit hoặc truy cập bất thường liên quan đến VPN.
```

---

## 9.2. Khi có IOC mới liên quan đến adversary đã biết

Nếu threat intelligence cung cấp IOC mới liên quan đến actor từng nhắm vào tổ chức hoặc ngành nghề của mình, nên hunt.

Ví dụ:

- Domain C2 mới.
- Hash malware mới.
- IP infrastructure mới.
- User-agent bất thường.
- Registry key persistence mới.
- Scheduled task name mới.

---

## 9.3. Khi phát hiện nhiều network anomalies

Một anomaly riêng lẻ có thể không nguy hiểm. Nhưng nhiều anomaly cùng lúc có thể là dấu hiệu attack.

Ví dụ:

- DNS query lạ tăng đột biến.
- Outbound traffic bất thường.
- SMB connection lạ.
- RDP đến nhiều server.
- VPN login từ quốc gia lạ.
- Endpoint tạo nhiều network connection hiếm.

---

## 9.4. Trong quá trình Incident Response

Khi có incident đã xác nhận, vẫn nên threat hunt song song.

Mục tiêu:

- Tìm host khác bị ảnh hưởng.
- Xác định full scope.
- Tìm lateral movement.
- Tìm persistence.
- Tìm additional IOC.
- Ngăn attacker quay lại.
- Hỗ trợ containment chính xác hơn.

---

## 9.5. Periodic Proactive Hunts

Threat hunting nên được thực hiện định kỳ.

Ví dụ:

- Weekly hunt.
- Monthly hunt.
- Quarterly hunt.
- Hunt theo MITRE tactic.
- Hunt theo threat actor.
- Hunt theo crown jewels.
- Hunt theo data source.

Thông điệp quan trọng:

```text
The best time to hunt is always now.
```

---

# 10. Threat Hunting và Risk Assessment

Risk Assessment giúp threat hunting có định hướng. Không nên hunt ngẫu nhiên. Nên ưu tiên dựa trên rủi ro.

Risk assessment giúp xác định:

- Critical assets.
- Crown jewels.
- Threat sources.
- Existing vulnerabilities.
- Business impact.
- Likelihood.
- Risk priority.
- Mitigation strategy.

---

## 10.1. Prioritizing Hunting Efforts

Nếu biết tài sản nào quan trọng nhất, hunter có thể ưu tiên hunt quanh các tài sản đó.

Ví dụ crown jewels:

- Domain Controllers.
- Database chứa dữ liệu khách hàng.
- Payment systems.
- Source code repository.
- Cloud admin accounts.
- VPN infrastructure.
- Backup servers.

---

## 10.2. Understanding Threat Landscape

Risk assessment giúp hiểu threat landscape.

Ví dụ:

- Ngành tài chính dễ bị phishing và credential theft.
- Healthcare dễ bị ransomware.
- Software company dễ bị supply chain attack.
- Cloud-heavy environment dễ bị IAM abuse.

Từ đó hunter xây hypothesis phù hợp.

---

## 10.3. Highlighting Vulnerabilities

Nếu tổ chức biết có vulnerability trong ứng dụng/hệ thống, hunter có thể tìm dấu hiệu exploit.

Ví dụ:

```text
Ứng dụng có lỗ hổng privilege escalation.
```

Hunt objective:

```text
Tìm user privilege anomalies hoặc process chain liên quan escalation.
```

---

## 10.4. Refining Incident Response Plans

Threat hunting và risk assessment cũng giúp cải thiện IR plan.

Ví dụ:

- Nếu crown jewel là database, cần IR playbook cho database compromise.
- Nếu risk cao là ransomware, cần playbook cho containment nhanh.
- Nếu risk là credential theft, cần playbook reset credential và revoke session.

---

# 11. Threat Hunting Process mẫu

Một quy trình hunting cơ bản:

```text
1. Select risk area or threat intel.
2. Build hypothesis.
3. Identify data sources.
4. Write queries.
5. Hunt across telemetry.
6. Analyze anomalies.
7. Validate findings.
8. Escalate true positives.
9. Document results.
10. Convert findings into detections.
```

---

## 11.1. Ví dụ hunt theo MITRE ATT&CK

```text
Tactic: Credential Access
Technique: OS Credential Dumping
Hypothesis: Attacker có thể truy cập LSASS để dump credential.
Data sources: Sysmon Event ID 10, Windows Event ID 4672, EDR telemetry.
Query: Process access to lsass.exe from non-security tools.
Outcome: Tìm process lạ truy cập LSASS và escalate nếu nghi ngờ.
```

---

# 12. Hunt output nên có gì?

Một hunt tốt cần có kết quả rõ ràng.

Hunt report nên gồm:

```text
Hunt name:
Hypothesis:
Risk area:
MITRE mapping:
Data sources:
Queries used:
Time range:
Findings:
False positives:
True positives:
Affected assets:
Recommended actions:
New detections:
Tuning notes:
Follow-up tasks:
```

---

# 13. Bảng ôn nhanh thuật ngữ

| Thuật ngữ | Nghĩa ngắn gọn |
|---|---|
| Threat Hunting | Chủ động tìm threat trong môi trường |
| Dwell Time | Thời gian từ compromise đến detection |
| Hypothesis | Giả thuyết hunting |
| TTP | Tactics, Techniques and Procedures |
| IOC | Indicator of Compromise |
| IOA | Indicator of Attack |
| Threat Intelligence | Tình báo mối đe dọa |
| Baseline | Hành vi bình thường |
| Crown Jewels | Tài sản quan trọng nhất |
| Risk Assessment | Đánh giá rủi ro |
| Incident Response | Ứng phó sự cố |
| DFIR | Digital Forensics and Incident Response |
| MITRE ATT&CK | Framework mô tả hành vi attacker |
| False Positive | Phát hiện sai |
| True Positive | Phát hiện đúng |

---

# 14. Câu hỏi ôn tập

## 1. Threat hunting là gì?

Threat hunting là hoạt động chủ động, do con người dẫn dắt, thường dựa trên giả thuyết để tìm kiếm threat ẩn trong môi trường.

## 2. Dwell time là gì?

Dwell time là khoảng thời gian từ lúc attacker xâm nhập đến khi tổ chức phát hiện được sự cố.

## 3. Mục tiêu chính của threat hunting là gì?

Mục tiêu chính là giảm dwell time và phát hiện threat đã vượt qua các cơ chế phòng thủ hiện có.

## 4. Threat hunting liên quan gì đến Incident Handling?

Threat hunting hỗ trợ preparation, detection & analysis, scope investigation, post-incident improvement và có thể hỗ trợ containment tùy tổ chức.

## 5. Khi nào nên threat hunt?

Nên hunt khi có threat intel mới, IOC mới, nhiều anomaly, trong quá trình incident response và theo lịch định kỳ.

## 6. Risk assessment giúp threat hunting như thế nào?

Risk assessment giúp ưu tiên hunt quanh critical assets, threat có khả năng xảy ra và khu vực có impact cao nhất.

## 7. Một hunt report nên có gì?

Nên có hypothesis, data sources, queries, findings, MITRE mapping, false positives, true positives, recommended actions và detection improvements.

---

# Key Takeaway

Threat hunting là bước chuyển từ phòng thủ bị động sang phòng thủ chủ động.

Một threat hunting program tốt giúp tổ chức giảm dwell time, tìm attacker sớm hơn, cải thiện detection coverage và tăng chất lượng incident response.

Threat hunting hiệu quả cần kết hợp threat intelligence, hiểu biết môi trường nội bộ, dữ liệu chất lượng cao, tư duy attacker, risk assessment và quy trình document rõ ràng.

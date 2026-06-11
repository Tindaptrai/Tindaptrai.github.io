---
layout: post
title: "SIEM Definition và Fundamentals"
date: 2026-06-11 04:00:00 +0700
categories: [SOC, SIEM]
tags: [soc, siem, log-management, threat-alerting, compliance, log-normalization, blue-team]
description: "Ghi chú tổng hợp về SIEM: định nghĩa, lịch sử phát triển, cách SIEM hoạt động, use cases, data flow, lợi ích và thuật ngữ quan trọng."
toc: true
---

# SIEM Definition và Fundamentals

**SIEM** là một trong những nền tảng quan trọng nhất trong SOC.

SIEM giúp tổ chức thu thập log, chuẩn hóa dữ liệu, tương quan sự kiện, tạo cảnh báo, hỗ trợ điều tra incident và đáp ứng yêu cầu compliance.

Nói ngắn gọn:

```text
SIEM = nơi tập trung log, phân tích sự kiện và phát hiện mối đe dọa.
```

---

## 1. SIEM là gì?

**SIEM** là viết tắt của:

```text
Security Information and Event Management
```

SIEM là giải pháp kết hợp giữa:

- Quản lý thông tin bảo mật.
- Quản lý sự kiện bảo mật.
- Thu thập log.
- Phân tích event.
- Tạo alert.
- Hỗ trợ incident response.
- Dashboard và reporting.

SIEM nhận dữ liệu từ nhiều nguồn khác nhau như firewall, IDS/IPS, endpoint, server, database, application, authentication system, cloud services và network devices.

Sau đó, SIEM chuẩn hóa, lưu trữ, phân tích và tạo cảnh báo khi phát hiện hành vi đáng ngờ.

---

## 2. Vì sao SIEM quan trọng?

Trong một tổ chức lớn, mỗi giờ có thể sinh ra hàng trăm đến hàng nghìn event.

Nếu không có SIEM, SOC analyst phải kiểm tra log rời rạc từ nhiều hệ thống khác nhau. Điều này rất khó và dễ bỏ sót dấu hiệu quan trọng.

SIEM giúp:

- Tập trung log từ nhiều nguồn.
- Tạo một giao diện giám sát chung.
- Tương quan các sự kiện liên quan.
- Phát hiện hành vi đáng ngờ.
- Tạo alert theo rule hoặc threshold.
- Hỗ trợ điều tra incident.
- Tạo report cho compliance.
- Giảm thời gian phản ứng sự cố.

Ví dụ:

```text
Firewall ghi nhận 5 lần đăng nhập sai
      ↓
Windows Server ghi nhận admin account bị lock
      ↓
VPN ghi nhận đăng nhập từ quốc gia lạ
      ↓
SIEM tương quan các event
      ↓
Tạo alert brute force hoặc account compromise
```

---

# 3. Sự phát triển của SIEM

Khái niệm SIEM xuất hiện từ sự kết hợp của hai công nghệ trước đó:

```text
SIM + SEM = SIEM
```

## SIM là gì?

**SIM** là viết tắt của **Security Information Management**.

SIM tập trung vào:

- Thu thập log.
- Lưu trữ log dài hạn.
- Tìm kiếm log.
- Reporting.
- Hỗ trợ compliance.
- Kết hợp log với threat intelligence.

SIM thiên về quản lý dữ liệu bảo mật và log lịch sử.

## SEM là gì?

**SEM** là viết tắt của **Security Event Management**.

SEM tập trung vào:

- Giám sát event bảo mật.
- Tương quan event.
- Tạo notification.
- Phát hiện event đáng ngờ.
- Gửi alert theo thời gian gần thực.

SEM thiên về phát hiện và phản ứng với event đang xảy ra.

## SIEM kết hợp SIM và SEM

| Thành phần | Trọng tâm |
|---|---|
| SIM | Thu thập, lưu trữ, báo cáo log |
| SEM | Giám sát, correlation, alerting |
| SIEM | Tập trung log, phân tích, cảnh báo và hỗ trợ incident response |

Nhờ vậy, SIEM trở thành nền tảng trung tâm cho threat detection và security monitoring.

---

# 4. SIEM hoạt động như thế nào?

Một SIEM thường hoạt động theo các bước:

```text
Data Collection
      ↓
Parsing
      ↓
Normalization
      ↓
Aggregation
      ↓
Correlation
      ↓
Alerting
      ↓
Investigation / Response
      ↓
Reporting
```

## 4.1. Data Collection

SIEM thu thập dữ liệu từ nhiều nguồn:

- Windows Event Logs.
- Linux syslog.
- Firewall logs.
- Proxy logs.
- DNS logs.
- VPN logs.
- EDR logs.
- Web server logs.
- Database logs.
- Cloud audit logs.
- IDS/IPS alerts.

Quá trình này còn gọi là **data ingestion** hoặc **data collection**.

## 4.2. Parsing

**Parsing** là quá trình tách log thô thành các trường dữ liệu dễ hiểu.

Ví dụ log thô:

```text
Jun 10 10:15:20 firewall deny src=10.10.10.5 dst=8.8.8.8 port=53
```

Sau parsing:

| Field | Value |
|---|---|
| timestamp | Jun 10 10:15:20 |
| device | firewall |
| action | deny |
| source_ip | 10.10.10.5 |
| destination_ip | 8.8.8.8 |
| destination_port | 53 |

## 4.3. Normalization

**Normalization** là chuẩn hóa log từ nhiều nguồn khác nhau về một format chung.

Ví dụ mỗi vendor có thể gọi IP nguồn khác nhau:

| Vendor | Field |
|---|---|
| Firewall A | src |
| Firewall B | sourceAddress |
| EDR | source.ip |
| SIEM normalized field | source_ip |

Normalization giúp SIEM dễ correlation và search hơn.

## 4.4. Aggregation

**Aggregation** là gom và tổng hợp log từ nhiều nguồn.

Ví dụ:

```text
Nhiều log đăng nhập sai
      ↓
Gom theo user
      ↓
Gom theo source IP
      ↓
Gom theo thời gian
```

Aggregation giúp giảm nhiễu và nhìn được pattern.

## 4.5. Correlation

**Correlation** là tương quan nhiều event để phát hiện hành vi có ý nghĩa bảo mật.

Một event đơn lẻ có thể chưa nguy hiểm:

```text
1 lần login fail
```

Nhưng nếu correlation nhiều event:

```text
50 lần login fail trong 5 phút
      +
Login thành công sau đó
      +
IP đến từ quốc gia lạ
```

Thì có thể là brute force thành công.

## 4.6. Alerting

Khi rule hoặc logic phát hiện được điều kiện đáng ngờ, SIEM tạo alert.

Alert có thể được gửi qua:

- Console.
- Email.
- Pop-up.
- Ticket.
- ChatOps.
- SMS.
- Phone call.
- SOAR/case management.

Alert giúp SOC biết cần kiểm tra sự kiện nào.

---

# 5. SIEM khác IDS/IPS như thế nào?

SIEM không thay thế IDS/IPS, firewall hoặc EDR.

SIEM hoạt động cùng các hệ thống đó bằng cách lấy log và phân tích tập trung.

| Công cụ | Vai trò |
|---|---|
| IDS | Phát hiện traffic đáng ngờ |
| IPS | Phát hiện và chặn traffic đáng ngờ |
| Firewall | Kiểm soát traffic vào/ra |
| EDR | Giám sát endpoint |
| SIEM | Tập trung log, correlation, alerting và reporting |

Ví dụ:

```text
IDS phát hiện exploit attempt
Firewall ghi nhận kết nối từ IP lạ
Web server log ghi nhận request bất thường
EDR ghi nhận process lạ
      ↓
SIEM gom các event lại
      ↓
Tạo alert có context đầy đủ hơn
```

---

# 6. SIEM Business Requirements & Use Cases

Một SIEM thường đáp ứng nhiều nhu cầu nghiệp vụ và bảo mật:

- Log aggregation.
- Log normalization.
- Threat alerting.
- Contextualization.
- Incident response.
- Compliance.
- Reporting.
- Dashboard.
- Investigation.
- Threat hunting.

## 6.1. Log Aggregation

**Log Aggregation** là thu thập và tập trung log từ nhiều hệ thống.

Nếu không có log aggregation, SOC sẽ thiếu visibility.

Log aggregation giúp SOC thấy được bức tranh tổng thể thay vì từng mảnh rời rạc.

## 6.2. Log Normalization

**Log Normalization** là chuẩn hóa log về format chung.

Lợi ích:

- Search dễ hơn.
- Correlation chính xác hơn.
- Dashboard nhất quán hơn.
- Rule dùng được cho nhiều nguồn log.
- Giảm phụ thuộc vào format riêng của từng vendor.

## 6.3. Threat Alerting

**Threat Alerting** là khả năng phát hiện và cảnh báo mối đe dọa.

SIEM có thể tạo alert dựa trên:

- Rule.
- Threshold.
- Correlation.
- Threat intelligence.
- Behavior analytics.
- Machine learning.
- AI-assisted detection.

Ví dụ alert:

- Nhiều lần login fail.
- Login từ quốc gia bất thường.
- PowerShell encoded command.
- Endpoint kết nối đến domain độc hại.
- Data transfer bất thường.

## 6.4. Contextualization

**Contextualization** là bổ sung ngữ cảnh cho alert.

Một alert tốt không chỉ nói:

```text
IP 10.10.10.5 kết nối đến IP lạ
```

Mà nên cho biết thêm:

- IP đó thuộc host nào.
- Host thuộc phòng ban nào.
- User nào đang đăng nhập.
- Host có critical không.
- Destination IP có nằm trong threat intelligence không.
- Có event liên quan trước đó không.

Context giúp giảm false positive và giúp analyst xử lý nhanh hơn.

## 6.5. Response

Một SIEM hiện đại có thể hỗ trợ response bằng cách tích hợp với:

- SOAR.
- Ticket system.
- EDR.
- Firewall.
- Email security gateway.
- TheHive.
- Slack/Teams.
- Automation scripts.

Ví dụ response:

```text
SIEM phát hiện account compromise
      ↓
Tạo alert
      ↓
Tạo ticket
      ↓
Gọi SOAR playbook
      ↓
Disable account tạm thời
      ↓
Thông báo SOC
```

---

# 7. Compliance

SIEM rất quan trọng trong compliance.

Nhiều tiêu chuẩn và quy định yêu cầu tổ chức phải:

- Thu thập log.
- Giám sát log.
- Lưu trữ log.
- Review log.
- Phát hiện sự kiện bảo mật.
- Có bằng chứng hệ thống được monitoring.

Ví dụ tiêu chuẩn/quy định:

- PCI DSS.
- HIPAA.
- GDPR.
- ISO 27001.
- Banking, Finance, Insurance, Healthcare requirements.

SIEM hỗ trợ compliance bằng:

- Log retention.
- Audit trail.
- Automated reporting.
- Security monitoring.
- Evidence for auditors.
- Dashboard theo yêu cầu kiểm toán.

---

# 8. Data Flow trong SIEM

Dữ liệu đi trong SIEM theo luồng cơ bản:

```text
Data Sources
      ↓
Log Collection / Ingestion
      ↓
Parsing
      ↓
Normalization
      ↓
Correlation Engine
      ↓
Detection Rules
      ↓
Alerts / Incidents
      ↓
Dashboards / Reports
```

## 8.1. Data Sources

Nguồn dữ liệu có thể là:

- PC.
- Server.
- Network device.
- Firewall.
- Database.
- Application.
- Cloud.
- Authentication system.
- IDS/IPS.
- EDR.

## 8.2. Ingestion

**Ingestion** là quá trình SIEM nhận log từ các nguồn.

Cách ingest có thể gồm:

- Agent.
- Syslog.
- API.
- Filebeat/Log shipper.
- Cloud connector.
- Event collector.
- Database connector.

## 8.3. Correlation Engine

**Correlation Engine** là phần xử lý logic để tìm mối quan hệ giữa các event.

Ví dụ rule correlation:

```text
Nếu một user có hơn 10 login fail trong 5 phút
và sau đó login success từ cùng IP
thì tạo alert brute force success.
```

## 8.4. Dashboard và Visualization

SIEM cung cấp dashboard giúp SOC nhìn nhanh tình trạng an ninh.

Ví dụ dashboard:

- Top alert.
- Failed logon trend.
- Malware detections.
- Firewall denies.
- Suspicious countries.
- High-risk users.
- Top destination domains.
- Endpoint health.
- Incident status.

---

# 9. Lợi ích của SIEM

## 9.1. Centralized Visibility

SIEM cung cấp cái nhìn tập trung về log và event trong toàn tổ chức.

Không cần mở từng hệ thống riêng lẻ để xem log.

## 9.2. Faster Incident Response

SIEM giúp SOC phản ứng nhanh hơn nhờ:

- Alert tập trung.
- Context rõ hơn.
- Timeline tốt hơn.
- Correlation tự động.
- Dashboard trực quan.
- Tích hợp ticket/case management.

## 9.3. Better Threat Detection

SIEM giúp phát hiện các pattern mà từng công cụ đơn lẻ khó thấy.

Ví dụ:

```text
Proxy thấy truy cập domain lạ
EDR thấy PowerShell chạy
DNS thấy query bất thường
Firewall thấy outbound traffic
      ↓
SIEM correlate thành possible malware infection
```

## 9.4. Reduced Alert Fatigue nếu tuning tốt

SIEM có thể sinh ra rất nhiều alert.

Nếu không tuning, SOC dễ bị ngập alert và nhiều false positive.

Một SIEM tốt cần:

- Rule tuning.
- Threshold phù hợp.
- Context enrichment.
- Suppression rule hợp lý.
- Severity chính xác.
- Use case rõ ràng.

## 9.5. Reporting và Audit

SIEM giúp tạo báo cáo:

- Compliance report.
- Incident report.
- Executive summary.
- Log review evidence.
- Audit trail.
- Detection metrics.
- SOC performance metrics.

## 9.6. AI và Behavior Analytics

SIEM hiện đại có thể tích hợp:

- Machine learning.
- User and Entity Behavior Analytics.
- AI-based alerting.
- Pattern analysis.
- Anomaly detection.

Ví dụ:

```text
User A thường đăng nhập từ Việt Nam giờ hành chính
      ↓
Hôm nay đăng nhập từ quốc gia lạ lúc 3 giờ sáng
      ↓
SIEM tạo alert anomaly
```

---

# 10. Ví dụ SIEM Use Cases

## Use Case 1: Brute Force Detection

```text
Condition:
- Same source IP
- More than 20 failed logins
- Within 5 minutes

Alert:
Possible brute force attack
```

## Use Case 2: Malware C2 Detection

```text
Condition:
- Endpoint connects to known malicious domain
- Domain appears in threat intelligence
- Process is suspicious

Alert:
Possible C2 communication
```

## Use Case 3: Suspicious PowerShell

```text
Condition:
- powershell.exe executed
- Command contains -enc or DownloadString
- Parent process is winword.exe

Alert:
Possible phishing payload execution
```

## Use Case 4: Privileged Account Abuse

```text
Condition:
- Admin account login
- Outside normal working hours
- From unusual source host
- Followed by privilege-related action

Alert:
Possible privileged account compromise
```

## Use Case 5: Data Exfiltration

```text
Condition:
- Large outbound data transfer
- To rare external destination
- From sensitive server
- Outside normal business hours

Alert:
Possible data exfiltration
```

---

# 11. Bảng ôn nhanh thuật ngữ

| Thuật ngữ | Nghĩa ngắn gọn |
|---|---|
| SIEM | Security Information and Event Management |
| SIM | Security Information Management |
| SEM | Security Event Management |
| Log Aggregation | Gom log từ nhiều nguồn |
| Log Normalization | Chuẩn hóa log về format chung |
| Data Ingestion | Quá trình đưa log vào SIEM |
| Parsing | Tách log thô thành field |
| Correlation | Tương quan nhiều event |
| Alert | Cảnh báo bảo mật |
| Event | Sự kiện được ghi nhận |
| Incident | Sự cố bảo mật cần xử lý |
| Dashboard | Bảng hiển thị trực quan |
| Visualization | Trực quan hóa dữ liệu |
| Threat Intelligence | Thông tin tình báo mối đe dọa |
| Contextualization | Bổ sung ngữ cảnh cho alert |
| Compliance | Tuân thủ quy định/tiêu chuẩn |
| Log Retention | Lưu trữ log theo thời gian quy định |
| False Positive | Cảnh báo sai |
| Alert Fatigue | Mệt mỏi do quá nhiều alert |
| Correlation Engine | Bộ máy tương quan sự kiện |
| UEBA | Phân tích hành vi user/entity |

---

# 12. Câu hỏi ôn tập

## 1. SIEM là gì?

SIEM là nền tảng thu thập, chuẩn hóa, phân tích log và event bảo mật từ nhiều nguồn để phát hiện threat, tạo alert, hỗ trợ incident response và compliance.

## 2. SIM và SEM khác nhau thế nào?

SIM tập trung vào thu thập, lưu trữ và reporting log. SEM tập trung vào giám sát event, correlation và alerting. SIEM kết hợp cả hai.

## 3. Vì sao normalization quan trọng?

Vì log từ nhiều nguồn có format khác nhau. Normalization giúp đưa chúng về format chung để search, correlation và alerting hiệu quả hơn.

## 4. Vì sao SIEM cần contextualization?

Vì alert không có context dễ gây false positive và làm analyst mất thời gian. Context giúp hiểu event ảnh hưởng hệ thống nào, user nào và có nghiêm trọng không.

## 5. SIEM có thay thế IDS/IPS không?

Không. SIEM không thay thế IDS/IPS. SIEM xử lý và tương quan log từ IDS/IPS, firewall, EDR và nhiều nguồn khác để phát hiện threat tốt hơn.

## 6. Vì sao SIEM quan trọng cho compliance?

Vì nhiều tiêu chuẩn yêu cầu tổ chức phải log, monitor, review và lưu trữ sự kiện bảo mật. SIEM cung cấp report, audit trail và evidence cho auditor.

---

# Key Takeaway

SIEM là nền tảng trung tâm của hoạt động SOC.

Một SIEM hiệu quả không chỉ thu thập log, mà còn phải chuẩn hóa dữ liệu, tương quan sự kiện, tạo alert có ngữ cảnh, hỗ trợ điều tra incident và cung cấp report cho compliance.

Tuy nhiên, SIEM không tự động giải quyết mọi vấn đề. Để SIEM thật sự có giá trị, tổ chức cần tuning rule, xây dựng use case rõ ràng, bổ sung context, giảm false positive và liên tục cải thiện detection.

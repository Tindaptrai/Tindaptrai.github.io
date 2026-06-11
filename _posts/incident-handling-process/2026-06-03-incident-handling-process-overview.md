---
layout: post
title: "Incident Handling Process Overview"
date: 2026-06-03 01:00:00 +0700
categories: [SOC, Incident Response]
tags: [soc, incident-response, incident-handling, nist, blue-team, dfir]
description: "Ghi chú tổng hợp về quy trình Incident Handling theo NIST, bao gồm Preparation, Detection & Analysis, Containment Eradication & Recovery và Post-Incident Activity."
toc: true
---

# Incident Handling Process Overview

Sau khi đã hiểu **Cyber Kill Chain** và các giai đoạn của một cuộc tấn công, SOC analyst có thể dự đoán tốt hơn bước tiếp theo của attacker và đề xuất biện pháp phản ứng phù hợp.

Tuy nhiên, khi xử lý sự cố bảo mật, defender không chỉ cần hiểu attacker đang làm gì, mà còn cần có một quy trình rõ ràng để:

- Chuẩn bị trước khi sự cố xảy ra.
- Phát hiện và phân tích sự cố.
- Cô lập, loại bỏ và khôi phục hệ thống.
- Rút kinh nghiệm sau sự cố.

Quy trình đó được gọi là **Incident Handling Process** hoặc **Incident Response Process**.

---

## Incident Handling Process là gì?

**Incident Handling Process** là quy trình giúp tổ chức chuẩn bị, phát hiện, phân tích, phản ứng và phục hồi sau các sự kiện bảo mật độc hại.

Quy trình này phù hợp cho việc xử lý các sự cố an toàn thông tin trong môi trường IT, ví dụ:

- Malware infection.
- Phishing attack.
- Credential compromise.
- Ransomware.
- Web server bị khai thác.
- Lateral movement trong mạng nội bộ.
- Data exfiltration.
- Unauthorized access.

Điểm quan trọng là **Incident Handling Process không tương ứng 1-1 với Cyber Kill Chain**.

Cyber Kill Chain mô tả vòng đời của attacker.

Incident Handling Process mô tả cách defender chuẩn bị, phát hiện, phản ứng và phục hồi.

---

## Cyber Kill Chain vs Incident Handling Process

| Tiêu chí | Cyber Kill Chain | Incident Handling Process |
|---|---|---|
| Góc nhìn | Attacker | Defender |
| Mục tiêu | Mô tả vòng đời tấn công | Mô tả quy trình xử lý sự cố |
| Trọng tâm | Recon, Delivery, Exploitation, C2, Impact | Preparation, Detection, Containment, Recovery |
| Dùng để | Hiểu attacker đang ở giai đoạn nào | Tổ chức hoạt động điều tra và phản ứng |
| Tính chất | Attack lifecycle | Response lifecycle |

Có thể hiểu đơn giản:

```text
Cyber Kill Chain = Attacker đang làm gì?
Incident Handling Process = Defender phải xử lý như thế nào?
```

---

# 4 giai đoạn Incident Handling theo NIST

Theo NIST, quy trình xử lý sự cố gồm 4 giai đoạn chính:

```text
Preparation
      ↓
Detection & Analysis
      ↓
Containment, Eradication & Recovery
      ↓
Post-Incident Activity
      ↺
```

Đây là một quy trình **có tính chu kỳ**, không phải quy trình tuyến tính tuyệt đối. Sau mỗi sự cố, tổ chức cần rút kinh nghiệm và cải thiện lại khả năng chuẩn bị, phát hiện và phản ứng.

---

## 1. Preparation

**Preparation** là giai đoạn chuẩn bị trước khi sự cố xảy ra.

Đây là một trong những giai đoạn quan trọng nhất, vì nếu tổ chức không chuẩn bị tốt thì khi incident xảy ra sẽ rất dễ bị rối, phản ứng chậm hoặc xử lý sai.

Mục tiêu của Preparation:

- Xây dựng quy trình incident response.
- Xác định vai trò và trách nhiệm.
- Chuẩn bị công cụ điều tra.
- Chuẩn bị logging và monitoring.
- Chuẩn bị playbook.
- Chuẩn bị phương án backup và recovery.
- Đào tạo đội ngũ SOC/IR.
- Kiểm thử khả năng phản ứng.

---

## Các công việc trong Preparation

Một tổ chức nên chuẩn bị:

### Chính sách và quy trình

- Incident Response Policy.
- Escalation Procedure.
- Communication Plan.
- Evidence Handling Procedure.
- Data Classification Policy.
- Backup and Restore Procedure.

### Nhân sự

- SOC Analyst.
- Incident Handler.
- Forensic Analyst.
- System Administrator.
- Network Administrator.
- Legal/Compliance.
- Management.
- PR/Communication team nếu incident ảnh hưởng lớn.

### Công cụ

- SIEM.
- EDR/XDR.
- Firewall logs.
- Proxy logs.
- DNS logs.
- Email security gateway.
- TheHive hoặc case management platform.
- Forensic tools.
- Malware analysis tools.
- Packet capture tools.
- Backup system.

### Playbook

Playbook là hướng dẫn xử lý cho từng loại incident cụ thể.

Ví dụ:

- Phishing Playbook.
- Malware Infection Playbook.
- Ransomware Playbook.
- Suspicious Login Playbook.
- Data Exfiltration Playbook.
- Web Shell Playbook.
- Privilege Escalation Playbook.

---

## 2. Detection & Analysis

**Detection & Analysis** là giai đoạn phát hiện và phân tích sự cố.

Incident handler thường dành rất nhiều thời gian ở hai giai đoạn đầu là **Preparation** và **Detection & Analysis**. Đây là nơi analyst liên tục cải thiện năng lực phát hiện và tìm kiếm sự kiện độc hại tiếp theo.

Nguồn phát hiện có thể đến từ:

- SIEM alert.
- EDR alert.
- User report.
- Email security alert.
- Firewall alert.
- IDS/IPS alert.
- Wazuh alert.
- Suricata/Snort alert.
- Threat intelligence.
- Unusual network traffic.
- Suspicious process.
- Authentication anomaly.

---

## Mục tiêu của Detection & Analysis

Khi có alert hoặc dấu hiệu bất thường, analyst cần xác định:

- Đây có phải true positive không?
- Sự cố thuộc loại nào?
- Asset nào bị ảnh hưởng?
- User nào liên quan?
- Attacker vào bằng cách nào?
- Có dấu hiệu persistence không?
- Có lateral movement không?
- Có data exfiltration không?
- Mức độ nghiêm trọng là gì?
- Cần containment ngay không?

Một câu hỏi quan trọng trong điều tra là tìm ra **patient zero**.

---

## Patient Zero là gì?

**Patient zero** là nạn nhân đầu tiên bị compromise trong incident.

Tìm được patient zero giúp analyst hiểu:

- Initial access xảy ra ở đâu.
- Attacker vào bằng vector nào.
- Timeline bắt đầu từ thời điểm nào.
- Những hệ thống nào có thể bị ảnh hưởng tiếp theo.
- Có cần hunt trên các máy tương tự không.

Ví dụ:

```text
User A mở email phishing
        ↓
Payload chạy trên máy User A
        ↓
Attacker lấy credential
        ↓
Attacker lateral movement sang file server
```

Trong ví dụ này, máy của User A là **patient zero**.

---

## Investigation cần làm gì?

Mục tiêu của hoạt động điều tra là:

1. Xác định patient zero.
2. Tạo incident timeline.
3. Xác định tool/malware attacker sử dụng.
4. Xác định hệ thống bị compromise.
5. Xác định hành động attacker đã thực hiện.
6. Xác định phạm vi ảnh hưởng.
7. Xác định dữ liệu có bị truy cập hoặc lấy ra ngoài không.
8. Đưa ra hướng containment phù hợp.

---

## Incident Timeline

Timeline giúp analyst sắp xếp sự kiện theo thứ tự thời gian.

Ví dụ:

| Time | Event |
|---|---|
| 08:15 | User nhận phishing email |
| 08:17 | User mở attachment |
| 08:18 | PowerShell chạy encoded command |
| 08:19 | Payload kết nối C2 |
| 08:25 | Credential dumping attempted |
| 08:40 | RDP connection sang server nội bộ |
| 09:10 | Suspicious file created on file server |

Timeline giúp trả lời:

- Sự cố bắt đầu khi nào?
- Attacker làm gì đầu tiên?
- Attacker đã di chuyển đến đâu?
- Containment nên bắt đầu từ đâu?
- Có hệ thống nào chưa được kiểm tra không?

---

## 3. Containment, Eradication & Recovery

Giai đoạn này là lúc tổ chức bắt đầu phản ứng trực tiếp với sự cố.

NIST gộp ba hoạt động vào một giai đoạn lớn:

```text
Containment
Eradication
Recovery
```

Tuy nhiên, cần hiểu rõ từng hoạt động.

---

## Containment

**Containment** là cô lập sự cố để ngăn attacker tiếp tục gây hại.

Mục tiêu:

- Ngăn malware lan rộng.
- Ngăn attacker tiếp tục điều khiển hệ thống.
- Ngăn data exfiltration.
- Bảo vệ hệ thống quan trọng.
- Giữ lại bằng chứng phục vụ điều tra.

Ví dụ containment:

- Isolate host khỏi network.
- Disable compromised account.
- Block malicious IP/domain.
- Block C2 traffic.
- Disable suspicious service.
- Remove host khỏi VPN.
- Apply firewall rule tạm thời.
- Quarantine email phishing.
- Reset session/token.

---

## Không được containment nửa vời

Một lỗi rất nguy hiểm là containment không đầy đủ.

Ví dụ tổ chức phát hiện 10 máy bị nhiễm nhưng chỉ cô lập 5 máy, sau đó bắt đầu eradication.

Điều này có thể khiến attacker nhận ra rằng họ đã bị phát hiện. Khi đó attacker có thể:

- Đổi C2.
- Xóa log.
- Tăng tốc exfiltration.
- Deploy ransomware sớm hơn.
- Tạo persistence mới.
- Di chuyển sang hệ thống khác.
- Phá hoại dữ liệu.

Vì vậy, trước khi chuyển sang eradication, cần đảm bảo containment đã bao phủ đầy đủ phạm vi incident.

---

## Eradication

**Eradication** là loại bỏ nguyên nhân và dấu vết độc hại khỏi môi trường.

Mục tiêu:

- Xóa malware.
- Xóa persistence.
- Vá lỗ hổng bị khai thác.
- Xóa backdoor.
- Xóa scheduled task độc hại.
- Xóa service độc hại.
- Reset credential bị lộ.
- Gỡ web shell.
- Reimage máy nếu cần.

Ví dụ eradication:

```text
Remove malicious service
Delete malware binary
Patch vulnerable application
Reset compromised passwords
Remove unauthorized admin account
Clean registry run key
```

---

## Recovery

**Recovery** là đưa hệ thống quay lại trạng thái hoạt động bình thường một cách an toàn.

Mục tiêu:

- Restore service.
- Restore dữ liệu từ backup.
- Kiểm tra hệ thống sạch trước khi đưa vào production.
- Theo dõi sau khi khôi phục.
- Đảm bảo attacker không còn quyền truy cập.
- Đưa business operation trở lại bình thường.

Ví dụ recovery:

- Restore server từ backup sạch.
- Rebuild endpoint.
- Re-enable account sau khi reset password.
- Monitor host sau khi đưa lại vào mạng.
- Validate business application.
- Confirm không còn C2 traffic.
- Confirm không còn suspicious process.

---

## Recovery Plan

Sau điều tra, đội IR cần tạo và triển khai recovery plan.

Recovery plan nên trả lời:

- Hệ thống nào cần restore?
- Restore từ backup nào?
- Có cần rebuild hoàn toàn không?
- Credential nào cần reset?
- Lỗ hổng nào cần vá?
- Service nào được ưu tiên khôi phục trước?
- Ai chịu trách nhiệm từng bước?
- Khi nào được đưa hệ thống trở lại production?
- Cần theo dõi thêm bao lâu?

---

## 4. Post-Incident Activity

**Post-Incident Activity** là giai đoạn sau khi incident đã được xử lý đầy đủ.

Đây là giai đoạn dùng để rút kinh nghiệm và cải thiện năng lực phòng thủ.

Hoạt động chính:

- Viết incident report.
- Tổ chức lessons learned meeting.
- Tính toán impact/cost.
- Cập nhật detection rule.
- Cập nhật playbook.
- Cập nhật policy.
- Cải thiện logging.
- Vá gap trong quy trình.
- Đào tạo lại user hoặc đội kỹ thuật nếu cần.

---

## Incident Report

Khi incident được xử lý hoàn toàn, tổ chức cần phát hành báo cáo.

Báo cáo nên bao gồm:

- Executive Summary.
- Incident Timeline.
- Initial Access Vector.
- Patient Zero.
- Affected Systems.
- Tools/Malware Used.
- Attacker Actions.
- Data Impact.
- Business Impact.
- Root Cause.
- Response Actions.
- Recovery Actions.
- Lessons Learned.
- Recommendations.

---

## Lessons Learned

**Lessons learned** giúp tổ chức hiểu cần làm gì để ngăn incident tương tự xảy ra trong tương lai.

Một số câu hỏi cần trả lời:

- Vì sao incident xảy ra?
- Vì sao detection chưa phát hiện sớm hơn?
- Logging đã đủ chưa?
- Alert có bị bỏ qua không?
- Playbook có hiệu quả không?
- Containment có chậm không?
- Backup có dùng được không?
- User có cần training thêm không?
- Có cần thêm detection rule không?
- Có cần thay đổi policy không?

---

# Hai hoạt động chính của Incident Handling

Incident handling có thể gom thành hai hoạt động lớn:

```text
Investigating
Recovering
```

---

## Investigating

Investigation nhằm hiểu rõ toàn bộ sự cố.

Mục tiêu:

- Tìm patient zero.
- Tạo timeline.
- Xác định tool/malware.
- Xác định compromised systems.
- Xác định attacker đã làm gì.
- Xác định scope.
- Xác định impact.

Investigation tốt giúp containment và recovery chính xác hơn.

---

## Recovering

Recovery nhằm đưa tổ chức trở lại hoạt động bình thường.

Mục tiêu:

- Tạo recovery plan.
- Thực hiện recovery plan.
- Restore service.
- Đảm bảo hệ thống sạch.
- Monitor sau recovery.
- Đảm bảo business operation được khôi phục.

---

# Vì sao Incident Handling là quy trình chu kỳ?

Incident Handling không phải quy trình làm một lần rồi kết thúc.

Sau mỗi incident, tổ chức nên cải thiện:

- Detection capability.
- Logging.
- Monitoring.
- Playbook.
- Security controls.
- User awareness.
- Backup strategy.
- Response speed.
- Communication plan.

Có thể hình dung:

```text
Prepare
Detect
Respond
Recover
Learn
Improve
Prepare again
```

---

# Ví dụ áp dụng thực tế

Giả sử có alert:

```text
Possible suspicious access to Windows admin shares
```

Analyst có thể xử lý như sau:

## Detection & Analysis

- Kiểm tra source host.
- Kiểm tra destination host.
- Kiểm tra user account.
- Kiểm tra SMB connection.
- Kiểm tra logon type.
- Kiểm tra có dùng admin share như C$, ADMIN$ không.
- Kiểm tra có credential compromise không.

## Mapping MITRE ATT&CK

```text
Tactic: Lateral Movement
Technique: Remote Services: SMB/Windows Admin Shares
ID: T1021.002
```

## Containment

- Disable account nếu nghi bị compromise.
- Isolate source host nếu có dấu hiệu malware.
- Block SMB traffic bất thường.
- Kiểm tra các host liên quan.

## Eradication

- Remove persistence.
- Reset password.
- Xóa tool độc hại.
- Vá lỗi cấu hình.
- Gỡ unauthorized admin account nếu có.

## Recovery

- Reconnect host sau khi xác nhận sạch.
- Monitor thêm.
- Kiểm tra không còn lateral movement.

## Post-Incident

- Viết report.
- Cập nhật detection rule.
- Cải thiện alert correlation.
- Review quyền admin share.

---

# Quick Reference

| Giai đoạn | Mục tiêu chính |
|---|---|
| Preparation | Chuẩn bị con người, quy trình, công cụ |
| Detection & Analysis | Phát hiện, xác minh, phân tích và xác định scope |
| Containment, Eradication & Recovery | Cô lập, loại bỏ nguyên nhân và khôi phục |
| Post-Incident Activity | Báo cáo, rút kinh nghiệm và cải thiện |

---

# Key Takeaway

**Incident Handling Process** là quy trình giúp tổ chức phản ứng có hệ thống trước sự cố bảo mật.

SOC analyst dành nhiều thời gian ở giai đoạn **Preparation** và **Detection & Analysis**, vì khả năng chuẩn bị và phát hiện tốt sẽ quyết định chất lượng phản ứng khi incident xảy ra.

Khi incident được xác nhận, cần containment đầy đủ trước khi eradication, tránh xử lý nửa vời khiến attacker phát hiện và thay đổi chiến thuật.

Sau cùng, mọi incident đều cần được báo cáo, rút kinh nghiệm và dùng để cải thiện năng lực phòng thủ cho các sự cố tiếp theo.

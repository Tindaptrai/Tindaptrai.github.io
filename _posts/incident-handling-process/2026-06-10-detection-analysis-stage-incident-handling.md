---
layout: post
title: "Detection & Analysis Stage trong Incident Handling"
date: 2026-06-10 23:12:00 +0700
categories: [SOC, Incident Response]
tags: [soc, incident-response, detection-analysis, cyber-kill-chain, mitre-attack, timeline, blue-team, thehive]
description: "Ghi chú về giai đoạn Detection & Analysis trong Incident Handling, cách phát hiện incident, xây dựng context, timeline, đánh giá mức độ nghiêm trọng và giao tiếp trong quá trình điều tra."
toc: true
---

# Detection \& Analysis Stage trong Incident Handling

Giai đoạn **Detection \& Analysis** là một trong những phần quan trọng nhất của quy trình **Incident Handling**.

Ở giai đoạn này, đội SOC/IR cần phát hiện sự cố, xác minh alert, xây dựng ngữ cảnh, tạo timeline và đánh giá mức độ nghiêm trọng của incident.

Nói ngắn gọn:

```text
Detection = phát hiện có dấu hiệu bất thường.
Analysis = hiểu dấu hiệu đó có ý nghĩa gì và có phải incident thật không.
```

\---

## 1\. Detection \& Analysis Stage là gì?

**Detection \& Analysis Stage** bao gồm tất cả hoạt động liên quan đến việc phát hiện và phân tích một incident.

Các yếu tố thường được dùng trong giai đoạn này:

* Sensors.
* Logs.
* SIEM alerts.
* EDR alerts.
* Trained personnel.
* Threat intelligence.
* Context sharing.
* Network visibility.
* Architecture segmentation.
* Asset information.

Mục tiêu là xác định xem một sự kiện bảo mật có thật sự là incident hay không, incident nghiêm trọng đến mức nào và cần xử lý theo hướng nào.

\---

## 2\. Vì sao cần hiểu Cyber Kill Chain trong Detection?

Trong Detection \& Analysis, Cyber Kill Chain giúp analyst hiểu attacker đang ở giai đoạn nào.

Cyber Kill Chain gồm các giai đoạn:

```text
Recon
Weaponize
Deliver
Exploit
Install
C\&C
Action
```

Mỗi giai đoạn có dấu hiệu phát hiện khác nhau.

Ví dụ:

|Kill Chain Stage|Dấu hiệu có thể phát hiện|
|-|-|
|Recon|Port scan, web enumeration, subdomain scan|
|Weaponize|Thường khó thấy trực tiếp vì xảy ra phía attacker|
|Delivery|Phishing email, malicious attachment, suspicious URL|
|Exploit|Process bất thường, exploit attempt, webshell upload|
|Install|Malware drop, persistence, service lạ|
|C\&C|Beaconing, DNS tunneling, outbound connection lạ|
|Action|Data exfiltration, ransomware, privilege abuse|

Mục tiêu của defender là phát hiện attacker càng sớm càng tốt, lý tưởng nhất là ở các giai đoạn đầu.

\---

## 3\. Nguồn phát hiện incident

Threat có thể được đưa vào tổ chức qua rất nhiều attack vector.

Detection có thể đến từ nhiều nguồn như:

* SIEM.
* EDR/XDR.
* IDS/IPS.
* Firewall logs.
* Proxy logs.
* DNS logs.
* Email security gateway.
* Web server logs.
* Authentication logs.
* User report.
* Threat intelligence.
* Network traffic analysis.
* Vulnerability scanner.
* Case management platform như TheHive.

Ví dụ:

```text
EDR alert: PowerShell encoded command
SIEM alert: Multiple failed login attempts
IDS alert: Exploit attempt
User report: Suspicious email
Firewall log: Outbound connection to suspicious IP
```

\---

## 4\. Cần xây dựng context trước khi phản ứng lớn

Khi phát hiện alert, không nên vội vàng gọi toàn bộ incident response team hoặc kích hoạt phản ứng toàn tổ chức ngay lập tức.

Trước tiên cần thực hiện **initial investigation** để xây dựng context.

Ví dụ có event:

```text
Administrative account connected to IP address at HH:MM:SS
```

Nếu không biết:

* IP đó là hệ thống gì.
* Timezone của log là gì.
* Account đó thuộc ai.
* Kết nối này có nằm trong lịch bảo trì không.
* Host đó có phải business-critical system không.

Thì analyst rất dễ kết luận sai.

Vì vậy, cần thu thập càng nhiều thông tin càng tốt trước khi quyết định escalation.

\---

## 5\. Những thông tin cần thu thập ban đầu

Khi có alert, analyst nên thu thập các thông tin sau:

* Alert đến từ nguồn nào.
* Thời gian xảy ra sự kiện.
* Timezone của log.
* Source IP.
* Destination IP.
* Hostname.
* Username.
* Process liên quan.
* Command line.
* File path.
* Hash.
* Domain/URL.
* Network connection.
* Asset owner.
* Business criticality.
* Có dấu hiệu lặp lại không.
* Có host khác bị ảnh hưởng không.

Mục tiêu là tránh điều tra thiếu ngữ cảnh.

\---

## 6\. Incident Timeline

Sau khi có thông tin ban đầu, analyst cần bắt đầu xây dựng **incident timeline**.

Timeline giúp sắp xếp các sự kiện theo thứ tự thời gian.

Điều này rất quan trọng vì trong quá trình điều tra, bằng chứng không phải lúc nào cũng được tìm thấy theo đúng thứ tự xảy ra.

Ví dụ:

```text
Analyst tìm thấy malware trên máy A hôm nay.
Sau đó phát hiện cùng payload đã xuất hiện trên máy B từ hai tuần trước.
```

Nếu không có timeline, analyst có thể nhầm máy A là patient zero.

\---

## 7\. Timeline nên có những cột nào?

Một bảng timeline cơ bản nên có:

|Time|Source|Event|Host/User|Evidence|Notes|
|-|-|-|-|-|-|
|08:15|Email Gateway|Phishing email received|user01|Message ID|Suspicious sender|
|08:17|Endpoint|Attachment opened|user01-PC|Process log|Word spawned PowerShell|
|08:18|EDR|Encoded PowerShell|user01-PC|Command line|Possible payload download|
|08:19|Firewall|Outbound connection|user01-PC|Destination IP|Possible C2|
|08:30|Windows Log|Admin login|file-server|Event ID|Possible lateral movement|

Timeline giúp trả lời:

* Incident bắt đầu khi nào.
* Sự kiện nào xảy ra trước.
* Host nào bị ảnh hưởng đầu tiên.
* Attacker đã làm gì.
* Có bằng chứng mới nào thuộc incident hiện tại không.
* Có dữ liệu nào không liên quan không.

\---

## 8\. MITRE ATT\&CK trong Detection \& Analysis

Trong quá trình xử lý incident, analyst nên map hành vi quan sát được vào **MITRE ATT\&CK**.

MITRE ATT\&CK giúp mô tả hành vi attacker theo:

* Tactic.
* Technique.
* Sub-technique.

Ví dụ:

|Hành vi|Tactic|Technique|ID|
|-|-|-|-|
|PowerShell encoded command|Execution|Command and Scripting Interpreter: PowerShell|T1059.001|
|LSASS dump|Credential Access|OS Credential Dumping: LSASS Memory|T1003.001|
|RDP lateral movement|Lateral Movement|Remote Services: RDP|T1021.001|
|SMB admin share access|Lateral Movement|SMB/Windows Admin Shares|T1021.002|
|Data encryption|Impact|Data Encrypted for Impact|T1486|

Mapping ATT\&CK giúp incident report rõ ràng và giúp SOC hiểu attacker đang cố đạt mục tiêu gì.

\---

## 9\. Làm việc với alert trong TheHive

Trong TheHive, analyst có thể:

* Assign alert cho chính mình.
* Tạo case từ alert.
* Thêm thông tin điều tra vào case.
* Gắn observables.
* Gắn MITRE ATT\&CK TTPs.
* Cập nhật timeline.
* Ghi findings.
* Ghi lessons learned.
* Đóng case sau khi investigation hoàn tất.

Workflow cơ bản:

```text
Alert received
      ↓
Assign to analyst
      ↓
Create case
      ↓
Add observables and context
      ↓
Investigate and enrich
      ↓
Map MITRE ATT\&CK
      ↓
Document findings
      ↓
Close case
```

\---

## 10\. Câu hỏi cần trả lời khi xử lý incident

Khi xử lý sự cố, analyst cần trả lời các câu hỏi để đánh giá mức độ nghiêm trọng và phạm vi ảnh hưởng.

Các câu hỏi quan trọng:

* Exploitation impact là gì?
* Exploitation requirements là gì?
* Business-critical systems có bị ảnh hưởng không?
* Có remediation steps được đề xuất không?
* Có bao nhiêu hệ thống bị ảnh hưởng?
* Exploit có đang được sử dụng ngoài thực tế không?
* Exploit có khả năng worm-like không?
* Attacker có đang lan rộng không?
* Có dấu hiệu credential compromise không?
* Có dấu hiệu exfiltration không?

Các incident có impact cao hoặc ảnh hưởng nhiều hệ thống cần được escalation nhanh.

\---

## 11\. Exploitation Impact

**Exploitation impact** là tác động nếu lỗ hổng hoặc kỹ thuật tấn công được khai thác thành công.

Ví dụ impact:

* Remote Code Execution.
* Privilege Escalation.
* Credential Theft.
* Data Exfiltration.
* Lateral Movement.
* Ransomware Deployment.
* Service Disruption.

Impact càng cao thì mức độ ưu tiên xử lý càng lớn.

\---

## 12\. Exploitation Requirements

**Exploitation requirements** là điều kiện cần để attacker khai thác thành công.

Ví dụ:

* Cần user click link.
* Cần user mở attachment.
* Cần authentication.
* Cần quyền local admin.
* Cần service expose Internet.
* Cần một version phần mềm cụ thể.
* Cần network access đến hệ thống.

Nếu yêu cầu khai thác thấp, rủi ro thường cao hơn.

\---

## 13\. Worm-like Capability

**Worm-like capability** là khả năng tự lan truyền giống worm.

Nếu exploit có khả năng tự lan qua mạng, incident có thể trở nên rất nghiêm trọng.

Ví dụ:

```text
Một máy bị nhiễm
      ↓
Tự scan mạng nội bộ
      ↓
Khai thác máy khác
      ↓
Lan rộng nhanh chóng
```

Các sự cố có worm-like behavior cần được containment nhanh.

\---

## 14\. Incident Confidentiality

Incident là chủ đề rất nhạy cảm.

Thông tin điều tra nên được giữ theo nguyên tắc:

```text
Need-to-know basis
```

Nghĩa là chỉ những người cần biết mới được biết.

Lý do:

* Attacker có thể là nhân viên nội bộ.
* Attacker có thể đang đọc email/chat nội bộ.
* Thông tin breach có thể ảnh hưởng pháp lý.
* Communication với khách hàng/bên ngoài cần thông qua người được chỉ định.
* Legal team có thể cần kiểm soát nội dung thông báo.

\---

## 15\. Communication trong incident

Khi investigation bắt đầu, cần đặt kỳ vọng rõ ràng với các bên liên quan.

Nội dung cần thông báo có thể gồm:

* Loại incident đang xử lý.
* Nguồn bằng chứng hiện có.
* Phạm vi điều tra ban đầu.
* Ước lượng thời gian điều tra.
* Mức độ ảnh hưởng dự kiến.
* Có khả năng xác định attacker hay không.
* Các bước tiếp theo.
* Khi nào có update mới.

Tuy nhiên, những thông tin này có thể thay đổi khi có bằng chứng mới.

Vì vậy, cần giữ management và các bên liên quan được cập nhật theo tiến độ điều tra.

\---

## 16\. Không nên kết luận quá sớm

Một lỗi phổ biến trong Detection \& Analysis là kết luận quá sớm khi chưa đủ context.

Ví dụ:

```text
Một admin account đăng nhập vào server lúc nửa đêm.
```

Có thể đây là:

* Hoạt động bảo trì hợp lệ.
* Script backup tự động.
* Admin làm việc theo timezone khác.
* Hoặc là attacker sử dụng credential bị đánh cắp.

Nếu không kiểm tra context, analyst có thể bỏ sót incident thật hoặc tạo false positive không cần thiết.

\---

## 17\. Workflow Detection \& Analysis

Một workflow đơn giản:

```text
Alert triggered
      ↓
Validate alert
      ↓
Collect context
      ↓
Identify affected asset/user
      ↓
Check business criticality
      ↓
Build initial timeline
      ↓
Map MITRE ATT\&CK
      ↓
Assess severity and scope
      ↓
Decide escalation
      ↓
Document in case
```

\---

## 18\. Bảng ôn nhanh thuật ngữ

|Thuật ngữ|Nghĩa ngắn gọn|
|-|-|
|Detection|Phát hiện dấu hiệu bất thường|
|Analysis|Phân tích để hiểu sự kiện có phải incident không|
|Sensor|Nguồn thu thập tín hiệu/log|
|Log|Dữ liệu ghi lại sự kiện|
|SIEM|Hệ thống tập trung và phân tích log|
|EDR|Giám sát và phản ứng trên endpoint|
|Threat Intelligence|Thông tin tình báo mối đe dọa|
|Context|Ngữ cảnh giúp hiểu đúng sự kiện|
|Timeline|Dòng thời gian sự kiện|
|MITRE ATT\&CK|Framework mô tả hành vi attacker|
|Tactic|Mục tiêu cấp cao của attacker|
|Technique|Kỹ thuật attacker sử dụng|
|Observable|Thông tin quan sát được như IP, hash, domain|
|IOC|Indicator of Compromise|
|Escalation|Nâng cấp mức xử lý|
|False Positive|Cảnh báo sai|
|True Positive|Cảnh báo đúng|
|Need-to-know|Chỉ chia sẻ cho người cần biết|
|Worm-like|Có khả năng tự lan truyền|
|Business-critical System|Hệ thống quan trọng với hoạt động kinh doanh|

\---

## 19\. Câu hỏi ôn tập

### 1\. Detection \& Analysis Stage dùng để làm gì?

Dùng để phát hiện, xác minh, phân tích incident, xây dựng context, timeline và đánh giá mức độ nghiêm trọng.

### 2\. Vì sao cần xây dựng context trước khi escalation?

Vì nếu thiếu context như hệ thống, timezone, user, business criticality, analyst có thể kết luận sai.

### 3\. Timeline giúp gì trong điều tra?

Timeline giúp sắp xếp sự kiện theo thứ tự thời gian, xác định incident bắt đầu khi nào, attacker làm gì và bằng chứng mới có liên quan hay không.

### 4\. Vì sao incident information cần giữ confidential?

Vì thông tin incident có thể nhạy cảm về pháp lý, attacker có thể là người nội bộ hoặc có thể đang theo dõi kênh communication.

### 5\. Vì sao cần map MITRE ATT\&CK?

Vì MITRE ATT\&CK giúp mô tả hành vi attacker rõ ràng, hỗ trợ detection, reporting và response.

\---

## Key Takeaway

Giai đoạn **Detection \& Analysis** giúp SOC/IR chuyển một alert rời rạc thành một incident có ngữ cảnh rõ ràng.

Analyst không nên chỉ nhìn alert đơn lẻ, mà cần thu thập thêm context, hiểu asset, user, thời gian, business impact và các sự kiện liên quan.

Timeline, MITRE ATT\&CK mapping, incident confidentiality và communication rõ ràng là những yếu tố quan trọng giúp quá trình điều tra chính xác và có tổ chức hơn.

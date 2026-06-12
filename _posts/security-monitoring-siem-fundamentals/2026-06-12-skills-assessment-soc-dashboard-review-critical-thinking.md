---
layout: post
title: "Skills Assessment: SOC Dashboard Review & Critical Thinking"
date: 2026-06-12 23:10:00 +0700
categories: [SOC, SIEM]
tags: [soc, siem, skills-assessment, dashboard-review, critical-thinking, alert-triage, kibana, blue-team]
description: "Ghi chú tổng hợp bài Skills Assessment về review SOC dashboard, phân tích các visualization và tư duy triage/escalation trong môi trường SOC Tier 1."
toc: true
---

# Skills Assessment: SOC Dashboard Review & Critical Thinking

Bài **Skills Assessment** này mô phỏng tình huống bạn vừa được tuyển vào công ty **Eagle** với vai trò **SOC Tier 1 Analyst**.

Nhiệm vụ của bạn là review dashboard `SOC-Alerts`, quan sát các visualization có sẵn và đưa ra nhận định ban đầu dựa trên bối cảnh môi trường doanh nghiệp.

Mục tiêu chính:

```text
Không chỉ nhìn alert, mà phải hiểu alert trong context của tổ chức.
```

---

## 1. Bối cảnh môi trường Eagle

Sau buổi onboarding, senior analyst cung cấp một số thông tin quan trọng về môi trường.

Các điểm cần nhớ:

- Tổ chức đã chuyển toàn bộ hosting lên cloud.
- DMZ cũ đã bị đóng, không còn server trong DMZ.
- IT Operations team chỉ có 4 người.
- 4 người này là nhóm duy nhất có high privileges.
- IT Operations thường dùng default administrator account dù đã được khuyến cáo không nên.
- Endpoint đã được hardening theo CIS baseline.
- Whitelisting có tồn tại nhưng còn giới hạn.
- IT Security đã tạo **Privileged Admin Workstation (PAW)**.
- Mọi admin activity phải được thực hiện từ PAW.
- Linux servers chủ yếu là hệ thống cũ còn sót lại.
- Linux hầu như không có nhiều hoạt động trong ngày bình thường.
- Root user không được dùng để remote login.
- User cần quyền cao trên Linux phải dùng `sudo`.
- Naming convention được tuân thủ nghiêm ngặt.
- Service account có chuỗi `-svc` trong tên.
- Service account dùng password dài, phức tạp và chỉ phục vụ nhiệm vụ rất cụ thể.

---

## 2. Tư duy quan trọng khi review dashboard

Trong bài này, không nên chỉ nhìn số lượng event.

SOC analyst cần kết hợp:

```text
Alert data + Environment context + Business rule + Security policy
```

Ví dụ:

- Nếu DMZ đã đóng mà vẫn có log từ DMZ thì đáng ngờ.
- Nếu admin login không từ PAW thì vi phạm policy.
- Nếu service account dùng RDP thì đáng ngờ.
- Nếu root SSH login xảy ra thì đáng ngờ.
- Nếu disabled user vẫn cố login thì cần điều tra.
- Nếu một SID lạ được thêm vào Administrators group thì cần hỏi IT hoặc escalate tùy context.

---

# 3. Dashboard cần review

Truy cập Kibana:

```text
http://[Target IP]:5601
```

Điều hướng:

```text
Side navigation toggle → Dashboard → SOC-Alerts
```

Nếu trước đó đã xóa dashboard trong lab, cần reset target bằng:

```text
Reset Target
```

Mục tiêu là lấy lại dashboard preconfigured.

---

# 4. Visualization 1: Failed logon attempts - All users

Visualization này giúp phát hiện các hành vi như:

- Brute force.
- Password guessing.
- Password spraying.
- Credential misuse.
- Service account dùng sai password.
- User hoặc endpoint có hành vi đăng nhập bất thường.

Thông thường, cần chú ý:

- Một user có nhiều failed attempts.
- Một source host thử nhiều user.
- Một user bị thử trên nhiều endpoint.
- Failed logon xảy ra ngoài giờ làm việc.
- Failed logon sau đó có successful logon.

Trong bài assessment, dữ liệu hiện tại không cho thấy brute force rõ ràng.

Tuy nhiên có một bất thường liên quan đến account:

```text
sql-svc1
```

---

## 4.1. Vì sao sql-svc1 đáng chú ý?

Theo naming convention, service accounts chứa:

```text
-svc
```

Do đó `sql-svc1` có khả năng là service account.

Service account trong môi trường này:

- Có password dài và phức tạp.
- Chỉ thực hiện nhiệm vụ cụ thể.
- Thường chạy service local trên machine.
- Không nên có hành vi login tương tác bất thường.
- Không nên bị failed logon nhiều nếu cấu hình đúng.

Nếu `sql-svc1` xuất hiện trong failed logon visualization, cần nghi ngờ:

- Service đang dùng password cũ.
- Có người cố dùng service account để login.
- Credential của service account bị lộ.
- Có misconfiguration trên service/scheduled task.
- Có attacker thử credential của service account.

---

# 5. Visualization 2: Failed logon attempts - Disabled user

Visualization này tập trung vào failed logon nhắm vào disabled users.

Trong bài assessment, có một incident liên quan đến user:

```text
Anni
```

Ý nghĩa:

```text
User Anni đã cố authenticate dù account đang disabled.
```

Điều này đáng chú ý vì disabled account không thể login thành công.

Cần kiểm tra:

- Vì sao Anni vẫn cố login?
- Đây là user thật hay attacker dùng credential cũ?
- Có source IP/host nào phát sinh attempt?
- Có phải application/service vẫn dùng credential cũ?
- Account Anni bị disable do nghỉ việc hay lý do bảo mật?
- Có failed attempts lặp lại không?

---

# 6. Visualization 3: Failed logon attempts - Admin users only

Visualization này chỉ tập trung vào admin users.

Hint quan trọng:

```text
Check if all events took place on either PAWs or Domain Controllers.
```

Theo policy, admin activity phải thực hiện từ:

```text
PAW - Privileged Admin Workstation
```

Hoặc trong một số trường hợp hợp lệ:

```text
Domain Controller
```

Nếu admin failed logon xảy ra từ endpoint thường, server thường hoặc host không phải PAW/DC, cần xem là bất thường.

---

## 6.1. Điều cần kiểm tra

SOC analyst nên kiểm tra:

- Username admin là ai?
- Event xảy ra trên host nào?
- Host có phải PAW không?
- Host có phải Domain Controller không?
- Logon type là gì?
- Source IP/WorkstationName là gì?
- Có nhiều failed attempts không?
- Có successful admin login sau đó không?
- Admin account có phải default administrator account không?

---

# 7. Visualization 4: RDP logon for service account

Visualization này rất quan trọng.

Service accounts trong môi trường Eagle có nhiệm vụ rất cụ thể và thường chỉ chạy service local.

Vì vậy, nếu service account dùng RDP thì rất đáng nghi.

Ví dụ service account:

```text
sql-svc1
```

Nếu account này có RDP logon, cần đặt câu hỏi:

```text
Vì sao service account lại đăng nhập RDP?
```

---

## 7.1. Vì sao service account RDP là đáng ngờ?

Service account thường không nên:

- Login tương tác.
- Dùng RDP.
- Dùng như user thường.
- Duyệt desktop.
- Truy cập server thủ công.
- Có hành vi ngoài nhiệm vụ dịch vụ.

RDP logon bằng service account có thể là dấu hiệu:

- Credential compromise.
- Lateral movement.
- Misuse by admin.
- Attacker dùng credential hợp lệ.
- Service account bị dùng sai mục đích.

---

## 7.2. Cần điều tra gì?

Cần kiểm tra:

- Service account nào dùng RDP?
- Source IP/machine nào khởi tạo RDP?
- Destination host là gì?
- Logon time có bất thường không?
- Account có quyền gì?
- Có process đáng ngờ sau RDP không?
- Có file transfer hoặc tool execution không?
- Có liên quan tới sql-svc1 không?

---

# 8. Visualization 5: User added or removed from a local group

Visualization này cho thấy có user được thêm hoặc xóa khỏi local group.

Trong bài assessment:

```text
An administrator added an individual represented only by SID into the Administrators group.
```

Điểm đáng chú ý:

```text
User chỉ hiện bằng SID, không hiện username rõ ràng.
```

Điều này có thể xảy ra khi:

- Account bị xóa nhưng SID vẫn còn.
- Domain lookup không resolve được.
- Log thiếu enrichment.
- Account đến từ domain khác.
- Có thao tác đáng ngờ thêm principal lạ vào local Administrators.

---

## 8.1. Escalate ngay hay hỏi IT Operations trước?

Trong tình huống này, nên **consult with IT Operations first** nếu chưa có dấu hiệu attack rõ ràng.

Lý do:

- IT Operations là nhóm quản trị core.
- Họ có thể biết thay đổi này là hợp lệ hay không.
- Có thể đang có maintenance/change request.
- SID cần được resolve để biết đó là ai.
- Escalate Tier 2/3 ngay khi thiếu context có thể gây nhiễu.

Tuy nhiên, nếu IT không xác nhận được hoặc hành động không có change record, cần escalate.

---

## 8.2. Cần hỏi IT Operations gì?

Nên hỏi:

- Ai thực hiện thay đổi?
- Thay đổi có change ticket không?
- SID này thuộc account nào?
- Vì sao thêm vào Administrators group?
- Host nào bị thay đổi?
- Thời điểm có nằm trong maintenance window không?
- Có phải hoạt động hợp lệ không?

---

# 9. Visualization 6: Admin logon not from PAW

Policy của Eagle:

```text
Administrators should exclusively utilize PAWs for server remote connections.
```

Nếu có admin logon không từ PAW, đây là policy violation.

Câu hỏi đặt ra:

```text
Escalate Tier 2/3 hay consult IT Operations trước?
```

Trong nhiều trường hợp, nên **consult with IT Operations first**, vì:

- IT admins có thể dùng default administrator account.
- Senior analyst đã nói họ thường dùng default admin dù được khuyến cáo không nên.
- Cần xác nhận đó là hoạt động hợp lệ hay không.
- Đây có thể là policy violation hơn là compromise ngay lập tức.

Tuy nhiên, nếu không được IT xác nhận hoặc có dấu hiệu bất thường, cần escalate.

---

## 9.1. Khi nào admin logon not from PAW cần escalation?

Nên escalate nếu:

- Admin login từ host lạ.
- Admin login ngoài giờ làm việc.
- Admin login vào critical server.
- Có failed logon trước đó.
- Có lateral movement sau đó.
- Có process đáng ngờ sau login.
- Admin account được dùng từ nhiều nguồn.
- IT Operations không xác nhận hoạt động.

---

# 10. Visualization 7: SSH Logins

Linux environment trong Eagle:

- Chủ yếu là server cũ.
- Ít hoạt động thường ngày.
- Root user không được dùng.
- Root bị block remote login.
- User cần quyền cao phải dùng `sudo`.

Do đó, nếu thấy SSH login bằng root thì rất đáng ngờ.

---

## 10.1. Vì sao root SSH login đáng ngờ?

Theo policy:

```text
root user account is not used
root remote login is blocked
```

Nếu dashboard vẫn hiển thị root SSH login, có thể có vấn đề:

- Policy không được enforce đúng.
- Server cấu hình sai.
- SSH config bị thay đổi.
- Attacker đã thay đổi cấu hình.
- Có unauthorized access.
- Log cho thấy hành vi bất thường.

---

## 10.2. Cần điều tra gì?

SOC analyst nên kiểm tra:

- User SSH là ai?
- Có phải root không?
- Source IP là gì?
- Destination Linux server nào?
- Login thành công hay thất bại?
- Có sudo command sau đó không?
- Có lệnh đáng ngờ không?
- Có thay đổi `/etc/ssh/sshd_config` không?
- Có failed attempts trước đó không?

---

# 11. Bảng tư duy nhanh theo visualization

| Visualization | Điều cần chú ý | Hướng xử lý |
|---|---|---|
| Failed logon all users | Bất thường liên quan `sql-svc1` | Điều tra service account misuse |
| Failed logon disabled user | User `Anni` authenticate dù disabled | Kiểm tra source/host/context |
| Failed logon admin only | Admin events có xảy ra ngoài PAW/DC không | Kiểm tra policy violation |
| RDP logon service account | Service account dùng RDP | Nghi ngờ cao, cần điều tra |
| Local group changes | SID được thêm vào Administrators | Consult IT trước, escalate nếu không hợp lệ |
| Admin logon not from PAW | Admin không dùng PAW | Consult IT trước, escalate nếu bất thường |
| SSH logins | Root SSH login | Nghi ngờ cao, kiểm tra Linux policy |

---

# 12. Tư duy Tier 1 Analyst khi làm assessment

SOC Tier 1 không chỉ điền đáp án, mà cần thể hiện tư duy:

```text
Context trước, kết luận sau.
```

Luồng tư duy nên là:

```text
1. Xem alert/visualization.
2. So với policy môi trường.
3. Xác định điểm bất thường.
4. Thu thập context.
5. Phân biệt false positive và true positive.
6. Consult IT nếu có khả năng là thay đổi hợp lệ.
7. Escalate nếu có dấu hiệu compromise hoặc không được xác nhận.
```

---

# 13. Khi nào consult IT Operations?

Nên consult IT Operations khi:

- Alert có thể liên quan đến maintenance.
- Có thể là change hợp lệ.
- Có thể là misconfiguration.
- Có thể là admin làm sai policy nhưng chưa chắc là compromise.
- Cần xác nhận SID/account/host.
- Cần biết system owner hoặc service owner.

Ví dụ:

```text
SID lạ được thêm vào Administrators group
Admin login không từ PAW nhưng có thể do IT admin thao tác
```

---

# 14. Khi nào escalate Tier 2/3?

Nên escalate Tier 2/3 khi:

- Có dấu hiệu compromise rõ.
- IT không xác nhận được hành vi.
- Alert liên quan privileged account.
- Có lateral movement.
- Có service account interactive/RDP login.
- Có root SSH login trái policy.
- Có hành vi xảy ra trên critical asset.
- Có nhiều event liên quan tạo thành attack chain.
- Có dấu hiệu data exfiltration hoặc malware.

---

# 15. Các KQL/field nên dùng khi điều tra

Một số field hữu ích:

```text
user.name
user.name.keyword
host.hostname
host.hostname.keyword
source.ip
destination.ip
event.code
event.action
event.outcome
winlog.logon.type
winlog.logon.type.keyword
winlog.event_data.TargetUserName
winlog.event_data.SubjectUserName
winlog.event_data.IpAddress
winlog.event_data.WorkstationName
@timestamp
```

Một số KQL mẫu:

```text
user.name:"sql-svc1"
```

```text
user.name:"Anni" OR user.name:"anni"
```

```text
event.code:4625
```

```text
winlog.logon.type.keyword:"RemoteInteractive"
```

```text
user.name:*svc*
```

```text
NOT user.name: *$ AND winlog.channel.keyword: Security
```

---

# 16. Câu hỏi ôn tập

## 1. Vì sao `sql-svc1` đáng chú ý trong failed logon all users?

Vì `sql-svc1` có dạng service account. Service account thường không nên có hành vi login bất thường hoặc failed logon nhiều lần.

## 2. User nào xuất hiện trong visualization disabled user?

User được nhắc đến là:

```text
Anni
```

## 3. Admin activity nên diễn ra ở đâu?

Theo policy, admin activity nên diễn ra trên:

```text
PAW - Privileged Admin Workstation
```

hoặc một số hoạt động hợp lệ trên:

```text
Domain Controller
```

## 4. Vì sao RDP logon bằng service account đáng ngờ?

Vì service account thường chỉ chạy service cụ thể, không nên dùng để đăng nhập tương tác hoặc RDP.

## 5. SID lạ được thêm vào Administrators group nên làm gì trước?

Nên consult IT Operations trước để xác nhận SID, change ticket và tính hợp lệ. Nếu không xác nhận được thì escalate.

## 6. Admin logon not from PAW nên làm gì?

Nên consult IT Operations trước nếu có khả năng là admin làm sai policy. Nếu không được xác nhận hoặc có dấu hiệu compromise thì escalate.

## 7. Vì sao root SSH login đáng ngờ?

Vì môi trường nói rõ root user không được dùng và bị block remote login. Nếu vẫn có root SSH login thì cần điều tra ngay.

---

# Key Takeaway

Bài assessment này kiểm tra khả năng review dashboard theo bối cảnh thực tế.

Điểm quan trọng không phải chỉ là đọc số liệu, mà là hiểu chính sách môi trường Eagle và phát hiện hành vi lệch khỏi baseline:

- Service account không nên RDP/login tương tác.
- Disabled user không nên tiếp tục authenticate.
- Admin phải dùng PAW.
- SID lạ trong Administrators group cần được xác minh.
- Root SSH login là tín hiệu rất đáng ngờ.
- Không phải mọi bất thường đều escalate ngay; một số case nên consult IT Operations trước để lấy context.

Một SOC Tier 1 analyst tốt cần biết khi nào nên điều tra thêm, khi nào nên hỏi IT và khi nào phải escalate.

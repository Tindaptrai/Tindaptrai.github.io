---
layout: post
title: "Preparation Stage: Biện pháp bảo vệ trong Incident Handling"
date: 2026-06-03 03:00:00 +0700
categories: [SOC, Incident Response]
tags: [soc, incident-response, preparation-stage, dmarc, dkim, spf, edr, amsi, hardening, mfa, blue-team]
description: "Ghi chú về các biện pháp bảo vệ trong giai đoạn Preparation như DMARC, endpoint hardening, EDR, network protection, MFA, vulnerability scanning và Purple Team."
toc: true
---

# Preparation Stage: Biện pháp bảo vệ trong Incident Handling

Trong giai đoạn **Preparation**, ngoài việc chuẩn bị đội ngũ, quy trình, tài liệu và công cụ, tổ chức còn cần triển khai các biện pháp bảo vệ nhằm giảm khả năng xảy ra sự cố bảo mật.

Mặc dù việc bảo vệ hệ thống không nhất thiết là trách nhiệm trực tiếp của đội **Incident Handling**, nhưng đội xử lý sự cố vẫn cần hiểu các biện pháp bảo vệ đang được triển khai.

Lý do là khi incident xảy ra, kiến thức này giúp analyst:

- Hiểu loại sự cố đang xảy ra.
- Đánh giá mức độ phức tạp của sự cố.
- Biết attacker có thể vượt qua lớp bảo vệ nào.
- Biết nên tìm bằng chứng ở đâu.
- Biết log hoặc artifact nào có thể hỗ trợ điều tra.
- Hiểu môi trường phòng thủ của tổ chức.

Nói ngắn gọn:

```text
Biết tổ chức đang bảo vệ như thế nào
= biết nên điều tra ở đâu khi sự cố xảy ra.
```

---

# 1. DMARC

## DMARC là gì?

**DMARC** là viết tắt của **Domain-based Message Authentication, Reporting and Conformance**.

DMARC là cơ chế bảo vệ email chống giả mạo và phishing, được xây dựng dựa trên hai cơ chế có sẵn là:

- **SPF**
- **DKIM**

Mục tiêu của DMARC là ngăn attacker gửi email giả mạo tên miền của tổ chức.

Ví dụ attacker gửi email giả mạo:

```text
From: ceo@company.com
Subject: Please pay this invoice urgently
```

Nếu email này không thật sự được gửi từ hệ thống hợp lệ của công ty, DMARC có thể giúp hệ thống email phát hiện và từ chối email đó.

---

## SPF là gì?

**SPF** là viết tắt của **Sender Policy Framework**.

SPF cho phép chủ sở hữu domain khai báo danh sách máy chủ hoặc dịch vụ được phép gửi email thay mặt domain đó.

Ví dụ:

```text
Chỉ mail server A và dịch vụ email B được phép gửi email cho domain company.com
```

Nếu một máy chủ lạ cố gửi email giả danh `company.com`, SPF có thể fail.

---

## DKIM là gì?

**DKIM** là viết tắt của **DomainKeys Identified Mail**.

DKIM sử dụng chữ ký số để xác minh rằng email thật sự được gửi từ domain hợp lệ và nội dung email không bị thay đổi trong quá trình gửi.

Hiểu đơn giản:

```text
SPF kiểm tra nơi gửi.
DKIM kiểm tra chữ ký và tính toàn vẹn.
DMARC dùng SPF/DKIM để quyết định xử lý email.
```

---

## DMARC giúp chống phishing như thế nào?

DMARC có thể yêu cầu hệ thống nhận email xử lý email không hợp lệ theo các chế độ:

| Policy | Ý nghĩa |
|---|---|
| none | Chỉ theo dõi, chưa chặn |
| quarantine | Đưa email nghi ngờ vào spam/quarantine |
| reject | Từ chối email giả mạo |

Ví dụ:

```text
Attacker giả mạo email từ domain công ty
        ↓
Email không đạt SPF/DKIM alignment
        ↓
DMARC fail
        ↓
Email bị quarantine hoặc reject
```

---

## Lưu ý khi triển khai DMARC

DMARC rất hữu ích, dễ triển khai và chi phí thấp. Tuy nhiên, cần kiểm tra kỹ trước khi bật chế độ chặn mạnh.

Nếu cấu hình sai, tổ chức có thể vô tình chặn email hợp pháp.

Ví dụ false positive thường gặp:

- Email gửi qua dịch vụ bên thứ ba.
- Email marketing.
- CRM gửi email thay mặt domain công ty.
- Hệ thống ticket gửi email tự động.
- Dịch vụ cloud gửi notification.
- Email được forward qua hệ thống trung gian.

Vì vậy, nên triển khai theo lộ trình:

```text
p=none
  ↓
theo dõi report
  ↓
sửa SPF/DKIM cho các dịch vụ hợp lệ
  ↓
p=quarantine
  ↓
p=reject
```

---

# 2. Email Filtering nâng cao với DMARC

Ngoài việc dùng DMARC cho domain của chính tổ chức, hệ thống email còn có thể kiểm tra DMARC đối với email từ các domain khác.

Một số mail system sẽ thêm header cho biết DMARC pass hay fail.

Dựa vào header này, tổ chức có thể tạo rule để phát hiện email phishing từ domain bên ngoài.

Tuy nhiên, cách này cần kiểm tra rộng rãi trước khi đưa vào production vì có thể gây nhiều false positive.

Ví dụ false positive:

```text
Một dịch vụ gửi email thay mặt domain khác
        ↓
Domain không alignment
        ↓
DMARC fail
        ↓
Email hợp lệ bị đánh dấu phishing
```

---

# 3. Endpoint Hardening và EDR

## Endpoint là gì?

**Endpoint** là thiết bị người dùng hoặc thiết bị cuối trong hệ thống.

Ví dụ:

- Workstation.
- Laptop.
- Desktop.
- Máy người dùng.
- Server trong một số ngữ cảnh.

Endpoint thường là điểm vào phổ biến của attacker vì người dùng thường:

- Duyệt web.
- Mở email.
- Tải file.
- Mở attachment.
- Chạy file thực thi.
- Dùng ứng dụng văn phòng.

---

## Endpoint Hardening là gì?

**Endpoint Hardening** là quá trình gia cố bảo mật cho máy trạm hoặc endpoint để giảm bề mặt tấn công.

Mục tiêu:

- Giảm khả năng bị khai thác.
- Chặn kỹ thuật phổ biến của attacker.
- Giảm quyền của user.
- Giảm khả năng malware chạy thành công.
- Tăng khả năng phát hiện hành vi bất thường.

Các baseline thường dùng:

- CIS Benchmark.
- Microsoft Security Baseline.

---

## Các biện pháp hardening quan trọng

### Tắt LLMNR và NetBIOS

**LLMNR** và **NetBIOS** có thể bị attacker lợi dụng để đánh cắp hash trong mạng nội bộ.

Ví dụ attack:

```text
User gõ sai tên share
        ↓
Máy hỏi LLMNR/NetBIOS trong LAN
        ↓
Attacker giả mạo trả lời
        ↓
User gửi NetNTLM hash
        ↓
Attacker relay hoặc crack hash
```

Vì vậy, nên tắt LLMNR/NetBIOS nếu không cần thiết.

---

### Triển khai LAPS

**LAPS** là viết tắt của **Local Administrator Password Solution**.

LAPS giúp quản lý mật khẩu local administrator trên từng máy Windows.

Lợi ích:

- Mỗi máy có mật khẩu local admin khác nhau.
- Giảm rủi ro dùng chung mật khẩu local admin.
- Hạn chế lateral movement.
- Khi một máy bị compromise, attacker không thể dùng cùng mật khẩu để truy cập nhiều máy khác.

---

### Xóa quyền admin khỏi user thông thường

Người dùng bình thường không nên có quyền local administrator.

Nếu user có quyền admin, malware chạy dưới user đó cũng có thể có quyền cao hơn, dễ cài persistence hoặc tắt security tool.

Nguyên tắc cần áp dụng:

```text
Least Privilege
```

Tức là chỉ cấp quyền tối thiểu cần thiết.

---

### Hạn chế PowerShell

PowerShell là công cụ quản trị mạnh, nhưng cũng thường bị attacker lợi dụng.

Một số biện pháp:

- Bật logging cho PowerShell.
- Bật Script Block Logging.
- Bật Module Logging.
- Bật Constrained Language Mode nếu phù hợp.
- Giám sát encoded command.
- Giám sát PowerShell gọi network.

Ví dụ hành vi đáng ngờ:

```powershell
powershell.exe -enc <base64>
```

---

### Bật Attack Surface Reduction Rules

**ASR** là viết tắt của **Attack Surface Reduction**.

ASR Rules trong Microsoft Defender giúp chặn các hành vi thường bị malware lợi dụng.

Ví dụ:

- Office tạo child process.
- Script tải executable.
- Credential stealing từ LSASS.
- Executable chạy từ thư mục đáng ngờ.
- Macro tạo process bất thường.

---

### Application Whitelisting

**Application Whitelisting** là cơ chế chỉ cho phép ứng dụng được phê duyệt chạy.

Dù khó triển khai toàn diện, tổ chức vẫn có thể áp dụng bước cơ bản:

- Chặn thực thi từ Downloads.
- Chặn thực thi từ Desktop.
- Chặn thực thi từ AppData.
- Chặn script lạ.
- Chặn file `.hta`, `.vbs`, `.cmd`, `.bat`, `.js`.

Các thư mục do user ghi thường là nơi payload ban đầu được tải xuống.

---

### Chú ý LOLBins

**LOLBins** là viết tắt của **Living Off the Land Binaries**.

Đây là các công cụ hợp pháp có sẵn trong Windows nhưng bị attacker lợi dụng.

Ví dụ:

- powershell.exe
- cmd.exe
- wscript.exe
- cscript.exe
- mshta.exe
- rundll32.exe
- regsvr32.exe
- certutil.exe
- bitsadmin.exe

Khi triển khai whitelist, không nên bỏ qua LOLBins vì attacker thường dùng chúng để bypass kiểm soát.

---

### Host-based Firewall

**Host-based Firewall** là firewall chạy trên từng máy endpoint/server.

Tối thiểu nên:

- Chặn workstation-to-workstation traffic không cần thiết.
- Chặn inbound không cần thiết.
- Giới hạn outbound traffic đáng ngờ.
- Chặn LOLBins kết nối Internet nếu không cần.
- Chỉ cho phép service hợp lệ giao tiếp.

Mục tiêu là giảm khả năng attacker lateral movement trong mạng nội bộ.

---

# 4. EDR và AMSI

## EDR là gì?

**EDR** là viết tắt của **Endpoint Detection and Response**.

EDR giúp giám sát, phát hiện và phản ứng với hành vi đáng ngờ trên endpoint.

EDR có thể theo dõi:

- Process.
- Command line.
- File creation.
- Registry change.
- Network connection.
- Parent-child process.
- Script execution.
- Credential access behavior.

---

## AMSI là gì?

**AMSI** là viết tắt của **Antimalware Scan Interface**.

AMSI là giao diện của Windows cho phép antivirus/EDR kiểm tra nội dung script trước khi script được thực thi.

AMSI đặc biệt hữu ích với:

- PowerShell.
- VBScript.
- JScript.
- Office macro.
- Script bị obfuscate.

Ví dụ:

```text
Script bị obfuscate
        ↓
PowerShell chuẩn bị chạy script
        ↓
AMSI gửi nội dung script cho antivirus/EDR kiểm tra
        ↓
Nếu độc hại thì bị chặn
```

Khi chọn EDR hoặc antimalware, nên ưu tiên sản phẩm tích hợp tốt với AMSI.

---

# 5. Network Protection

## Network Segmentation

**Network Segmentation** là phân đoạn mạng.

Mục tiêu là chia mạng thành các vùng riêng biệt để hạn chế attacker di chuyển tự do.

Ví dụ:

- User VLAN.
- Server VLAN.
- DMZ.
- Management VLAN.
- Backup network.
- Domain Controller network.
- Production network.

Nguyên tắc:

```text
Hệ thống quan trọng phải được cô lập.
Chỉ cho phép kết nối khi có nhu cầu nghiệp vụ rõ ràng.
```

---

## DMZ

**DMZ** là vùng mạng trung gian dùng để đặt các hệ thống cần public ra Internet.

Ví dụ:

- Web server public.
- Reverse proxy.
- VPN gateway.
- Mail gateway.

Tài nguyên nội bộ không nên expose trực tiếp ra Internet. Nếu bắt buộc public dịch vụ, nên đặt trong DMZ và kiểm soát chặt traffic.

---

## IDS/IPS

**IDS** là **Intrusion Detection System**.

**IPS** là **Intrusion Prevention System**.

| Hệ thống | Chức năng |
|---|---|
| IDS | Phát hiện xâm nhập |
| IPS | Phát hiện và chặn xâm nhập |

IDS/IPS có thể phát hiện:

- Scan.
- Exploit.
- Malware traffic.
- C2 traffic.
- Suspicious DNS.
- Lateral movement.
- Known attack signatures.

IDS/IPS hiệu quả hơn khi có khả năng kiểm tra nội dung traffic. Nếu traffic bị mã hóa SSL/TLS, hệ thống cần SSL/TLS inspection để nhìn thấy nội dung.

---

## 802.1X

**802.1X** là cơ chế kiểm soát truy cập mạng ở tầng truy cập.

Nó giúp đảm bảo chỉ thiết bị được phê duyệt mới được kết nối vào mạng công ty.

Mục tiêu:

- Giảm rủi ro BYOD không kiểm soát.
- Chặn thiết bị lạ cắm vào mạng.
- Xác thực thiết bị/người dùng trước khi cấp network access.

---

## Conditional Access

**Conditional Access** là chính sách truy cập có điều kiện, thường dùng trong môi trường cloud như Microsoft Entra ID.

Ví dụ chỉ cho phép truy cập tài nguyên nếu:

- Thiết bị do công ty quản lý.
- User bật MFA.
- Thiết bị compliant.
- Không đến từ quốc gia rủi ro.
- Không có dấu hiệu đăng nhập bất thường.

---

# 6. Privileged Identity Management, MFA và Password

## Vì sao tài khoản đặc quyền nguy hiểm?

Trong môi trường Active Directory, đánh cắp credential của tài khoản đặc quyền là một trong những đường leo thang phổ biến nhất.

Nếu attacker lấy được tài khoản admin, họ có thể:

- Lateral movement.
- Dump credential.
- Tạo persistence.
- Tắt security tools.
- Truy cập dữ liệu nhạy cảm.
- Triển khai ransomware.

---

## Password yếu nhưng có vẻ phức tạp

Một ví dụ mật khẩu yếu nhưng nhìn có vẻ phức tạp:

```text
Password1!
```

Mật khẩu này có:

- Chữ hoa.
- Chữ thường.
- Số.
- Ký tự đặc biệt.

Nhưng vẫn yếu vì rất dễ đoán và thường có trong wordlist attacker dùng.

---

## Passphrase

**Passphrase** là cụm mật khẩu dài, dễ nhớ nhưng khó đoán hơn mật khẩu ngắn.

Ví dụ:

```text
i LIK3 my coffeE warm
```

Passphrase tốt vì:

- Dài hơn.
- Dễ nhớ hơn.
- Khó brute-force hơn.
- Có thể kết hợp nhiều ngôn ngữ.

---

## MFA

**MFA** là viết tắt của **Multi-Factor Authentication**.

MFA yêu cầu nhiều hơn một yếu tố xác thực.

Ví dụ:

- Mật khẩu.
- OTP.
- Push notification.
- Hardware token.
- Biometrics.

MFA nên được triển khai ít nhất cho mọi truy cập quản trị vào ứng dụng và thiết bị.

---

# 7. Vulnerability Scanning

**Vulnerability Scanning** là quá trình quét lỗ hổng trên hệ thống.

Mục tiêu:

- Phát hiện lỗ hổng.
- Xác định hệ thống chưa vá.
- Ưu tiên khắc phục lỗi nghiêm trọng.
- Giảm khả năng attacker khai thác.

Tổ chức nên thực hiện quét lỗ hổng liên tục và ưu tiên khắc phục:

- Critical vulnerabilities.
- High vulnerabilities.

Nếu chưa thể vá ngay, cần có biện pháp giảm thiểu như:

- Phân đoạn hệ thống.
- Chặn truy cập không cần thiết.
- Tắt dịch vụ rủi ro.
- Giới hạn network exposure.

---

# 8. User Awareness Training

**User Awareness Training** là đào tạo người dùng nhận biết rủi ro bảo mật.

Mục tiêu:

- Nhận biết phishing.
- Báo cáo email đáng ngờ.
- Không chạy file lạ.
- Không cắm USB lạ.
- Không chia sẻ password.
- Nhận biết social engineering.

Các bài kiểm tra định kỳ có thể gồm:

- Email phishing giả lập.
- USB drop test.
- Bài kiểm tra nhận thức.
- Mô phỏng tình huống social engineering.

Dù không thể đạt 100%, đào tạo tốt có thể giảm đáng kể số lượng compromise thành công.

---

# 9. Active Directory Security Assessment

**Active Directory Security Assessment** là đánh giá bảo mật Active Directory.

Mục tiêu là tìm cấu hình sai hoặc đường leo thang đặc quyền từ góc nhìn attacker.

Các vấn đề thường gặp:

- User có quyền quá cao.
- Service account yếu.
- Kerberoasting path.
- Local admin dùng chung password.
- GPO cấu hình sai.
- Delegation sai.
- ACL nguy hiểm.
- Domain Admin dùng sai cách.
- Legacy protocol còn bật.

Đánh giá AD giúp loại bỏ các “easy win” mà attacker có thể lợi dụng sau khi compromise một endpoint.

---

# 10. Purple Team Exercise

## Purple Team là gì?

**Purple Team** là hoạt động kết hợp giữa:

- **Red Team**: mô phỏng attacker.
- **Blue Team**: phát hiện, phòng thủ và phản ứng.

Mục tiêu của Purple Team không chỉ là tấn công, mà là cải thiện khả năng phòng thủ.

---

## Lợi ích của Purple Team

Purple Team giúp tổ chức:

- Kiểm tra logging.
- Kiểm tra monitoring.
- Kiểm tra detection rule.
- Kiểm tra alert triage.
- Kiểm tra incident playbook.
- Tìm điểm mù trong phòng thủ.
- Cải thiện khả năng phản ứng của Blue Team.

Nếu một hành vi tấn công không bị phát hiện, đó là cơ hội để cải thiện detection.

Nếu một hành vi đã bị phát hiện, Blue Team có thể kiểm tra quy trình xử lý có hiệu quả không.

---

# Bảng ôn nhanh thuật ngữ

| Thuật ngữ | Nghĩa ngắn gọn |
|---|---|
| DMARC | Cơ chế chống giả mạo email dựa trên SPF/DKIM |
| SPF | Khai báo server được phép gửi email cho domain |
| DKIM | Chữ ký số xác thực email |
| Endpoint Hardening | Gia cố bảo mật endpoint |
| EDR | Endpoint Detection and Response |
| AMSI | Giao diện cho antimalware kiểm tra script trước khi chạy |
| LLMNR | Cơ chế phân giải tên nội bộ có thể bị lạm dụng |
| NetBIOS | Cơ chế tên mạng cũ trong Windows |
| LAPS | Quản lý mật khẩu local administrator |
| ASR | Attack Surface Reduction |
| Whitelisting | Chỉ cho phép ứng dụng được phê duyệt chạy |
| LOLBins | Công cụ hợp pháp bị attacker lạm dụng |
| Host-based Firewall | Firewall trên từng endpoint/server |
| Network Segmentation | Phân đoạn mạng |
| DMZ | Vùng mạng trung gian cho dịch vụ public |
| IDS | Hệ thống phát hiện xâm nhập |
| IPS | Hệ thống ngăn chặn xâm nhập |
| 802.1X | Kiểm soát truy cập mạng |
| Conditional Access | Truy cập có điều kiện |
| MFA | Xác thực đa yếu tố |
| Passphrase | Cụm mật khẩu dài, dễ nhớ, khó đoán |
| Vulnerability Scanning | Quét lỗ hổng |
| AD Security Assessment | Đánh giá bảo mật Active Directory |
| Purple Team | Kết hợp Red Team và Blue Team để cải thiện phòng thủ |

---

# Câu hỏi ôn tập

## 1. Vì sao đội Incident Handling cần biết các biện pháp bảo vệ?

Vì khi incident xảy ra, hiểu các biện pháp bảo vệ giúp analyst biết attacker có thể đã vượt qua lớp phòng thủ nào và nên tìm bằng chứng ở đâu.

## 2. DMARC dùng để làm gì?

DMARC dùng để chống email spoofing và phishing bằng cách xác minh email dựa trên SPF và DKIM.

## 3. Vì sao nên tắt LLMNR/NetBIOS?

Vì attacker có thể lợi dụng LLMNR/NetBIOS để đánh cắp hash hoặc thực hiện relay attack trong mạng nội bộ.

## 4. Vì sao LAPS quan trọng?

LAPS giúp mỗi máy có mật khẩu local admin riêng, giảm khả năng attacker dùng một mật khẩu để lateral movement sang nhiều máy.

## 5. Vì sao MFA cần thiết cho tài khoản quản trị?

Vì nếu attacker biết mật khẩu, họ vẫn cần yếu tố xác thực thứ hai để truy cập.

## 6. Purple Team giúp gì cho Blue Team?

Purple Team giúp kiểm tra detection, logging, monitoring và playbook xử lý sự cố trong môi trường thực tế.

---

# Key Takeaway

Giai đoạn **Preparation** không chỉ là chuẩn bị tài liệu và công cụ cho đội Incident Handling, mà còn bao gồm việc hiểu và triển khai các biện pháp bảo vệ quan trọng.

Các biện pháp như **DMARC**, **endpoint hardening**, **EDR**, **network segmentation**, **MFA**, **vulnerability scanning**, **AD security assessment** và **Purple Team exercise** giúp giảm khả năng incident xảy ra và tăng khả năng phát hiện khi attacker hoạt động.

Đối với SOC analyst, hiểu các lớp phòng thủ này giúp điều tra chính xác hơn, xác định artifact nhanh hơn và đưa ra phản ứng phù hợp hơn.

---

# References

- DMARCly - What is DMARC: <https://dmarcly.com/blog/how-to-implement-dmarc-dkim-spf-to-stop-email-spoofing-phishing-the-definitive-guide#what-is-dmarc>
- DMARCly - What is DKIM: <https://dmarcly.com/blog/how-to-implement-dmarc-dkim-spf-to-stop-email-spoofing-phishing-the-definitive-guide#what-is-dkim>
- Microsoft Learn - How AMSI helps: <https://learn.microsoft.com/en-us/windows/win32/amsi/how-amsi-helps>

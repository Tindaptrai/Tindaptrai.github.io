\---

title: "Incident Handling"

date: 2026-06-02 00:00:00 +0700

categories: \[HTB, CDSA]

tags: \[incident-handling, dfir, soc, blue-team]

description: "HTB/CDSA note về xử lý sự cố bảo mật, vòng đời ứng phó sự cố, các loại incident thực tế và cách viết báo cáo theo hướng DFIR/IR."

\---



\# Incident Handling



HTB/CDSA note về \*\*xử lý sự cố bảo mật\*\*, vòng đời ứng phó sự cố, các loại incident thực tế và cách viết báo cáo theo hướng DFIR/IR.



Bài này được ghi chú theo style kỹ thuật: \*\*Concept → Flow → Use cases → Scenario → Quick Reference\*\*.



\---



\## Concept



> \*\*Incident Handling\*\* là tập hợp các quy trình rõ ràng để quản lý, điều tra và phản ứng với các sự cố bảo mật trong môi trường máy tính hoặc mạng.



Nói dễ hiểu hơn, Incident Handling là toàn bộ quá trình từ lúc tổ chức \*\*chuẩn bị\*\*, \*\*phát hiện sự cố\*\*, \*\*phân tích\*\*, \*\*cô lập\*\*, \*\*loại bỏ nguyên nhân\*\*, \*\*khôi phục hệ thống\*\* cho đến \*\*rút kinh nghiệm sau sự cố\*\*.



\---



\## Event vs Incident vs IT Security Incident



| Thuật ngữ | Ý nghĩa |

|---|---|

| \*\*Event\*\* | Một hành động xảy ra trong hệ thống hoặc mạng. Ví dụ: user gửi email, click chuột, firewall cho phép kết nối. |

| \*\*Incident\*\* | Một event có hậu quả tiêu cực. Ví dụ: hệ thống bị sập, truy cập trái phép dữ liệu. |

| \*\*IT Security Incident\*\* | Một sự kiện có chủ ý gây hại đến hệ thống máy tính, dữ liệu hoặc dịch vụ. |



Ví dụ về \*\*security incident\*\*:



\- Đánh cắp dữ liệu.

\- Trộm tiền.

\- Truy cập trái phép dữ liệu nhạy cảm.

\- Cài malware hoặc remote access tool.

\- Tấn công gây gián đoạn dịch vụ.



\---



\## Incident Handling Lifecycle



Vòng đời xử lý sự cố thường gồm các giai đoạn chính:



```text

Preparation

&#x20;  ↓

Detection \& Analysis

&#x20;  ↓

Containment, Eradication \& Recovery

&#x20;  ↓

Post-Incident Activity

&#x20;  ↺

Continuous Improvement

```



| Giai đoạn | Mục tiêu |

|---|---|

| \*\*Preparation\*\* | Chuẩn bị quy trình, công cụ, nhân sự, logging, playbook. |

| \*\*Detection \& Analysis\*\* | Phát hiện dấu hiệu bất thường, phân tích log, xác định có phải incident không. |

| \*\*Containment\*\* | Cô lập hệ thống bị ảnh hưởng để ngăn sự cố lan rộng. |

| \*\*Eradication\*\* | Loại bỏ malware, tài khoản độc hại, persistence, lỗ hổng bị khai thác. |

| \*\*Recovery\*\* | Khôi phục dịch vụ, đưa hệ thống trở lại trạng thái an toàn. |

| \*\*Post-Incident Activity\*\* | Rút kinh nghiệm, cập nhật rule, vá quy trình, viết báo cáo. |



\---



\## Scope



Incident Handling \*\*không chỉ giới hạn ở việc bị hacker xâm nhập\*\*.



Các loại sự cố cũng thuộc phạm vi xử lý:



\- Insider threat.

\- Mất tính sẵn sàng của dịch vụ.

\- Mất dữ liệu hoặc tài sản trí tuệ.

\- Ransomware.

\- Lộ thông tin đăng nhập.

\- Cấu hình sai.

\- Hệ thống chưa vá lỗi.

\- Malware infection.

\- Data exfiltration.

\- Unauthorized access.



Một event có thể \*\*chưa chắc là incident ngay từ đầu\*\*, nhưng nếu đáng ngờ thì nên xử lý như incident cho đến khi điều tra chứng minh ngược lại.



\---



\## Why Incident Handling Matters



Mục tiêu chính của xử lý sự cố:



```text

Giảm thiểu thiệt hại

Giảm thời gian gián đoạn

Ngăn mất dữ liệu

Khôi phục hoạt động bình thường

Rút kinh nghiệm để phòng thủ tốt hơn

```



Điểm quan trọng:



\- Incident càng nghiêm trọng thì càng cần ưu tiên xử lý nhanh.

\- Cần có \*\*incident response team\*\* để phản ứng có quy trình.

\- Mỗi quyết định trước, trong và sau incident đều ảnh hưởng đến mức độ thiệt hại.

\- Người điều phối chính thường là \*\*Incident Manager\*\*.

\- Một quy trình IR tốt giúp tổ chức phản ứng nhất quán, tránh xử lý cảm tính.



\---



\## Role: Incident Manager



\*\*Incident Manager\*\* là đầu mối điều phối toàn bộ quá trình xử lý sự cố.



Nhiệm vụ chính:



\- Theo dõi trạng thái điều tra.

\- Điều phối SOC, IT, network, system, legal, management.

\- Yêu cầu nhân sự liên quan thực hiện hành động cần thiết.

\- Đảm bảo mọi hoạt động được ghi nhận.

\- Là điểm liên lạc chính trong quá trình incident.

\- Theo dõi tiến độ containment, eradication và recovery.

\- Tổng hợp thông tin để báo cáo cho cấp quản lý.



Thông thường vai trò này có thể là:



\- SOC Manager.

\- CISO/CIO.

\- Người phụ trách bảo mật nội bộ.

\- Nhà cung cấp IR bên thứ ba.



\---



\# Common Real-World Incident Types



\## 1. Leaked Credentials



Ví dụ: \*\*Colonial Pipeline Ransomware Attack\*\*



Nguyên nhân chính:



```text

Mật khẩu VPN bị lộ

Tài khoản không còn hoạt động nhưng chưa bị vô hiệu hóa

Không bật MFA

```



Tác động:



\- Attacker dùng credential bị lộ để truy cập hệ thống.

\- Dẫn đến ransomware và gián đoạn hoạt động lớn.

\- Gây ảnh hưởng đến vận hành, tài chính và uy tín tổ chức.



Bài học:



\- Bắt buộc MFA cho VPN và tài khoản quan trọng.

\- Không để tài khoản cũ tồn tại nếu không còn sử dụng.

\- Giám sát đăng nhập bất thường.

\- Kiểm tra credential leak trên dark web hoặc public breach database.



\---



\## 2. Default / Weak Credentials



Ví dụ: \*\*Mirai Botnet\*\*



```text

IoT devices dùng tài khoản mặc định

Ví dụ: admin/admin

Attacker scan hàng loạt thiết bị

Thiết bị bị đưa vào botnet DDoS

```



Tác động:



\- Hàng trăm nghìn thiết bị IoT bị điều khiển.

\- Botnet được dùng để DDoS quy mô lớn.

\- Gây gián đoạn dịch vụ của nhiều tổ chức.



Bài học:



\- Không dùng mật khẩu mặc định.

\- Bắt buộc đổi password khi triển khai.

\- Hạn chế remote access không cần thiết.

\- Dùng password policy đủ mạnh.

\- Kiểm tra định kỳ thiết bị IoT/OT trong mạng.



\---



\## 3. Outdated Software / Unpatched Systems



Ví dụ: \*\*Equifax 2017\*\*



```text

Apache Struts có CVE đã công bố

Bản vá đã có

Tổ chức không vá kịp thời

Attacker khai thác thành công

```



Ví dụ khác: \*\*WannaCry 2017\*\*



```text

Khai thác SMB EternalBlue

Lan truyền dạng worm

Ảnh hưởng hơn 200.000 hệ thống

Nguyên nhân: Windows chưa vá MS17-010

```



Bài học:



\- Patch management cực kỳ quan trọng.

\- CVE public mà không vá là rủi ro rất cao.

\- Hệ thống legacy cần được cô lập.

\- Cần vulnerability scanning định kỳ.

\- Ưu tiên vá các lỗ hổng có exploit public hoặc đang bị khai thác thực tế.



\---



\## 4. Insider Threat



Ví dụ: \*\*Cash App / Block Inc.\*\*



```text

Cựu nhân viên lạm dụng quyền truy cập hợp pháp

Truy cập dữ liệu người dùng

Kiểm soát nội bộ và giám sát chưa đủ chặt

```



Tác động:



\- Dữ liệu người dùng bị truy cập trái phép.

\- Tổ chức bị điều tra, mất uy tín và có thể bị phạt.

\- Khó phát hiện hơn attacker bên ngoài vì hành vi đến từ tài khoản hợp lệ.



Bài học:



\- Cần nguyên tắc \*\*least privilege\*\*.

\- Thu hồi quyền ngay khi nhân viên nghỉ việc.

\- Giám sát truy cập dữ liệu nhạy cảm.

\- Tách quyền theo vai trò.

\- Log đầy đủ hành vi truy cập và thay đổi dữ liệu.



\---



\## 5. Phishing / Social Engineering



Ví dụ: \*\*Twitter Account Hijacking 2020\*\*



```text

Attacker dùng social engineering

Truy cập được công cụ quản trị nội bộ

Chiếm tài khoản nổi tiếng

Đăng lừa đảo bitcoin

```



Tác động:



\- Nhiều tài khoản nổi tiếng bị chiếm.

\- Attacker đăng nội dung lừa đảo.

\- Gây thiệt hại về uy tín và niềm tin người dùng.



Bài học:



\- Con người là mắt xích quan trọng.

\- Cần MFA, security training, kiểm soát admin tool.

\- Không chỉ bảo vệ hệ thống, phải bảo vệ quy trình vận hành.

\- Hạn chế số người có quyền truy cập công cụ quản trị nhạy cảm.

\- Giám sát hành vi bất thường của nhân viên nội bộ.



\---



\## 6. Supply Chain Attack



Ví dụ: \*\*SolarWinds Orion 2020\*\*



```text

Attacker xâm phạm môi trường build/release

Chèn backdoor vào bản cập nhật hợp pháp

Khách hàng tải update bị nhiễm

```



Tác động:



\- Ảnh hưởng chính phủ và doanh nghiệp lớn.

\- Khó phát hiện vì malware đi qua kênh update tin cậy.

\- Dẫn đến hoạt động gián điệp kéo dài.



Bài học:



\- Cần bảo vệ CI/CD pipeline.

\- Kiểm tra integrity của update.

\- Theo dõi hành vi bất thường sau khi cập nhật phần mềm.

\- Áp dụng code signing.

\- Giám sát vendor risk và third-party risk.



\---



\# Incident Report Format



Một báo cáo incident tốt nên viết theo hướng \*\*tuần tự theo giai đoạn tấn công\*\*.



Ví dụ flow:



```text

Initial Access

&#x20;  ↓

Execution

&#x20;  ↓

Persistence

&#x20;  ↓

Privilege Escalation

&#x20;  ↓

Discovery

&#x20;  ↓

Lateral Movement

&#x20;  ↓

Collection / Exfiltration

&#x20;  ↓

Impact

```



Có thể map theo:



\- \*\*Cyber Kill Chain\*\*.

\- \*\*MITRE ATT\&CK\*\*.

\- Timeline sự kiện.

\- Evidence từ log, SIEM, EDR, firewall, endpoint.

\- Root cause.

\- Impact.

\- Remediation.

\- Lessons learned.



\---



\## Incident-Specific Reports vs Global IR Reports



| Loại báo cáo | Mục đích |

|---|---|

| \*\*Incident-specific report\*\* | Mô tả chi tiết một vụ việc cụ thể: attacker vào bằng cách nào, làm gì, bị phát hiện ra sao, tác động thế nào. |

| \*\*Global incident-response report\*\* | Tổng hợp dữ liệu từ nhiều incident để rút ra xu hướng, thống kê, threat trend và khuyến nghị cấp cao. |



Ví dụ nguồn báo cáo thực tế:



\- The DFIR Report.

\- Mandiant.

\- Palo Alto Unit 42.

\- Proofpoint.

\- Cybereason.



\---



\# Scenario: Insight Nexus



Nạn nhân giả lập: \*\*Insight Nexus\*\*



Đây là công ty nghiên cứu thị trường toàn cầu, xử lý dữ liệu cạnh tranh nhạy cảm cho khách hàng cao cấp trong lĩnh vực CNTT.



\## Mô hình tổng quan



```text

Internet Threat Actors

&#x20;       ↓

Firewall

&#x20;       ↓

Public-facing Web Apps

&#x20;┌───────────────────────────────┐

&#x20;│ ManageEngine ADManager Plus   │

&#x20;│ PHP Client Report Portal      │

&#x20;└───────────────────────────────┘

&#x20;       ↓

Internal Network

&#x20;┌───────────────────────────────┐

&#x20;│ Domain Controller             │

&#x20;│ Windows Workstations          │

&#x20;│ Database Server               │

&#x20;│ File Server                   │

&#x20;└───────────────────────────────┘

&#x20;       ↓

SIEM → TheHive → SOC Analyst

```



\---



\# Attack Chain — Threat Actor 1



\## Bước 1 — Initial Access



```text

Ứng dụng internet-facing: ManageEngine ADManager Plus

Lỗi: admin quên đổi credential mặc định

Credential: admin/admin

```



Attacker đăng nhập thành công vào ứng dụng vì tài khoản quản trị mặc định chưa được thay đổi sau khi cập nhật sản phẩm.



Điểm yếu:



\- Internet-facing admin panel.

\- Credential mặc định.

\- Không bắt buộc đổi password sau update.

\- Có thể thiếu MFA.



\---



\## Bước 2 — Reconnaissance



Sau khi vào được hệ thống, attacker thực hiện:



```text

User enumeration

Machine mapping

Domain reconnaissance

Kiểm tra quyền và tài nguyên nội bộ

```



Mục tiêu là hiểu cấu trúc Active Directory và tìm đường pivot sâu hơn.



Dấu hiệu có thể thấy:



\- Nhiều truy vấn AD bất thường.

\- Liệt kê user/group/computer.

\- Truy cập các chức năng quản trị bất thường.

\- Tần suất truy vấn tăng đột biến.



\---



\## Bước 3 — Privilege Abuse



Attacker tạo thêm các tài khoản Active Directory có quyền cao.



```text

Compromised Web App

&#x20;       ↓

Create privileged AD accounts

&#x20;       ↓

Use new accounts for lateral movement

```



Điểm nguy hiểm:



\- Tài khoản mới có quyền cao.

\- Attacker có thể duy trì truy cập.

\- Có thể dùng tài khoản này để pivot sang hệ thống khác.



Dấu hiệu cần theo dõi:



\- Account mới được tạo.

\- Account được thêm vào privileged group.

\- Thay đổi quyền bất thường.

\- Event log liên quan đến user/group management.



\---



\## Bước 4 — Lateral Movement



Attacker phát hiện một dịch vụ \*\*RDP exposed\*\* do cấu hình sai.



```text

Privileged AD Account

&#x20;       ↓

Misconfigured External RDP

&#x20;       ↓

Internal Access

```



Điểm yếu chính:



\- RDP bị lộ.

\- Cấu hình firewall/access control chưa chặt.

\- Tài khoản đặc quyền bị lạm dụng.

\- Thiếu network segmentation.



Dấu hiệu cần theo dõi:



\- RDP login từ IP bất thường.

\- Đăng nhập ngoài giờ làm việc.

\- Login thành công sau nhiều lần thất bại.

\- RDP session đến nhiều máy khác nhau.



\---



\## Bước 5 — Impact / Malware Deployment



Attacker sử dụng \*\*Group Policy Object — GPO\*\* để triển khai spyware qua gói MSI trên nhiều endpoint.



```text

GPO

&#x20;↓

MSI Package

&#x20;↓

Multiple Windows Endpoints

&#x20;↓

Spyware Deployment

```



Đây là giai đoạn nguy hiểm vì attacker đã có khả năng điều khiển triển khai trên diện rộng.



Điểm nguy hiểm:



\- Malware được triển khai qua cơ chế quản trị hợp pháp.

\- Nhiều endpoint bị ảnh hưởng cùng lúc.

\- Khó phân biệt hoạt động quản trị thật và hoạt động độc hại nếu không có baseline.



Dấu hiệu cần theo dõi:



\- GPO mới được tạo.

\- GPO bị chỉnh sửa bất thường.

\- MSI lạ được chạy trên nhiều máy.

\- Endpoint xuất hiện process/persistence bất thường.

\- Network beaconing ra ngoài.



\---



\# Detection Points



Các điểm có thể phát hiện trong scenario:



| Giai đoạn | Dấu hiệu cần theo dõi |

|---|---|

| Login vào ADManager | Đăng nhập admin bất thường từ IP lạ |

| Recon | Query AD nhiều bất thường |

| Tạo account | Account đặc quyền mới được tạo |

| RDP | Kết nối RDP từ nguồn không hợp lệ |

| GPO | GPO mới hoặc GPO bị chỉnh sửa |

| MSI deployment | Nhiều máy chạy file MSI lạ |

| Endpoint | Spyware process, persistence, network beaconing |



\---



\# Defensive Lessons



Các bài học phòng thủ chính:



\- Không để credential mặc định trên hệ thống production.

\- Bắt buộc MFA cho ứng dụng quản trị.

\- Hạn chế public-facing admin panel.

\- Kiểm soát RDP bằng VPN, allowlist, MFA.

\- Theo dõi việc tạo tài khoản đặc quyền.

\- Alert khi GPO bị chỉnh sửa.

\- SIEM cần gom log từ domain controller, endpoint, firewall và ứng dụng.

\- Có playbook xử lý cho credential compromise, malware deployment và lateral movement.

\- Áp dụng least privilege.

\- Kiểm tra định kỳ các tài khoản đặc quyền.

\- Phân đoạn mạng để hạn chế lateral movement.



\---



\# Quick Reference — Summary



```text

\# Core idea

Incident Handling = quy trình quản lý và phản ứng với security incident.



\# Event vs Incident

Event    = hành động xảy ra trong hệ thống.

Incident = event có hậu quả tiêu cực.

Security Incident = event có chủ đích gây hại đến hệ thống/dữ liệu.



\# Lifecycle

Preparation

Detection \& Analysis

Containment

Eradication

Recovery

Post-Incident Activity



\# Key role

Incident Manager = người điều phối chính trong toàn bộ quá trình xử lý sự cố.



\# Common causes

Leaked credentials

Default / weak passwords

Unpatched systems

Insider threat

Phishing / social engineering

Supply chain compromise



\# Report style

Viết theo timeline hoặc attack chain:

Initial Access → Execution → Persistence → Privilege Escalation

→ Discovery → Lateral Movement → Exfiltration / Impact



\# Scenario root cause

ManageEngine ADManager Plus dùng credential mặc định admin/admin.



\# Scenario impact

Attacker tạo tài khoản AD đặc quyền, pivot qua RDP cấu hình sai,

sau đó dùng GPO triển khai spyware bằng MSI trên nhiều endpoint.

```



\---



\# Key Takeaway



Incident Handling không chỉ là \*\*“bị hack rồi xử lý”\*\*, mà là một quy trình đầy đủ gồm:



```text

Chuẩn bị

Phát hiện

Phân tích

Cô lập

Loại bỏ

Phục hồi

Rút kinh nghiệm

```



Điểm quan trọng nhất là phải có:



\- Quy trình rõ ràng.

\- Phân quyền trách nhiệm cụ thể.

\- Log đầy đủ.

\- Khả năng phản ứng nhanh.

\- Playbook xử lý theo từng loại incident.

\- Cơ chế cải thiện liên tục sau mỗi sự cố.



Mục tiêu cuối cùng là \*\*giảm thiểu thiệt hại\*\*, \*\*khôi phục hoạt động nhanh nhất có thể\*\*, và \*\*nâng cấp khả năng phòng thủ sau mỗi incident\*\*.


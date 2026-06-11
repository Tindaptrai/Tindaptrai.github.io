---
layout: post
title: "Preparation Stage trong Incident Handling"
date: 2026-06-03 02:00:00 +0700
categories: [SOC, Incident Response]
tags: [soc, incident-response, preparation-stage, jump-bag, forensic, blue-team, dfir]
description: "Ghi chú tổng hợp về giai đoạn Preparation trong Incident Handling, các chính sách, tài liệu, công cụ cần chuẩn bị và bảng thuật ngữ chuyên ngành."
toc: true
---

# Preparation Stage trong Incident Handling

Trong quy trình **Incident Handling**, giai đoạn **Preparation** là giai đoạn chuẩn bị trước khi sự cố bảo mật xảy ra.

Đây là giai đoạn rất quan trọng vì nếu tổ chức không chuẩn bị sẵn con người, quy trình, tài liệu và công cụ thì khi incident xảy ra, đội xử lý sẽ dễ bị rối, phản ứng chậm hoặc bỏ sót bằng chứng quan trọng.

---

## Tóm tắt nhanh: Preparation Stage

Trong giai đoạn **Preparation**, đội Incident Handling cần chuẩn bị trước khi sự cố xảy ra.

Có **2 mục tiêu chính**:

### 1. Xây dựng năng lực xử lý sự cố trong tổ chức

Tổ chức cần chuẩn bị:

- Có đội **Incident Handling**.
- Có người được đào tạo.
- Có quy trình, chính sách và tài liệu.
- Có công cụ phục vụ điều tra và phản ứng.
- Có khả năng ghi nhận, báo cáo và phối hợp khi sự cố xảy ra.

Nói dễ hiểu, tổ chức phải có sẵn đội ngũ và phương án xử lý, thay vì đợi đến khi bị tấn công mới bắt đầu tìm người, tìm công cụ và viết quy trình.

### 2. Phòng ngừa sự cố bảo mật

Ngoài việc chuẩn bị để xử lý incident, tổ chức cũng cần triển khai các biện pháp phòng ngừa để giảm khả năng sự cố xảy ra.

Một số biện pháp quan trọng:

- **Hardening endpoint/server**.
- **Active Directory tiering**.
- **Multi-Factor Authentication**.
- **Privileged Access Management**.
- Chuẩn bị khả năng phát hiện và phản ứng nhanh.

Các hoạt động phòng ngừa này không phải lúc nào cũng do riêng đội Incident Handling thực hiện, nhưng chúng ảnh hưởng trực tiếp đến hiệu quả xử lý sự cố.

---

# Những thứ cần chuẩn bị

Trong giai đoạn Preparation, có hai nhóm quan trọng cần chuẩn bị:

```text
Policies & Documentation
Tools
```

---

## 1. Policies & Documentation

Tổ chức cần có tài liệu rõ ràng để khi incident xảy ra, đội xử lý biết phải liên hệ ai, làm gì, làm theo quy trình nào và ghi nhận thông tin ra sao.

Các tài liệu cần có:

- Thông tin liên hệ của đội Incident Response.
- Thông tin liên hệ của legal, compliance, management, IT support, ISP, law enforcement.
- Incident response policy, plan, procedures.
- Network diagrams.
- Asset inventory.
- Baseline hệ thống/network.
- Tài khoản đặc quyền dùng khi incident xảy ra.
- Forensic/investigative cheat sheets.
- Quy trình ghi nhận bằng chứng và báo cáo.

---

## Ghi chú trong quá trình điều tra

Trong quá trình xử lý incident, analyst cần ghi chú đầy đủ.

Nguyên tắc cần nhớ:

```text
Who, What, When, Where, Why, How
```

Ý nghĩa:

| Câu hỏi | Ý nghĩa |
|---|---|
| Who | Ai thực hiện hoặc ai liên quan? |
| What | Chuyện gì đã xảy ra? |
| When | Xảy ra khi nào? |
| Where | Xảy ra ở đâu, hệ thống nào? |
| Why | Vì sao hành động đó được thực hiện? |
| How | Sự cố xảy ra hoặc được xử lý như thế nào? |

Ghi chú tốt giúp tạo timeline, viết báo cáo, phục vụ pháp lý và rút kinh nghiệm sau sự cố.

---

## 2. Tools

Đội Incident Handling cần chuẩn bị sẵn các công cụ để điều tra, thu thập bằng chứng, phân tích log, phân tích hệ thống và khôi phục sau sự cố.

Các công cụ cần chuẩn bị:

- Forensic workstation/laptop.
- Disk imaging tools.
- Memory capture tools.
- Log analysis tools.
- Network capture tools.
- Write blockers.
- Hard drives.
- Network cables/switches.
- Chain of custody forms.
- Encryption software.
- Ticket tracking system.
- Secure investigation/storage facility.
- Incident handling system độc lập với hạ tầng công ty.

---

# Vì sao cần hệ thống độc lập?

Trong một incident nghiêm trọng, cần giả định rằng hạ tầng nội bộ có thể đã bị compromise.

Vì vậy, không nên phụ thuộc hoàn toàn vào:

- Email nội bộ.
- Domain nội bộ.
- File server nội bộ.
- Chat nội bộ.
- Ticket system nằm trong domain bị compromise.

Tổ chức nên có kênh liên lạc và hệ thống quản lý incident độc lập, để nếu hạ tầng chính bị ảnh hưởng thì đội xử lý vẫn có thể làm việc.

---

# Jump Bag

Một phần quan trọng trong Preparation là chuẩn bị **Jump Bag**.

**Jump Bag** là túi hoặc bộ công cụ phản ứng sự cố được chuẩn bị sẵn để có thể cầm đi ngay khi incident xảy ra.

Nói dễ hiểu:

```text
Jump Bag = túi đồ nghề cấp cứu của đội Incident Response
```

Trong Jump Bag có thể có:

- Laptop forensic.
- Ổ cứng.
- USB toolkit.
- Network cable.
- Write blocker.
- Power cable.
- Screwdriver.
- Chain of custody forms.
- Công cụ thu thập log.
- Công cụ memory capture.
- Công cụ network capture.

Nếu không chuẩn bị Jump Bag từ trước, khi incident xảy ra đội xử lý có thể mất nhiều giờ hoặc nhiều ngày chỉ để tìm đủ công cụ cần thiết.

---

# Ghi chú thuật ngữ chuyên ngành

## Incident Handling

**Incident Handling** nghĩa là **xử lý sự cố bảo mật**.

Đây là toàn bộ quá trình từ chuẩn bị, phát hiện, phân tích, cô lập, loại bỏ, khôi phục cho đến báo cáo sau sự cố.

Ví dụ incident:

- Máy tính bị nhiễm malware.
- Tài khoản bị compromise.
- Web server bị khai thác.
- Có attacker truy cập trái phép.
- Có dấu hiệu ransomware.
- Có dữ liệu bị đánh cắp.

---

## Preparation Stage

**Preparation Stage** nghĩa là **giai đoạn chuẩn bị**.

Đây là giai đoạn diễn ra trước khi sự cố xảy ra.

Mục tiêu là chuẩn bị sẵn:

- Con người.
- Quy trình.
- Chính sách.
- Tài liệu.
- Công cụ.
- Hệ thống giám sát.
- Phương án khôi phục.

---

## Incident Handling Team

**Incident Handling Team** là **đội xử lý sự cố bảo mật**.

Đội này có nhiệm vụ điều tra và xử lý khi có sự cố xảy ra.

Thành viên có thể gồm:

- SOC Analyst.
- Incident Responder.
- Forensic Analyst.
- System Administrator.
- Network Administrator.
- Security Engineer.
- Legal/Compliance.

---

## Security Awareness

**Security Awareness** nghĩa là **nhận thức an toàn thông tin**.

Đây là các hoạt động giúp người dùng hiểu các nguy cơ bảo mật và biết cách xử lý khi gặp dấu hiệu đáng ngờ.

Ví dụ:

- Nhận biết email phishing.
- Không tải file lạ.
- Không cắm USB lạ.
- Không chia sẻ mật khẩu.
- Báo cáo sự cố khi thấy dấu hiệu bất thường.

---

## Policy

**Policy** nghĩa là **chính sách**.

Trong an ninh mạng, policy là tài liệu quy định tổ chức phải làm gì và không được làm gì.

Ví dụ:

- Password Policy.
- Incident Response Policy.
- Access Control Policy.
- Backup Policy.
- Data Classification Policy.

---

## Documentation

**Documentation** nghĩa là **tài liệu hóa** hoặc **tài liệu kỹ thuật/quy trình**.

Documentation giúp đội xử lý biết phải làm gì khi sự cố xảy ra.

Ví dụ:

- Sơ đồ mạng.
- Danh sách server.
- Danh sách người liên hệ.
- Quy trình phản ứng sự cố.
- Hướng dẫn thu thập log.
- Hướng dẫn cô lập máy.
- Hướng dẫn restore backup.

---

## Baseline

**Baseline** nghĩa là **trạng thái chuẩn hoặc bình thường của hệ thống**.

Ví dụ baseline:

- Server bình thường chạy những service nào.
- Máy người dùng bình thường có process nào.
- Lưu lượng mạng bình thường khoảng bao nhiêu.
- Tài khoản admin bình thường đăng nhập từ đâu.
- Port nào được phép mở.

Baseline giúp analyst biết điều gì là bất thường.

---

## Golden Image

**Golden Image** là **bản image hệ thống sạch và chuẩn**.

Đây là bản hệ điều hành đã được cấu hình sẵn, vá lỗi, hardening và không có malware.

Dùng để:

- Triển khai máy mới.
- Rebuild máy bị nhiễm.
- Khôi phục môi trường sạch.
- So sánh với máy nghi bị compromise.

---

## Network Diagram

**Network Diagram** nghĩa là **sơ đồ mạng**.

Sơ đồ mạng cho biết:

- Các subnet.
- Firewall.
- Router.
- Switch.
- Server.
- DMZ.
- Đường kết nối.
- Hệ thống quan trọng.
- Luồng traffic.

Khi incident xảy ra, network diagram giúp xác định attacker có thể di chuyển từ đâu sang đâu.

---

## Asset Management Database

**Asset Management Database** là **cơ sở dữ liệu quản lý tài sản CNTT**.

Nó chứa thông tin về các thiết bị trong tổ chức.

Ví dụ:

- Tên máy.
- IP.
- MAC.
- Owner.
- Hệ điều hành.
- Vai trò của máy.
- Vị trí.
- Mức độ quan trọng.
- Phần mềm đang chạy.

---

## Excessive Privileges

**Excessive Privileges** nghĩa là **quyền vượt mức cần thiết**.

Ví dụ: một tài khoản nhân viên bình thường nhưng lại có quyền Domain Admin thì đó là excessive privileges.

Tài khoản quyền cao nên được kiểm soát kỹ, chỉ bật khi cần và phải reset password sau khi sử dụng.

---

## PAM

**PAM** là viết tắt của **Privileged Access Management**, nghĩa là **quản lý tài khoản đặc quyền**.

PAM kiểm soát các tài khoản có quyền cao như:

- Domain Admin.
- Local Administrator.
- Root.
- Database Admin.
- Firewall Admin.
- Cloud Admin.

PAM giúp giảm nguy cơ attacker lạm dụng tài khoản đặc quyền.

---

## Forensic Workstation

**Forensic Workstation** là **máy chuyên dùng để điều tra số**.

Máy này thường dùng để:

- Phân tích disk image.
- Phân tích memory dump.
- Phân tích log.
- Phân tích malware.
- Chạy tool forensic.

---

## Disk Imaging

**Disk Imaging** nghĩa là **tạo bản sao toàn bộ ổ đĩa**.

Đây không phải copy file thông thường, mà là sao chép toàn bộ nội dung ổ đĩa ở mức thấp.

Dùng để:

- Bảo toàn bằng chứng.
- Phân tích offline.
- Không làm thay đổi dữ liệu gốc.
- Phục vụ forensic.

---

## Memory Capture

**Memory Capture** là **thu thập RAM của máy tại thời điểm điều tra**.

RAM có thể chứa:

- Process đang chạy.
- Malware đang hoạt động.
- Network connection.
- Token.
- Credential.
- Command đã chạy.
- Dữ liệu chưa ghi xuống disk.

---

## Live Response

**Live Response** là **điều tra trên hệ thống đang chạy**.

Dùng khi máy chưa tắt và cần thu thập thông tin nhanh.

Ví dụ thu thập:

- Process.
- Network connection.
- Logged-in users.
- Scheduled tasks.
- Services.
- Autoruns.
- Recent files.
- Event logs.

---

## Log Analysis

**Log Analysis** nghĩa là **phân tích log**.

Log có thể đến từ:

- Windows Event Logs.
- Linux syslog.
- Firewall logs.
- Proxy logs.
- DNS logs.
- Web server logs.
- VPN logs.
- EDR logs.
- SIEM logs.

---

## Network Capture

**Network Capture** nghĩa là **bắt gói tin mạng**.

Dùng để xem traffic thực tế giữa các máy.

Tool thường dùng:

- Wireshark.
- tcpdump.
- Zeek.
- Arkime.

---

## Write Blocker

**Write Blocker** là thiết bị hoặc phần mềm giúp **ngăn ghi dữ liệu vào ổ đĩa gốc** khi phân tích forensic.

Mục tiêu là bảo toàn bằng chứng.

Ví dụ: analyst cắm ổ cứng qua write blocker để đọc dữ liệu nhưng không làm thay đổi dữ liệu trên ổ.

---

## Chain of Custody

**Chain of Custody** là **chuỗi quản lý bằng chứng**.

Nó ghi lại bằng chứng đã được ai thu thập, ai giữ, chuyển cho ai, vào lúc nào.

Thông tin thường có:

- Tên bằng chứng.
- Người thu thập.
- Thời gian thu thập.
- Vị trí thu thập.
- Người bàn giao.
- Người nhận.
- Hash của bằng chứng.
- Mục đích sử dụng.

---

## Ticket Tracking System

**Ticket Tracking System** là **hệ thống quản lý ticket/công việc**.

Dùng để theo dõi tiến trình xử lý incident.

Ví dụ:

- Jira.
- ServiceNow.
- TheHive.
- GLPI.
- OTRS.

---

## Secure Facility

**Secure Facility** là **khu vực an toàn để lưu trữ và điều tra**.

Dùng để:

- Lưu bằng chứng.
- Phân tích malware.
- Phân tích ổ đĩa.
- Làm việc với dữ liệu nhạy cảm.

Khu vực này nên có kiểm soát truy cập, camera, khóa vật lý và quy trình bàn giao rõ ràng.

---

## Jump Bag

**Jump Bag** là **túi đồ nghề phản ứng sự cố chuẩn bị sẵn**.

Đây là thứ mà đội Incident Response có thể cầm đi ngay khi có sự cố.

Trong Jump Bag thường có:

- Laptop forensic.
- Ổ cứng.
- USB toolkit.
- Dây mạng.
- Write blocker.
- Power cable.
- Công cụ thu thập log.
- Công cụ memory capture.
- Chain of custody forms.

---

# Bảng ôn nhanh thuật ngữ

| Thuật ngữ | Nghĩa ngắn gọn |
|---|---|
| Incident Handling | Xử lý sự cố bảo mật |
| Preparation Stage | Giai đoạn chuẩn bị |
| Incident Handling Team | Đội xử lý sự cố |
| Security Awareness | Nhận thức an toàn thông tin |
| Policy | Chính sách |
| Documentation | Tài liệu/quy trình |
| Baseline | Trạng thái chuẩn của hệ thống |
| Golden Image | Bản image sạch chuẩn |
| Network Diagram | Sơ đồ mạng |
| Asset Management Database | CSDL quản lý tài sản IT |
| Excessive Privileges | Quyền vượt mức cần thiết |
| PAM | Quản lý tài khoản đặc quyền |
| Forensic Workstation | Máy điều tra số |
| Disk Imaging | Tạo bản sao ổ đĩa |
| Memory Capture | Thu thập RAM |
| Live Response | Điều tra trên máy đang chạy |
| Log Analysis | Phân tích log |
| Network Capture | Bắt gói tin mạng |
| Write Blocker | Thiết bị chặn ghi vào ổ gốc |
| Chain of Custody | Chuỗi quản lý bằng chứng |
| Ticket Tracking System | Hệ thống quản lý ticket |
| Secure Facility | Khu vực điều tra/lưu trữ an toàn |
| Jump Bag | Túi đồ nghề phản ứng sự cố chuẩn bị sẵn |


---

# Key Takeaway

Giai đoạn **Preparation** quyết định rất lớn đến hiệu quả xử lý incident.

Một tổ chức chuẩn bị tốt sẽ có:

- Đội xử lý sự cố có kỹ năng.
- Nhân sự được đào tạo.
- Chính sách và tài liệu rõ ràng.
- Công cụ điều tra đầy đủ.
- Hệ thống quản lý incident độc lập.
- Jump bag luôn sẵn sàng để phản ứng nhanh.

Khi incident xảy ra, chuẩn bị tốt giúp đội SOC/IR phản ứng nhanh hơn, điều tra chính xác hơn và giảm thiểu thiệt hại cho tổ chức.

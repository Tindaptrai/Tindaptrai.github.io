---
layout: post
title: "Containment, Eradication và Recovery Stage"
date: 2026-06-10 02:00:00 +0700
categories: [SOC, Incident Response]
tags: [soc, incident-response, containment, eradication, recovery, forensic, blue-team]
description: "Ghi chú về giai đoạn Containment, Eradication và Recovery trong Incident Handling: cô lập sự cố, loại bỏ nguyên nhân, khôi phục hệ thống và giám sát sau khôi phục."
toc: true
---

# Containment, Eradication và Recovery Stage

Sau khi quá trình điều tra đã hoàn tất, đội Incident Handling đã hiểu được loại sự cố, phạm vi ảnh hưởng và tác động đến hoạt động kinh doanh, bước tiếp theo là đi vào giai đoạn:

```text
Containment, Eradication and Recovery
```

Mục tiêu của giai đoạn này là:

- Ngăn incident gây thêm thiệt hại.
- Cô lập hệ thống bị ảnh hưởng.
- Bảo toàn bằng chứng.
- Loại bỏ malware, backdoor hoặc root cause.
- Khôi phục hệ thống về trạng thái hoạt động bình thường.
- Giám sát hệ thống sau khi khôi phục.

---

## 1. Tổng quan quy trình

Giai đoạn này thường đi theo luồng:

```text
Investigation is complete
      ↓
Containment Strategy
      ↓
Evidence Gathering
      ↓
Identify the Attacking Host
      ↓
Eradication and Recovery
      ↓
Bring Systems Back to Normal Operation
```

Trong thực tế, các bước có thể lặp lại hoặc thay đổi tùy theo mức độ nghiêm trọng của incident.

---

# 2. Containment là gì?

**Containment** là giai đoạn cô lập sự cố để ngăn sự cố lan rộng hoặc gây thêm thiệt hại.

Mục tiêu của containment:

- Ngăn malware lan sang hệ thống khác.
- Ngăn attacker tiếp tục truy cập.
- Ngăn data exfiltration.
- Bảo vệ hệ thống quan trọng.
- Giữ lại bằng chứng để phân tích forensic.
- Tạo thời gian để xây dựng remediation strategy.

Containment không chỉ là “tắt máy bị nhiễm”. Đôi khi tắt máy quá sớm sẽ làm mất bằng chứng quan trọng trong RAM.

---

## 3. Vì sao containment phải được phối hợp đồng thời?

Containment cần được phối hợp và thực hiện trên tất cả hệ thống liên quan cùng lúc.

Nếu chỉ xử lý một phần hệ thống, attacker có thể nhận ra họ đang bị phát hiện.

Khi đó attacker có thể:

- Thay đổi C2.
- Xóa log.
- Tạo persistence mới.
- Di chuyển sang host khác.
- Tăng tốc exfiltration.
- Deploy ransomware sớm hơn.
- Đổi tool hoặc technique.

Ví dụ:

```text
Có 10 máy bị compromise
      ↓
Team chỉ isolate 3 máy
      ↓
Attacker thấy một phần access bị mất
      ↓
Attacker chuyển sang 7 máy còn lại
      ↓
Incident trở nên khó xử lý hơn
```

Vì vậy, containment cần được lên kế hoạch rõ ràng và thực hiện đồng bộ.

---

# 4. Short-term Containment

**Short-term Containment** là các hành động cô lập ngắn hạn, thường để giảm thiệt hại ngay lập tức nhưng vẫn cố gắng giữ nguyên trạng thái hệ thống càng nhiều càng tốt.

Mục tiêu:

- Chặn attacker nhanh.
- Giảm thiệt hại.
- Giữ hệ thống ít bị thay đổi.
- Tạo thời gian để chuẩn bị remediation dài hạn.
- Bảo toàn bằng chứng.

Ví dụ short-term containment:

- Đưa hệ thống vào VLAN cô lập.
- Rút dây mạng khỏi máy bị compromise.
- Chặn domain C2 bằng DNS.
- Trỏ C2 DNS name về hệ thống do ta kiểm soát.
- Trỏ C2 DNS name về domain không tồn tại.
- Chặn IP/domain độc hại tạm thời.
- Disable account bị compromise.
- Tạm ngắt VPN access của user đáng ngờ.

---

## 5. Backup substage trong containment

Trong short-term containment, vì hệ thống được giữ gần như nguyên trạng, đội điều tra có cơ hội thu thập forensic evidence.

Các hoạt động có thể gồm:

- Tạo forensic image.
- Thu thập memory dump.
- Export event logs.
- Thu thập network connections.
- Thu thập process list.
- Thu thập file đáng ngờ.
- Lưu lại registry artifact.
- Ghi nhận timestamp.
- Ghi chain of custody.

Đây đôi khi được gọi là **backup substage** của containment.

---

## 6. Khi containment cần shutdown hệ thống

Nếu short-term containment yêu cầu tắt hệ thống, cần cân nhắc kỹ.

Trước khi shutdown nên xác định:

- Hệ thống có business-critical không?
- RAM có chứa bằng chứng quan trọng không?
- Có cần memory dump trước không?
- Ai có quyền phê duyệt shutdown?
- Có ảnh hưởng đến production không?
- Business đã được thông báo chưa?

Không nên tự ý tắt server quan trọng nếu chưa có phê duyệt phù hợp.

---

# 7. Long-term Containment

**Long-term Containment** là các hành động cô lập dài hạn và bền vững hơn.

Mục tiêu:

- Giảm khả năng attacker quay lại.
- Áp dụng thay đổi bảo mật lâu dài.
- Hạn chế attack path.
- Chuẩn bị cho eradication và recovery.

Ví dụ long-term containment:

- Đổi mật khẩu user bị compromise.
- Đổi mật khẩu tài khoản đặc quyền.
- Áp dụng firewall rules.
- Cài hoặc bật Host Intrusion Detection System.
- Vá lỗ hổng hệ thống.
- Shutdown hệ thống nếu cần.
- Tăng logging.
- Chặn protocol không cần thiết.
- Thay đổi segmentation.
- Tạm disable service rủi ro.

---

## 8. Patching không có nghĩa incident đã kết thúc

Một điểm rất quan trọng:

```text
Hệ thống được patch không có nghĩa là incident đã kết thúc.
```

Patch chỉ giúp chặn một attack vector hoặc lỗ hổng.

Sau patch vẫn cần:

- Eradication.
- Recovery.
- Kiểm tra persistence.
- Kiểm tra credential compromise.
- Kiểm tra lateral movement.
- Kiểm tra hệ thống khác bị ảnh hưởng.
- Post-incident review.

Ví dụ:

```text
Web server bị khai thác lỗ hổng
      ↓
Team patch web server
      ↓
Nhưng attacker đã cài web shell
      ↓
Nếu không eradication, attacker vẫn có thể quay lại
```

---

# 9. Eradication là gì?

**Eradication** là giai đoạn loại bỏ nguyên nhân gốc và các dấu vết còn lại của incident.

Mục tiêu:

- Loại bỏ attacker khỏi hệ thống.
- Loại bỏ malware.
- Loại bỏ persistence.
- Loại bỏ backdoor.
- Vá root cause.
- Đảm bảo attack path không thể lặp lại.

Eradication bắt đầu sau khi incident đã được containment.

---

## 10. Các hoạt động trong Eradication

Một số hoạt động thường gặp:

- Remove malware.
- Remove web shell.
- Remove malicious service.
- Remove scheduled task độc hại.
- Remove persistence registry key.
- Rebuild hệ thống.
- Restore hệ thống từ backup sạch.
- Reset credential bị lộ.
- Remove unauthorized account.
- Patch lỗ hổng.
- Hardening hệ thống.
- Kiểm tra lại configuration.
- Kiểm tra toàn mạng xem còn artifact tương tự không.

---

## 11. Eradication có thể mở rộng ngoài hệ thống bị ảnh hưởng

Trong nhiều trường hợp, hoạt động eradication không chỉ áp dụng cho hệ thống bị compromise.

Có thể cần mở rộng ra toàn môi trường.

Ví dụ:

- Một server bị khai thác vì cấu hình SMB yếu.
- Sau điều tra phát hiện nhiều server khác cũng có cấu hình tương tự.
- Eradication nên áp dụng hardening cho toàn bộ nhóm server liên quan.

Nói cách khác:

```text
Không chỉ dọn máy bị nhiễm.
Phải sửa điều kiện khiến incident xảy ra.
```

---

# 12. Recovery là gì?

**Recovery** là giai đoạn đưa hệ thống trở lại hoạt động bình thường.

Mục tiêu:

- Khôi phục dịch vụ.
- Khôi phục dữ liệu.
- Đưa hệ thống trở lại production.
- Đảm bảo hệ thống sạch.
- Đảm bảo business operation hoạt động lại.
- Theo dõi chặt sau khôi phục.

---

## 13. Business verification trong Recovery

Trước khi đưa hệ thống về production, business hoặc owner cần xác nhận:

- Hệ thống hoạt động đúng chức năng.
- Dữ liệu cần thiết còn đầy đủ.
- Ứng dụng chạy ổn định.
- Người dùng có thể truy cập.
- Không còn lỗi nghiệp vụ.
- Không còn dấu hiệu compromise.

Security team xác nhận hệ thống sạch về mặt bảo mật, nhưng business owner cần xác nhận hệ thống dùng được về mặt vận hành.

---

## 14. Monitoring sau Recovery

Tất cả hệ thống được restore hoặc rebuild sau incident cần được giám sát chặt.

Lý do: hệ thống từng bị compromise thường có thể bị attacker nhắm lại nếu attacker lấy lại được access.

Các sự kiện đáng nghi nên monitor:

- Unusual logons.
- User hoặc service account chưa từng đăng nhập trước đó.
- Process lạ.
- Network connection bất thường.
- Registry changes ở vị trí thường bị malware dùng.
- Scheduled task mới.
- Service mới.
- C2 beaconing.
- File creation trong thư mục đáng ngờ.
- Authentication failure bất thường.
- Lateral movement attempt.

---

## 15. Recovery trong incident lớn có thể kéo dài nhiều tháng

Trong các incident lớn, recovery có thể được thực hiện theo nhiều phase.

### Early phases

Tập trung vào các quick wins:

- Chặn attack path rõ ràng.
- Reset credential quan trọng.
- Tăng logging.
- Tắt service rủi ro.
- Vá lỗi nghiêm trọng.
- Cô lập hệ thống quan trọng.
- Loại bỏ low-hanging fruit.

### Later phases

Tập trung vào thay đổi dài hạn:

- Thiết kế lại segmentation.
- Triển khai PAM.
- Hardening toàn môi trường.
- Nâng cấp EDR/SIEM.
- Cải thiện identity security.
- Cải thiện backup strategy.
- Cập nhật playbook.
- Đào tạo nhân sự.
- Đánh giá lại kiến trúc bảo mật.

---

# 16. Bảng so sánh Containment, Eradication và Recovery

| Giai đoạn | Mục tiêu | Ví dụ hành động |
|---|---|---|
| Containment | Ngăn incident lan rộng | Isolate host, block C2, disable account |
| Eradication | Loại bỏ root cause và dấu vết độc hại | Remove malware, patch, rebuild, remove persistence |
| Recovery | Đưa hệ thống trở lại hoạt động bình thường | Restore service, verify business, monitor closely |

---

# 17. Workflow tổng hợp

```text
Investigation complete
      ↓
Understand incident type and impact
      ↓
Plan containment strategy
      ↓
Execute short-term containment
      ↓
Preserve evidence
      ↓
Execute long-term containment
      ↓
Remove malware and persistence
      ↓
Patch root cause
      ↓
Rebuild or restore systems
      ↓
Business verification
      ↓
Return to production
      ↓
Heavy monitoring
```

---

# 18. Bảng ôn nhanh thuật ngữ

| Thuật ngữ | Nghĩa ngắn gọn |
|---|---|
| Containment | Cô lập sự cố để ngăn lan rộng |
| Short-term Containment | Cô lập ngắn hạn, ít thay đổi hệ thống |
| Long-term Containment | Cô lập dài hạn bằng thay đổi bền vững |
| Eradication | Loại bỏ nguyên nhân và dấu vết độc hại |
| Recovery | Khôi phục hệ thống về hoạt động bình thường |
| Forensic Image | Bản sao phục vụ điều tra số |
| Evidence Preservation | Bảo toàn bằng chứng |
| C2 | Command and Control |
| VLAN Isolation | Cô lập máy bằng VLAN riêng |
| HIDS | Host Intrusion Detection System |
| Patch | Bản vá lỗi |
| Persistence | Cơ chế giúp attacker quay lại |
| Business Verification | Xác nhận hệ thống hoạt động đúng nghiệp vụ |
| Production Environment | Môi trường vận hành thật |
| Heavy Monitoring | Giám sát chặt sau incident |
| Low-hanging Fruit | Điểm yếu dễ khai thác, dễ sửa |

---

# 19. Câu hỏi ôn tập

## 1. Containment dùng để làm gì?

Containment dùng để ngăn incident lan rộng và ngăn attacker tiếp tục gây thiệt hại.

## 2. Vì sao containment cần thực hiện đồng bộ?

Vì nếu chỉ xử lý một phần, attacker có thể nhận ra đang bị phát hiện và thay đổi kỹ thuật để tiếp tục tồn tại trong môi trường.

## 3. Short-term containment khác long-term containment như thế nào?

Short-term containment tập trung giảm thiệt hại nhanh và ít thay đổi hệ thống. Long-term containment tập trung vào các thay đổi bền vững như đổi mật khẩu, áp firewall rules, patch và hardening.

## 4. Vì sao patch xong chưa chắc incident đã kết thúc?

Vì attacker có thể đã cài persistence, web shell, backdoor hoặc đã đánh cắp credential trước khi hệ thống được patch.

## 5. Recovery cần làm gì?

Recovery đưa hệ thống trở lại hoạt động bình thường, xác nhận business hoạt động đúng và giám sát chặt hệ thống sau khi khôi phục.

---

# Key Takeaway

Giai đoạn **Containment, Eradication và Recovery** là lúc đội Incident Handling chuyển từ điều tra sang hành động.

Containment giúp ngăn thiệt hại lan rộng. Eradication giúp loại bỏ attacker, malware, persistence và root cause. Recovery giúp đưa hệ thống trở lại production một cách an toàn.

Điều quan trọng là không được xử lý nửa vời. Cần bảo toàn bằng chứng, phối hợp containment đồng bộ, loại bỏ root cause và giám sát kỹ sau khi hệ thống được khôi phục.

---
title: Incident Handling
date: 2026-06-01 20:00:00 +0700
categories:
  - SOC
  - Incident Response
tags:
  - incident-handling
  - soc
  - blue-team
  - dfir
description: Ghi chú quy trình xử lý sự cố cho SOC analyst mới học.
---

Incident handling là quy trình giúp SOC analyst chuyển từ cảnh báo rời rạc sang hành động có kiểm soát. Mục tiêu không chỉ là đóng alert, mà là hiểu chuyện gì đã xảy ra, phạm vi ảnh hưởng ở đâu và cần khắc phục như thế nào.

## Tóm tắt

| Phase | Mục tiêu | Output |
| --- | --- | --- |
| Identification | Xác định alert có đáng tin không | Initial triage notes |
| Containment | Giảm phạm vi ảnh hưởng | Isolated host/user/block rule |
| Recovery | Đưa hệ thống về trạng thái an toàn | Clean host và monitoring plan |

## 1. Preparation

- Chuẩn bị playbook cho các nhóm sự cố phổ biến như phishing, malware, brute force và data exfiltration.
- Đảm bảo log từ endpoint, firewall, proxy, DNS, VPN và cloud được thu thập đầy đủ.
- Kiểm tra quyền truy cập công cụ SIEM, EDR, ticketing và kênh liên lạc nội bộ.

## 2. Identification

Khi có alert, cần xác định nguồn phát hiện, độ tin cậy và tài sản liên quan. Một alert tốt nên trả lời được:

- Host hoặc user nào bị ảnh hưởng?
- Dấu hiệu ban đầu là gì?
- Có bằng chứng lặp lại trên nguồn log khác không?
- Đây là true positive, false positive hay cần theo dõi thêm?

## 3. Containment

Containment nên cân bằng giữa giảm thiểu rủi ro và tránh phá hỏng bằng chứng. Ví dụ:

- Cô lập endpoint qua EDR.
- Disable tài khoản nghi ngờ bị compromise.
- Chặn domain, IP hoặc hash độc hại.
- Thu thập memory hoặc triage artifact trước khi xóa file nếu cần điều tra DFIR.

## 4. Eradication And Recovery

Sau khi kiểm soát phạm vi, xử lý nguyên nhân gốc như malware persistence, credential theft, rule firewall sai hoặc cấu hình yếu. Recovery cần theo dõi lại telemetry để chắc chắn sự cố không tái diễn.

## 5. Lessons Learned

Mỗi incident nên kết thúc bằng ghi chú ngắn:

- Timeline sự kiện.
- Root cause.
- Tác động.
- Hành động đã thực hiện.
- Detection rule hoặc hardening cần bổ sung.

Một ticket tốt là tài liệu học tập cho lần xử lý tiếp theo.

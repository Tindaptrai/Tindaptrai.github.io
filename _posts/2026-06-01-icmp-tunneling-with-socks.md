---
layout: post
title: "ICMP Tunneling with SOCKS"
date: 2026-06-01 20:30:00 +0700
categories: [Network Security, HTB]
tags: [icmp, tunneling, socks, pivoting]
description: "Ghi chú lab về ICMP tunneling và SOCKS proxy trong môi trường hợp pháp."
toc: true
---

ICMP tunneling là kỹ thuật đóng gói traffic vào ICMP để đi qua môi trường bị giới hạn kết nối TCP/UDP. Trong lab CTF hoặc HTB, kỹ thuật này thường được dùng để học về pivoting, egress filtering và network detection.

Chỉ thực hành trong lab được phép. Không dùng kỹ thuật tunneling để vượt kiểm soát trên hệ thống không thuộc quyền kiểm tra.

## Tóm tắt

| Thành phần | Vai trò |
| --- | --- |
| ICMP | Kênh vận chuyển trong lab bị giới hạn TCP/UDP |
| SOCKS proxy | Cho phép công cụ đi qua tunnel |
| Detection | Theo dõi payload, tần suất và host bất thường |

## Khi nào cần nghĩ tới tunneling?

- Host nội bộ không truy cập trực tiếp được từ máy attacker.
- Firewall chỉ cho phép ICMP echo request/reply.
- Cần tạo đường đi tạm thời để proxy công cụ qua mạng trung gian.

## Luồng ý tưởng

1. Xác nhận ICMP được phép giữa hai host.
2. Chạy agent/server tunneling trong phạm vi lab.
3. Tạo SOCKS proxy local.
4. Cấu hình công cụ như browser, proxychains hoặc scanner đi qua SOCKS.
5. Ghi log traffic để hiểu dấu hiệu phát hiện.

## Detection Ideas

Blue team có thể theo dõi:

- ICMP payload lớn bất thường.
- Tần suất echo request/reply cao và đều.
- ICMP giữa các host không thường xuyên giao tiếp.
- Mẫu payload có entropy cao.
- Dòng traffic ICMP kéo dài trong thời gian dài.

## Ghi chú học tập

Khi làm HTB, tôi nên ghi lại sơ đồ mạng, command đã dùng trong lab, lỗi gặp phải và IOC sinh ra. Điều này giúp hiểu cả offensive workflow lẫn defensive detection.

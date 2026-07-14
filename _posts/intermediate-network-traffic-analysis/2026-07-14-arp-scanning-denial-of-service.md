---
layout: post
title: "ARP Scanning & Denial-of-Service"
date: 2026-07-14 22:46:43 +0700
categories: ["Intermediate Network Traffic Analysis"]
tags: [arp, arp-scanning, denial-of-service, arp-poisoning, host-discovery, wireshark, tshark, pcap, threat-hunting, incident-response, cdsa]
description: "Phân tích ARP scanning và ARP-based DoS: sequential probing, live-host discovery, duplicate gateway IP và quy trình containment."
toc: true
---

# ARP Scanning & Denial-of-Service

Bài này thuộc module **Intermediate Network Traffic Analysis**, tập trung vào việc phát hiện ARP scanning và ARP-based denial-of-service.

> Chỉ thực hành trong HTB, lab hoặc mạng có ủy quyền rõ ràng.

## Timeline cập nhật

```text
Created at: 2026-07-14 22:46:43 +0700
Timezone: Asia/Ho_Chi_Minh
Module: Intermediate Network Traffic Analysis
Section: ARP Scanning & Denial-of-Service
Focus: Sequential ARP scan, live-host discovery, subnet poisoning, containment
```

## 1. ARP Scanning là gì?

ARP scanning sử dụng broadcast ARP requests để tìm host đang hoạt động trong cùng broadcast domain.

Pattern thường gặp:

```text
Who has 192.168.10.1?
Who has 192.168.10.2?
Who has 192.168.10.3?
...
```

Host đang hoạt động sẽ trả ARP reply, giúp attacker nhanh chóng xây danh sách mục tiêu.

## 2. Dấu hiệu ARP Scanning

- Broadcast ARP requests tới các IP tuần tự.
- Requests tới nhiều IP không tồn tại.
- Volume ARP cao từ một source MAC.
- Một endpoint probe gần như toàn subnet.
- Active hosts phản hồi liên tục sau requests.

## 3. Bộ lọc Wireshark

Tất cả ARP:

```wireshark
arp
```

Chỉ requests:

```wireshark
arp.opcode == 1
```

Chỉ replies:

```wireshark
arp.opcode == 2
```

Theo source MAC:

```wireshark
eth.src == aa:bb:cc:dd:ee:ff && arp.opcode == 1
```

## 4. Phân biệt scan hợp lệ và đáng ngờ

Có thể hợp lệ:

- Asset discovery được phê duyệt.
- Network scanner của SOC.
- Monitoring ở mức thấp.

Đáng ngờ:

```text
Một user endpoint quét toàn /24 trong vài giây.
Source MAC không thuộc scanner được quản lý.
Hoạt động xảy ra ngoài maintenance window.
Sau scan xuất hiện poisoning hoặc connection attempts.
```

## 5. Từ reconnaissance sang DoS/MITM

Sau khi tìm live hosts, attacker có thể:

```text
1. Gửi ARP reply giả tới router.
2. Tuyên bố nhiều victim IP cùng trỏ về attacker MAC.
3. Gửi reply giả tới client.
4. Tuyên bố gateway IP trỏ về attacker MAC.
5. Làm hỏng ARP cache ở cả hai phía.
```

Nếu attacker không forward traffic, kết quả là **DoS**. Nếu có forwarding, attacker có thể trở thành **MITM**.

## 6. Dấu hiệu ARP-based DoS

- Một MAC tuyên bố sở hữu nhiều IP sống.
- Gateway IP xuất hiện với MAC mới.
- Duplicate IP cho địa chỉ gateway.
- Nhiều client mất kết nối cùng lúc.
- ARP replies tăng đột biến.
- TCP sessions timeout hàng loạt.

Ví dụ gateway bị giả mạo:

```text
192.168.10.1 is at 08:00:27:53:0c:ba
```

trong khi MAC thật của router khác hoàn toàn.

## 7. Lọc duplicate address

```wireshark
arp.duplicate-address-detected
```

Chỉ duplicate replies:

```wireshark
arp.duplicate-address-detected && arp.opcode == 2
```

## 8. Tshark Hunt

Đếm source MAC gửi ARP requests:

```bash
tshark -r ARP_Scan.pcapng   -Y "arp.opcode == 1"   -T fields -e eth.src | sort | uniq -c | sort -nr
```

Liệt kê source MAC và target IP:

```bash
tshark -r ARP_Scan.pcapng   -Y "arp.opcode == 1"   -T fields -e eth.src -e arp.dst.proto_ipv4
```

## 9. Detection Engineering

Logic gợi ý:

```text
Một MAC gửi ARP requests tới hơn N IP duy nhất trong T giây.
```

Threshold lab tham khảo:

```text
> 50 target IP duy nhất trong 10 giây
```

Cần baseline theo VLAN để tránh false positive từ scanner hợp lệ.

## 10. Containment

```text
[ ] Xác định source MAC và switch port.
[ ] Disable port hoặc chuyển host sang quarantine VLAN.
[ ] Block MAC nếu phù hợp.
[ ] Clear ARP cache trên gateway/victim sau containment.
[ ] Thu PCAP và endpoint evidence.
[ ] Kiểm tra khả năng data interception.
```

## 11. Hardening

- Dynamic ARP Inspection.
- DHCP Snooping.
- Port Security.
- 802.1X/NAC.
- VLAN segmentation.
- Switch rate limiting phù hợp.
- SPAN/TAP hoặc NDR để theo dõi ARP fan-out.

## 12. Mapping với CDSA

**Security Operations & Monitoring:** baseline ARP rate và unique target count theo VLAN.

**Incident Response & Forensics:** xác định transition từ discovery sang poisoning và isolate source ở switch layer.

**Threat Hunting:** hunt sequential probes, gateway duplicate IP và one-MAC-to-many-IP mappings.

## Key Takeaway

```text
ARP scanning tạo requests tuần tự tới toàn subnet.
Live hosts trả lời và giúp attacker xây target list.
Khi một MAC bắt đầu tuyên bố sở hữu nhiều IP hoặc gateway IP,
hoạt động đã chuyển từ reconnaissance sang DoS/MITM.
```

---
layout: post
title: "ARP Spoofing & Abnormality Detection"
date: 2026-07-14 22:46:16 +0700
categories: ["Intermediate Network Traffic Analysis", "Layer 2 & Network Attacks"]
tags: [arp, arp-spoofing, arp-poisoning, mitm, denial-of-service, wireshark, tcpdump, packet-analysis, threat-hunting, incident-response, cdsa]
description: "Phân tích ARP Spoofing và ARP Poisoning bằng Wireshark/tcpdump: duplicate IP, IP-to-MAC mismatch, TCP anomalies và quy trình phản ứng."
toc: true
---

# ARP Spoofing & Abnormality Detection

Bài này thuộc module **Intermediate Network Traffic Analysis**, tập trung vào cách phát hiện ARP Spoofing/Poisoning trong PCAP.

> Chỉ thực hành trong HTB, lab hoặc mạng có ủy quyền rõ ràng.

## Timeline cập nhật

```text
Created at: 2026-07-14 22:46:16 +0700
Timezone: Asia/Ho_Chi_Minh
Module: Intermediate Network Traffic Analysis
Section: ARP Spoofing & Abnormality Detection
Focus: ARP poisoning, duplicate IP, MITM, Wireshark, tcpdump
```

## 1. ARP hoạt động như thế nào?

ARP ánh xạ địa chỉ IPv4 sang địa chỉ MAC trong cùng broadcast domain:

```text
Host A kiểm tra ARP cache
→ nếu chưa có mapping, broadcast "Who has x.x.x.x?"
→ Host B trả lời "x.x.x.x is at aa:bb:cc:dd:ee:ff"
→ Host A cập nhật ARP cache
```

ARP không có xác thực mạnh, vì vậy attacker có thể gửi ARP reply giả.

## 2. ARP Poisoning và MITM

Trong một cuộc tấn công điển hình:

```text
Victim tin gateway IP → attacker MAC
Gateway tin victim IP → attacker MAC
```

Nếu attacker bật IP forwarding, lưu lượng trở thành:

```text
Victim ↔ Attacker ↔ Gateway
```

Nếu attacker không forward, kết quả thường là DoS.

## 3. Thu thập PCAP

```bash
sudo apt install tcpdump -y
sudo tcpdump -i eth0 -w arp-spoof.pcapng
wireshark arp-spoof.pcapng
```

## 4. Bộ lọc Wireshark

Tất cả ARP:

```wireshark
arp
```

ARP request và reply:

```wireshark
arp.opcode == 1
arp.opcode == 2
```

Duplicate IP:

```wireshark
arp.duplicate-address-detected
```

Duplicate IP trong reply:

```wireshark
arp.duplicate-address-detected && arp.opcode == 2
```

## 5. Dấu hiệu bất thường

- Một IP được ánh xạ tới hai MAC khác nhau.
- Một MAC mới tuyên bố sở hữu IP gateway hoặc victim.
- Nhiều unsolicited ARP replies.
- Một host phát ARP với volume bất thường.
- Wireshark báo `duplicate use detected`.
- TCP retransmission, reset hoặc timeout xuất hiện sau ARP anomaly.

## 6. Xác định MAC đáng ngờ

Ví dụ MAC nghi vấn:

```text
08:00:27:53:0c:ba
```

Lọc toàn bộ ARP liên quan:

```wireshark
(arp.opcode) && (
  (eth.src == 08:00:27:53:0c:ba) ||
  (eth.dst == 08:00:27:53:0c:ba)
)
```

So sánh hai MAC:

```wireshark
eth.addr == 50:eb:f6:ec:0e:7f ||
eth.addr == 08:00:27:53:0c:ba
```

Mục tiêu là xác định MAC attacker ban đầu dùng IP nào, sau đó bắt đầu tuyên bố sở hữu IP nào.

## 7. Xác minh trên endpoint

```bash
arp -a
ip neigh
arp -a | grep -i "08:00:27:53:0c:ba"
```

Nếu cùng một IP xuất hiện với nhiều MAC, cần điều tra ngay.

## 8. Phân biệt DoS và MITM

**DoS:**

- Victim mất kết nối.
- TCP retransmission nhiều.
- Session timeout.
- Attacker không forward traffic.

**MITM:**

- Traffic vẫn hoạt động.
- Có luồng gần đối xứng Victim → Attacker và Attacker → Gateway.
- Có thể xuất hiện DNS spoofing hoặc SSL stripping ở lớp trên.

## 9. Response

```text
[ ] Xác định switch port chứa MAC nghi vấn.
[ ] Kiểm tra DHCP lease, CAM table và ARP table.
[ ] Isolate host hoặc đưa vào quarantine VLAN.
[ ] Thu PCAP và triage endpoint.
[ ] Kiểm tra DNS spoofing và credential exposure.
[ ] Clear ARP cache sau containment nếu cần.
```

## 10. Hardening

- Dynamic ARP Inspection.
- DHCP Snooping.
- Port Security.
- 802.1X/NAC.
- VLAN segmentation.
- Static ARP cho hệ thống đặc biệt quan trọng.
- SPAN/TAP, NDR hoặc IDS để giám sát lớp liên kết.

## 11. Mapping với CDSA

**Security Operations & Monitoring:** theo dõi duplicate IP/MAC, ARP rate và switch telemetry.

**Incident Response & Forensics:** thu PCAP, xác định MAC/switch port và kiểm tra dữ liệu bị chặn bắt.

**Threat Hunting:** hunt one-IP-to-many-MAC, one-MAC-to-many-IP và ARP reply burst.

## Key Takeaway

```text
ARP Spoofing thường lộ qua duplicate IP, IP-to-MAC mismatch và ARP reply bất thường.
Nếu traffic bị ngắt, nghiêng về DoS; nếu tiếp tục qua attacker, nghiêng về MITM.
Wireshark phải được correlate với ARP cache, DHCP và switch telemetry.
```

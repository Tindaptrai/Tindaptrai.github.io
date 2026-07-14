---
layout: post
title: "802.11 Denial-of-Service"
date: 2026-07-14 22:47:08 +0700
categories: ["Intermediate Network Traffic Analysis"]
tags: [wifi, 80211, deauthentication, denial-of-service, monitor-mode, airodump-ng, wireshark, wids, wips, threat-hunting, incident-response, cdsa]
description: "Phân tích 802.11 deauthentication DoS: monitor mode, capture Wi-Fi frames, Wireshark filters, reason-code rotation và biện pháp 802.11w/WPA3."
toc: true
---

# 802.11 Denial-of-Service

Bài này thuộc module **Intermediate Network Traffic Analysis**, tập trung vào việc phát hiện deauthentication/disassociation attacks trong mạng Wi-Fi.

> Chỉ thực hành trong HTB, lab hoặc mạng Wi-Fi có ủy quyền rõ ràng.

## Timeline cập nhật

```text
Created at: 2026-07-14 22:47:08 +0700
Timezone: Asia/Ho_Chi_Minh
Module: Intermediate Network Traffic Analysis
Section: 802.11 Denial-of-Service
Focus: Monitor mode, deauthentication, reason codes, WIDS/WIPS
```

## 1. Vì sao cần Monitor Mode?

Managed mode không hiển thị đầy đủ raw 802.11 management frames. Monitor mode cho phép bắt:

- Beacon.
- Probe.
- Authentication.
- Association.
- Deauthentication.
- Disassociation.
- Control frames.

Kiểm tra interface:

```bash
iwconfig
```

## 2. Bật Monitor Mode

Dùng airmon-ng:

```bash
sudo airmon-ng start wlan0
```

Hoặc thủ công:

```bash
sudo ifconfig wlan0 down
sudo iwconfig wlan0 mode monitor
sudo ifconfig wlan0 up
```

Xác minh:

```bash
iwconfig
```

Cần thấy:

```text
Mode:Monitor
```

Tên interface có thể là `wlan0mon` hoặc vẫn là `wlan0`.

## 3. Capture bằng airodump-ng

```bash
sudo airodump-ng   -c 4   --bssid F8:14:FE:4D:E6:F1   wlan0mon   -w raw
```

| Tham số | Ý nghĩa |
|---|---|
| `-c` | Channel |
| `--bssid` | MAC của AP |
| `-w` | Output prefix |
| `wlan0mon` | Monitor interface |

## 4. Deauthentication Attack

Attacker giả mạo deauthentication frame như thể nó đến từ AP hợp lệ.

Mục tiêu:

- Bắt WPA handshake để crack offline.
- Gây DoS.
- Ép client rời mạng.
- Dẫn dụ client sang rogue AP.

Client khó phân biệt frame giả nếu không có Protected Management Frames.

## 5. Bộ lọc theo BSSID

```wireshark
wlan.bssid == F8:14:FE:4D:E6:F1
```

Chỉ management deauthentication frames:

```wireshark
(wlan.bssid == F8:14:FE:4D:E6:F1) &&
(wlan.fc.type == 0) &&
(wlan.fc.type_subtype == 12)
```

## 6. Reason Code 7

Nhiều công cụ cơ bản thường dùng:

```text
Reason code 7
```

Ý nghĩa thường gặp:

```text
Class 3 frame received from nonassociated STA
```

Filter:

```wireshark
(wlan.bssid == F8:14:FE:4D:E6:F1) &&
(wlan.fc.type == 0) &&
(wlan.fc.type_subtype == 12) &&
(wlan.fixed.reason_code == 7)
```

Một lượng lớn frame trong thời gian ngắn là dấu hiệu deauth flood.

## 7. Reason-Code Rotation

Attacker tinh vi có thể xoay reason code để né rule chỉ theo code 7:

```wireshark
wlan.fixed.reason_code == 1
wlan.fixed.reason_code == 2
wlan.fixed.reason_code == 3
```

Detection nên dựa trên:

```text
Deauth frame rate
Unique destination clients
Source/BSSID consistency
Sequence-number anomalies
Reason-code diversity
```

## 8. Authentication và Association bất thường

Lọc association/authentication:

```wireshark
(wlan.bssid == F8:14:FE:4D:E6:F1) &&
(
  (wlan.fc.type_subtype == 0) ||
  (wlan.fc.type_subtype == 1) ||
  (wlan.fc.type_subtype == 11)
)
```

| Subtype | Ý nghĩa |
|---:|---|
| 0 | Association Request |
| 1 | Association Response |
| 11 | Authentication |

Quá nhiều requests từ một client có thể là brute-force, misconfiguration hoặc thiết bị giả mạo.

## 9. Dấu hiệu tấn công

- Deauth/disassociation frames tăng đột biến.
- Một source nhắm nhiều clients.
- Một client nhận hàng trăm deauth frames.
- Reason codes thay đổi nhanh.
- BSSID bị spoof.
- Client disconnect/reconnect liên tục.
- WPA handshake xuất hiện lặp lại.
- Association/authentication failures tăng cao.

## 10. Hardening

- Bật IEEE 802.11w / Protected Management Frames.
- Ưu tiên WPA3-SAE.
- Bật WIDS/WIPS.
- Tune rule theo frame rate, không chỉ reason code.
- Theo dõi rogue AP.
- Phân tách guest và employee networks.
- Dùng 802.1X cho enterprise Wi-Fi.

## 11. Response

```text
[ ] Xác định BSSID, channel và client bị tác động.
[ ] Thu raw 802.11 capture.
[ ] Dùng WIDS/WIPS triangulation nếu có.
[ ] Tìm thiết bị phát sóng bất thường.
[ ] Kiểm tra rogue AP và handshake capture.
[ ] Bật PMF/WPA3 nếu tương thích.
[ ] Chuyển channel tạm thời khi cần.
```

## 12. Mapping với CDSA

**Security Operations & Monitoring:** monitor deauth/disassoc rate theo AP/channel và correlate WLC với WIDS/WIPS.

**Incident Response & Forensics:** thu raw capture, xác định thời điểm disconnect/reconnect và kiểm tra rogue AP.

**Threat Hunting:** hunt reason-code rotation, one-source-to-many-client deauth và repeated WPA handshakes.

## Key Takeaway

```text
802.11 deauthentication là link-layer DoS và thường là bước đầu để bắt WPA handshake.
Monitor mode là bắt buộc để quan sát raw management frames.
Detection không nên chỉ dựa vào reason code 7 vì attacker có thể xoay reason codes.
IEEE 802.11w, WPA3-SAE và WIDS/WIPS là các biện pháp phòng thủ chính.
```

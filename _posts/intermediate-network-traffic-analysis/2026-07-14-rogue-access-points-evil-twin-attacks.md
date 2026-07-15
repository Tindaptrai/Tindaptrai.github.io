---
layout: post
title: "Rogue Access Points & Evil-Twin Attacks"
date: 2026-07-14 22:54:57 +0700
categories: ["Intermediate Network Traffic Analysis", "Layer 2 & Network Attacks"]
tags: [wifi, rogue-access-point, evil-twin, "80211", airodump-ng, wireshark, beacon-frame, rsn, hostile-portal, wireless-security, threat-hunting, incident-response, cdsa]
description: "Phân tích Rogue Access Point và Evil-Twin: phát hiện bằng Airodump-ng, Beacon/RSN comparison, xác định client bị ảnh hưởng và quy trình ứng phó."
toc: true
---

# Rogue Access Points & Evil-Twin Attacks

Bài này thuộc module **Intermediate Network Traffic Analysis**, tập trung vào việc phân biệt **Rogue Access Point** với **Evil-Twin**, phát hiện AP giả bằng Airodump-ng/Wireshark và xác định client đã kết nối nhầm.

```text
Mục tiêu: nhận diện cùng ESSID nhưng khác BSSID/security profile,
so sánh Beacon/RSN information và xác định client bị ảnh hưởng.
```

> Chỉ thực hành trong HTB, lab hoặc mạng Wi-Fi có ủy quyền rõ ràng. Không dựng Evil-Twin hoặc captive portal giả trên mạng ngoài phạm vi cho phép.

---

## Timeline cập nhật

```text
Created at: 2026-07-14 22:54:57 +0700
Timezone: Asia/Ho_Chi_Minh
Module: Intermediate Network Traffic Analysis
Section: Rogue Access Points & Evil-Twin Attacks
Focus: Rogue AP, Evil-Twin, Beacon/RSN analysis, client attribution, incident response
```

---

# 1. Rogue Access Point là gì?

**Rogue Access Point** là AP không được tổ chức cho phép nhưng được kết nối trực tiếp vào mạng nội bộ.

Mục tiêu thường là:

- Bỏ qua network segmentation.
- Bypass perimeter controls.
- Cung cấp truy cập trái phép vào LAN.
- Tạo một bridge từ Wi-Fi vào mạng nội bộ.
- Vô tình hoặc cố ý mở đường vào air-gapped/isolated segment.

```text
Điểm quan trọng: Rogue AP thường có kết nối trực tiếp với mạng doanh nghiệp.
```

Ví dụ:

```text
Internet/Wi-Fi client
        ↓
Rogue AP
        ↓
Corporate LAN
```

---

# 2. Evil-Twin là gì?

**Evil-Twin** là AP giả mạo được attacker dựng với ESSID giống AP hợp lệ.

Khác Rogue AP, Evil-Twin thường:

- Không nối trực tiếp vào mạng nội bộ.
- Hoạt động độc lập.
- Dùng captive portal giả.
- Thu thập Wi-Fi password hoặc domain credential.
- Làm MITM cho client đã kết nối.
- Kết hợp deauthentication để ép client rời AP thật.

```text
Điểm quan trọng: Evil-Twin giả danh AP hợp lệ để lừa client.
```

---

# 3. Rogue AP và Evil-Twin khác nhau thế nào?

| Đặc điểm | Rogue AP | Evil-Twin |
|---|---|---|
| Kết nối trực tiếp LAN | Thường có | Không bắt buộc |
| Giả ESSID hợp lệ | Có thể | Thường có |
| Mục tiêu chính | Bypass perimeter | Lừa client, credential theft |
| Captive portal giả | Không bắt buộc | Thường gặp |
| Deauthentication hỗ trợ | Không bắt buộc | Thường dùng |

---

# 4. Dấu hiệu Evil-Twin

Các dấu hiệu đáng chú ý:

- Cùng ESSID nhưng nhiều BSSID không được quản lý.
- Một AP dùng WPA2, AP còn lại là Open.
- Cipher/authentication khác AP thật.
- Channel, vendor information hoặc supported rates khác.
- AP mới có tín hiệu rất mạnh.
- Deauthentication frames xuất hiện gần thời điểm AP giả hoạt động.
- Client bắt đầu kết nối với BSSID lạ.

---

# 5. Phát hiện bằng Airodump-ng

Ví dụ lọc theo ESSID:

```bash
sudo airodump-ng   -c 4   --essid HTB-Wireless   wlan0mon   -w raw
```

Output có thể hiển thị:

```text
BSSID              ENC   CIPHER AUTH ESSID
F8:14:FE:4D:E6:F1  WPA2  CCMP   PSK  HTB-Wireless
F8:14:FE:4D:E6:F2  OPN               HTB-Wireless
```

Phân tích:

```text
Cùng ESSID
Khác BSSID
Một AP WPA2
Một AP Open
```

Đây là dấu hiệu Evil-Twin rất rõ.

---

# 6. Vì sao attacker dùng Open AP?

Open Evil-Twin giúp client:

- Kết nối dễ hơn.
- Không cần biết WPA2 password.
- Bị chuyển tới captive portal giả.
- Gửi credential qua HTTP/plaintext.
- Trở thành mục tiêu MITM.

Attacker có thể dựng portal giống:

- Microsoft 365.
- VPN login.
- Wi-Fi reauthentication.
- Domain login.
- SSO portal nội bộ.

---

# 7. Phân tích Beacon Frames

Bộ lọc Beacon:

```wireshark
(wlan.fc.type == 0) &&
(wlan.fc.type_subtype == 8)
```

Hoặc theo ESSID:

```wireshark
wlan.fc.type_subtype == 8 &&
wlan.ssid == "HTB-Wireless"
```

Beacon frame chứa:

- SSID.
- BSSID.
- Channel.
- Supported rates.
- Capability information.
- RSN information.
- Vendor-specific information.
- Country information.
- Beacon interval.

---

# 8. So sánh RSN Information

**RSN - Robust Security Network** mô tả security capabilities của AP.

AP hợp lệ có thể quảng bá:

```text
WPA2
CCMP/AES
TKIP compatibility
PSK authentication
```

Evil-Twin Open thường:

```text
Không có RSN information
```

Đây là dấu hiệu mạnh khi hai AP cùng ESSID nhưng security profile khác nhau.

---

# 9. Evil-Twin tinh vi hơn

Attacker có thể cấu hình cùng:

- WPA/WPA2 mode.
- Cipher.
- Channel.
- Supported rates.
- Beacon interval.

Khi đó cần kiểm tra sâu hơn:

- Vendor-specific information.
- OUI của BSSID.
- Country code.
- HT/VHT/HE capabilities.
- Timestamp/sequence behavior.
- Transmit power.
- WPS fields.
- RSN capability flags.
- PMF support.
- Channel width.

```text
Không nên chỉ dựa vào ESSID để xác định AP hợp lệ.
```

---

# 10. Lọc riêng BSSID nghi vấn

Ví dụ Evil-Twin:

```text
F8:14:FE:4D:E6:F2
```

Filter:

```wireshark
wlan.bssid == F8:14:FE:4D:E6:F2
```

Chỉ Beacon của AP nghi vấn:

```wireshark
wlan.bssid == F8:14:FE:4D:E6:F2 &&
wlan.fc.type_subtype == 8
```

---

# 11. Tìm client đã kết nối nhầm

Khi lọc theo BSSID nghi vấn, cần tìm:

- Authentication frames.
- Association requests/responses.
- DHCP.
- ARP.
- DNS.
- HTTP.
- Client MAC address.
- Hostname nếu có.

Ví dụ:

```wireshark
wlan.bssid == F8:14:FE:4D:E6:F2
```

Nếu thấy ARP từ client:

```text
Who has 169.254.63.254?
```

hoặc DHCP request, client có khả năng đã kết nối vào AP giả.

---

# 12. Xác định client bị ảnh hưởng

Cần ghi lại:

```text
Client MAC address
Hostname
IP address
Time connected
BSSID connected
DNS/HTTP destinations
Credentials possibly submitted
```

Hostname có thể lấy từ:

- DHCP option 12.
- NBNS.
- mDNS.
- LLMNR.
- SMB traffic.
- EDR/asset inventory correlation.

---

# 13. Deauthentication liên quan Evil-Twin

Attacker thường ép client rời AP thật bằng deauthentication frames.

Filter:

```wireshark
wlan.fc.type_subtype == 12
```

Theo AP hợp lệ:

```wireshark
wlan.bssid == F8:14:FE:4D:E6:F1 &&
wlan.fc.type_subtype == 12
```

Attack chain:

```text
Deauth client khỏi AP thật
        ↓
Client scan lại Wi-Fi
        ↓
Kết nối Evil-Twin cùng ESSID
        ↓
Captive portal/MITM/credential theft
```

---

# 14. Phát hiện Rogue AP

Rogue AP thường phát hiện qua:

- Network device inventory.
- Switch MAC/CAM table.
- LLDP/CDP.
- DHCP server.
- NAC/802.1X.
- Wireless controller.
- WIDS/WIPS.
- Strong unknown SSID/BSSID gần văn phòng.
- AP Open nằm gần corporate wireless environment.

Nếu Rogue AP dùng Windows hotspot hoặc travel router, MAC OUI có thể cho biết vendor.

---

# 15. Network-Side Investigation

Các bước:

```text
1. Xác định BSSID/MAC của AP nghi vấn.
2. Tra OUI/vendor.
3. Kiểm tra switch CAM table.
4. Map MAC tới switch port.
5. Kiểm tra DHCP lease.
6. Xác định user/device owner.
7. Isolate port hoặc quarantine VLAN.
8. Thu PCAP và endpoint evidence.
```

---

# 16. Airodump-ng Detection Checklist

```text
[ ] Cùng ESSID có bao nhiêu BSSID?
[ ] Encryption có giống nhau không?
[ ] Cipher/Auth có giống nhau không?
[ ] Channel có hợp lý không?
[ ] Vendor/OUI có khớp inventory không?
[ ] Signal strength có bất thường không?
[ ] Có deauth activity không?
[ ] Client nào đang associate với BSSID lạ?
```

---

# 17. Wireshark Filters tổng hợp

Beacon frames:

```wireshark
wlan.fc.type_subtype == 8
```

Theo ESSID:

```wireshark
wlan.ssid == "HTB-Wireless"
```

Theo Evil-Twin BSSID:

```wireshark
wlan.bssid == F8:14:FE:4D:E6:F2
```

Deauthentication:

```wireshark
wlan.fc.type_subtype == 12
```

Association/Authentication:

```wireshark
wlan.fc.type_subtype == 0 ||
wlan.fc.type_subtype == 1 ||
wlan.fc.type_subtype == 11
```

ARP của client trên BSSID nghi vấn:

```wireshark
wlan.bssid == F8:14:FE:4D:E6:F2 && arp
```

---

# 18. Detection Engineering

Detection logic nên kết hợp:

```text
Same ESSID
+ Unknown BSSID
+ Security profile mismatch
+ Client association
+ Deauthentication activity
```

High-confidence example:

```text
Corporate ESSID appears as WPA2 and Open simultaneously.
Client disconnects from approved BSSID.
Client associates to unapproved Open BSSID.
HTTP/DNS traffic begins through unapproved BSSID.
```

---

# 19. WIDS/WIPS Use Cases

Wireless IDS/IPS có thể:

- Baseline approved APs.
- Detect unknown BSSID.
- Detect Evil-Twin.
- Detect deauth flood.
- Locate rogue AP by triangulation.
- Block/quarantine unauthorized AP.
- Alert client association to rogue BSSID.

---

# 20. Response

```text
[ ] Xác định AP là Rogue AP hay Evil-Twin.
[ ] Xác định client đã kết nối.
[ ] Isolate affected clients.
[ ] Reset credential nếu có khả năng bị nhập vào portal giả.
[ ] Revoke sessions/tokens.
[ ] Tìm switch port nếu AP nối trực tiếp LAN.
[ ] Thu wireless PCAP.
[ ] Block BSSID qua WIPS nếu có.
[ ] Điều tra endpoint dựng AP.
[ ] Kiểm tra credential theft và lateral movement.
```

---

# 21. Hardening

- WPA2-Enterprise/WPA3-Enterprise.
- 802.1X với certificate authentication.
- Protected Management Frames.
- WIDS/WIPS.
- NAC cho wired ports.
- Disable unauthorized hotspot/Internet Connection Sharing.
- Certificate validation cho enterprise Wi-Fi.
- MDM Wi-Fi profiles.
- User awareness về Open AP và captive portal giả.
- Approved AP/BSSID inventory.

---

# 22. Mapping với CDSA

## Security Operations & Monitoring

- Baseline approved ESSID/BSSID.
- Monitor deauth và unknown AP.
- Correlate WLC/WIDS/NAC logs.

## Incident Response & Forensics

- Xác định client bị ảnh hưởng.
- Thu PCAP và portal artifacts.
- Reset credentials, revoke sessions.
- Tìm Rogue AP qua switch telemetry.

## Threat Hunting

- Hunt same-ESSID security mismatch.
- Hunt client association với BSSID chưa được phê duyệt.
- Hunt deauth → reassociation chain.
- Hunt Open AP gần corporate ESSID.

## Detection Engineering

- WIDS rule cho unknown BSSID.
- Correlation theo ESSID/BSSID/RSN.
- SIEM enrichment với approved AP inventory.
- High-confidence alert khi corporate ESSID xuất hiện ở Open mode.

---

# 23. Lab Setup

Môi trường tối thiểu:

```text
1 AP hợp lệ
1 AP lab mô phỏng Evil-Twin
1 wireless adapter hỗ trợ monitor mode
Wireshark
Aircrack-ng suite
WIDS/WIPS hoặc Splunk/ELK nhận wireless logs
```

Workflow:

```text
1. Bắt Beacon frames.
2. So sánh BSSID và RSN.
3. Xác định AP giả.
4. Quan sát deauth/association.
5. Xác định client bị ảnh hưởng.
6. Viết detection và IR timeline.
```

---

# Key Takeaway

```text
Rogue AP thường nối trực tiếp vào mạng nội bộ để bypass perimeter controls.
Evil-Twin giả ESSID hợp lệ để lừa client và thu credential.
Cùng ESSID nhưng khác BSSID hoặc khác RSN/security profile là dấu hiệu quan trọng.
Beacon analysis, client association và deauthentication phải được correlate.
WIDS/WIPS, NAC, 802.1X và PMF là các biện pháp phòng thủ chính.
```

Rogue Access Points & Evil-Twin Attacks là chủ đề điển hình của CDSA vì cần kết hợp wireless packet analysis, asset inventory, incident response và behavior-based detection.

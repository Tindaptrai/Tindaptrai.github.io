---
layout: post
title: "LLMNR/NBT-NS Poisoning from Linux"
date: 2026-07-06 23:34:58 +0700
categories: ["Active Directory Enumeration & Attacks", "Credential Attacks"]
tags: [active-directory, ad-enumeration-attacks, llmnr, nbtns, mdns, responder, netntlmv2, hashcat, credential-capture, poisoning, mitm, blue-team, red-team]
description: "Tóm tắt ngắn bài LLMNR/NBT-NS Poisoning from Linux: cơ chế LLMNR/NBT-NS, Responder, bắt NetNTLMv2 hash, crack bằng Hashcat và các lưu ý phòng thủ trong môi trường AD."
toc: true
---

# LLMNR/NBT-NS Poisoning from Linux

Bài này thuộc module **Active Directory Enumeration & Attacks**, tập trung vào kỹ thuật **LLMNR/NBT-NS poisoning** từ máy Linux để thu thập NetNTLMv2 hash trong môi trường AD được ủy quyền kiểm thử.

```text
Mục tiêu: bắt hash xác thực qua LLMNR/NBT-NS/MDNS, crack offline và dùng credential hợp lệ để bắt đầu credentialed enumeration.
```

---

## Timeline cập nhật

```text
Created at: 2026-07-06 23:34:58 +0700
Timezone: Asia/Ho_Chi_Minh
Module: Active Directory Enumeration & Attacks
Topic: LLMNR/NBT-NS Poisoning from Linux
Focus: Responder, LLMNR, NBT-NS, MDNS, NetNTLMv2, Hashcat, initial foothold
```

---

# 1. Bối cảnh

Sau giai đoạn initial enumeration, ta đã có một số thông tin cơ bản như:

- Users.
- Groups.
- Hosts.
- Domain Controller.
- Naming scheme.
- Một số service/role quan trọng trong domain.

Ở giai đoạn tiếp theo, mục tiêu là lấy **valid cleartext credentials** để có foothold trong domain.

Hai hướng thường đi song song:

```text
Network poisoning
Password spraying
```

Phần này tập trung vào **network poisoning** bằng Responder.

---

# 2. LLMNR và NBT-NS là gì?

## LLMNR

**LLMNR** là viết tắt của:

```text
Link-Local Multicast Name Resolution
```

LLMNR được Windows dùng khi DNS không resolve được hostname.

Đặc điểm:

```text
UDP 5355
Hoạt động trên cùng local link
Dựa trên format giống DNS
```

## NBT-NS

**NBT-NS** là viết tắt của:

```text
NetBIOS Name Service
```

NBT-NS dùng để resolve NetBIOS name trong local network.

Đặc điểm:

```text
UDP 137
Dùng NetBIOS name
Được dùng khi LLMNR/DNS không đáp ứng được
```

---

# 3. Vì sao LLMNR/NBT-NS nguy hiểm?

Khi DNS không resolve được host, Windows có thể hỏi broadcast/multicast:

```text
Có ai biết host này ở đâu không?
```

Vấn đề là:

```text
Bất kỳ host nào trong local network cũng có thể trả lời.
```

Attacker chạy Responder có thể giả vờ là host được hỏi và khiến victim gửi authentication request đến máy attacker.

Kết quả có thể bắt được:

```text
NetNTLMv1 hash
NetNTLMv2 hash
Cleartext credential trong một số trường hợp/protocol
```

---

# 4. Attack Flow tổng quan

Ví dụ flow đơn giản:

```text
1. User gõ sai \print01 thành \printer01.
2. DNS trả lời không biết host này.
3. Máy victim broadcast hỏi local network.
4. Responder trả lời: "Tôi là printer01".
5. Victim tin phản hồi và gửi authentication request.
6. Attacker bắt được NetNTLMv2 hash.
7. Hash được crack offline bằng Hashcat/John.
```

---

# 5. Công cụ thường dùng

Một số công cụ có thể dùng cho poisoning/spoofing:

| Tool | Mô tả |
|---|---|
| Responder | Tool phổ biến để poison LLMNR, NBT-NS, MDNS và bắt hash |
| Inveigh | Cross-platform MITM/spoofing tool, có bản C# và PowerShell legacy |
| Metasploit | Có module hỗ trợ spoofing/poisoning |

Trong Linux attack host, **Responder** là lựa chọn rất phổ biến.

---

# 6. Responder Analyze Mode

Trước khi poison thật, có thể chạy chế độ phân tích:

```bash
sudo responder -I ens224 -A
```

Ý nghĩa:

```text
-A = analyze mode
Chỉ nghe request, không trả lời/poison
```

Analyze mode hữu ích để quan sát môi trường mà chưa tạo poisoned response.

---

# 7. Chạy Responder để Poison

Chạy Responder với interface phù hợp:

```bash
sudo responder -I ens224
```

Nếu chưa biết interface:

```bash
ip a
```

Responder sẽ lắng nghe và trả lời các request liên quan:

- LLMNR.
- NBT-NS.
- MDNS.
- SMB.
- HTTP.
- WPAD nếu bật.
- Một số protocol khác tùy cấu hình.

---

# 8. Một số option Responder quan trọng

| Option | Ý nghĩa |
|---|---|
| `-I` | Chọn network interface |
| `-A` | Analyze mode, chỉ nghe không poison |
| `-w` | Bật WPAD rogue proxy |
| `-f` | Fingerprint remote host |
| `-v` | Verbose output |
| `-F` | Force WPAD authentication |
| `-P` | Force proxy authentication |

Gợi ý:

```text
-w, -F, -P có thể gây login prompt hoặc ảnh hưởng môi trường.
Chỉ dùng khi được phép và hiểu rõ tác động.
```

---

# 9. Port Responder thường dùng

Responder cần một số port để hoạt động tốt, ví dụ:

```text
UDP 137
UDP 138
UDP 53
UDP 5355
UDP 5353
TCP 80
TCP 135
TCP 139
TCP 445
TCP 21
TCP 25
TCP 110
TCP 587
TCP 3128
TCP 3141
TCP 1433
UDP/TCP 389
UDP 1434
```

Nếu dịch vụ rogue server không cần thiết, có thể tắt trong:

```text
Responder.conf
```

---

# 10. Responder Logs

Khi bắt được hash, Responder sẽ in ra console và lưu log.

Thư mục log thường gặp:

```bash
/usr/share/responder/logs
```

Ví dụ file log:

```text
SMB-NTLMv2-SSP-172.16.5.25.txt
HTTP-NTLMv2-172.16.5.200.txt
Proxy-Auth-NTLMv2-172.16.5.200.txt
```

Tìm hash đã bắt:

```bash
ls /usr/share/responder/logs
grep -Ri "NTLMv2" /usr/share/responder/logs
```

---

# 11. Format NetNTLMv2 Hash

Hash thường có dạng:

```text
USERNAME::DOMAIN:challenge:response:blob
```

Ví dụ format:

```text
FOREND::INLANEFREIGHT:4af70a79938ddf8a:0f85ad1e80baa52d732719dbf62c34cc:010100...
```

Khi đưa vào file để crack:

```text
Mỗi hash nằm trên 1 dòng.
Không kèm prefix log như [SMB] NTLMv2-SSP Hash :
Không xuống dòng giữa hash.
```

---

# 12. Crack NetNTLMv2 bằng Hashcat

NetNTLMv2 dùng Hashcat mode:

```text
5600
```

Lệnh crack:

```bash
hashcat -m 5600 hash.txt /usr/share/wordlists/rockyou.txt
```

Xem kết quả đã crack:

```bash
hashcat -m 5600 hash.txt --show
```

Kết quả thường có dạng:

```text
USER::DOMAIN:challenge:response:blob:cleartext_password
```

Phần cuối cùng sau dấu `:` là password cleartext.

---

# 13. NetNTLMv2 dùng được Pass-the-Hash không?

Điểm quan trọng:

```text
NetNTLMv2 hash thường không dùng trực tiếp cho pass-the-hash.
```

Thông thường phải:

```text
Crack offline để lấy cleartext password
hoặc
Relay authentication nếu điều kiện phù hợp
```

SMB Relay cần điều kiện như SMB signing không bắt buộc và sẽ được học sâu hơn ở lateral movement.

---

# 14. Quy trình thực hành trong lab

Workflow an toàn trong lab:

```text
1. Xác định interface.
2. Chạy Responder.
3. Chờ LLMNR/NBT-NS/MDNS request.
4. Bắt NetNTLMv2 hash.
5. Lưu hash vào hash.txt.
6. Crack bằng Hashcat mode 5600.
7. Submit cleartext password nếu bài lab yêu cầu.
```

Command mẫu:

```bash
ip a
sudo responder -I ens224

grep -Ri "NTLMv2" /usr/share/responder/logs

hashcat -m 5600 hash.txt /usr/share/wordlists/rockyou.txt
hashcat -m 5600 hash.txt --show
```

---

# 15. Blue Team Detection

Blue Team có thể phát hiện các dấu hiệu:

- LLMNR/NBT-NS traffic bất thường.
- Nhiều host hỏi tên không tồn tại.
- Máy lạ trả lời LLMNR/NBT-NS.
- SMB authentication đến host không phải server hợp lệ.
- NetNTLM authentication bất thường.
- WPAD request bất thường.
- Responder-like behavior trong subnet.

Log/telemetry nên quan sát:

- DNS logs.
- Windows event logs.
- Network IDS/IPS.
- Zeek/Suricata.
- SMB authentication logs.
- DHCP/NAC asset inventory.
- EDR network events.

---

# 16. Hardening và Mitigation

Các biện pháp phòng thủ:

```text
Disable LLMNR nếu không cần.
Disable NBT-NS nếu không cần.
Disable WPAD nếu không dùng.
Bật SMB signing nơi phù hợp.
Dùng strong password/passphrase.
Triển khai MFA cho remote access.
Giới hạn local admin.
Phân đoạn mạng.
Monitor broadcast name resolution.
```

GPO thường dùng để tắt LLMNR:

```text
Computer Configuration
→ Administrative Templates
→ Network
→ DNS Client
→ Turn off multicast name resolution
```

NBT-NS thường cấu hình qua adapter/WINS hoặc DHCP option tùy môi trường.

---

# 17. Red Team Notes

Trong pentest hợp pháp, Responder thường được dùng để:

- Lấy foothold ban đầu.
- Bắt hash từ user/machine.
- Crack password offline.
- Xác định password reuse.
- Chuẩn bị credentialed enumeration.

Nhưng cần tuân thủ:

```text
Scope
Rules of Engagement
Thời gian được phép test
Không gây gián đoạn dịch vụ
Không relay hoặc ép auth nếu chưa được phép
```

---

# Key Takeaway

```text
LLMNR/NBT-NS là cơ chế fallback khi DNS resolve thất bại.
Bất kỳ host nào trong local network cũng có thể trả lời request này.
Responder lợi dụng điểm đó để poison request và bắt NetNTLMv2 hash.
NetNTLMv2 thường cần crack offline bằng Hashcat mode 5600.
Hash bắt được được lưu trong /usr/share/responder/logs.
Mitigation chính là tắt LLMNR/NBT-NS/WPAD không cần thiết, bật SMB signing và monitor name-resolution traffic.
```

LLMNR/NBT-NS poisoning là kỹ thuật rất phổ biến để lấy foothold trong AD lab và pentest nội bộ, nhưng chỉ được thực hiện trong môi trường có ủy quyền rõ ràng.

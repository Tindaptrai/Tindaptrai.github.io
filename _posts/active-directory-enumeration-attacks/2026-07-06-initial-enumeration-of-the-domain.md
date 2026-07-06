---
layout: post
title: "Initial Enumeration of the Domain"
date: 2026-07-06 23:36:56 +0700
categories: ["Active Directory Enumeration & Attacks"]
tags: [active-directory, ad-enumeration-attacks, initial-enumeration, internal-pentest, wireshark, tcpdump, responder, fping, nmap, kerbrute, kerberos, domain-controller, blue-team, red-team]
description: "Tóm tắt bài Initial Enumeration of the Domain: thiết lập internal pentest, passive/active host discovery, Wireshark, tcpdump, Responder analyze mode, fping, Nmap, Kerbrute user enumeration và các lưu ý scope khi kiểm thử AD."
toc: true
---

# Initial Enumeration of the Domain

Bài này thuộc module **Active Directory Enumeration & Attacks**, nói về giai đoạn đầu khi bắt đầu pentest nội bộ vào domain **INLANEFREIGHT.LOCAL**.

```text
Mục tiêu: xác định host, dịch vụ quan trọng, Domain Controller, user hợp lệ và các hướng có thể tạo foothold ban đầu.
```

---

## Timeline cập nhật

```text
Created at: 2026-07-06 23:36:56 +0700
Timezone: Asia/Ho_Chi_Minh
Module: Active Directory Enumeration & Attacks
Topic: Initial Enumeration of the Domain
Focus: Internal network discovery, passive/active enumeration, host discovery, Kerbrute, Nmap, Responder analyze mode
```

---

# 1. Bối cảnh pentest

Ta đang ở giai đoạn rất đầu của một bài **AD-focused internal penetration test**.

Thông tin ban đầu:

- Khách hàng: Inlanefreight.
- Network range được cung cấp: `172.16.5.0/23`.
- Không có internal network map chi tiết.
- Bắt đầu từ góc nhìn unauthenticated.
- Có Linux attack host nằm trong mạng nội bộ.
- Có Windows host có thể dùng để load tools nếu cần.
- Kiểu test: grey box.
- Phong cách test: non-evasive.

```text
Grey box = có một phần thông tin như range mạng.
Non-evasive = không cố tình né detection, nhưng vẫn phải tránh gây gián đoạn.
```

---

# 2. Các kiểu setup thường gặp khi internal pentest

Khách hàng có thể cung cấp nhiều kiểu môi trường test khác nhau:

- Linux pentest VM bên trong hạ tầng nội bộ.
- Physical dropbox cắm vào Ethernet và callback về VPN.
- Tester có mặt tại văn phòng, cắm laptop vào mạng.
- Linux VM trong Azure/AWS có route vào internal network.
- VPN access vào internal network.
- Corporate laptop kết nối VPN.
- Managed Windows workstation tại văn phòng.
- VDI qua Citrix hoặc nền tảng tương tự.

Lưu ý:

```text
VPN đôi khi hạn chế một số attack như LLMNR/NBT-NS poisoning.
```

---

# 3. Nhiệm vụ chính

Trong section này, các task chính gồm:

```text
1. Enumerate internal network.
2. Xác định hosts.
3. Xác định critical services.
4. Tìm potential foothold.
5. Kết hợp passive và active enumeration.
6. Document mọi finding quan trọng.
```

Các kết quả scan/log/tool output nên được lưu lại để dùng sau.

---

# 4. Key Data Points cần tìm

| Data Point | Ý nghĩa |
|---|---|
| AD Users | User hợp lệ để phục vụ password spraying hoặc credentialed enumeration |
| AD Joined Computers | DC, file server, SQL server, web server, Exchange, database server |
| Key Services | Kerberos, NetBIOS, LDAP, DNS, SMB |
| Vulnerable Hosts/Services | Host/service có thể tạo foothold nhanh |

```text
Mục tiêu sớm nhất: tìm được domain user credential hoặc SYSTEM access trên domain-joined host.
```

---

# 5. Tư duy enumeration

AD enumeration rất dễ bị loạn nếu không có kế hoạch.

Quy trình nên đi từng lớp:

```text
1. Passive identification.
2. Active validation.
3. Service enumeration.
4. User enumeration.
5. Potential vulnerability review.
6. Regroup và phân tích dữ liệu.
```

Điều quan trọng là xây dựng methodology có thể lặp lại.

---

# 6. Passive Host Discovery bằng Wireshark

Ở bước đầu, có thể “nghe mạng” bằng Wireshark:

```bash
sudo -E wireshark
```

Quan sát traffic như:

- ARP requests/replies.
- MDNS.
- Layer 2 broadcast.
- Hostnames trong local segment.

Ví dụ ARP/MDNS có thể phát hiện host:

```text
172.16.5.5
172.16.5.25
172.16.5.50
172.16.5.100
172.16.5.125
ACADEMY-EA-WEB01
```

```text
Passive capture giúp có host đầu tiên mà chưa gửi scan chủ động.
```

---

# 7. Passive Capture bằng tcpdump

Nếu attack host không có GUI, dùng tcpdump:

```bash
sudo tcpdump -i ens224
```

Nên lưu PCAP để phân tích lại:

```bash
sudo tcpdump -i ens224 -w initial-capture.pcap
```

Sau đó có thể mở file bằng Wireshark ở máy khác.

Lợi ích của việc lưu PCAP:

- Review lại sau.
- Tìm hostname hoặc protocol bị bỏ sót.
- Làm bằng chứng trong report.
- Ghi lại thời điểm phát hiện.

---

# 8. Responder Analyze Mode

Responder có thể dùng ở chế độ passive/analyze để quan sát LLMNR/NBT-NS/MDNS mà không poison.

Lệnh:

```bash
sudo responder -I ens224 -A
```

Ý nghĩa:

```text
-A = Analyze mode
Chỉ lắng nghe request, không gửi poisoned response.
```

Responder analyze mode giúp phát hiện thêm host hoặc tên máy chưa thấy trong Wireshark.

---

# 9. Active Host Discovery bằng fping

Sau passive checks, dùng fping để xác thực host đang sống trong subnet.

Lệnh mẫu:

```bash
fping -asgq 172.16.5.0/23
```

Ý nghĩa flags:

| Flag | Ý nghĩa |
|---|---|
| `-a` | Chỉ show alive targets |
| `-s` | In stats cuối scan |
| `-g` | Generate target list từ CIDR |
| `-q` | Quiet mode, không spam từng kết quả |

Ví dụ output có thể thấy 9 live hosts:

```text
172.16.5.5
172.16.5.25
172.16.5.50
172.16.5.100
172.16.5.125
172.16.5.200
172.16.5.225
172.16.5.238
172.16.5.240
```

---

# 10. Chuẩn bị hosts.txt

Sau khi có kết quả từ Wireshark, Responder và fping, gom IP vào `hosts.txt`.

Ví dụ:

```text
172.16.5.5
172.16.5.25
172.16.5.50
172.16.5.100
172.16.5.125
172.16.5.200
172.16.5.225
172.16.5.238
172.16.5.240
```

Nên deduplicate IP để tránh scan trùng.

---

# 11. Nmap Scanning

Dùng Nmap để xác định service, role và host quan trọng.

Lệnh mẫu:

```bash
sudo nmap -v -A -iL hosts.txt -oN /home/htb-student/Documents/host-enum
```

Nên dùng `-oA` để lưu nhiều format:

```bash
sudo nmap -v -A -iL hosts.txt -oA /home/htb-student/Documents/host-enum
```

```text
-oA giúp lưu normal, grepable và XML output để dùng lại với tool khác.
```

---

# 12. Nhận diện Domain Controller

Một host có thể là Domain Controller nếu thấy các port/service như:

```text
53/tcp   DNS
88/tcp   Kerberos
135/tcp  MSRPC
139/tcp  NetBIOS
389/tcp  LDAP
445/tcp  SMB
464/tcp  kpasswd
593/tcp  RPC over HTTP
636/tcp  LDAPS
3268/tcp Global Catalog LDAP
3269/tcp Global Catalog LDAPS
3389/tcp RDP
```

Ví dụ host:

```text
ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
172.16.5.5
```

Dữ liệu Nmap có thể tiết lộ:

- NetBIOS domain name.
- DNS domain name.
- Computer name.
- Product version.
- Certificate subject.
- LDAP domain.
- AD site name.

---

# 13. Nhận diện legacy/vulnerable host

Một host chạy OS hoặc service cũ là điểm cần chú ý.

Ví dụ:

```text
Windows Server 2008 R2
Microsoft IIS 7.5
Microsoft SQL Server 2008 R2 RTM
SMB signing enabled but not required
```

Điều này có thể gợi ý:

- Legacy OS.
- EOL software.
- Missing patches.
- Potential old exploits.
- SMB relay opportunity nếu signing không bắt buộc.

Nhưng cần lưu ý:

```text
Không khai thác legacy host nếu chưa có approval rõ ràng.
Một số exploit có thể làm treo service hoặc sập host.
```

---

# 14. Lưu ý khi dùng Nmap

Một số Nmap scripts có thể active và gây ảnh hưởng hệ thống.

Cần thận trọng với:

- Vulnerability scripts.
- Aggressive scanning trên thiết bị yếu.
- Scan trên ICS/OT/sensors/logic controllers.
- Scan toàn mạng quá rộng.
- Rate quá cao.

```text
Hiểu scan mình chạy trước khi chạy trong môi trường khách hàng.
```

---

# 15. Identifying Users

Nếu khách hàng không cấp user ban đầu, cần tìm cách có foothold:

- Cleartext credential.
- NTLM/NetNTLM hash.
- SYSTEM shell trên domain-joined host.
- Shell dưới context domain user.
- Valid domain user qua Kerberos enumeration.

Một valid low-privilege user vẫn rất giá trị vì có thể mở ra credentialed enumeration.

---

# 16. Kerbrute Internal AD Username Enumeration

**Kerbrute** dùng Kerberos pre-authentication để enumerate valid AD users.

Ưu điểm:

```text
Nhanh.
Hữu ích từ unauthenticated perspective.
Có thể ít noisy hơn một số cách khác.
```

Cảnh báo:

```text
Failed Kerberos Pre-Auth vẫn có thể tính là failed login và có rủi ro lockout.
```

Lệnh mẫu:

```bash
kerbrute userenum -d INLANEFREIGHT.LOCAL --dc 172.16.5.5 jsmith.txt -o valid_ad_users
```

Kết quả có thể trả về:

```text
[+] VALID USERNAME: jjones@INLANEFREIGHT.LOCAL
[+] VALID USERNAME: sbrown@INLANEFREIGHT.LOCAL
[+] VALID USERNAME: tjohnson@INLANEFREIGHT.LOCAL
```

---

# 17. Kerbrute setup nhanh

Có thể clone và build:

```bash
sudo git clone https://github.com/ropnop/kerbrute.git
cd kerbrute
make all
```

Binary output thường nằm trong:

```text
dist/
```

Di chuyển binary vào PATH:

```bash
sudo mv kerbrute_linux_amd64 /usr/local/bin/kerbrute
```

Kiểm tra:

```bash
kerbrute
```

---

# 18. SYSTEM access trên domain-joined host

`NT AUTHORITY\SYSTEM` là quyền rất cao trên Windows host.

Nếu host đã join domain, SYSTEM context có thể enumerate AD bằng computer account.

```text
SYSTEM trên domain-joined host gần tương đương một domain user để enumeration.
```

Cách có thể đạt SYSTEM:

- Remote exploit như MS08-067, EternalBlue, BlueKeep.
- Abuse service chạy SYSTEM.
- Abuse SeImpersonatePrivilege trên hệ cũ.
- Local privilege escalation.
- Có local admin rồi dùng PsExec để mở SYSTEM shell.

---

# 19. Làm gì được với SYSTEM trong domain?

Từ SYSTEM trên domain-joined host có thể:

- Enumerate domain bằng built-in tools.
- Chạy BloodHound/SharpHound.
- Dùng PowerView.
- Kerberoasting.
- ASREPRoasting.
- Chạy Inveigh để bắt NetNTLMv2.
- SMB relay nếu điều kiện phù hợp.
- Token impersonation.
- ACL attacks.

---

# 20. Word of Caution

Phải luôn bám scope và style của engagement.

Nếu là non-evasive pentest:

```text
Có thể noisy hơn, nhưng vẫn phải tránh downtime.
```

Nếu là evasive/red team:

```text
Phải kiểm soát stealth, tool noise và detection threshold.
```

Không nên mặc định ném Nmap toàn mạng nếu chưa hiểu môi trường.

---

# 21. Suggested Workflow

Workflow gọn cho section này:

```text
1. Đọc scope: 172.16.5.0/23.
2. Passive capture bằng Wireshark/tcpdump.
3. Responder analyze mode để nghe LLMNR/NBT-NS/MDNS.
4. fping để tìm live hosts.
5. Gom IP vào hosts.txt.
6. Nmap scan để xác định services/roles.
7. Nhận diện DC, web, SQL, file server.
8. Chạy Kerbrute để tìm valid users.
9. Lưu output đầy đủ.
10. Dừng lại phân tích foothold path.
```

---

# 22. Blue Team Checklist

Blue Team nên monitor:

- ICMP sweep bất thường.
- Nmap service scan.
- Kerberos pre-auth failures tăng đột biến.
- LLMNR/NBT-NS/MDNS requests bất thường.
- Host lạ trong subnet.
- SMB/RDP/LDAP scanning.
- Nhiều kết nối đến DC port 88/389/445.
- Responder/Inveigh-like behavior.
- Tool execution trên internal hosts.

---

# 23. Red Team Notes

Red Team/Pentest nên ghi chú:

- Scope rõ ràng.
- Tất cả live hosts.
- Hostnames.
- Ports/services.
- Domain Controller.
- Domain name.
- AD site.
- Legacy hosts.
- SMB signing status.
- Valid usernames.
- Potential foothold vectors.
- Evidence/screenshot/output paths.

```text
Document tốt giúp giai đoạn sau nhanh hơn rất nhiều.
```

---

# Key Takeaway

```text
Initial enumeration bắt đầu bằng passive listening rồi active validation.
Wireshark/tcpdump giúp thấy ARP, MDNS và traffic nền.
Responder -A giúp nghe LLMNR/NBT-NS/MDNS không poison.
fping giúp xác định live hosts nhanh trong CIDR.
Nmap giúp nhận diện services, DC, legacy hosts và potential attack surface.
Kerbrute giúp enumerate valid AD usernames qua Kerberos.
SYSTEM trên domain-joined host có thể mở ra domain enumeration như một user.
```

Giai đoạn này không phải để “đánh ngay”, mà là để xây bản đồ ban đầu, tìm foothold khả thi và chuẩn bị dữ liệu cho các bước AD enumeration/attacks tiếp theo.

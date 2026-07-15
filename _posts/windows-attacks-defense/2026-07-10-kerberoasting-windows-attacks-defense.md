---
layout: post
title: "Kerberoasting"
date: 2026-07-10 22:48:13 +0700
categories: ["Windows Attacks & Defense", "Windows Internals & Authentication"]
tags: [active-directory, windows-attacks-defense, kerberos, kerberoasting, tgs, spn, rubeus, hashcat, john, event-4769, rc4, aes, honeypot, sigma, splunk, elk, threat-hunting, cdsa]
description: "Tóm tắt Kerberoasting: SPN, TGS, Rubeus, cracking TGS hash, prevention bằng gMSA/strong password và detection bằng Event ID 4769."
toc: true
---

# Kerberoasting

Bài này thuộc module **Windows Attacks & Defense**, tập trung vào kỹ thuật **Kerberoasting** trong Active Directory và cách phòng thủ/phát hiện theo góc nhìn SOC/CDSA.

```text
Mục tiêu: hiểu vì sao service account có SPN có thể bị request TGS, vì sao ticket có thể bị crack offline, và cách detect bằng Windows Event ID 4769.
```

> Chỉ thực hành trong môi trường lab/HTB hoặc hệ thống có ủy quyền rõ ràng.

---

## Timeline cập nhật

```text
Created at: 2026-07-10 22:48:13 +0700
Timezone: Asia/Ho_Chi_Minh
Module: Windows Attacks & Defense
Topic: Kerberoasting
Focus: SPN, TGS-REP, Rubeus, Hashcat mode 13100, John, Event ID 4769, RC4, honeypot detection
```

---

# 1. Kerberoasting là gì?

**Kerberoasting** là kỹ thuật post-exploitation trong AD. Attacker dùng domain user hợp lệ để request **TGS/service ticket** cho account có **SPN**. Ticket này được mã hóa bằng secret/hash của service account, nên có thể đem crack offline.

```text
Domain user hợp lệ -> tìm SPN -> request TGS -> lưu hash -> crack offline
```

---

# 2. SPN và TGS

**SPN - Service Principal Name** là định danh duy nhất của một service instance, ví dụ:

```text
HTTP/web01.eagle.local
MSSQLSvc/sql01.eagle.local:1433
CIFS/fileserver.eagle.local
```

Khi client truy cập service, Kerberos cấp TGS cho SPN đó. Nếu account service dùng password yếu, TGS có thể bị crack.

---

# 3. Encryption Type

| EType | Nhận xét |
|---|---|
| AES | Mạnh hơn, crack chậm hơn |
| RC4 | Dựa trên NTLM hash, crack nhanh hơn AES |
| DES | Legacy, nên bị disable |

Trong lab, RC4 thường xuất hiện với:

```text
Ticket Encryption Type: 0x17
```

---

# 4. Rubeus Kerberoast

Lệnh request Kerberoastable tickets:

```powershell
.\Rubeus.exe kerberoast /outfile:spn.txt
```

Nếu không chỉ định user, Rubeus có thể lấy ticket cho mọi account có SPN.

![Rubeus Kerberoast output](/assets/img/windows-attacks-defense/kerberoasting/rubeus-kerberoast-output.png)

---

# 5. Crack bằng Hashcat

RC4 TGS-REP dùng Hashcat mode:

```text
13100 = Kerberos 5, etype 23, TGS-REP
```

Lệnh:

```bash
hashcat -m 13100 -a 0 spn.txt passwords.txt --outfile="cracked.txt"
```

Xem kết quả:

```bash
cat cracked.txt
hashcat -m 13100 spn.txt passwords.txt --show
```

Ví dụ password crack được trong lab:

```text
Slavi123
```

---

# 6. Crack bằng John

```bash
sudo john spn.txt --fork=4 --format=krb5tgs --wordlist=passwords.txt --pot=results.pot
```

---

# 7. Prevention

Biện pháp phòng thủ:

- Cleanup SPN không còn dùng.
- Giảm số account có SPN không cần thiết.
- Dùng password service account dài, random, 100+ ký tự nếu app hỗ trợ.
- Dùng **gMSA** cho service hỗ trợ.
- Bật AES và hạn chế RC4 nếu có thể.
- Không cấp quyền cao cho service account nếu không cần.
- Review service account định kỳ.

---

# 8. Detection bằng Event ID 4769

Kerberos service ticket request tạo log:

```text
Event ID 4769 - A Kerberos service ticket was requested
```

Field quan trọng:

| Field | Ý nghĩa |
|---|---|
| Account Name | User request ticket |
| Service Name | SPN/service được request |
| Client Address | IP/host nguồn |
| Ticket Encryption Type | RC4/AES/DES |
| Failure Code | Trạng thái request |

![Event 4769 field breakdown](/assets/img/windows-attacks-defense/kerberoasting/event-4769-field-breakdown.png)

---

# 9. Detection Logic

## RC4 TGS request

Splunk:

```spl
index=wineventlog sourcetype=WinEventLog:Security EventCode=4769 Ticket_Encryption_Type=0x17
| stats count values(Service_Name) as services by Account_Name, Client_Address
| sort - count
```

ELK/KQL:

```kql
event.code: "4769" and winlog.event_data.TicketEncryptionType: "0x17"
```

Sigma:

```yaml
title: Kerberos TGS Request Using RC4
logsource:
  product: windows
  service: security
detection:
  selection:
    EventID: 4769
    TicketEncryptionType: '0x17'
  condition: selection
level: medium
```

## Volume anomaly

Alert khi một requester hoặc source IP request nhiều TGS trong thời gian ngắn:

```spl
index=wineventlog sourcetype=WinEventLog:Security EventCode=4769
| bin _time span=1m
| stats count dc(Service_Name) as unique_services values(Service_Name) as services by _time, Account_Name, Client_Address
| where count > 10 OR unique_services > 10
```

![Event 4769 volume by source IP](/assets/img/windows-attacks-defense/kerberoasting/event-4769-volume-by-source-ip.png)

## Honeypot SPN account

Tạo service account giả có SPN hợp lệ nhưng không dùng thật. Mọi request TGS tới account này đều đáng alert.

![Event 4769 honeypot triggered](/assets/img/windows-attacks-defense/kerberoasting/event-4769-honeypot-triggered.png)

---

# 10. Mapping với CDSA

## Security Operations & Monitoring

Monitor Event ID 4769, RC4 usage, TGS request volume, request tới honeypot SPN và requester/source host bất thường.

## Incident Response & Forensics

Khi có alert, cần xác định requester, source host, service account bị request, tool execution như Rubeus, file artifact như `spn.txt`, và dấu hiệu lateral movement sau đó.

## Threat Hunting

Hunt theo spike 4769, RC4 usage, rare SPN, user thường request nhiều service ticket, hoặc request tới service account cũ/quyền cao.

---

# 11. IR Checklist

```text
[ ] Xác định Account Name requester.
[ ] Xác định Client Address/source host.
[ ] Kiểm tra process execution trên source host.
[ ] Tìm Rubeus/PowerShell/cmd/unusual binary execution.
[ ] Tìm artifact: spn.txt, cracked.txt, hash output.
[ ] Xác định service account bị request.
[ ] Rotate password service account nếu nghi compromise.
[ ] Review SPN và quyền của service accounts.
[ ] Kiểm tra lateral movement sau thời điểm request.
```

---

# Key Takeaway

```text
Kerberoasting lợi dụng việc domain user có thể request TGS cho account có SPN.
TGS được mã hóa bằng secret của service account nên có thể bị crack offline.
RC4 dễ crack hơn AES, nên RC4 TGS request là tín hiệu đáng chú ý.
Hashcat mode 13100 dùng cho Kerberos 5 TGS-REP etype 23.
Phòng thủ tốt nhất là gMSA, password service account rất dài, cleanup SPN và hạn chế RC4.
Detection trọng tâm là Event ID 4769 kết hợp RC4, volume anomaly, rare SPN và honeypot SPN.
```

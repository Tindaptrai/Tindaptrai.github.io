---
layout: post
title: "AS-REProasting"
date: 2026-07-12 20:59:00 +0700
categories: ["Windows Attacks & Defense"]
tags: [active-directory, windows-attacks-defense, kerberos, as-reproasting, rubeus, hashcat, event-4768, rc4, preauthentication, honeypot, splunk, elk, sigma, threat-hunting, cdsa]
description: "Tóm tắt AS-REProasting: Kerberos pre-authentication, Rubeus, Hashcat mode 18200, Event ID 4768, prevention và SOC detection."
toc: true
---

# AS-REProasting

Bài này thuộc module **Windows Attacks & Defense**, tập trung vào kỹ thuật **AS-REProasting** và cách phòng thủ/phát hiện theo góc nhìn SOC/CDSA.

> Chỉ thực hành trong HTB, lab hoặc hệ thống có ủy quyền rõ ràng.

## Timeline cập nhật

```text
Created at: 2026-07-12 20:59:00 +0700
Timezone: Asia/Ho_Chi_Minh
Module: Windows Attacks & Defense
Section: AS-REProasting
Focus: Kerberos pre-authentication, Rubeus, Hashcat 18200, Event ID 4768
```

## 1. Cơ chế

AS-REProasting nhắm vào user có thuộc tính:

```text
Do not require Kerberos preauthentication
```

Luồng:

```text
1. Tìm user có DONT_REQ_PREAUTH.
2. Gửi AS-REQ không kèm pre-authentication.
3. KDC trả AS-REP.
4. Lưu AS-REP hash.
5. Crack offline.
```

Khác biệt chính:

```text
AS-REProasting: nhắm user không cần pre-auth.
Kerberoasting: nhắm account có SPN và TGS.
```

## 2. Thu thập bằng Rubeus

```powershell
.\Rubeus.exe asreproast /outfile:asrep.txt
```

Ví dụ lab:

```text
SamAccountName: anni
AS-REQ w/o preauth successful
Hash written to asrep.txt
```

## 3. Chuẩn hóa hash

Hashcat cần format:

```text
$krb5asrep$23$anni@eagle.local:...
```

Nếu output thiếu `23$`, thêm ngay sau `$krb5asrep$`.

## 4. Crack bằng Hashcat

```bash
sudo hashcat -m 18200 -a 0 asrep.txt passwords.txt --outfile asrepcrack.txt
```

Xem kết quả:

```bash
hashcat -m 18200 asrep.txt passwords.txt --show
```

Trong lab, password được crack là:

```text
Slavi123
```

## 5. Prevention

- Không bật `Do not require Kerberos preauthentication` nếu không cần.
- Review account có flag này hàng quý.
- Áp password tối thiểu 20 ký tự cho account bắt buộc dùng.
- Hạn chế privilege.
- Ưu tiên AES và giảm RC4.
- Monitor thay đổi `userAccountControl`.

## 6. Detection bằng Event ID 4768

```text
Event ID 4768 - A Kerberos authentication ticket (TGT) was requested
```

Field cần xem:

- Account Name.
- Client Address.
- Ticket Encryption Type.
- Pre-Authentication Type.
- Result Code.

![Event 4768 AS-REP request](/assets/img/windows-attacks-defense/as-reproasting/event-4768-asrep-request.png)

Dấu hiệu đáng chú ý:

```text
Pre-Authentication Type: 0
Ticket Encryption Type: 0x17
```

`0x17` là RC4-HMAC.

## 7. Splunk

```spl
index=wineventlog sourcetype=WinEventLog:Security EventCode=4768
Pre_Authentication_Type=0
| table _time host Account_Name Client_Address Ticket_Encryption_Type Result_Code
```

RC4 + no pre-auth:

```spl
index=wineventlog sourcetype=WinEventLog:Security EventCode=4768
Pre_Authentication_Type=0 Ticket_Encryption_Type=0x17
| stats count values(Client_Address) as sources by Account_Name
```

## 8. ELK/KQL

```kql
event.code: "4768" and winlog.event_data.PreAuthType: "0"
```

```kql
event.code: "4768"
and winlog.event_data.PreAuthType: "0"
and winlog.event_data.TicketEncryptionType: "0x17"
```

## 9. Sigma

```yaml
title: Kerberos AS-REP Request Without Preauthentication
status: experimental
logsource:
  product: windows
  service: security
detection:
  selection:
    EventID: 4768
    PreAuthType: '0'
  condition: selection
falsepositives:
  - Legacy accounts intentionally configured without preauthentication
level: high
```

## 10. Honeypot

Một honeypot account có pre-authentication disabled nhưng không dùng thật có thể tạo high-confidence alert.

```text
Account Name: svc-iam
Client Address: 172.16.18.25
Pre-Authentication Type: 0
```

![Event 4768 honeypot triggered](/assets/img/windows-attacks-defense/as-reproasting/event-4768-honeypot-triggered.png)

Honeypot nên:

- Trông đủ cũ và hợp lý.
- Có password cực mạnh.
- Không dùng production.
- Có alert riêng cho mọi hoạt động.
- Không phải account duy nhất có pre-auth disabled.

## 11. IR Checklist

```text
[ ] Xác định account và source host.
[ ] Xác nhận Event 4768, PreAuthType 0.
[ ] Kiểm tra RC4 0x17.
[ ] Tìm Rubeus, PowerShell, Event 4688, Sysmon Event 1.
[ ] Tìm asrep.txt/asrepcrack.txt.
[ ] Reset password nếu nghi compromise.
[ ] Xóa DONT_REQ_PREAUTH nếu không cần.
[ ] Hunt lateral movement sau thời điểm alert.
```

## 12. Mapping CDSA

**Security Operations & Monitoring:** thu Security log từ DC, parse 4768, correlate account/source/EType.

**Incident Response & Forensics:** triage source workstation, process execution, PowerShell logs và file artifacts.

**Threat Hunting:** hunt `4768 + PreAuthType 0`, RC4 `0x17`, honeypot activity và request từ VLAN bất thường.

**Detection Engineering:** viết Sigma rồi chuyển sang Splunk SPL hoặc Elastic rule.

## Key Takeaway

```text
AS-REProasting nhắm account không yêu cầu Kerberos preauthentication.
Hashcat mode 18200 dùng cho AS-REP etype 23.
Detection trọng tâm là Event ID 4768 + Pre-Authentication Type 0.
RC4 0x17, source IP và honeypot giúp tăng độ tin cậy.
```

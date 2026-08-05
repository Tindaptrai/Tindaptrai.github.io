---
layout: post
title: "Detecting Kerberoasting & AS-REPRoasting"
date: 2026-08-06 00:27:12 +0700
categories: ["Detecting Windows Attacks with Splunk", "Credential Access"]
tags: [kerberoasting, asreproasting, kerberos, rubeus, spn, tgs, tgt, event-id-4768, event-id-4769, event-id-4648, silketw, silkservice, ldap, splunk, detection-engineering, threat-hunting, cdsa]
description: "Phát hiện Kerberoasting và AS-REPRoasting bằng LDAP telemetry, Windows Security Logs và Splunk."
toc: true
---

# Detecting Kerberoasting & AS-REPRoasting

## Công thức nhớ nhanh

```text
KERBEROASTING:
SPN ENUM → TGS REQUEST → NO SERVICE LOGON → OFFLINE CRACK

AS-REPROASTING:
PREAUTH-DISABLED ENUM → AS-REQ/TGT → PREAUTH TYPE 0 → OFFLINE CRACK
```

# 1. Kerberoasting

Kerberoasting nhắm vào **service account có SPN**.

```text
Enumerate SPN accounts
→ request TGS
→ lấy ticket được mã hóa bằng secret của service account
→ crack offline
```

Tool thường gặp:

```text
Rubeus kerberoast
PowerView
Impacket GetUserSPNs.py
BloodHound/SharpHound
Hashcat
John the Ripper
```

# 2. Kerberos Flow Hợp Lệ

```text
4768 → TGT Request
4769 → TGS Request
4624 → Successful Logon
4648 → Explicit Credentials
```

Luồng hợp lệ:

```text
Client xin TGT
→ xin TGS cho SPN
→ trình TGS cho service
→ server ghi nhận logon
```

> `4769` tự nó không chứng minh Kerberoasting.

# 3. Event IDs Quan Trọng

| Event ID | Ý nghĩa |
|---|---|
| `4768` | Kerberos TGT Request |
| `4769` | Kerberos Service Ticket Request |
| `4624` | Successful Logon |
| `4648` | Explicit Credentials |

Field cần xem:

```text
user
service_name
src_ip
Ticket_Options
Ticket_Encryption_Type
Target_Server_Name
Additional_Information
Pre_Authentication_Type
```

# 4. Detect SPN Enumeration

LDAP filter:

```text
(&(samAccountType=805306368)(servicePrincipalName=*))
```

Telemetry:

```text
Microsoft-Windows-LDAP-Client
→ SilkETW/SilkService
→ Windows Event Log
→ Splunk
```

```spl
index=main
earliest=1690448444 latest=1690454437
source="WinEventLog:SilkService-Log"
| spath input=Message
| rename XmlEventData.* AS *
| table _time, ComputerName, ProcessName, DistinguishedName, SearchFilter
| search SearchFilter="*(&(samAccountType=805306368)(servicePrincipalName=*)*"
```

Dấu hiệu trong bài:

```text
ProcessName: rundll32
```

`rundll32.exe` thực hiện LDAP enumeration là context bất thường.

# 5. Benign TGS Request

```spl
index=main
earliest=1690388417 latest=1690388630
EventCode=4648 OR (EventCode=4769 AND service_name=iis_svc)
| dedup RecordNumber
| rex field=user "(?<username>[^@]+)"
| table _time, ComputerName, EventCode, name, username,
        Account_Name, Account_Domain, src_ip, service_name,
        Ticket_Options, Ticket_Encryption_Type,
        Target_Server_Name, Additional_Information
```

Pattern hợp lệ:

```text
4769 request TGS
+ 4648 explicit credential use
→ user thực sự truy cập service
```

# 6. Detect Kerberoasting — TGS Không Có 4648

```spl
index=main
earliest=1690450374 latest=1690450483
EventCode=4648 OR (EventCode=4769 AND service_name=iis_svc)
| dedup RecordNumber
| rex field=user "(?<username>[^@]+)"
| bin span=2m _time
| search username!=*$
| stats values(EventCode) AS Events,
        values(service_name) AS service_name,
        values(Additional_Information) AS Additional_Information,
        values(Target_Server_Name) AS Target_Server_Name
  BY _time, username
| where !match(Events,"4648")
```

Logic:

```text
4769 có mặt
+ không có 4648 trong cùng cửa sổ
→ nghi ticket được lấy để crack
```

`username!=*$` loại computer account.

# 7. Detect Bằng `transaction`

```spl
index=main
earliest=1690450374 latest=1690450483
EventCode=4648 OR (EventCode=4769 AND service_name=iis_svc)
| dedup RecordNumber
| rex field=user "(?<username>[^@]+)"
| search username!=*$
| transaction username keepevicted=true maxspan=5s
    startswith=(EventCode=4769)
    endswith=(EventCode=4648)
| where closed_txn=0 AND EventCode=4769
| table _time, EventCode, service_name, username
```

Keyword SPL:

```text
transaction
keepevicted
maxspan
startswith
endswith
closed_txn
```

# 8. Giới Hạn Detection Kerberoasting

Không phải mọi truy cập hợp lệ đều tạo `4648`.

Cần thêm context:

```text
SPN enumeration trước đó
Burst nhiều 4769
Nhiều service_name
RC4 encryption
Process bất thường
User không có baseline request ticket
Không có 4624 trên service host
```

# 9. AS-REPRoasting

AS-REPRoasting nhắm vào account có:

```text
Kerberos pre-authentication disabled
```

Chuỗi:

```text
Enumerate DONT_REQ_PREAUTH users
→ gửi AS-REQ
→ nhận AS-REP/TGT material
→ crack offline
```

Tool:

```text
Rubeus asreproast
Impacket GetNPUsers.py
PowerView
```

# 10. Kerberos Pre-Authentication

Khi bật pre-auth:

```text
AS-REQ + PA-ENC-TIMESTAMP
→ KDC verify timestamp bằng user key
→ đúng thì cấp TGT
```

Khi tắt:

```text
Không cần timestamp validation
→ attacker có thể xin AS-REP mà không biết password
```

# 11. LDAP Filter Cho Pre-Auth Disabled

```text
(samAccountType=805306368)
(userAccountControl:1.2.840.113556.1.4.803:=4194304)
```

```text
4194304 = DONT_REQ_PREAUTH
1.2.840.113556.1.4.803 = LDAP bitwise AND rule
```

# 12. Detect AS-REPRoast Enumeration

```spl
index=main
earliest=1690392745 latest=1690393283
source="WinEventLog:SilkService-Log"
| spath input=Message
| rename XmlEventData.* AS *
| table _time, ComputerName, ProcessName, DistinguishedName, SearchFilter
| search SearchFilter="*(samAccountType=805306368)(userAccountControl:1.2.840.113556.1.4.803:=4194304)*"
```

Dấu hiệu trong bài:

```text
ProcessName: Rubeus
```

# 13. Detect AS-REPRoasting Qua 4768

```spl
index=main
earliest=1690392745 latest=1690393283
source="WinEventLog:Security"
EventCode=4768
Pre_Authentication_Type=0
| rex field=src_ip "(\:\:ffff\:)?(?<src_ip>[0-9\.]+)"
| table _time, src_ip, user,
        Pre_Authentication_Type,
        Ticket_Options,
        Ticket_Encryption_Type
```

Logic:

```text
4768
+ Pre_Authentication_Type=0
→ TGT request không sử dụng pre-authentication
```

Regex chuẩn hóa:

```text
::ffff:10.0.10.101
→ 10.0.10.101
```

# 14. So Sánh Nhanh

| Thuộc tính | Kerberoasting | AS-REPRoasting |
|---|---|---|
| Target | Service account có SPN | User không yêu cầu pre-auth |
| Ticket | TGS | AS-REP/TGT material |
| Event chính | `4769` | `4768` |
| LDAP keyword | `servicePrincipalName=*` | `DONT_REQ_PREAUTH` |
| Giá trị UAC | Không cố định | `4194304` |
| Cracking | Offline | Offline |
| Tool | Rubeus kerberoast | Rubeus asreproast |

# 15. Tuning Và False Positives

Kerberoasting false positive:

```text
Legitimate service access
Application servers
Monitoring systems
Scheduled integrations
Service account authentication
```

AS-REP false positive:

```text
Legacy accounts
Compatibility requirements
Known pre-auth-disabled users
Authorized security testing
```

Tune theo:

```text
Known service accounts
Known client hosts
Expected SPNs
Ticket volume baseline
Encryption type
Business hours
LDAP query context
Process and parent process
```

# 16. Investigation Workflow

```text
1. Xác định source host và user
2. Kiểm tra LDAP precursor
3. Xem process tạo query
4. Kiểm tra 4769 hoặc 4768
5. Xem SPN/service_name
6. Kiểm tra Ticket_Encryption_Type
7. Correlate 4648 và 4624
8. Xem có burst nhiều account/SPN
9. Hunt credential dumping/lateral movement
10. Reset hoặc rotate credential bị ảnh hưởng
```

Hunt tiếp:

```text
PowerShell
Rubeus
SharpHound
LSASS access
PsExec
WMI
SMB
Remote service creation
```

# Keyword Cần Nhớ

```text
Kerberoasting
AS-REPRoasting
Rubeus
SPN
servicePrincipalName
TGT
TGS
KDC
4768
4769
4624
4648
Pre_Authentication_Type
PA-ENC-TIMESTAMP
DONT_REQ_PREAUTH
4194304
1.2.840.113556.1.4.803
SilkETW
SilkService
LDAP
transaction
closed_txn
Ticket_Encryption_Type
Hashcat
John the Ripper
```

# Key Takeaway

```text
Kerberoasting:
SPN query + TGS request + thiếu service access hợp lệ
```

```text
AS-REPRoasting:
DONT_REQ_PREAUTH query + 4768 PreAuthType=0
```

```text
Detection tốt =
LDAP precursor
+ Kerberos event
+ process/source context
+ correlation
```

> Chỉ thực hành trên HTB, lab hoặc hệ thống được cấp phép.

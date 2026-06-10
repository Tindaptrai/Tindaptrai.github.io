---
layout: post
title: "Detection & Analysis Stage Part 2: IOCs, Investigation và AI"
date: 2026-06-10 01:00:00 +0700
categories: [SOC, Incident Response]
tags: [soc, incident-response, detection-analysis, ioc, stix, yara, thehive, live-response, chain-of-custody, ai, threat-detection]
description: "Ghi chú về Detection & Analysis Part 2: cách điều tra incident, tạo và dùng IOC, xác định hệ thống bị ảnh hưởng, thu thập bằng chứng, chain of custody và ứng dụng AI trong threat detection."
toc: true
---

# Detection & Analysis Stage Part 2: IOCs, Investigation và AI

Trong phần trước, ta đã nói về giai đoạn **Detection & Analysis** ở mức tổng quan: phát hiện alert, xây dựng context, tạo timeline, đánh giá severity và giao tiếp trong quá trình điều tra.

Ở phần này, trọng tâm là quá trình điều tra sâu hơn:

- Hiểu **what happened**.
- Hiểu **how it happened**.
- Tạo và sử dụng **Indicators of Compromise**.
- Tìm hệ thống bị ảnh hưởng.
- Thu thập và phân tích dữ liệu.
- Bảo toàn bằng chứng.
- Hiểu vai trò của AI trong threat detection và incident response.

---

## 1. Mục tiêu của Investigation

Khi một cuộc điều tra bắt đầu, mục tiêu chính là hiểu:

```text
What happened?
How did it happen?
```

Tức là:

```text
Chuyện gì đã xảy ra?
Nó đã xảy ra bằng cách nào?
```

Đội Incident Handling cần có kiến thức kỹ thuật và kinh nghiệm thực tế để phân tích dữ liệu liên quan đến incident một cách chính xác.

---

## 2. Vì sao không đơn giản rebuild hệ thống?

Một câu hỏi thường gặp là:

```text
Tại sao phải điều tra kỹ?
Sao không rebuild máy bị ảnh hưởng rồi quên chuyện đó đi?
```

Câu trả lời là: nếu không biết attacker vào bằng cách nào và ảnh hưởng đến đâu, các bước remediation có thể không đủ để ngăn attacker quay lại.

Ví dụ:

```text
Máy bị nhiễm malware
      ↓
Team rebuild máy
      ↓
Nhưng tài khoản admin đã bị lộ
      ↓
Attacker dùng credential cũ đăng nhập lại
```

Nếu chỉ rebuild hệ thống mà không hiểu root cause, attacker có thể lặp lại cùng attack path.

Điều tra tốt giúp trả lời:

- Attacker vào bằng vector nào?
- Attacker dùng tool gì?
- Hệ thống nào bị ảnh hưởng?
- Credential nào bị lộ?
- Có persistence không?
- Attack path có thể bị lặp lại không?
- Cần remediation ở đâu?

---

# 3. Chu trình điều tra 3 bước

Investigation bắt đầu từ dữ liệu ban đầu đã thu thập được.

Từ dữ liệu ban đầu đó, đội Incident Handling sẽ đi qua một quy trình lặp lại gồm 3 bước:

```text
Initial Investigation Data
      ↓
Create and use IOCs
      ↓
Identify new leads and impacted systems
      ↓
Collect and analyze data
      ↓
Update timeline and repeat
```

Ba bước chính:

1. **Creation and usage of IOCs**.
2. **Identification of new leads and impacted systems**.
3. **Data collection and analysis from new leads and impacted systems**.

Quá trình này không diễn ra một lần rồi kết thúc. Nó lặp lại nhiều lần khi có bằng chứng mới.

---

## 4. Initial Investigation Data

**Initial Investigation Data** là dữ liệu ban đầu mà đội xử lý có về incident.

Ví dụ:

- Alert từ SIEM.
- Alert từ EDR.
- IP đáng ngờ.
- Hash file malware.
- Host bị nghi compromise.
- User liên quan.
- Thời điểm xảy ra sự kiện.
- Command line đáng ngờ.
- Domain hoặc URL độc hại.
- File path lạ.
- Process bất thường.

Từ dữ liệu này, analyst bắt đầu tìm lead mới.

---

## 5. Không nên điều tra quá hẹp

Một sai lầm phổ biến là chỉ tập trung vào một dấu hiệu duy nhất, ví dụ một tool độc hại đã biết.

Nếu chỉ tập trung vào một finding, đội điều tra có thể:

- Bỏ sót hệ thống khác bị ảnh hưởng.
- Kết luận quá sớm.
- Không hiểu toàn bộ impact.
- Không phát hiện attack path đầy đủ.
- Không tìm thấy persistence khác.
- Không tìm thấy credential compromise.

Vì vậy, trong suốt quá trình điều tra, team cần liên tục đặt câu hỏi:

```text
Còn lead nào khác không?
IOC này có xuất hiện ở hệ thống khác không?
Có bằng chứng nào cho thấy attacker đã di chuyển tiếp không?
```

---

# 6. IOC là gì?

**IOC** là viết tắt của **Indicator of Compromise**.

IOC là dấu hiệu cho thấy một incident hoặc compromise đã xảy ra.

Hiểu đơn giản:

```text
IOC = dấu hiệu nhận biết hệ thống có thể đã bị xâm nhập.
```

Ví dụ IOC:

- IP address độc hại.
- Domain C2.
- URL phishing.
- File hash.
- File name.
- Registry key.
- Mutex.
- Process name.
- Scheduled task.
- Service name.
- Email sender.
- Command line pattern.

---

## 7. Vì sao IOC quan trọng?

IOC giúp analyst tìm kiếm dấu hiệu compromise trên nhiều hệ thống.

Ví dụ:

```text
Tìm thấy malware hash trên máy A
      ↓
Dùng hash đó search toàn bộ endpoint
      ↓
Phát hiện máy B và C cũng có cùng file
      ↓
Mở rộng phạm vi điều tra
```

IOC giúp trả lời:

- Incident có lan sang host khác không?
- Có bao nhiêu hệ thống bị ảnh hưởng?
- IOC có xuất hiện trong log cũ không?
- Có campaign tương tự không?
- IOC có phải false positive không?

---

## 8. Các định dạng dùng để mô tả IOC

IOC có thể được lưu và chia sẻ bằng nhiều định dạng hoặc ngôn ngữ chuẩn.

Một số định dạng phổ biến:

| Định dạng | Mục đích |
|---|---|
| OpenIOC | Định dạng mô tả IOC có cấu trúc |
| YARA | Rule nhận diện file/malware dựa trên pattern |
| STIX | Định dạng chuẩn để trao đổi threat intelligence |
| JSON IOC | IOC ở dạng JSON, dễ xử lý tự động |

---

## 9. OpenIOC

**OpenIOC** là một định dạng dùng để mô tả IOC theo cấu trúc chuẩn.

Nó giúp security team mô tả artifact phát hiện trong quá trình điều tra.

Ví dụ:

- File name.
- Hash.
- Registry path.
- IP address.
- Domain.
- Process.

Một số công cụ như **Mandiant IOC Editor** có thể dùng để tạo hoặc chỉnh sửa IOC.

---

## 10. YARA

**YARA** là công cụ/rule language dùng để nhận diện file hoặc malware dựa trên pattern.

YARA có thể tìm:

- Chuỗi đặc trưng trong file.
- Header bất thường.
- Section name.
- Byte pattern.
- Malware family indicator.

Ví dụ ứng dụng:

```text
Có malware sample
      ↓
Tạo YARA rule từ string/hash/pattern
      ↓
Scan file system hoặc repository malware
      ↓
Tìm sample liên quan
```

YARA rất hữu ích trong malware analysis và threat hunting.

---

## 11. STIX

**STIX** là viết tắt của **Structured Threat Information eXpression**.

STIX là ngôn ngữ mã nguồn mở, machine-readable, thường dùng dưới dạng JSON để trao đổi cyber threat intelligence.

STIX có thể mô tả:

- Malware.
- File.
- Hash.
- Indicator.
- Threat actor.
- Campaign.
- Attack pattern.
- Infrastructure.
- Relationship giữa các đối tượng.

Ví dụ IOC trong STIX có thể chứa:

- MD5.
- SHA-1.
- SHA-256.
- SHA-512.
- File name.
- File size.
- PE metadata.
- Section hash.

Các tổ chức như CISA có thể công bố IOC ở định dạng STIX để các tổ chức khác tự động ingest và sử dụng.

---

# 12. IOC trong TheHive

Trong **TheHive**, IOC có thể được thêm vào phần **Observables** của alert hoặc case.

Observable có thể là:

- IP.
- Domain.
- FQDN.
- Hostname.
- Hash.
- File.
- Filename.
- URL.
- Autonomous System.

Trong TheHive, analyst có thể đánh dấu một observable là IOC nếu nó là dấu hiệu compromise thật sự.

Ví dụ workflow:

```text
Alert received
      ↓
Open Observables
      ↓
Add IP/hash/domain
      ↓
Mark as IOC
      ↓
Tag TLP/PAP if needed
      ↓
Use IOC to pivot/hunt
```

---

## 13. Dùng IOC để hunt trên môi trường

Để tận dụng IOC, tổ chức cần công cụ tìm kiếm IOC trên nhiều endpoint hoặc log source.

Ví dụ công cụ/cách làm:

- EDR search.
- SIEM search.
- PowerShell.
- WMI.
- Velociraptor.
- Osquery.
- TheHive + Cortex.
- YARA scanning.
- Log search.

Trong môi trường Windows, có thể dùng:

```text
WMI
PowerShell
WinRM
EDR Live Query
```

---

# 14. Cẩn thận với credential khi điều tra

Một cảnh báo rất quan trọng: khi điều tra trên hệ thống có thể bị compromise, cần tránh để credential đặc quyền bị cache trên máy đó.

Nếu dùng tài khoản đặc quyền để kết nối vào hệ thống bị compromise, attacker có thể lấy credential từ memory hoặc cache.

Vì vậy, cần biết rõ công cụ mình dùng để lại dấu vết gì.

---

## 15. WinRM và Logon Type 3

Windows logon với **logon type 3**, còn gọi là **Network Logon**, thường không cache credential trên remote system.

Ví dụ:

- Kết nối bằng WinRM.
- Một số truy cập mạng dạng remote command an toàn hơn nếu dùng đúng cách.

Tuy nhiên, vẫn cần kiểm tra tool cụ thể.

---

## 16. PsExec và rủi ro credential cache

**PsExec** là ví dụ rất tốt cho câu “know your tools”.

Nếu dùng PsExec với explicit credentials, credential có thể bị cache trên remote machine.

Ví dụ rủi ro:

```text
psexec.exe \\host -u DOMAIN\Admin -p Password123 cmd.exe
```

Cách này có thể làm credential bị lưu lại trên máy remote.

Nhưng nếu dùng PsExec không truyền explicit credential, thông qua session của user hiện tại, credential có thể không bị cache giống cách trên.

Bài học:

```text
Cùng một tool nhưng cách dùng khác nhau có thể để lại dấu vết khác nhau.
```

---

# 17. Identification of New Leads & Impacted Systems

Sau khi search IOC, analyst có thể nhận được nhiều kết quả.

Những kết quả đó có thể là:

- Hệ thống thật sự bị compromise.
- Hệ thống liên quan gián tiếp.
- False positive.
- Artifact cũ không liên quan.
- IOC quá generic nên match nhiều nơi.

Nhiệm vụ của analyst là:

```text
Identify true leads
Eliminate false positives
Prioritize impacted systems
```

---

## 18. False Positive là gì?

**False positive** là cảnh báo hoặc kết quả khớp IOC nhưng không phải độc hại thật.

Ví dụ:

```text
IOC là filename: svchost.exe
```

Tên này quá generic vì Windows có file hợp lệ tên `svchost.exe`.

Nếu search chỉ theo filename, sẽ có rất nhiều hit nhưng phần lớn không độc hại.

Vì vậy, IOC cần đủ chính xác, ví dụ kết hợp:

- File path.
- Hash.
- Parent process.
- Command line.
- Signature.
- Creation time.
- Network destination.

---

## 19. Prioritization

Nếu có quá nhiều IOC hits, cần ưu tiên phân tích.

Nên ưu tiên:

- Business-critical systems.
- Domain Controllers.
- File servers.
- Servers có dữ liệu nhạy cảm.
- Host có nhiều IOC match.
- Host có dấu hiệu lateral movement.
- Host có dấu hiệu credential dumping.
- Host có dấu hiệu C2.
- Host có timeline gần patient zero.

Mục tiêu là chọn hệ thống có khả năng đem lại lead mới sau forensic analysis.

---

# 20. Data Collection and Analysis

Khi đã xác định hệ thống có IOC hoặc lead đáng quan tâm, cần thu thập và bảo toàn trạng thái hệ thống.

Mục tiêu:

- Thu thập bằng chứng.
- Giữ nguyên giá trị forensic.
- Tìm lead mới.
- Trả lời câu hỏi điều tra.
- Xác định attacker đã làm gì.

---

## 21. Live Response

**Live response** là thu thập dữ liệu trên hệ thống đang chạy.

Đây là cách phổ biến nhất trong incident response.

Dữ liệu thường thu thập:

- Process list.
- Network connections.
- Logged-on users.
- Services.
- Scheduled tasks.
- Autoruns.
- Registry keys.
- Event logs.
- Prefetch.
- Amcache.
- ShimCache.
- Recent files.
- Command history.
- Memory dump nếu cần.

Live response hữu ích vì nhiều artifact chỉ tồn tại khi hệ thống còn chạy.

---

## 22. Khi nào không nên tắt máy ngay?

Không nên tắt máy ngay nếu cần bảo toàn dữ liệu trong RAM.

RAM có thể chứa:

- Malware đang chạy.
- Injected code.
- Encryption key.
- Network session.
- Credential.
- Command line.
- Process memory.
- C2 connection.
- Unsaved data.

Nếu tắt máy, những dữ liệu này sẽ mất.

Vì vậy, trong nhiều trường hợp cần memory capture trước khi shutdown.

---

## 23. Minimal Interaction

Khi thu thập dữ liệu, cần hạn chế tương tác với hệ thống.

Lý do:

- Tránh thay đổi bằng chứng.
- Tránh ghi đè artifact.
- Tránh làm mất timestamp.
- Tránh kích hoạt malware.
- Tránh làm attacker nhận ra.
- Tránh làm kết quả forensic bị tranh cãi.

Nguyên tắc:

```text
Collect what is needed.
Change as little as possible.
Document every action.
```

---

## 24. Analysis

Sau khi thu thập dữ liệu, analyst bắt đầu phân tích.

Đây thường là phần tốn thời gian nhất của incident.

Các loại phân tích phổ biến:

- Malware analysis.
- Disk forensics.
- Memory forensics.
- Log analysis.
- Timeline analysis.
- Network traffic analysis.
- Registry analysis.
- Event log analysis.

Mọi lead mới được xác thực nên được thêm vào timeline.

Timeline sẽ được cập nhật liên tục trong suốt quá trình điều tra.

---

## 25. Memory Forensics

**Memory forensics** ngày càng quan trọng, đặc biệt trong các cuộc tấn công nâng cao.

Memory forensics có thể phát hiện:

- Malware chỉ chạy trong memory.
- Process injection.
- Hidden process.
- Network connection.
- Credential artifact.
- Command line.
- DLL injection.
- Rootkit behavior.

Tool phổ biến:

- Volatility.
- Rekall.
- MemProcFS.
- WinPmem.
- DumpIt.

---

# 26. Chain of Custody

**Chain of Custody** là chuỗi quản lý bằng chứng.

Khi thu thập dữ liệu, cần ghi lại chain of custody để đảm bảo bằng chứng có thể được chấp nhận nếu có hành động pháp lý.

Chain of custody ghi lại:

- Bằng chứng là gì?
- Ai thu thập?
- Thu thập khi nào?
- Thu thập ở đâu?
- Hash của bằng chứng là gì?
- Ai nhận bằng chứng?
- Bằng chứng được lưu ở đâu?
- Có ai truy cập hoặc thay đổi không?

Nếu chain of custody không rõ ràng, bằng chứng có thể bị nghi ngờ về tính toàn vẹn.

---

# 27. Use of AI in Threat Detection

AI đang thay đổi cách tổ chức phát hiện, triage và phản ứng với incident.

Trong workflow truyền thống, analyst phải tự đọc:

- Log.
- Alert.
- Report.
- Timeline.
- Detection result.

Việc này có thể mất hàng giờ hoặc nhiều ngày.

AI có thể hỗ trợ bằng cách:

- Tự động phân tích nhiều alert.
- Gom nhóm alert liên quan.
- Ưu tiên cảnh báo quan trọng.
- Tái dựng timeline.
- Tìm behavioral anomaly.
- Tóm tắt attack story.
- Gợi ý MITRE ATT&CK mapping.
- Hỗ trợ post-incident analysis.

---

## 28. AI Attack Discovery

Một ví dụ là tính năng **Attack Discovery** trong Elastic Security.

Tính năng này dùng LLM để phân tích nhiều alert trong môi trường và tạo summary về cuộc tấn công.

AI có thể cho biết:

- Host nào liên quan.
- User nào liên quan.
- Các bước attack.
- Alert nào có quan hệ với nhau.
- MITRE ATT&CK mapping.
- Timeline tấn công.
- Hoạt động chính của attacker.

Ví dụ, AI có thể gom nhiều alert thành một câu chuyện:

```text
Host SRVNIX05
      ↓
root giải nén file đáng ngờ
      ↓
copy payload vào /dev/shm
      ↓
chmod 755
      ↓
execute payload
      ↓
cleanup file gốc
      ↓
detected as Linux.Trojan.BPFDoor
```

Điều này giúp analyst hiểu nhanh hơn thay vì đọc từng alert riêng lẻ.

---

## 29. Các use case của AI trong Incident Response

AI có thể hỗ trợ nhiều hoạt động:

| Use Case | Ý nghĩa |
|---|---|
| Automated Triage & Alert Prioritization | Tự động phân loại và ưu tiên alert |
| Incident Correlation | Gom nhóm alert liên quan |
| Timeline Reconstruction | Tái dựng timeline |
| Automated Response Playbooks | Gợi ý hoặc kích hoạt playbook |
| Post-Incident Analysis | Hỗ trợ rút kinh nghiệm sau incident |
| Threat Hunting Support | Gợi ý giả thuyết hunt |
| MITRE ATT&CK Mapping | Gợi ý tactic/technique phù hợp |

---

## 30. Lưu ý khi dùng AI

AI rất hữu ích nhưng không thay thế hoàn toàn analyst.

Cần lưu ý:

- AI có thể hiểu sai context.
- AI có thể hallucinate.
- AI cần dữ liệu đầu vào tốt.
- Analyst vẫn phải xác minh evidence.
- Không nên gửi dữ liệu nhạy cảm vào công cụ không được phê duyệt.
- Cần tuân thủ chính sách bảo mật và pháp lý.

Nói ngắn gọn:

```text
AI hỗ trợ analyst, nhưng analyst vẫn chịu trách nhiệm xác minh.
```

---

# 31. Workflow Investigation tổng hợp

Một workflow điều tra tổng hợp:

```text
Initial alert
      ↓
Collect initial context
      ↓
Create IOCs
      ↓
Search IOCs across environment
      ↓
Identify impacted systems
      ↓
Eliminate false positives
      ↓
Prioritize systems
      ↓
Collect evidence
      ↓
Preserve state
      ↓
Analyze data
      ↓
Find new leads
      ↓
Update timeline
      ↓
Repeat until scope is clear
```

---

# 32. Bảng ôn nhanh thuật ngữ

| Thuật ngữ | Nghĩa ngắn gọn |
|---|---|
| Investigation | Quá trình điều tra sự cố |
| Lead | Manh mối điều tra |
| IOC | Dấu hiệu compromise |
| OpenIOC | Định dạng mô tả IOC |
| YARA | Rule nhận diện file/malware |
| STIX | Chuẩn trao đổi threat intelligence |
| Observable | Thông tin quan sát được trong TheHive |
| False Positive | Kết quả dương tính giả |
| Impacted System | Hệ thống bị ảnh hưởng |
| Live Response | Thu thập dữ liệu trên hệ thống đang chạy |
| Memory Forensics | Phân tích RAM |
| Disk Forensics | Phân tích ổ đĩa |
| Chain of Custody | Chuỗi quản lý bằng chứng |
| WinRM | Công cụ quản trị từ xa trên Windows |
| Logon Type 3 | Network logon |
| PsExec | Công cụ remote execution của Sysinternals |
| Threat Detection | Phát hiện mối đe dọa |
| AI Attack Discovery | AI gom nhóm và tóm tắt alert thành attack story |
| LLM | Large Language Model |
| Automated Triage | Tự động phân loại và ưu tiên alert |

---

# 33. Câu hỏi ôn tập

## 1. Vì sao cần biết incident xảy ra như thế nào?

Vì nếu không biết attacker vào bằng cách nào, remediation có thể không ngăn attacker lặp lại cùng attack path.

## 2. IOC là gì?

IOC là dấu hiệu cho thấy hệ thống có thể đã bị compromise, ví dụ IP, hash, domain, file name hoặc registry key.

## 3. Vì sao IOC quá generic có thể gây vấn đề?

Vì nó có thể tạo nhiều false positive và làm analyst điều tra sai hướng.

## 4. Vì sao không nên tắt máy ngay trong nhiều incident?

Vì nhiều artifact quan trọng chỉ tồn tại trong RAM và sẽ mất nếu tắt máy.

## 5. Chain of custody dùng để làm gì?

Dùng để ghi lại quá trình thu thập, lưu trữ và bàn giao bằng chứng, đảm bảo bằng chứng giữ được tính toàn vẹn.

## 6. AI giúp gì trong Detection & Analysis?

AI có thể hỗ trợ triage alert, correlation, timeline reconstruction, attack summary, MITRE mapping và post-incident analysis.

---

# Key Takeaway

Trong **Detection & Analysis Part 2**, điều quan trọng nhất là không chỉ biết có incident, mà phải hiểu rõ:

```text
What happened?
How did it happen?
What was impacted?
How can we prevent it from happening again?
```

IOC giúp mở rộng điều tra và tìm hệ thống bị ảnh hưởng. Live response và forensic analysis giúp trả lời các câu hỏi kỹ thuật. Chain of custody giúp đảm bảo bằng chứng có giá trị. AI có thể tăng tốc quá trình triage và correlation, nhưng analyst vẫn phải xác minh mọi kết luận bằng evidence.

---
layout: post
title: "Network Traffic Analysis: Practical Workflow"
date: 2026-06-25 23:42:24 +0700
categories: [SOC, Network Traffic Analysis]
tags: [soc, network-traffic-analysis, nta, pcap, netflow, wireshark, tcpdump, traffic-analysis, baseline, incident-response, blue-team]
description: "Tóm tắt quy trình phân tích lưu lượng mạng trong thực tế: descriptive, diagnostic, predictive, prescriptive analysis, scope, capture, filtering, baseline, note-taking và các phương pháp tìm bất thường."
toc: true
---

# Network Traffic Analysis: Practical Workflow

Phân tích lưu lượng mạng là một quy trình động. Nó thay đổi theo công cụ, quyền truy cập, vị trí đặt sensor và mức độ visibility trong mạng.

```text
NTA Practical Workflow = xác định vấn đề → giới hạn phạm vi → capture traffic → lọc noise → phân tích → tóm tắt → đề xuất hành động.
```

---

## Timeline cập nhật

```text
Created at: 2026-06-25 23:42:24 +0700
Timezone: Asia/Ho_Chi_Minh
Topic: Network Traffic Analysis
Focus: Practical analysis workflow, baseline, capture, filtering, reporting
```

---

# 1. Phân tích lưu lượng mạng trong thực tế

Traffic analysis là quá trình kiểm tra chi tiết một event hoặc process để xác định:

- Chuyện gì đang xảy ra.
- Nguồn gốc vấn đề.
- Phạm vi ảnh hưởng.
- Traffic có khớp baseline không.
- Có dấu hiệu malicious không.
- Có cần incident response không.

Trong network, analyst thường tìm:

- Remote access trái phép qua RDP/SSH/Telnet.
- Malware download.
- C2 communication.
- File transfer đáng ngờ.
- Host-to-host traffic bất thường.
- Network misconfiguration.
- Protocol errors.
- Traffic lệch baseline.

---

# 2. Vì sao visibility quan trọng?

Nếu không có khả năng giám sát traffic, đội phòng thủ bị thiếu một phần rất lớn của bức tranh.

NTA giúp thấy:

- Network usage.
- Top talkers.
- Host-to-host communication.
- Internal communication.
- Protocol usage.
- Dòng traffic bất thường.
- Dấu hiệu compromise.

```text
Càng thấy sớm → càng giảm thiệt hại tiềm ẩn.
```

---

# 3. Không nên chỉ dựa vào tool

Các công cụ như IDS/IPS, firewall, SIEM, Splunk, ELK, Security Onion, Wireshark hay tcpdump rất hữu ích.

Nhưng analyst không nên chỉ phụ thuộc vào tool vì:

- Tool có thể bị bypass.
- Signature có thể chưa có.
- False positive/false negative vẫn xảy ra.
- Attacker luôn tìm cách tránh detection.
- Context môi trường cần con người đánh giá.

```text
Tool hỗ trợ phân tích, nhưng mắt người và tư duy analyst vẫn rất quan trọng.
```

---

# 4. Descriptive Analysis

**Descriptive analysis** trả lời câu hỏi:

```text
Chuyện gì đang xảy ra?
Ta đang nhìn vào dataset nào?
Phạm vi điều tra là gì?
```

Đây là bước mô tả dataset và xác định phạm vi.

## 4.1. What is the issue?

Cần xác định vấn đề:

```text
Nghi ngờ vi phạm?
Hay chỉ là vấn đề network?
```

Ví dụ:

```text
Nhiều máy chủ có khả năng tải xuống file độc hại từ bad.example.com.
```

## 4.2. Define scope and goal

Cần xác định:

- Ta đang tìm gì?
- Tìm trong khoảng thời gian nào?
- Có IOC/file name/domain/IP nào không?
- Có protocol nào liên quan không?

Ví dụ:

```text
Goal:
Tìm host có thể tải file độc hại.

Time period:
48 giờ qua + 2 giờ monitoring từ hiện tại.

Supporting information:
superbad.exe
new-crypto-miner.exe
bad.example.com
```

## 4.3. Define targets

Cần xác định target theo:

- Network.
- Host.
- Protocol.
- Port.
- Time window.
- Known indicator.

Ví dụ:

```text
Scope:
192.168.100.0/24

Protocols:
HTTP
FTP
```

---

# 5. Diagnostic Analysis

**Diagnostic analysis** trả lời câu hỏi:

```text
Vì sao chuyện này xảy ra?
Nguyên nhân là gì?
Ảnh hưởng thế nào?
```

Đây là bước capture, filter và hiểu traffic.

## 5.1. Capture network traffic

Có thể capture live traffic hoặc lấy dữ liệu lịch sử.

Nguồn dữ liệu:

- PCAP.
- NetFlow.
- SIEM logs.
- Firewall logs.
- Proxy logs.
- IDS/IPS alerts.
- Zeek logs.
- Full packet capture.

Ví dụ hành động:

```text
Cắm vào link có access đến mạng 192.168.100.0/24.
Capture live traffic để tìm file đang được truyền.
Yêu cầu admin lấy PCAP hoặc NetFlow từ SIEM cho dữ liệu lịch sử.
```

## 5.2. Identify required traffic components

Sau khi có traffic, cần lọc bớt noise.

Giữ lại traffic liên quan:

- HTTP.
- FTP.
- Subnet trong scope.
- Request đến domain đáng ngờ.
- File transfer.
- GET request đến file nghi vấn.
- Traffic chứa file name nghi vấn.

Ví dụ filter trong Wireshark:

```text
http.request.method == "GET"
```

```text
ftp-data
```

Ý tưởng:

```text
Loại traffic khớp baseline bình thường.
Giữ traffic liên quan đến bad.example.com, superbad.exe, new-crypto-miner.exe.
```

## 5.3. Understand captured traffic

Khi noise đã giảm, analyst cần hiểu traffic còn lại.

Cần tìm:

- Host nào tải file?
- File được tải qua protocol nào?
- Thời điểm tải?
- Từ domain/IP nào?
- Có transfer nội bộ tiếp theo không?
- Có nhiều host liên quan không?
- Có dấu hiệu lan truyền không?

---

# 6. Predictive Analysis

**Predictive analysis** dùng dữ liệu hiện tại và lịch sử để dự đoán khả năng xảy ra hoặc xu hướng tiếp theo.

Trong NTA, bước này giúp:

- So sánh traffic với baseline.
- Xác định deviation.
- Nhận diện host có khả năng bị compromise.
- Tìm pattern có thể tái diễn.
- Ước lượng phạm vi ảnh hưởng.

## 6.1. Note-taking and mind mapping

Ghi chú là bắt buộc trong điều tra.

Cần ghi:

- Time window capture.
- Host đáng ngờ.
- Conversation liên quan.
- Timestamp.
- Packet number.
- File name.
- Domain/IP.
- Protocol.
- Evidence.
- Filter đã dùng.
- Finding tạm thời.

Ví dụ note:

```text
Capture window:
2026-06-25 21:00 - 2026-06-25 23:00

Suspicious host:
192.168.100.25

Indicator:
bad.example.com
superbad.exe

Evidence:
HTTP GET /superbad.exe
Packet No. 18422
```

## 6.2. Summary of analysis

Summary nên trả lời:

```text
Chúng ta tìm thấy gì?
Host nào bị ảnh hưởng?
Traffic nào chứng minh điều đó?
Dữ liệu có khớp IOC không?
Có cần isolate host không?
Có cần IR escalation không?
```

---

# 7. Prescriptive Analysis

**Prescriptive analysis** trả lời:

```text
Nên làm gì tiếp theo?
Làm sao loại bỏ hoặc ngăn vấn đề tái diễn?
```

Các hành động có thể gồm:

- Isolate affected hosts.
- Block domain/IP.
- Remove malware.
- Reset credentials.
- Patch vulnerability.
- Update firewall rule.
- Tune IDS/IPS.
- Create detection.
- Create incident ticket.
- Preserve evidence.
- Start full incident response.

## 7.1. Lessons learned

Sau khi xử lý, cần review lại:

- Điều gì làm đúng?
- Điều gì không giúp ích?
- Detection nào bị thiếu?
- Visibility có đủ không?
- Baseline có cần cập nhật không?
- Có cần thêm log source không?
- Có cần cải thiện playbook không?

---

# 8. Full Workflow Template

```text
1. What is the issue?
   - Nghi ngờ vi phạm hay vấn đề network?

2. Define scope and goal
   - Tìm gì?
   - Time period nào?
   - IOC/file/domain/IP nào?

3. Define targets
   - Network/host/protocol/port nào?

4. Capture network traffic
   - Live capture hay PCAP/NetFlow lịch sử?

5. Identify required components
   - Lọc traffic không liên quan.
   - Giữ HTTP/FTP/domain/file/protocol trong scope.

6. Understand captured traffic
   - Host nào liên quan?
   - File nào được truyền?
   - Flow nào đáng ngờ?

7. Note-taking and mind mapping
   - Ghi timestamp, packet number, host, evidence.

8. Summary of analysis
   - Tóm tắt finding, impact, recommendation.
```

---

# 9. Các thành phần chính của phân tích hiệu quả

## 9.1. Know your environment

Analyst phải hiểu môi trường.

Cần có:

- Asset inventory.
- Network map.
- IP ranges.
- VLANs.
- Critical servers.
- Normal traffic baseline.
- Common protocols.
- Approved remote access paths.
- Security policy.

Nếu không biết host có thuộc mạng hay không, rất khó xác định nó là rogue hay hợp lệ.

## 9.2. Location is key

Vị trí capture rất quan trọng.

```text
Đặt sensor/capture point càng gần nguồn vấn đề càng tốt.
```

Ví dụ:

- Nếu traffic đến từ Internet: capture ở inbound edge/link.
- Nếu vấn đề nằm trong LAN: capture cùng segment với host nghi vấn.
- Nếu cần traffic routed giữa VLAN: đặt sensor ở Layer 3 link hoặc dùng TAP/SPAN phù hợp.

## 9.3. Be persistent

Không phải threat nào cũng xuất hiện liên tục.

Ví dụ C2 có thể beacon:

```text
Mỗi vài giờ một lần
Một lần mỗi ngày
Hoặc interval rất dài để né detection
```

Nếu không thấy ngay lần đầu, cần tiếp tục monitor hoặc mở rộng time window.

---

# 10. Phương pháp phân tích traffic

## 10.1. Start with standard protocols first

Bắt đầu với protocol phổ biến:

```text
HTTP/S
FTP
Email
TCP
UDP
SSH
RDP
Telnet
```

Cần đối chiếu với policy:

```text
Tổ chức có cho phép RDP từ Internet không?
Telnet có được phép không?
SSH có whitelist không?
```

## 10.2. Look for patterns

Tìm các mẫu lặp lại:

```text
Host connect Internet cùng giờ mỗi ngày
Beacon đều theo interval
DNS query lặp lại
Outbound traffic định kỳ
Same destination nhiều lần
```

Đây có thể là dấu hiệu C2.

## 10.3. Check host-to-host traffic

Trong môi trường chuẩn, user workstations hiếm khi nói chuyện trực tiếp với nhau.

Cần chú ý:

```text
PC người dùng kết nối SMB đến PC người dùng khác
Host-to-host trên port 445
Host-to-host trên port 8080
Workstation truy cập ADMIN$
Internal lateral movement pattern
```

Traffic bình thường thường là:

```text
Client → DNS
Client → DHCP
Client → Domain Controller
Client → File server
Client → Web/Intranet app
Client → Internet proxy/gateway
```

## 10.4. Look for unique events

Các event hiếm có thể đáng chú ý:

- User-Agent lạ.
- Port chỉ xuất hiện 1-2 lần.
- Host nói chuyện với Internet host lạ.
- Pattern truy cập website thay đổi.
- Protocol chạy trên port không chuẩn.
- Service mới xuất hiện.
- Flow có data size bất thường.

## 10.5. Ask for help

Sau thời gian dài nhìn PCAP/log, analyst có thể bỏ sót chi tiết.

Nên nhờ người khác review khi:

- Traffic quá lớn.
- Kết quả mơ hồ.
- Không chắc filter đúng.
- Finding có impact lớn.
- Cần xác nhận trước khi escalate.

---

# 11. Ví dụ scenario

Tình huống:

```text
Nhiều user host báo latency cao.
File lạ xuất hiện trên desktop.
Nghi ngờ có malware download hoặc lateral movement.
```

Không có baseline:

```text
Phải kiểm tra gần như mọi conversation.
Tốn thời gian.
Khó biết host nào bình thường.
Dễ bị chìm trong noise.
```

Có baseline:

```text
Loại bỏ known-good communication.
Tập trung top talkers.
So sánh host traffic với baseline.
Phát hiện vài user PCs nói chuyện qua 8080 và 445.
```

Finding có thể là:

```text
User-to-user SMB/8080 bất thường
→ nghi lateral movement hoặc file transfer nội bộ
→ tạo incident ticket
→ yêu cầu IR hỗ trợ
```

---

# 12. Bảng tóm tắt workflow

| Giai đoạn | Câu hỏi chính | Output |
|---|---|---|
| Descriptive | Chuyện gì đang xảy ra? | Scope, goal, target |
| Diagnostic | Vì sao xảy ra? | Captured traffic, filtered evidence |
| Predictive | Có pattern/xu hướng gì? | Baseline comparison, impact estimate |
| Prescriptive | Nên làm gì tiếp? | Recommendation, containment, detection |

---

# Key Takeaway

Phân tích traffic không phải quy trình một lần là xong. Nó thường lặp lại theo chu kỳ.

```text
Define issue
→ capture
→ filter
→ analyze
→ note
→ summarize
→ act
→ monitor
→ repeat if needed
```

Muốn làm NTA tốt, analyst cần hiểu môi trường, đặt capture point đúng vị trí, kiên trì theo dõi, bắt đầu từ protocol phổ biến, tìm pattern, kiểm tra host-to-host traffic bất thường và không ngại nhờ người khác review.

---
layout: post
title: "Network Traffic Analysis Process"
date: 2026-06-25 22:46:40 +0700
categories: [SOC, Network Traffic Analysis]
tags: [soc, network-traffic-analysis, nta, traffic-capture, pcap, baseline, passive-capture, active-capture, span-port, network-tap, blue-team]
description: "Tóm tắt quy trình phân tích lưu lượng mạng: mục tiêu NTA, visibility, active/passive capture, các phụ thuộc khi capture traffic, baseline và cách phát hiện bất thường."
toc: true
---

# Network Traffic Analysis Process

**Network Traffic Analysis (NTA)** là quá trình kiểm tra chi tiết lưu lượng mạng để hiểu chuyện gì đang xảy ra, xác định nguồn gốc/tác động của sự kiện và hỗ trợ phòng ngừa hoặc phản ứng với incident.

```text
NTA Process = capture traffic → lọc noise → phân tích baseline/bất thường → xác định vấn đề → hành động và monitor.
```

---

## Timeline cập nhật

```text
Created at: 2026-06-25 22:46:40 +0700
Timezone: Asia/Ho_Chi_Minh
Topic: Network Traffic Analysis Process
Focus: traffic analysis workflow, passive/active capture, dependencies, baseline
```

---

# 1. Phân tích lưu lượng mạng là gì?

Phân tích lưu lượng mạng là một quy trình động, phụ thuộc vào công cụ, quyền được cấp, vị trí đặt capture device và mức độ visibility trong network.

NTA không chỉ là mở Wireshark xem packet. Nó là quá trình chia nhỏ dữ liệu mạng thành các phần dễ hiểu, sau đó kiểm tra xem có gì lệch khỏi hành vi bình thường hay không.

---

# 2. Mục tiêu của NTA

NTA giúp defender:

- Xác định nguồn gốc sự kiện.
- Xác định tác động của sự kiện.
- Tìm traffic độc hại tiềm ẩn.
- Tìm remote access trái phép như RDP, SSH, Telnet.
- Phát hiện lỗi network/protocol.
- So sánh traffic với baseline bình thường.
- Tạo cảnh báo hoặc hành động phòng ngừa.
- Hỗ trợ incident response và threat hunting.

Ví dụ:

```text
Nếu thấy nhiều kết nối RDP từ Internet vào host nội bộ,
đây có thể là brute force hoặc remote access trái phép.
```

---

# 3. Vì sao visibility quan trọng?

Không có traffic visibility thì defender bị thiếu một phần lớn bức tranh.

NTA cung cấp visibility về:

- Host nào nói chuyện nhiều nhất.
- Server nào nhận nhiều traffic nhất.
- Protocol nào đang được dùng.
- Internal communication diễn ra thế nào.
- Có host nào nói chuyện bất thường không.
- Có port/protocol lạ không.
- Có data transfer lớn không.

Visibility giúp admin và security team phát hiện vấn đề trước hoặc ngay sau khi nó xảy ra.

---

# 4. Baseline trong NTA

**Baseline** là hình ảnh bình thường của network.

Baseline có thể gồm:

- Host nào thường giao tiếp với server nào.
- Port nào thường được dùng.
- Protocol nào bình thường trong từng VLAN.
- Lưu lượng trung bình theo giờ/ngày.
- Top talkers bình thường.
- DNS/web/SMB/RDP pattern bình thường.

Nếu có baseline, analyst có thể nhanh chóng lọc traffic hợp lệ và tập trung vào outlier.

---

# 5. NTA trong phòng thủ nâng cao

Trong môi trường nâng cao, NTA thường kết hợp với:

- IDS/IPS.
- Firewall logs.
- Host logs.
- Network logs.
- Splunk.
- ELK/Elastic Stack.
- Zeek.
- Suricata.
- Threat intelligence.

Tool tự động rất hữu ích, nhưng không nên phụ thuộc hoàn toàn vào chúng.

```text
Tool có thể bị bypass.
Human analyst vẫn cần review, validate và đặt câu hỏi đúng.
```

---

# 6. Passive Capture vs Active Capture

Thu thập traffic có thể theo hai hướng:

```text
Passive Capture
Active / In-line Capture
```

---

## 6.1. Passive Capture

Passive capture nghĩa là chỉ sao chép traffic để quan sát, không tương tác trực tiếp với packet.

Ví dụ:

- SPAN port.
- Port mirroring.
- Capture trên cùng VLAN.
- Passive network tap.
- Sensor chỉ nhận bản copy traffic.

Ưu điểm:

- Ít ảnh hưởng production traffic.
- An toàn hơn.
- Phù hợp monitoring.

Hạn chế:

- Cần đúng visibility.
- Nếu switch/VLAN không mirror traffic thì không thấy được.
- Wireless passive capture phức tạp hơn.

---

## 6.2. Active / In-line Capture

Active capture hoặc in-line capture nghĩa là đặt thiết bị vào đường đi traffic.

Ví dụ:

- Network tap in-line.
- Host có nhiều NIC đứng giữa link.
- Sensor nằm trực tiếp trong path.

Ưu điểm:

- Visibility tốt trên link được đặt tap.
- Có thể thấy routed traffic giữa các segment.

Hạn chế:

- Có thể cần thay đổi topology.
- Cần phần cứng tốt.
- Nếu cấu hình sai có thể ảnh hưởng traffic.
- Nhu cầu storage/processing lớn hơn.

---

# 7. Các phụ thuộc khi capture traffic

| Dependency | Passive | Active | Ý nghĩa |
|---|---|---|---|
| Permission | Có | Có | Cần quyền hợp lệ trước khi capture |
| Mirrored Port | Có | Không bắt buộc | Copy traffic từ switch/router |
| Capture Tool | Có | Có | tcpdump/Wireshark/Zeek/Suricata... |
| In-line Placement | Không | Có | Đặt thiết bị vào đường đi traffic |
| Network Tap / Multi-NIC Host | Không | Có | Cho traffic đi qua và copy để phân tích |
| Storage/Processing Power | Có | Có | PCAP lớn cần CPU/RAM/disk tốt |
| Baseline | Rất nên có | Rất nên có | Giúp lọc normal traffic và tìm outlier |

---

## 7.1. Permission

Luôn cần quyền rõ ràng trước khi capture traffic.

Traffic có thể chứa dữ liệu nhạy cảm. Trong các ngành như ngân hàng hoặc y tế, capture traffic không đúng quy định có thể vi phạm chính sách hoặc pháp luật.

```text
Always get written permission.
```

---

## 7.2. Mirrored Port / SPAN Port

Mirrored port hoặc SPAN port copy traffic từ port/VLAN/link khác sang port analyst đang cắm sensor.

Trong switched network, switch không gửi toàn bộ traffic đến mọi port. Nếu muốn thấy traffic của VLAN cụ thể, cần cắm vào đúng segment hoặc mirror traffic về port capture.

---

## 7.3. Capture Tool

Cần công cụ để thu và đọc traffic:

- tcpdump.
- Wireshark.
- TShark.
- NetMiner.
- Zeek.
- Suricata.

PCAP có thể rất lớn rất nhanh. Mỗi lần apply filter trong Wireshark có thể tốn CPU/RAM, nên máy phân tích cần đủ storage và processing power.

---

## 7.4. Network Tap hoặc host nhiều NIC

Active capture thường cần network tap hoặc host có nhiều NIC để traffic vẫn đi qua bình thường trong khi analyst có thể copy/inspect data.

Vị trí tốt thường là:

```text
Layer 3 link giữa các network segment.
```

Vì tại đây có thể thấy routed traffic đi ra ngoài local network.

---

# 8. Vì sao baseline giúp phân tích nhanh hơn?

Nếu có phân đoạn mạng bị lỗi, một số host báo latency cao và xuất hiện file lạ, analyst có thể capture traffic để điều tra.

Nếu không có baseline:

- Không biết host nào bình thường.
- Không biết port nào thường dùng.
- Không biết traffic nào hợp lệ.
- Phải kiểm tra từng conversation.
- Phân tích chậm và dễ bỏ sót.

Nếu có baseline:

- Loại nhanh traffic đã biết.
- Tập trung vào top talkers bất thường.
- So sánh host với hành vi bình thường của nó.
- Nhìn ra outlier sớm hơn.

---

# 9. Ví dụ phân tích từ baseline

Nếu thấy:

```text
User PC A nói chuyện với User PC B qua port 8080 và 445
```

Port 8080 và 445 không nhất thiết độc hại.

Nhưng hai máy user nói chuyện trực tiếp với nhau qua các port này có thể bất thường vì:

- Web traffic thường từ client đến web server.
- SMB traffic thường từ client đến file server/domain resource.
- User-to-user SMB có thể gợi ý lateral movement.
- Port 8080 có thể là staging server, proxy hoặc tool transfer.

Kết luận ban đầu:

```text
Có thể có compromise hoặc unauthorized peer-to-peer communication.
```

---

# 10. NTA workflow thực tế

Một workflow có thể lặp lại:

```text
1. Xác định vấn đề
2. Xác định vị trí capture
3. Xin permission
4. Capture traffic
5. Lọc noise
6. So sánh với baseline
7. Xác định outlier
8. Correlate với log khác
9. Tạo finding/incident ticket
10. Fix và monitor tiếp
```

---

# 11. Câu hỏi analyst nên đặt

Khi xem traffic, hỏi:

```text
Traffic này có thuộc baseline không?
Host này có nên nói chuyện với host kia không?
Port này có bình thường không?
Protocol này có đúng với port không?
Có data transfer lớn bất thường không?
Có user-to-user SMB/RDP/SSH không?
Có beacon interval đều không?
Có top talker nào tăng đột biến không?
Có dấu hiệu scan hoặc lateral movement không?
```

---

# 12. Key Takeaway

NTA không chỉ là capture packet. Giá trị lớn nhất của NTA là **visibility**.

```text
Visibility + baseline + filtering + human analysis = phát hiện bất thường nhanh hơn.
```

Tool tự động như IDS/IPS, SIEM hay Zeek rất hữu ích, nhưng analyst vẫn cần kiểm tra thủ công, hiểu network bình thường và biết cách đặt câu hỏi đúng.

Càng hiểu traffic bình thường trong network, càng dễ phát hiện traffic bất thường trước khi thiệt hại lan rộng.

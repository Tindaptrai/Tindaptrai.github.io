---
layout: post
title: "Tcpdump Basic Lab"
date: 2026-06-26 23:10:40 +0700
categories: [SOC, Network Traffic Analysis]
tags: [soc, network-traffic-analysis, tcpdump, pcap, packet-capture, basic-lab, wireshark, blue-team]
description: "Tóm tắt phòng thí nghiệm cơ bản về tcpdump: kiểm tra cài đặt, liệt kê interface, bắt gói, dùng switch cơ bản, lưu PCAP và đọc lại PCAP."
toc: true
---

# Tcpdump Basic Lab

Bài lab này giúp làm quen với **tcpdump** trong terminal.

Mục tiêu chính là thực hành các thao tác cơ bản:

```text
Kiểm tra tcpdump
→ liệt kê interface
→ bắt gói
→ dùng switch cơ bản
→ lưu traffic thành PCAP
→ đọc lại PCAP
```

Tcpdump thường được dùng để kiểm tra host, server hoặc network segment cụ thể nhằm hiểu chúng đang giao tiếp với ai, qua port/protocol nào và có dấu hiệu bất thường hay không.

---

## Timeline cập nhật

```text
Created at: 2026-06-26 23:10:40 +0700
Timezone: Asia/Ho_Chi_Minh
Topic: Tcpdump Basic Lab
Focus: Validate tcpdump, capture traffic, save/read PCAP, basic switches
```

---

# 1. Mục đích của lab

Phòng thí nghiệm này giúp chúng ta:

- Làm quen với terminal.
- Xác định tcpdump đã được cài chưa.
- Liệt kê network interfaces.
- Bắt traffic từ interface.
- Dùng các switch cơ bản.
- Ghi traffic vào file `.pcap`.
- Đọc lại file `.pcap`.
- Quan sát traffic trong broadcast domain local.
- Chuẩn bị nền tảng cho phân tích traffic sâu hơn.

---

# 2. Bối cảnh lab

Trong vai trò network administrator mới, chúng ta được giao nhiệm vụ capture một phần traffic để hỗ trợ baseline và validate network.

Yêu cầu:

```text
1. Đảm bảo tcpdump đã được cài.
2. Xác định interface có thể capture.
3. Capture một lượng traffic nhỏ.
4. Dùng switch để thay đổi output.
5. Lưu capture vào file PCAP.
6. Đọc lại PCAP bằng tcpdump.
```

---

# 3. Vì sao lab này quan trọng?

Tcpdump có thể được dùng để:

- Theo dõi communication của một host.
- Ghi lại packet gửi/nhận để phân tích.
- Tìm backdoor hoặc communication bất thường.
- Capture evidence trong incident response.
- Kiểm tra DNS/HTTP/FTP/SMB traffic.
- Tạo PCAP để mở bằng Wireshark sau đó.
- Xác thực traffic có đi qua interface hay không.

Nói ngắn gọn:

```text
tcpdump giúp analyst nhìn thấy traffic thật đi qua host hoặc network segment.
```

---

# 4. Task 1: Validate tcpdump is installed

Trước khi bắt đầu, kiểm tra tcpdump đã có trên máy chưa.

## Command

```bash
which tcpdump
```

## Expected output

Nếu tcpdump đã cài, thường thấy:

```text
/usr/sbin/tcpdump
```

Nếu không có output, cần cài:

```bash
sudo apt install tcpdump
```

---

# 5. Task 2: Start a capture

Trước khi capture, cần biết interface nào có thể dùng.

## Step 1: Liệt kê interface

```bash
tcpdump -D
```

Hoặc nếu cần quyền:

```bash
sudo tcpdump -D
```

Output có thể có:

```text
1.eth0
2.any
3.lo
```

## Step 2: Bắt đầu capture

```bash
tcpdump -i [interface name or #]
```

Ví dụ:

```bash
sudo tcpdump -i eth0
```

Hoặc dùng số interface:

```bash
sudo tcpdump -i 1
```

---

# 6. Task 3: Utilize Basic Capture Filters

Sau khi capture được traffic, ta có thể thay đổi cách output hiển thị.

Yêu cầu trong lab:

```text
Thêm verbosity
Hiển thị nội dung dạng ASCII và Hex
```

## Command

```bash
tcpdump -i [interface name or #] -vX
```

Ví dụ:

```bash
sudo tcpdump -i eth0 -vX
```

## Ý nghĩa switch

| Switch | Ý nghĩa |
|---|---|
| `-v` | Tăng độ chi tiết output |
| `-X` | Hiển thị packet ở dạng hex và ASCII |

---

## 6.1. Thử thêm switch khác

Tắt name resolution:

```bash
sudo tcpdump -i eth0 -nn
```

Hiển thị Ethernet header:

```bash
sudo tcpdump -i eth0 -e
```

Hiển thị sequence number tuyệt đối:

```bash
sudo tcpdump -i eth0 -S
```

Kết hợp nhiều switch:

```bash
sudo tcpdump -i eth0 -nnvX
```

---

# 7. Task 4: Save a Capture to a PCAP file

Để lưu traffic vào file `.pcap`, dùng switch `-w`.

## Command

```bash
tcpdump -i [interface name or #] -nvw [/path/of/filename.pcap]
```

Ví dụ:

```bash
sudo tcpdump -i eth0 -nvw ~/baseline.pcap
```

## Ý nghĩa switch

| Switch | Ý nghĩa |
|---|---|
| `-i eth0` | Capture trên interface eth0 |
| `-n` | Không resolve hostname/port |
| `-v` | Verbose output |
| `-w` | Write output vào file PCAP |

Khi dùng `-w`, tcpdump sẽ ghi trực tiếp vào file và thường không in packet ra terminal.

---

# 8. Task 5: Read the Capture from a PCAP file

Sau khi có file PCAP, có thể đọc lại bằng `-r`.

## Command

```bash
tcpdump -nnSXr [file/to/read.pcap]
```

Ví dụ:

```bash
sudo tcpdump -nnSXr ~/baseline.pcap
```

## Ý nghĩa switch

| Switch | Ý nghĩa |
|---|---|
| `-nn` | Không resolve hostname hoặc port |
| `-S` | Hiển thị TCP sequence number tuyệt đối |
| `-X` | Hiển thị hex và ASCII |
| `-r` | Read từ file PCAP |

---

# 9. Full Lab Command Checklist

```bash
which tcpdump
```

```bash
sudo apt install tcpdump
```

```bash
sudo tcpdump -D
```

```bash
sudo tcpdump -i eth0
```

```bash
sudo tcpdump -i eth0 -vX
```

```bash
sudo tcpdump -i eth0 -nn
```

```bash
sudo tcpdump -i eth0 -nvw ~/baseline.pcap
```

```bash
sudo tcpdump -nnSXr ~/baseline.pcap
```

---

# 10. Analyst Notes khi làm lab

Khi capture traffic, nên ghi chú:

- Interface dùng để capture.
- Thời gian bắt đầu/kết thúc.
- Command đã chạy.
- File PCAP lưu ở đâu.
- Host/IP nào xuất hiện nhiều.
- Protocol nào xuất hiện.
- Có traffic DNS/HTTP/HTTPS/SMB/FTP không.
- Có port nào lạ không.
- Có packet clear-text nào đọc được không.

Ví dụ note:

```text
Capture file: ~/baseline.pcap
Interface: eth0
Time: 2026-06-26 23:10 - 23:15
Observed protocols: DNS, HTTPS
Interesting hosts: 172.16.146.1, 172.16.146.2
```

---

# 11. Những lỗi thường gặp

## 11.1. Permission denied

Nếu tcpdump báo thiếu quyền:

```bash
sudo tcpdump -i eth0
```

## 11.2. Không biết interface nào

Dùng:

```bash
sudo tcpdump -D
```

## 11.3. File PCAP quá lớn

Giới hạn số packet:

```bash
sudo tcpdump -i eth0 -c 100 -w sample.pcap
```

Hoặc dùng filter:

```bash
sudo tcpdump -i eth0 host 192.168.1.10 -w host-only.pcap
```

## 11.4. Output bị resolve tên khó đọc

Dùng:

```bash
sudo tcpdump -i eth0 -nn
```

---

# 12. Khi nào dùng tcpdump trong thực tế?

Tcpdump phù hợp khi:

- Cần kiểm tra nhanh traffic trên Linux server.
- Không có GUI/Wireshark.
- Đang SSH vào server từ xa.
- Cần xác minh host có giao tiếp ra ngoài không.
- Cần bắt traffic để gửi team khác phân tích.
- Cần tạo PCAP evidence cho incident.
- Cần kiểm tra DNS, HTTP, FTP, SMB, SSH, RDP.

---

# 13. Câu hỏi ôn tập

## 1. Lệnh nào kiểm tra tcpdump đã cài chưa?

```bash
which tcpdump
```

## 2. Switch nào liệt kê interface capture?

```bash
-D
```

## 3. Switch nào chọn interface?

```bash
-i
```

## 4. Switch nào lưu capture vào PCAP?

```bash
-w
```

## 5. Switch nào đọc PCAP?

```bash
-r
```

## 6. Switch nào tắt hostname/port resolution?

```bash
-nn
```

## 7. Switch nào hiển thị hex và ASCII?

```bash
-X
```

## 8. Lệnh đọc PCAP với sequence number tuyệt đối và hex/ASCII?

```bash
tcpdump -nnSXr file.pcap
```

---

# Key Takeaway

Lab này là bước nền để dùng tcpdump trong Network Traffic Analysis.

```text
Biết kiểm tra tcpdump
→ chọn interface
→ capture traffic
→ dùng switch phù hợp
→ lưu PCAP
→ đọc lại PCAP
```

Khi đã quen với tcpdump basic lab, analyst có thể tiến tới filter nâng cao bằng BPF, phân tích protocol cụ thể và phát hiện traffic bất thường trong incident response hoặc threat hunting.

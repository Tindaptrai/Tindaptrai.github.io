---
layout: post
title: "Tcpdump Fundamentals"
date: 2026-06-26 23:01:15 +0700
categories: [SOC, Network Traffic Analysis]
tags: [soc, network-traffic-analysis, tcpdump, pcap, libpcap, bpf, packet-capture, wireshark, blue-team]
description: "Tóm tắt Tcpdump Fundamentals: cách kiểm tra/cài đặt tcpdump, basic capture options, đọc output, dùng switches, lưu PCAP bằng -w và đọc PCAP bằng -r."
toc: true
---

# Tcpdump Fundamentals

**tcpdump** là công cụ command-line packet sniffer dùng để capture và phân tích network traffic trực tiếp từ interface hoặc từ file PCAP.

```text
tcpdump = capture traffic từ interface/file → filter → đọc nhanh packet → lưu/đọc PCAP.
```

---

## Timeline cập nhật

```text
Created at: 2026-06-26 23:01:15 +0700
Timezone: Asia/Ho_Chi_Minh
Topic: Tcpdump Fundamentals
Focus: tcpdump usage, switches, output reading, PCAP input/output
```

---

# 1. Tcpdump là gì?

Tcpdump có thể:

- Capture traffic từ network interface.
- Đọc traffic từ file PCAP.
- Hiển thị packet summary.
- Hiển thị Ethernet header.
- Hiển thị hex/ASCII payload.
- Filter traffic bằng BPF syntax.
- Lưu capture ra file.

Tcpdump dùng thư viện:

```text
pcap / libpcap
```

và thường cần quyền:

```text
root / administrator
```

Trên Linux thường chạy bằng:

```bash
sudo tcpdump
```

---

# 2. Kiểm tra và cài đặt tcpdump

Kiểm tra tcpdump:

```bash
which tcpdump
```

Cài đặt trên Ubuntu/Debian:

```bash
sudo apt install tcpdump
```

Kiểm tra version:

```bash
sudo tcpdump --version
```

---

# 3. Basic Capture Options

| Switch | Ý nghĩa |
|---|---|
| `-D` | Liệt kê interface có thể capture |
| `-i` | Chọn interface để capture |
| `-n` | Không resolve address/port sang tên |
| `-nn` | Không resolve cả hostnames và port names |
| `-e` | Hiển thị Ethernet header |
| `-X` | Hiển thị packet ở hex và ASCII |
| `-XX` | Giống `-X` nhưng thêm Ethernet header |
| `-v`, `-vv`, `-vvv` | Tăng độ chi tiết output |
| `-c` | Capture số packet nhất định rồi dừng |
| `-s` | Định nghĩa snap length |
| `-S` | Hiển thị absolute TCP sequence number |
| `-q` | Giảm bớt protocol information |
| `-r file.pcap` | Đọc từ file PCAP |
| `-w file.pcap` | Ghi traffic vào file PCAP |

---

# 4. Liệt kê interface

```bash
sudo tcpdump -D
```

Ví dụ interface:

```text
eth0
any
lo
bluetooth0
nflog
nfqueue
```

Ý nghĩa:

- `eth0`: interface Ethernet.
- `any`: capture trên mọi interface.
- `lo`: loopback.

---

# 5. Capture từ interface

```bash
sudo tcpdump -i eth0
```

Ví dụ output:

```text
10:58:33.719241 IP 172.16.146.2.55260 > 172.67.1.1.https: Flags [P.], seq ..., ack ..., length 81
```

Một dòng output thường gồm:

- Timestamp.
- Protocol.
- Source IP/port.
- Destination IP/port.
- TCP flags.
- Sequence/Acknowledgement.
- Packet length.

---

# 6. Disable name resolution

```bash
sudo tcpdump -i eth0 -nn
```

`-nn` giúp output rõ và nhanh hơn:

```text
172.16.146.2.55272 > 172.67.1.1.443
```

Thay vì:

```text
172.16.146.2.55272 > 172.67.1.1.https
```

Trong SOC/NTA, `-nn` rất hay dùng vì không phát sinh DNS lookup và dễ copy IP/port để điều tra.

---

# 7. Hiển thị Ethernet Header

```bash
sudo tcpdump -i eth0 -e
```

Switch `-e` hiển thị Layer 2:

- Source MAC.
- Destination MAC.
- Ethertype.
- Frame length.

Ví dụ:

```text
00:0c:29:97:52:65 > 8a:66:5a:11:8d:64, ethertype IPv4
```

---

# 8. Hiển thị Hex và ASCII

```bash
sudo tcpdump -i eth0 -X
```

Switch `-X` giúp thấy payload dạng:

```text
Hex bên trái
ASCII bên phải
```

Hữu ích khi phân tích clear-text protocols như:

- HTTP.
- FTP.
- DNS.
- SMTP.
- Telnet.

---

# 9. Kết hợp switches

Best practice là chain switches:

```bash
sudo tcpdump -i eth0 -nnvXX
```

Ý nghĩa:

| Switch | Tác dụng |
|---|---|
| `-i eth0` | Capture trên interface eth0 |
| `-nn` | Không resolve host/port |
| `-v` | Verbose output |
| `-XX` | Hex/ASCII + Ethernet header |

---

# 10. Đọc tcpdump output

| Field | Ý nghĩa |
|---|---|
| Timestamp | Thời điểm packet được capture |
| Protocol | IP, ARP, TCP, UDP... |
| Source IP.Port | Nguồn gửi packet |
| Destination IP.Port | Đích nhận packet |
| Flags | TCP flags như SYN, ACK, FIN, PSH |
| Seq/Ack | Sequence và acknowledgement numbers |
| Options | TCP options như MSS, window scale, SACK |
| Length | Độ dài payload |
| Notes/Next Header | Thông tin protocol tiếp theo như FTP/HTTP/DNS |

TCP flags thường gặp:

| Flag | Ý nghĩa |
|---|---|
| `S` | SYN |
| `.` | ACK |
| `F` | FIN |
| `P` | PUSH |
| `R` | RST |

---

# 11. Lưu capture ra file PCAP

Dùng `-w`:

```bash
sudo tcpdump -i eth0 -w ~/output.pcap
```

Khi dùng `-w`, tcpdump ghi trực tiếp vào file và không in packet ra terminal.

Có thể giới hạn số packet:

```bash
sudo tcpdump -i eth0 -c 100 -w ~/output.pcap
```

---

# 12. Đọc file PCAP

Dùng `-r`:

```bash
sudo tcpdump -r ~/output.pcap
```

Đọc không resolve names:

```bash
sudo tcpdump -nnr ~/output.pcap
```

Đọc chi tiết:

```bash
sudo tcpdump -nnvXXr ~/output.pcap
```

---

# 13. Filter cơ bản

Capture host cụ thể:

```bash
sudo tcpdump -i eth0 -nn host 192.168.1.10
```

Capture port cụ thể:

```bash
sudo tcpdump -i eth0 -nn port 443
```

Capture DNS:

```bash
sudo tcpdump -i eth0 -nn udp port 53
```

Capture HTTP:

```bash
sudo tcpdump -i eth0 -nn tcp port 80
```

Capture traffic giữa 2 host:

```bash
sudo tcpdump -i eth0 -nn host 192.168.1.10 and host 10.10.10.5
```

---

# 14. SOC/NTA use cases

Tcpdump hữu ích khi:

- SSH vào server và cần capture nhanh.
- Không có GUI/Wireshark.
- Cần capture PCAP để phân tích sau.
- Cần xác minh host có gửi traffic không.
- Cần kiểm tra DNS/HTTP/FTP clear-text.
- Cần kiểm tra TCP flags.
- Cần capture trên Linux sensor.

Ví dụ:

```text
Host bị nghi C2 → tcpdump kiểm tra destination IP/port.
DNS lỗi → tcpdump udp port 53.
HTTP nghi tải malware → tcpdump tcp port 80 -X.
SMB bất thường → tcpdump tcp port 445.
```

---

# 15. Cẩn thận khi dùng tcpdump

Lưu ý:

- Cần quyền phù hợp.
- Không capture dữ liệu nhạy cảm khi chưa được phép.
- PCAP có thể chứa credential/token/cookie.
- File PCAP lớn rất nhanh.
- Nên giới hạn scope bằng filter.
- Nên ghi chú time window capture.
- Nên bảo quản PCAP như evidence nếu liên quan incident.

---

# 16. Bảng ôn nhanh

| Lệnh | Mục đích |
|---|---|
| `which tcpdump` | Kiểm tra tcpdump |
| `sudo apt install tcpdump` | Cài tcpdump |
| `sudo tcpdump --version` | Kiểm tra version |
| `sudo tcpdump -D` | List interfaces |
| `sudo tcpdump -i eth0` | Capture eth0 |
| `sudo tcpdump -i eth0 -nn` | Không resolve names |
| `sudo tcpdump -i eth0 -e` | Ethernet header |
| `sudo tcpdump -i eth0 -X` | Hex/ASCII |
| `sudo tcpdump -i eth0 -nnvXX` | Output chi tiết |
| `sudo tcpdump -i eth0 -w output.pcap` | Ghi PCAP |
| `sudo tcpdump -r output.pcap` | Đọc PCAP |

---

# Key Takeaway

Tcpdump là công cụ cực kỳ quan trọng cho Network Traffic Analysis.

```text
Dùng tcpdump tốt = biết chọn interface, dùng switch phù hợp, filter đúng scope, lưu PCAP an toàn và đọc output chính xác.
```

Trong SOC/Blue Team, tcpdump giúp analyst kiểm tra nhanh traffic ngay trên host/sensor, tạo PCAP để điều tra sâu hơn bằng Wireshark hoặc SIEM/NDR pipeline.

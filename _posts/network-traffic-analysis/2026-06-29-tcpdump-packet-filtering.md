---
layout: post
title: "Tcpdump Packet Filtering"
date: 2026-06-29 21:25:12 +0700
categories: [SOC, Network Traffic Analysis]
tags: [soc, network-traffic-analysis, tcpdump, packet-filtering, bpf, pcap, tcp-flags, blue-team]
description: "Tóm tắt Tcpdump Packet Filtering: host/src/dst/net/proto/port/portrange/less/greater/and/or/not filters, capture-time vs post-capture filtering, grep output và săn TCP SYN flag."
toc: true
---

# Tcpdump Packet Filtering

**Tcpdump packet filtering** giúp analyst giảm noise khi capture hoặc đọc PCAP. Thay vì xem toàn bộ traffic, ta có thể lọc theo host, network, protocol, port, kích thước packet hoặc TCP flags.

```text
Packet filtering = chỉ giữ traffic liên quan → dễ đọc hơn, ít tốn disk hơn, phân tích nhanh hơn.
```

---

## Timeline cập nhật

```text
Created at: 2026-06-29 21:25:12 +0700
Timezone: Asia/Ho_Chi_Minh
Topic: Tcpdump Packet Filtering
Focus: host/src/dst/net/proto/port filters, logical operators, size filters, SYN hunting
```

---

# 1. Vì sao cần filter trong tcpdump?

Traffic capture thường rất nhiều noise. Filter giúp:

- Giảm packet hiển thị.
- Giảm dung lượng PCAP.
- Tập trung vào host/protocol/port nghi vấn.
- Phân tích nhanh theo hướng điều tra.
- Tránh bị chìm trong broadcast, multicast hoặc traffic không liên quan.

---

# 2. Các filter hữu ích

| Filter | Ý nghĩa |
|---|---|
| `host` | Lọc traffic liên quan đến một host |
| `src` / `dst` | Chỉ định source hoặc destination |
| `net` | Lọc traffic theo network/subnet |
| `proto` | Lọc theo protocol number |
| `tcp`, `udp`, `icmp` | Lọc theo protocol phổ biến |
| `port` | Lọc theo port, cả source hoặc destination |
| `portrange` | Lọc theo dải port |
| `less` / `greater` | Lọc theo kích thước packet |
| `and` / `&&` | Cả hai điều kiện đều phải đúng |
| `or` / `||` | Một trong hai điều kiện đúng |
| `not` / `!` | Loại trừ điều kiện |

---

# 3. Host filter

```bash
sudo tcpdump -i eth0 host 172.16.146.2
```

Ý nghĩa:

```text
Hiển thị packet có source hoặc destination là 172.16.146.2.
```

Use case:

- Xem host đang nói chuyện với ai.
- Kiểm tra host nghi vấn có kết nối ra ngoài không.
- Triage C2 hoặc malware download.
- Theo dõi server/service cụ thể.

---

# 4. Source và Destination filter

Source host:

```bash
sudo tcpdump -i eth0 src host 172.16.146.2
```

Destination host:

```bash
sudo tcpdump -i eth0 dst host 172.16.146.2
```

Source port:

```bash
sudo tcpdump -i eth0 tcp src port 80
```

Destination port:

```bash
sudo tcpdump -i eth0 tcp dst port 443
```

`src` và `dst` rất hữu ích khi cần hiểu hướng giao tiếp.

---

# 5. Net filter

```bash
sudo tcpdump -i eth0 net 172.16.146.0/24
```

Destination network:

```bash
sudo tcpdump -i eth0 dst net 172.16.146.0/24
```

Use case:

- Xem traffic vào/ra một subnet.
- Điều tra segment nội bộ.
- Kiểm tra traffic routed đến VLAN/subnet cụ thể.

---

# 6. Protocol filter

Dùng tên protocol:

```bash
sudo tcpdump -i eth0 udp
sudo tcpdump -i eth0 tcp
sudo tcpdump -i eth0 icmp
```

Dùng protocol number:

```bash
sudo tcpdump -i eth0 proto 17
```

Protocol number phổ biến:

| Protocol | Number |
|---|---|
| ICMP | 1 |
| TCP | 6 |
| UDP | 17 |

---

# 7. Port filter

`port` là hai chiều, source hoặc destination port khớp đều được hiển thị.

HTTPS:

```bash
sudo tcpdump -i eth0 tcp port 443
```

DNS TCP hoặc UDP:

```bash
sudo tcpdump -i eth0 port 53
```

Chỉ UDP DNS:

```bash
sudo tcpdump -i eth0 udp port 53
```

Chỉ TCP DNS:

```bash
sudo tcpdump -i eth0 tcp port 53
```

---

# 8. Portrange filter

```bash
sudo tcpdump -i eth0 portrange 0-1024
```

Ý nghĩa:

```text
Hiển thị traffic có source hoặc destination port trong dải 0-1024.
```

Use case:

- Xem well-known ports.
- Tìm traffic service phổ biến.
- Kiểm tra port lạ ngoài expected service.
- So sánh traffic với baseline.

---

# 9. Less / Greater filter

Packet nhỏ hơn 64 bytes:

```bash
sudo tcpdump -i eth0 less 64
```

Packet lớn hơn 500 bytes:

```bash
sudo tcpdump -i eth0 greater 500
```

Use case:

- Tìm file transfer.
- Tìm payload lớn.
- Giảm packet nhỏ không liên quan.
- Tập trung vào data transfer.

---

# 10. AND / OR / NOT filters

AND:

```bash
sudo tcpdump -i eth0 host 192.168.0.1 and port 23
```

OR:

```bash
sudo tcpdump -r sus.pcap icmp or host 172.16.146.1
```

NOT:

```bash
sudo tcpdump -r sus.pcap not icmp
```

Ví dụ loại DNS noise:

```bash
sudo tcpdump -i eth0 not port 53
```

---

# 11. Capture-time filter vs Post-capture filter

## Filter lúc capture

```bash
sudo tcpdump -i eth0 host 192.168.1.10 -w host.pcap
```

Ưu điểm:

- File nhỏ hơn.
- Ít tốn disk.
- Tập trung đúng mục tiêu.

Nhược điểm:

```text
Packet không khớp filter sẽ không được lưu, có thể mất evidence cần thiết.
```

## Filter khi đọc PCAP

```bash
sudo tcpdump -r full.pcap host 192.168.1.10
```

Ưu điểm:

- Không làm thay đổi PCAP gốc.
- Có thể thử nhiều filter khác nhau.
- Giữ dữ liệu đầy đủ cho điều tra sau.

---

# 12. Mẹo dùng tcpdump

## Absolute sequence number với `-S`

```bash
sudo tcpdump -S -r file.pcap
```

`-S` hữu ích khi cần correlate sequence number với tool hoặc log khác.

## ASCII output với `-A`

```bash
sudo tcpdump -Ar telnet.pcap
```

Hữu ích với clear-text protocol như Telnet, HTTP, FTP, SMTP.

## Pipe sang grep

```bash
sudo tcpdump -Ar http.cap -l | grep 'mailto:*'
```

Use case:

- Tìm email address.
- Tìm URL.
- Tìm keyword.
- Tìm User-Agent.
- Tìm string trong clear-text HTTP.

---

# 13. Săn TCP SYN flag

```bash
sudo tcpdump -i eth0 'tcp[13] & 2 != 0'
```

Ý nghĩa:

```text
tcp[13] = byte chứa TCP flags.
& 2 != 0 = kiểm tra bit SYN có bật hay không.
```

Use case:

- Tìm TCP connection attempts.
- Điều tra port scan.
- Theo dõi SYN flood.
- Xem host nào đang mở kết nối TCP mới.

---

# 14. Quick SOC examples

| Mục tiêu | Command |
|---|---|
| Host nghi vấn | `sudo tcpdump -i eth0 -nn host 192.168.1.50` |
| DNS traffic | `sudo tcpdump -i eth0 -nn port 53` |
| HTTP clear-text | `sudo tcpdump -i eth0 -nnA tcp port 80` |
| SMB traffic | `sudo tcpdump -i eth0 -nn tcp port 445` |
| Loại ICMP noise | `sudo tcpdump -i eth0 -nn not icmp` |
| Packet lớn | `sudo tcpdump -i eth0 -nn greater 1000` |
| SYN packet | `sudo tcpdump -i eth0 -nn 'tcp[13] & 2 != 0'` |

---

# 15. Bảng ôn nhanh

| Mục tiêu | Command |
|---|---|
| Host cụ thể | `tcpdump host 172.16.146.2` |
| Source host | `tcpdump src host 172.16.146.2` |
| Destination host | `tcpdump dst host 172.16.146.2` |
| Network | `tcpdump net 172.16.146.0/24` |
| Destination network | `tcpdump dst net 172.16.146.0/24` |
| UDP | `tcpdump udp` |
| TCP | `tcpdump tcp` |
| ICMP | `tcpdump icmp` |
| Protocol number | `tcpdump proto 17` |
| Port | `tcpdump port 53` |
| TCP port | `tcpdump tcp port 443` |
| Port range | `tcpdump portrange 0-1024` |
| Packet nhỏ | `tcpdump less 64` |
| Packet lớn | `tcpdump greater 500` |
| AND | `tcpdump host 192.168.0.1 and port 23` |
| OR | `tcpdump icmp or host 172.16.146.1` |
| NOT | `tcpdump not icmp` |
| SYN flag | `tcpdump 'tcp[13] & 2 != 0'` |

---

# Key Takeaway

Tcpdump filter là kỹ năng cực kỳ quan trọng khi làm Network Traffic Analysis.

```text
Filter đúng giúp giảm noise, giảm dung lượng PCAP, tăng tốc điều tra và tập trung vào traffic có ý nghĩa.
```

Khi làm SOC/Blue Team, nên nắm chắc `host`, `src`, `dst`, `net`, `proto`, `port`, `portrange`, `less`, `greater`, `and`, `or`, `not` và cách đọc TCP flags để phát hiện scan, C2, file transfer hoặc traffic bất thường.

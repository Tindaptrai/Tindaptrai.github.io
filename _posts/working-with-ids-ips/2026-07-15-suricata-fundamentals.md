---
layout: post
title: "Suricata Fundamentals"
date: 2026-07-15 15:14:48 +0700
categories: ["Working with IDS/IPS", "Suricata"]
tags: [suricata, ids, ips, nsm, eve-json, af-packet, nfqueue, pcap, file-extraction, protocol-anomalies, detection-engineering, splunk, elk, threat-hunting, incident-response, cdsa]
description: "Suricata fundamentals: IDS/IPS/NSM, PCAP and live input, EVE JSON, custom rules, anomaly detection, file extraction, ruleset updates and validation."
toc: true
---

# Suricata Fundamentals

Suricata là một engine mã nguồn mở cho **IDS**, **IPS** và **Network Security Monitoring**. Công cụ phân tích packet, TCP stream và application-layer transaction để tạo alert, metadata và bằng chứng phục vụ SOC/DFIR.

> Chỉ replay PCAP hoặc thử inline IPS trong HTB, lab hay hệ thống có ủy quyền. Rule `drop` phải được kiểm thử kỹ trước production.

## Timeline cập nhật

```text
Created at: 2026-07-15 15:14:48 +0700
Timezone: Asia/Ho_Chi_Minh
Module: Working with IDS/IPS
Section: Suricata Fundamentals
Focus: IDS/IPS/NSM, EVE JSON, rules, anomalies, file extraction
Documentation baseline: Suricata 8.0.6
HTB target example: Suricata 6.0.13
```

## 1. Suricata trong SOC

```text
SPAN/TAP hoặc inline gateway
        ↓
Suricata
        ↓
eve.json
        ↓
Filebeat/Logstash
        ↓
Splunk hoặc ELK
        ↓
SOC alert, hunting và incident response
```

Suricata có thể tạo metadata cho DNS, HTTP, TLS, SSH, SMB, flow, file và anomaly events.

## 2. Các mô hình vận hành

### IDS

Suricata quan sát traffic và tạo alert nhưng không chặn.

```text
SPAN/TAP → Suricata sensor → alert/log
```

### IPS inline

Traffic phải đi qua Suricata hoặc hàng đợi do Suricata kiểm tra. Rule `drop`/`reject` mới có hiệu lực ngăn chặn trong đúng inline mode.

```text
Internet → firewall/NFQUEUE/AF_PACKET inline → Suricata → LAN
```

### NSM

Suricata ghi flow và application metadata để hunting, retrospective analysis và incident reconstruction.

## 3. Offline và live input

Phân tích PCAP:

```bash
sudo suricata -r /home/htb-student/pcaps/suspicious.pcap
```

Bỏ qua checksum và chọn thư mục log:

```bash
sudo suricata   -r /home/htb-student/pcaps/suspicious.pcap   -k none   -l ./suricata-output
```

Live LibPCAP:

```bash
sudo suricata --pcap=ens160 -vv
```

AF_PACKET:

```bash
sudo suricata --af-packet=ens160
```

NFQUEUE inline trong lab:

```bash
sudo iptables -I FORWARD -j NFQUEUE --queue-num 0
sudo suricata -q 0
```

Xóa rule sau lab:

```bash
sudo iptables -D FORWARD -j NFQUEUE --queue-num 0
```

## 4. Replay traffic trong lab

Terminal 1:

```bash
sudo suricata --af-packet=ens160
```

Terminal 2:

```bash
sudo tcpreplay   -i ens160   /home/htb-student/pcaps/suspicious.pcap
```

> Không replay PCAP độc hại vào interface nối production hoặc Internet.

## 5. HOME_NET

Trong `/etc/suricata/suricata.yaml`:

```yaml
vars:
  address-groups:
    HOME_NET: "[192.168.10.0/24,192.168.20.0/24,192.168.30.0/24]"
    EXTERNAL_NET: "!$HOME_NET"

    HTTP_SERVERS: "$HOME_NET"
    DNS_SERVERS: "$HOME_NET"
    DC_SERVERS: "$HOME_NET"
```

`HOME_NET` sai có thể khiến rule match sai hướng, tăng false positive hoặc bỏ sót attack.

## 6. Custom rule cơ bản

```suricata
alert dns $HOME_NET any -> $EXTERNAL_NET any (
  msg:"LAB Suspicious DNS query";
  dns.query;
  content:"example-bad-domain.test";
  nocase;
  sid:1000001;
  rev:1;
)
```

Cấu trúc:

```text
Action Protocol Source Direction Destination (Options)
```

Các action chính:

| Action | Ý nghĩa |
|---|---|
| `alert` | Tạo cảnh báo |
| `drop` | Chặn trong IPS |
| `reject` | Chặn và gửi reset/error |
| `pass` | Bỏ qua traffic khớp rule |

Thêm custom rule:

```bash
sudo nano /etc/suricata/rules/local.rules
```

Trong YAML:

```yaml
rule-files:
  - suricata.rules
  - local.rules
```

Kiểm tra:

```bash
sudo suricata -T -c /etc/suricata/suricata.yaml
```

## 7. EVE JSON

`eve.json` là output JSON chính. Event type thường gồm:

```text
alert, anomaly, flow, dns, http, tls, ssh, smb, fileinfo, drop, stats
```

Lọc alert:

```bash
jq -c   'select(.event_type == "alert")'   /var/log/suricata/eve.json
```

DNS đầu tiên:

```bash
jq -c   'select(.event_type == "dns")'   /var/log/suricata/eve.json |
head -1 |
jq .
```

Top signatures:

```bash
jq -r   'select(.event_type == "alert") | .alert.signature'   /var/log/suricata/eve.json |
sort | uniq -c | sort -nr
```

### flow_id

`flow_id` giúp correlate alert, DNS/HTTP/TLS transaction, flow và fileinfo thuộc cùng network flow.

```bash
jq -c   'select(.flow_id == 1959965318909019)'   /var/log/suricata/eve.json
```

### pcap_cnt

`pcap_cnt` cho biết thứ tự packet Suricata đã xử lý, hỗ trợ quay lại đúng packet và xây timeline.

## 8. fast.log và stats.log

`fast.log` chứa alert dạng text, hữu ích cho triage nhanh.

`stats.log` phản ánh sensor health. Counter đáng chú ý:

```text
capture.kernel_drops
decoder.invalid
tcp.reassembly_gap
detect.alert
detect.alert_queue_overflow
flow.memuse
```

Packet drop hoặc reassembly gap cao có thể tạo blind spot.

## 9. Protocol anomaly detection

Suricata nhận diện application protocol độc lập với port. Đây là cách phát hiện:

- HTTP chạy ngoài 80/8080.
- Port 80 nhưng payload không phải HTTP.
- SSH chạy ngoài 22.
- UDP/53 nhưng traffic không phải DNS.
- HTTP cleartext trên 443.

Ví dụ:

```suricata
alert tcp any any -> any ![80,8080] (
  msg:"LAB HTTP on unusual port";
  flow:to_server;
  app-layer-protocol:http;
  sid:1000101;
  rev:1;
)
```

```suricata
alert tcp any any -> any 80 (
  msg:"LAB Port 80 traffic is not HTTP";
  flow:to_server;
  app-layer-protocol:!http;
  sid:1000102;
  rev:1;
)
```

```suricata
alert tcp any any -> any !22 (
  msg:"LAB SSH on unusual port";
  flow:to_server;
  app-layer-protocol:ssh;
  sid:1000103;
  rev:1;
)
```

```suricata
alert udp any any -> any 53 (
  msg:"LAB UDP 53 traffic is not DNS";
  flow:to_server;
  app-layer-protocol:!dns;
  sid:1000104;
  rev:1;
)
```

Luôn kiểm tra rule bằng `suricata -T` và PCAP đại diện.

## 10. File extraction

Suricata có thể trích xuất file từ các parser được hỗ trợ như HTTP, SMTP, FTP, NFS, SMB và HTTP/2.

Cấu hình:

```yaml
file-store:
  version: 2
  enabled: yes
  force-filestore: yes
```

EVE file metadata:

```yaml
outputs:
  - eve-log:
      types:
        - files:
            force-magic: no
            force-hash: [md5, sha256]
```

Rule lab:

```suricata
alert http any any -> any any (
  msg:"LAB Store HTTP files";
  filestore;
  sid:1000201;
  rev:1;
)
```

File được lưu theo SHA-256:

```text
filestore/f9/f9bc6d...
```

Kiểm tra an toàn:

```bash
file filestore/21/<SHA256>
xxd filestore/21/<SHA256> | head
sha256sum filestore/21/<SHA256>
strings -a filestore/21/<SHA256> | head
```

Không chạy file trực tiếp; chuyển sang malware-analysis VM/sandbox cô lập.

## 11. Live rule reload và update

Test trước:

```bash
sudo suricata -T -c /etc/suricata/suricata.yaml
```

Reload bằng signal khi deployment hỗ trợ:

```bash
sudo kill -USR2 "$(pidof suricata)"
```

Cập nhật rules:

```bash
sudo suricata-update
sudo suricata-update list-sources
sudo suricata-update enable-source et/open
sudo suricata-update
```

Sau đó test và reload/restart theo change management:

```bash
sudo suricata -T -c /etc/suricata/suricata.yaml
sudo systemctl restart suricata
```

## 12. Quy trình rule deployment

```text
1. Viết local rule.
2. Kiểm tra SID không trùng.
3. Chạy suricata -T.
4. Test malicious PCAP.
5. Test benign PCAP.
6. Đánh giá false positive.
7. Đưa vào IDS alert-only.
8. Theo dõi và tune.
9. Chỉ chuyển drop khi đủ tin cậy.
10. Chuẩn bị rollback.
```

## 13. SIEM correlation

Splunk:

```spl
index=suricata event_type=alert
| stats count
        values(alert.signature) as signatures
        values(dest_ip) as destinations
  by src_ip
| sort -count
```

Theo `flow_id`:

```spl
index=suricata flow_id=1959965318909019
| sort _time
| table _time event_type src_ip src_port dest_ip dest_port app_proto alert.signature
```

Elastic/KQL:

```kql
event_type: "alert"
```

```kql
event_type: "fileinfo"
```

```kql
alert.signature: "*unusual port*"
```

## 14. Mapping với CDSA

### Security Operations & Monitoring

- EVE ingestion.
- Alert triage.
- Sensor-health monitoring.
- Protocol anomaly detection.

### Incident Response & Forensics

- PCAP replay.
- `flow_id` và `pcap_cnt`.
- File extraction.
- Timeline reconstruction.

### Malware Analysis & Reverse Engineering

- Trích xuất PE/script.
- Hashing và YARA.
- Static/dynamic analysis.
- C2 protocol identification.

### Threat Hunting

- Non-standard protocols.
- Rare DNS/TLS/SSH destinations.
- File downloads.
- Multi-event flow correlation.

### Detection Engineering

- Local rules.
- Suricata-update.
- Offline regression tests.
- IDS-to-IPS promotion.

## 15. Lab checklist

```bash
sudo suricata -T -c /etc/suricata/suricata.yaml

mkdir -p ~/suricata-lab-output

sudo suricata   -r ~/pcaps/suspicious.pcap   -k none   -l ~/suricata-lab-output

jq -c   'select(.event_type == "alert")'   ~/suricata-lab-output/eve.json

less ~/suricata-lab-output/stats.log

find ~/suricata-lab-output/filestore -type f 2>/dev/null
```

## Lưu ý phiên bản

Lab HTB dùng Suricata `6.0.13`, trong khi tài liệu tham chiếu hiện tại là `8.0.6`.

```bash
suricata --version
suricata --build-info
suricata-update --version
```

Luôn kiểm tra lại configuration key, rule keyword, output schema và IPS setup theo đúng phiên bản đang triển khai.

## Key Takeaway

```text
Suricata không chỉ alert theo signature.
Nó còn cung cấp application metadata, anomaly detection,
flow correlation, file extraction và NSM telemetry.

Một deployment tốt cần:
HOME_NET chính xác
+ input phù hợp
+ EVE đầy đủ
+ rule testing
+ sensor-health monitoring
+ SIEM/EDR correlation.
```

## Tài liệu tham khảo

- [Suricata 8.0.6 User Guide](https://docs.suricata.io/en/suricata-8.0.6/)
- [Protocol Anomalies Detection](https://redmine.openinfosecfoundation.org/projects/suricata/wiki/Protocol_Anomalies_Detection)
- [EVE JSON](https://docs.suricata.io/en/suricata-8.0.6/output/eve/index.html)
- [File Extraction](https://docs.suricata.io/en/suricata-8.0.6/file-extraction/file-extraction.html)
- [Suricata-Update](https://docs.suricata.io/en/suricata-8.0.6/rule-management/suricata-update.html)
- [Rule Reloads](https://docs.suricata.io/en/suricata-8.0.6/rule-management/rule-reload.html)
- [AF_PACKET](https://docs.suricata.io/en/suricata-8.0.6/capture-hardware/af-packet.html)
- [Linux Inline IPS](https://docs.suricata.io/en/suricata-8.0.6/ips/setting-up-ipsinline-for-linux.html)

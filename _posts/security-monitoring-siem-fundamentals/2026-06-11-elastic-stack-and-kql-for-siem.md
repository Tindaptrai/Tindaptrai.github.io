---
layout: post
title: "Elastic Stack và Kibana Query Language trong SIEM"
date: 2026-06-11 05:00:00 +0700
categories: [SOC, SIEM]
tags: [soc, siem, elastic-stack, elasticsearch, logstash, kibana, beats, kql, ecs, blue-team]
description: "Ghi chú tổng hợp về Elastic Stack, các thành phần Elasticsearch, Logstash, Kibana, Beats, cách dùng Elastic như SIEM và các truy vấn KQL cơ bản cho SOC analyst."
toc: true
---

# Elastic Stack và Kibana Query Language trong SIEM

**Elastic Stack** là một bộ công cụ rất phổ biến trong SOC để thu thập, lưu trữ, tìm kiếm, phân tích và trực quan hóa log.

Trong môi trường Blue Team/SOC, Elastic Stack có thể được dùng như một giải pháp **SIEM** để phân tích dữ liệu bảo mật từ nhiều nguồn như firewall, IDS/IPS, endpoint, server và Windows Event Logs.

---

## 1. Elastic Stack là gì?

**Elastic Stack**, do Elastic phát triển, là một bộ công cụ mã nguồn mở chủ yếu gồm:

- **Elasticsearch**
- **Logstash**
- **Kibana**
- **Beats**

Trước đây, bộ này thường được gọi là **ELK Stack**:

```text
E = Elasticsearch
L = Logstash
K = Kibana
```

Sau này Elastic bổ sung **Beats**, nên thường gọi là **Elastic Stack**.

---

## 2. Mục tiêu của Elastic Stack

Elastic Stack giúp người dùng:

- Thu thập log từ nhiều nguồn.
- Chuẩn hóa và xử lý log.
- Lưu trữ dữ liệu.
- Tìm kiếm dữ liệu rất nhanh.
- Trực quan hóa dữ liệu bằng dashboard.
- Phân tích sự kiện theo thời gian thực.
- Hỗ trợ SOC analyst điều tra security event.

Nói ngắn gọn:

```text
Elastic Stack = Collect → Process → Store → Search → Visualize
```

---

# 3. Các thành phần chính của Elastic Stack

Elastic Stack thường gồm các thành phần sau:

```text
Beats → Logstash → Elasticsearch → Kibana
```

Hoặc trong mô hình đơn giản hơn:

```text
Beats → Elasticsearch → Kibana
```

---

## 3.1. Elasticsearch

**Elasticsearch** là công cụ tìm kiếm và phân tích dữ liệu phân tán, dựa trên JSON và có RESTful API.

Trong Elastic Stack, Elasticsearch là thành phần cốt lõi dùng để:

- Lưu trữ log.
- Index dữ liệu.
- Truy vấn dữ liệu.
- Tìm kiếm dữ liệu nhanh.
- Phân tích dữ liệu.
- Hỗ trợ correlation và detection.

Elasticsearch giống như “kho dữ liệu tìm kiếm nhanh” của Elastic Stack.

Ví dụ:

```text
Log từ firewall
      ↓
Được xử lý
      ↓
Được lưu vào Elasticsearch
      ↓
SOC analyst tìm kiếm qua Kibana
```

---

## 3.2. Logstash

**Logstash** là công cụ dùng để thu thập, xử lý, chuyển đổi và gửi log.

Logstash hoạt động chủ yếu qua 3 giai đoạn:

```text
Input → Filter → Output
```

### Input

Logstash nhận dữ liệu từ nhiều nguồn như:

- File.
- TCP socket.
- Syslog.
- Beats.
- Message queue.
- Application logs.

### Filter

Logstash xử lý và làm giàu dữ liệu.

Ví dụ:

- Parse log.
- Tách field.
- Chuẩn hóa timestamp.
- Thêm metadata.
- GeoIP enrichment.
- Đổi format dữ liệu.
- Loại bỏ field không cần thiết.

### Output

Sau khi xử lý xong, Logstash gửi dữ liệu đến đích.

Đích phổ biến nhất là:

```text
Elasticsearch
```

---

## 3.3. Kibana

**Kibana** là giao diện trực quan hóa và phân tích dữ liệu cho Elasticsearch.

Kibana giúp SOC analyst:

- Tìm kiếm log.
- Viết truy vấn KQL.
- Xem bảng event.
- Tạo dashboard.
- Tạo visualization.
- Phân tích timeline.
- Điều tra alert.
- Quan sát security event.

Trong SOC dùng Elastic Stack, Kibana thường là giao diện chính mà analyst làm việc hằng ngày.

---

## 3.4. Beats

**Beats** là các data shipper nhẹ, được cài trên máy endpoint/server để gửi dữ liệu về Logstash hoặc Elasticsearch.

Beats giúp đơn giản hóa việc thu thập dữ liệu từ nhiều nguồn.

Một số Beats phổ biến:

| Beat | Mục đích |
|---|---|
| Filebeat | Thu thập log file |
| Winlogbeat | Thu thập Windows Event Logs |
| Metricbeat | Thu thập metrics hệ thống |
| Packetbeat | Thu thập network traffic metadata |
| Auditbeat | Thu thập audit data |
| Heartbeat | Kiểm tra uptime/service availability |

Ví dụ:

```text
Winlogbeat trên Windows Server
      ↓
Gửi Windows Event Logs
      ↓
Logstash hoặc Elasticsearch
      ↓
Kibana để phân tích
```

---

# 4. Các mô hình triển khai Elastic Stack

## 4.1. Mô hình đầy đủ

```text
Beats → Logstash → Elasticsearch → Kibana
```

Mô hình này phù hợp khi cần:

- Xử lý log phức tạp.
- Parse nhiều loại log.
- Enrichment dữ liệu.
- Filter dữ liệu.
- Kiểm soát pipeline tốt hơn.

---

## 4.2. Mô hình đơn giản

```text
Beats → Elasticsearch → Kibana
```

Mô hình này bỏ qua Logstash.

Phù hợp khi:

- Dữ liệu đã được Beats parse tốt.
- Không cần filter phức tạp.
- Muốn giảm độ phức tạp hệ thống.
- Môi trường nhỏ hoặc lab.

---

## 4.3. Mô hình có Message Queue

Trong môi trường lớn, có thể thêm:

- Kafka.
- RabbitMQ.
- Redis.

Mục tiêu:

- Buffer dữ liệu.
- Tăng khả năng chịu lỗi.
- Tránh mất log khi Elasticsearch/Logstash quá tải.
- Tách rời collector và processing pipeline.

Ví dụ:

```text
Beats → Kafka/Redis → Logstash → Elasticsearch → Kibana
```

---

# 5. Elastic Stack như một giải pháp SIEM

Elastic Stack có thể được dùng như một giải pháp **SIEM**.

Nó có thể thu thập, lưu trữ, phân tích và trực quan hóa dữ liệu bảo mật từ nhiều nguồn.

Nguồn dữ liệu có thể gồm:

- Firewall.
- IDS/IPS.
- Endpoint.
- Windows Event Logs.
- Linux logs.
- Web server logs.
- DNS logs.
- Proxy logs.
- VPN logs.
- Cloud logs.
- Authentication logs.

---

## 5.1. Vai trò của Elasticsearch trong SIEM

Elasticsearch dùng để:

- Lưu trữ security logs.
- Index dữ liệu.
- Tìm kiếm nhanh.
- Thực hiện query.
- Hỗ trợ correlation.
- Hỗ trợ detection.

---

## 5.2. Vai trò của Kibana trong SIEM

Kibana dùng để:

- Tìm kiếm event.
- Điều tra incident.
- Xem dashboard.
- Tạo visualization.
- Phân tích timeline.
- Dùng KQL để lọc log.
- Hiểu security event.

Với SOC analyst, Kibana là giao diện chính khi làm việc với Elastic Stack.

---

## 5.3. Vai trò của Logstash trong SIEM

Logstash dùng để:

- Nhận log từ nhiều nguồn.
- Parse log.
- Normalize field.
- Enrich dữ liệu.
- Gửi dữ liệu vào Elasticsearch.

Ví dụ:

```text
Firewall log thô
      ↓
Logstash parse source IP, destination IP, action
      ↓
Elasticsearch lưu dưới field chuẩn
      ↓
Kibana hiển thị và truy vấn
```

---

# 6. Kibana Query Language là gì?

**KQL** là viết tắt của:

```text
Kibana Query Language
```

KQL là ngôn ngữ truy vấn thân thiện dùng trong Kibana để tìm kiếm và lọc dữ liệu trong Elasticsearch.

KQL đơn giản hơn Elasticsearch Query DSL và rất hữu ích cho SOC analyst khi điều tra log.

---

## 7. Cấu trúc cơ bản của KQL

Cấu trúc cơ bản:

```text
field:value
```

Ví dụ:

```text
event.code:4625
```

Ý nghĩa:

```text
Tìm các event có mã event.code là 4625.
```

Trong Windows, event ID 4625 thường liên quan đến **failed logon**.

---

## 8. Ví dụ KQL: Failed Logon

```text
event.code:4625
```

Truy vấn này giúp tìm các lần đăng nhập thất bại trên Windows.

SOC analyst có thể dùng để điều tra:

- Brute force.
- Password spraying.
- Credential guessing.
- Tài khoản bị thử mật khẩu.
- Đăng nhập thất bại bất thường.

---

## 9. Free Text Search

KQL hỗ trợ tìm kiếm văn bản tự do.

Ví dụ:

```text
"svc-sql1"
```

Truy vấn này tìm chuỗi `svc-sql1` trong bất kỳ field nào được index.

Cách này hữu ích khi chưa biết chính xác field name.

Ví dụ dùng khi tìm:

- Hostname.
- Username.
- Service account.
- File name.
- Domain.
- IP.
- Error message.

---

## 10. Logical Operators

KQL hỗ trợ các toán tử logic:

- `AND`
- `OR`
- `NOT`

Ví dụ:

```text
event.code:4625 AND winlog.event_data.SubStatus:0xC0000072
```

Ý nghĩa:

```text
Tìm failed logon event 4625
và SubStatus là 0xC0000072.
```

Trong Windows, SubStatus `0xC0000072` cho biết tài khoản đang bị disabled.

Hành vi này cần được điều tra vì attacker có thể đang thử dùng credential của tài khoản đã bị vô hiệu hóa.

---

## 11. Comparison Operators

KQL hỗ trợ toán tử so sánh như:

- `>`
- `>=`
- `<`
- `<=`
- `:`

Ví dụ:

```text
event.code:4625 AND winlog.event_data.SubStatus:0xC0000072 AND @timestamp >= "2023-03-03T00:00:00.000Z" AND @timestamp <= "2023-03-06T23:59:59.999Z"
```

Ý nghĩa:

```text
Tìm failed logon của disabled accounts
trong khoảng thời gian từ 03/03/2023 đến 06/03/2023.
```

---

## 12. Wildcards

KQL hỗ trợ wildcard để tìm theo mẫu.

Ví dụ:

```text
event.code:4625 AND user.name: admin*
```

Ý nghĩa:

```text
Tìm failed logon event
với username bắt đầu bằng admin.
```

Ví dụ match:

- admin
- administrator
- admin123
- admin_backup

Truy vấn này hữu ích khi muốn phát hiện các nỗ lực đăng nhập nhắm vào tài khoản quản trị.

---

# 13. Cách xác định field và value có sẵn

Khi dùng Kibana, đôi khi analyst chưa biết field nào tồn tại trong dữ liệu.

Có hai cách phổ biến để xác định field/value:

1. Dùng free text search trong Discover.
2. Tham khảo tài liệu Elastic.

---

## 13.1. Cách 1: Dùng free text search

Ví dụ muốn tìm failed logon Windows Event ID 4625.

Có thể search:

```text
"4625"
```

Sau đó mở rộng event trả về để xem field nào chứa giá trị này.

Có thể thấy các field như:

- `event.code`
- `winlog.event_id`
- `@timestamp`

Trong đó:

- `event.code` liên quan đến Elastic Common Schema.
- `winlog.event_id` liên quan đến Winlogbeat.
- `@timestamp` thường là thời gian được trích xuất từ event gốc.

---

## 13.2. Tìm SubStatus

Nếu muốn tìm disabled account status:

```text
"0xC0000072"
```

Sau đó mở event để xác định field liên quan.

Ví dụ:

```text
winlog.event_data.SubStatus
```

Từ đó có thể xây dựng truy vấn:

```text
event.code:4625 AND winlog.event_data.SubStatus:0xC0000072
```

---

## 13.3. Cách 2: Dùng tài liệu Elastic

Một số tài liệu hữu ích:

- Elastic Common Schema.
- ECS event fields.
- Winlogbeat fields.
- Winlogbeat ECS fields.
- Winlogbeat Security module fields.
- Filebeat fields.
- Filebeat ECS fields.

Việc đọc tài liệu giúp analyst hiểu field nào nên dùng và field nào phù hợp với từng source.

---

# 14. Elastic Common Schema

**ECS** là viết tắt của:

```text
Elastic Common Schema
```

ECS là một bộ từ vựng chuẩn, có thể mở rộng cho event và log trong Elastic Stack.

Mục tiêu của ECS là chuẩn hóa field name trên nhiều nguồn dữ liệu khác nhau.

---

## 14.1. Vì sao nên dùng ECS?

Nếu tổ chức dùng Elastic cho nhiều văn phòng hoặc nhiều bộ phận bảo mật, nên ưu tiên dùng ECS field.

Lý do:

- Dữ liệu thống nhất hơn.
- Query dễ viết hơn.
- Correlation tốt hơn.
- Dashboard dễ tái sử dụng hơn.
- Tương thích tốt hơn với Elastic Security.
- Dễ tích hợp với Elastic Machine Learning.
- Future-proof với các tính năng mới của Elastic.

---

## 14.2. Unified Data View

ECS giúp tạo góc nhìn thống nhất trên nhiều nguồn dữ liệu.

Ví dụ:

```text
Windows logs
Firewall logs
Endpoint events
Cloud logs
```

Đều có thể dùng các field chuẩn như:

```text
source.ip
destination.ip
user.name
host.name
event.code
```

---

## 14.3. Improved Search Efficiency

Khi field được chuẩn hóa, analyst không cần nhớ mỗi vendor dùng tên field khác nhau.

Ví dụ thay vì phải nhớ:

```text
src
sourceAddress
source_ip
client_ip
```

Có thể dùng field ECS:

```text
source.ip
```

---

## 14.4. Enhanced Correlation

ECS giúp correlation giữa nhiều nguồn dữ liệu dễ hơn.

Ví dụ:

```text
source.ip xuất hiện trong firewall log
      +
source.ip xuất hiện trong endpoint log
      +
source.ip xuất hiện trong VPN log
```

SOC analyst có thể liên kết các event lại để hiểu toàn cảnh incident.

---

## 14.5. Better Visualizations

Khi dữ liệu theo cùng schema, dashboard Kibana dễ xây dựng và tái sử dụng hơn.

Ví dụ:

- Top source IP.
- Top destination IP.
- Failed logon by user.
- Authentication by host.
- Alerts by event category.

---

# 15. Các truy vấn KQL mẫu cho SOC

## 15.1. Tìm failed logon

```text
event.code:4625
```

## 15.2. Tìm failed logon của disabled account

```text
event.code:4625 AND winlog.event_data.SubStatus:0xC0000072
```

## 15.3. Tìm failed logon theo khoảng thời gian

```text
event.code:4625 AND @timestamp >= "2023-03-03T00:00:00.000Z" AND @timestamp <= "2023-03-06T23:59:59.999Z"
```

## 15.4. Tìm username bắt đầu bằng admin

```text
event.code:4625 AND user.name: admin*
```

## 15.5. Tìm service account

```text
"svc-sql1"
```

## 15.6. Tìm source IP cụ thể

```text
source.ip:10.10.10.5
```

## 15.7. Tìm host cụ thể

```text
host.name:"DC01"
```

---

# 16. Bảng ôn nhanh thuật ngữ

| Thuật ngữ | Nghĩa ngắn gọn |
|---|---|
| Elastic Stack | Bộ công cụ Elasticsearch, Logstash, Kibana, Beats |
| ELK Stack | Elasticsearch, Logstash, Kibana |
| Elasticsearch | Công cụ lưu trữ, index, tìm kiếm và phân tích |
| Logstash | Công cụ thu thập, xử lý, chuyển đổi và gửi log |
| Kibana | Giao diện tìm kiếm, trực quan hóa và dashboard |
| Beats | Data shipper nhẹ gửi log/metric |
| Filebeat | Beat dùng để gửi log file |
| Winlogbeat | Beat dùng để gửi Windows Event Logs |
| Metricbeat | Beat dùng để gửi metric |
| Packetbeat | Beat dùng để thu thập network metadata |
| KQL | Kibana Query Language |
| ECS | Elastic Common Schema |
| @timestamp | Thời gian event theo Elastic |
| event.code | Mã event theo ECS |
| winlog.event_id | Event ID từ Winlogbeat |
| SubStatus | Mã lý do đăng nhập thất bại trong Windows |
| Discover | Khu vực tìm kiếm và khám phá dữ liệu trong Kibana |
| Visualization | Trực quan hóa dữ liệu |
| Dashboard | Bảng điều khiển hiển thị nhiều visualization |

---

# 17. Câu hỏi ôn tập

## 1. Elastic Stack gồm những thành phần chính nào?

Elastic Stack gồm Elasticsearch, Logstash, Kibana và Beats.

## 2. Elasticsearch dùng để làm gì?

Elasticsearch dùng để lưu trữ, index, tìm kiếm và phân tích dữ liệu log.

## 3. Logstash hoạt động qua mấy giai đoạn chính?

Logstash thường hoạt động qua 3 giai đoạn: input, filter và output.

## 4. Kibana dùng để làm gì?

Kibana dùng để tìm kiếm, trực quan hóa, tạo dashboard và phân tích dữ liệu trong Elasticsearch.

## 5. Beats là gì?

Beats là các data shipper nhẹ được cài trên máy endpoint/server để gửi log hoặc metric về Logstash hoặc Elasticsearch.

## 6. KQL là gì?

KQL là Kibana Query Language, dùng để tìm kiếm và lọc dữ liệu trong Kibana.

## 7. Truy vấn `event.code:4625` dùng để tìm gì?

Truy vấn này dùng để tìm các Windows failed logon events.

## 8. Vì sao nên dùng ECS field?

Vì ECS chuẩn hóa field name trên nhiều nguồn dữ liệu, giúp query, correlation và visualization hiệu quả hơn.

---

# Key Takeaway

Elastic Stack là một nền tảng mạnh để xây dựng SIEM mã nguồn mở hoặc phục vụ security monitoring.

Trong SOC, analyst thường dùng Kibana để truy vấn log bằng KQL, phân tích event, tìm IOC, xây dựng dashboard và điều tra incident.

Để làm việc hiệu quả với Elastic, cần hiểu vai trò của Elasticsearch, Logstash, Kibana, Beats, cách dữ liệu đi qua pipeline và cách dùng các field chuẩn theo Elastic Common Schema.


---

# References

- Elastic Docs - Winlogbeat exported fields: <https://www.elastic.co/docs/reference/beats/winlogbeat/exported-fields-winlog>
- Elastic Docs - Winlogbeat ECS fields: <https://www.elastic.co/docs/reference/beats/winlogbeat/exported-fields-ecs>
- Elastic Docs - Winlogbeat Security module fields: <https://www.elastic.co/docs/reference/beats/winlogbeat/exported-fields-security>
- Elastic Docs - Filebeat exported fields: <https://www.elastic.co/docs/reference/beats/filebeat/exported-fields>
- Elastic Docs - Filebeat ECS fields: <https://www.elastic.co/docs/reference/beats/filebeat/exported-fields-ecs>
- Elastic Docs - Elastic Common Schema: <https://www.elastic.co/docs/reference/ecs>
- Elastic Docs - ECS Event fields: <https://www.elastic.co/docs/reference/ecs/ecs-event>

---
layout: post
title: "HTTP Requests with cURL"
date: 2026-07-21 22:54:55 +0700
categories: ["JavaScript Deobfuscation", "HTTP Requests"]
tags: [curl, http, post-request, web-security, javascript, endpoint, cdsa]
description: "Tóm tắt ngắn cách dùng cURL gửi GET và POST request để mô phỏng hành vi JavaScript."
toc: true
---

# HTTP Requests with cURL

## Công thức dễ nhớ

```text
Xem trang
→ gửi POST
→ thêm dữ liệu
→ kiểm tra phản hồi
```

## 1. GET Request

```bash
curl http://SERVER_IP:PORT/
```

Dùng để lấy nội dung HTML của trang.

## 2. POST Request

```bash
curl -s -X POST http://SERVER_IP:PORT/serial.php
```

Trong đó:

```text
-s       = ẩn thông tin thừa
-X POST  = chọn phương thức POST
```

## 3. Gửi POST Data

```bash
curl -s -X POST   -d "param1=sample"   http://SERVER_IP:PORT/serial.php
```

`-d` dùng để gửi dữ liệu trong request body.

## Liên Hệ Với Bài Trước

JavaScript đã thực hiện:

```text
POST /serial.php
Body rỗng
```

Ta dùng cURL để mô phỏng lại đúng request này.

## Keyword Cần Nhớ

```text
curl
GET
POST
-X POST
-s
-d
request body
/serial.php
```

## Key Takeaway

```text
curl -s -X POST http://SERVER_IP:PORT/serial.php
```

Đây là lệnh chính để tái tạo request từ `secret.js`.

> Chỉ thực hiện trên HTB, lab hoặc hệ thống được cấp phép.

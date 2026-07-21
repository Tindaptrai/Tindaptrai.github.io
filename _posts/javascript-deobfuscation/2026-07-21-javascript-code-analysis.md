---
layout: post
title: "JavaScript Code Analysis"
date: 2026-07-21 22:50:28 +0700
categories: ["JavaScript Deobfuscation", "Code Analysis"]
tags: [javascript, code-analysis, xmlhttprequest, http-post, endpoint, web-security, deobfuscation]
description: "Tóm tắt ngắn cách phân tích JavaScript sau khi deobfuscate và xác định HTTP request ẩn."
toc: true
---

# JavaScript Code Analysis

## Công thức dễ nhớ

```text
Deobfuscate
→ đọc biến
→ đọc hàm
→ xác định request
→ kiểm tra endpoint
```

# 1. Hàm Chính

```javascript
function generateSerial() {
  var xhr = new XMLHttpRequest();
  var url = "/serial.php";

  xhr.open("POST", url, true);
  xhr.send(null);
}
```

Hàm `generateSerial()` chỉ gửi một HTTP POST request tới:

```text
/serial.php
```

# 2. Ý Nghĩa Từng Dòng

```text
XMLHttpRequest()      = tạo HTTP request
url                   = endpoint cùng domain
xhr.open("POST", ...) = cấu hình POST request
xhr.send(null)        = gửi request không có body
```

# 3. Nhận Định Chính

- Không có dữ liệu POST.
- Không xử lý response trong đoạn code.
- Endpoint có thể là chức năng chưa được đưa lên giao diện.
- Có thể kiểm tra thủ công để xem server có phản hồi hay không.

# 4. Kiểm Tra Request

```bash
curl -X POST http://SERVER_IP:PORT/serial.php
```

Hoặc dùng:

```text
Browser DevTools
Burp Suite
curl
```

> Chỉ kiểm thử trên HTB, lab hoặc hệ thống được cấp phép.

# Keyword Cần Nhớ

```text
generateSerial
XMLHttpRequest
POST
/serial.php
xhr.open
xhr.send
hidden endpoint
server-side
```

# Key Takeaway

```text
Code analysis giúp biến JavaScript đã deobfuscate
thành hành vi HTTP cụ thể.
```

Trong bài này:

```text
generateSerial()
→ POST /serial.php
→ không có request body
```

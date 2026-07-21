---
layout: post
title: "JavaScript Deobfuscation Basics"
date: 2026-07-21 22:46:31 +0700
categories: ["JavaScript Deobfuscation", "Deobfuscation"]
tags: [javascript, deobfuscation, beautify, pretty-print, unpacking, reverse-engineering, devtools, web-security]
description: "Tóm tắt ngắn về quy trình beautify, unpack và reverse engineer JavaScript bị obfuscate."
toc: true
---

# JavaScript Deobfuscation Basics

## Công thức dễ nhớ

```text
Minified
→ Beautify
→ Deobfuscate / Unpack
→ Analyze logic
```

# 1. Beautify

Beautify chỉ định dạng lại code:

- Thêm xuống dòng.
- Căn lề.
- Tách block.
- Giúp dễ đọc hơn.

Cách nhanh:

```text
Browser DevTools
→ mở file .js
→ nhấn nút { }
→ Pretty Print
```

Công cụ:

```text
Prettier
Beautifier
Browser DevTools
```

> Beautify không loại bỏ obfuscation; nó chỉ làm code gọn và dễ nhìn.

# 2. Deobfuscate / Unpack

Với packer dạng:

```javascript
eval(function(p,a,c,k,e,d){...})
```

có thể dùng UnPacker để khôi phục code gần với bản gốc.

Kết quả thường lộ:

```javascript
var xhr = new XMLHttpRequest();
var url = "/serial.php";
xhr.open("POST", url, true);
xhr.send(null);
```

Từ đó xác định được:

- Hàm chính.
- Endpoint.
- HTTP method.
- Tham số.
- Logic phía client.

# 3. Mẹo Unpack

Thay vì để code gọi `eval`, có thể sửa để in phần code được dựng lại:

```javascript
console.log(decoded_code);
```

Mục tiêu:

```text
In code đã giải ra
thay vì thực thi trực tiếp
```

# 4. Khi Tool Tự Động Không Đủ

Nếu code dùng:

- Custom obfuscator.
- Nhiều lớp encoding.
- Dynamic string decoding.
- Control-flow flattening.
- Runtime-generated code.

thì cần reverse engineering thủ công bằng:

```text
DevTools Debugger
Breakpoints
Console
Call stack
Network tab
Variable inspection
```

# Keyword Cần Nhớ

```text
Beautify
Pretty Print
Deobfuscate
Unpack
eval
UnPacker
Prettier
DevTools
Reverse engineering
XMLHttpRequest
Endpoint
```

# Key Takeaway

```text
BEAUTIFY ≠ DEOBFUSCATE
```

```text
Beautify  = làm code dễ nhìn
Unpack    = khôi phục logic
Analyze   = hiểu hành vi thật
```

Chỉ phân tích mã JavaScript trong lab, CTF hoặc hệ thống được phép kiểm thử.

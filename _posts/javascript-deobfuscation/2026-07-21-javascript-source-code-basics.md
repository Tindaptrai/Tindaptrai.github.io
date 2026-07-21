---
layout: post
title: "JavaScript Source Code Basics"
date: 2026-07-21 22:22:58 +0700
categories: ["JavaScript Deobfuscation", "Fundamentals"]
tags: [javascript, html, css, source-code, obfuscation, client-side, web-security]
description: "Tóm tắt ngắn về cách xem HTML, CSS và JavaScript client-side trước khi deobfuscation."
toc: true
---

# JavaScript Source Code Basics

## Công thức dễ nhớ

```text
HTML = cấu trúc
CSS = giao diện
JavaScript = chức năng
```

## 1. Xem source code

Phím tắt:

```text
Ctrl + U
```

Hoặc:

```text
view-source:http://SERVER_IP:PORT
```

Source HTML có thể chứa:

- Comments.
- Đường dẫn file CSS/JS.
- Endpoint.
- Thông tin nhạy cảm bị để quên.

## 2. CSS

CSS có thể nằm trong:

```html
<style>
  ...
</style>
```

Hoặc file ngoài:

```html
<link rel="stylesheet" href="style.css">
```

## 3. JavaScript

JavaScript có thể nằm trong:

```html
<script>
  ...
</script>
```

Hoặc file ngoài:

```html
<script src="secret.js"></script>
```

Mở trực tiếp `secret.js` để đọc logic phía client.

## 4. Obfuscation

Nếu JavaScript có dạng khó đọc như:

```javascript
eval(function(p,a,c,k,e,d){...})
```

thì code có thể đã bị **obfuscate**.

Obfuscation dùng để:

- Làm code khó đọc.
- Che giấu logic.
- Cản trở phân tích.
- Không đồng nghĩa với mã hóa.
- Không đảm bảo an toàn vì code vẫn chạy trên trình duyệt người dùng.

## Keyword cần nhớ

```text
HTML
CSS
JavaScript
Ctrl+U
view-source
<script src="">
client-side
obfuscation
eval
```

## Key Takeaway

```text
Muốn hiểu chức năng phía client:

Xem HTML
→ tìm file JavaScript
→ mở file .js
→ xác định code có bị obfuscate hay không
```

---
layout: post
title: "Code Obfuscation"
date: 2026-07-21 22:31:06 +0700
categories: ["JavaScript Deobfuscation", "Obfuscation"]
tags: [javascript, obfuscation, deobfuscation, client-side, evasion, web-security]
description: "Tóm tắt ngắn về code obfuscation, mục đích sử dụng và rủi ro bảo mật."
toc: true
---

# Code Obfuscation

## Công thức dễ nhớ

```text
Code gốc
→ Obfuscator
→ Code khó đọc
→ Vẫn chạy như cũ
```

# 1. Obfuscation Là Gì?

Obfuscation là kỹ thuật làm code khó đọc với con người nhưng vẫn giữ nguyên chức năng.

Cách thường gặp:

```text
Đổi tên biến
Ẩn chuỗi
Dùng dictionary
Chia nhỏ logic
Dùng eval
Làm rối control flow
```

# 2. Vì Sao JavaScript Hay Bị Obfuscate?

JavaScript chạy ở **client-side**, nên source code được gửi trực tiếp tới trình duyệt người dùng.

Vì vậy developer hoặc attacker thường dùng obfuscation để làm việc phân tích khó hơn.

# 3. Mục Đích Sử Dụng

## Hợp lệ

- Hạn chế sao chép code.
- Làm reverse engineering khó hơn.
- Che giấu logic kinh doanh.

## Độc hại

- Che giấu script độc.
- Né IDS/IPS và antivirus.
- Ẩn phishing, skimmer hoặc malware loader.

# 4. Điều Cần Nhớ

```text
Obfuscation ≠ Encryption
Obfuscation ≠ Security
```

Code vẫn phải được giải mã hoặc dựng lại khi chạy, nên analyst vẫn có thể deobfuscate.

Không nên đặt authentication hoặc encryption quan trọng hoàn toàn ở client-side.

# Keyword

```text
Obfuscation
Deobfuscation
Client-side
Dictionary
eval
Control flow
Evasion
Reverse engineering
```

# Key Takeaway

```text
Obfuscation chỉ làm code khó đọc,
không làm code trở nên an toàn.
```

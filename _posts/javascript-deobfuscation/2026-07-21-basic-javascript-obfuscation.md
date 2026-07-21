---
layout: post
title: "Basic JavaScript Obfuscation"
date: 2026-07-21 22:34:43 +0700
categories: ["JavaScript Deobfuscation", "Obfuscation"]
tags: [javascript, obfuscation, minification, packing, eval, deobfuscation, client-side]
description: "Tóm tắt ngắn về minification và packing trong JavaScript, cách nhận diện và ý nghĩa khi phân tích."
toc: true
---

# Basic JavaScript Obfuscation

## Công thức dễ nhớ

```text
Cleartext
→ Minify
→ Pack
→ Khó đọc hơn nhưng vẫn chạy
```

# 1. Chạy JavaScript

Ví dụ:

```javascript
console.log('HTB JavaScript Deobfuscation Module');
```

Có thể kiểm tra nhanh trên JS Console.

# 2. Minification

Minification:

- Xóa khoảng trắng và xuống dòng.
- Gộp code thành một dòng.
- Giảm kích thước file.
- Thường dùng đuôi `.min.js`.

```text
Minification làm code khó đọc hơn,
nhưng không phải obfuscation mạnh.
```

# 3. Packing

Packing biến code thành dạng khó đọc và tự dựng lại khi chạy.

Dấu hiệu phổ biến:

```javascript
eval(function(p,a,c,k,e,d){...})
```

Cách hoạt động:

```text
Tạo dictionary từ từ khóa/ký hiệu
→ thay bằng chỉ số
→ dùng eval dựng lại code gốc
```

# 4. Điểm Cần Nhớ

```text
Minify = nén cách trình bày
Pack   = che logic bằng dictionary/eval
```

Packing vẫn có thể để lộ strings quan trọng, nên analyst có thể dùng chúng để suy luận chức năng.

# Keyword

```text
console.log
minify
.min.js
packing
eval
function(p,a,c,k,e,d)
dictionary
deobfuscation
```

# Key Takeaway

```text
MINIFY = khó đọc hơn
PACK    = khó phân tích hơn
```

Cả hai vẫn phải được trình duyệt khôi phục và thực thi, nên vẫn có thể deobfuscate.

---
layout: post
title: "Hunting Evil with YARA on the Web"
date: 2026-07-23 23:43:04 +0700
categories: ["Introduction to YARA & Sigma", "YARA"]
tags: [yara, unpacme, malware-hunting, online-dataset, threat-hunting, soc, cdsa]
description: "Tóm tắt ngắn cách dùng Unpac.Me để chạy YARA rule trên tập dữ liệu malware trực tuyến."
toc: true
---

# Hunting Evil with YARA on the Web

## Công thức dễ nhớ

```text
RULE → VALIDATE → SCAN → MATCH
```

## 1. Unpac.Me Là Gì?

Unpac.Me là nền tảng hỗ trợ:

- Unpack malware.
- Lưu trữ nhiều mẫu malware.
- Chạy YARA rule trên tập dữ liệu trực tuyến.

Phù hợp khi SOC analyst không có sẵn malware dataset nội bộ.

## 2. Quy Trình YARA Hunt

```text
Đăng ký tài khoản
→ mở Yara Hunt
→ New Hunt
→ dán YARA rule
→ Validate
→ Scan
→ xem kết quả
```

## 3. Kết Quả Có Thể Xem

```text
Số file match
File type
File size
First seen
Last seen
Scan coverage
Rule validation
```

## 4. Giá Trị Với SOC

- Kiểm tra rule trên nhiều mẫu thực tế.
- Đánh giá độ bao phủ.
- Phát hiện malware cùng family.
- Tìm false positive hoặc pattern quá yếu.
- Hỗ trợ Threat Hunting và Detection Engineering.

## Keyword Cần Nhớ

```text
Unpac.Me
Yara Hunt
New Hunt
Validate
Scan
Match
Online malware dataset
Detection coverage
```

## Key Takeaway

```text
Unpac.Me giúp kiểm thử YARA rule
trên kho malware trực tuyến
mà không cần tự thu thập dataset lớn.
```

> Chỉ sử dụng dữ liệu và rule trong phạm vi lab, nghiên cứu hoặc hệ thống được cấp phép.

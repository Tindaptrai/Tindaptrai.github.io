---
title: Hashcat Cheatsheet
date: 2026-06-01 21:00:00 +0700
categories:
  - Cheatsheet
  - Password Security
tags:
  - hashcat
  - password
  - cracking
  - cheatsheet
description: Cheatsheet Hashcat cho lab password security và kiểm thử hợp pháp.
---

Hashcat là công cụ phục hồi mật khẩu mạnh, thường dùng trong lab password security để kiểm tra độ yếu của hash, wordlist và rule. Chỉ sử dụng với dữ liệu thuộc quyền kiểm thử hợp pháp.

## Tóm tắt mode

| Hash type | Mode | Ghi chú |
| --- | --- | --- |
| MD5 | 0 | Hash cơ bản trong lab |
| NTLM | 1000 | Thường gặp trong Windows lab |
| bcrypt | 3200 | Chậm hơn, cần tối ưu wordlist |

## Kiểm tra thiết bị

```bash
hashcat -I
```

## Ví dụ mode phổ biến

```bash
# MD5
hashcat -m 0 hashes.txt rockyou.txt

# NTLM
hashcat -m 1000 hashes.txt rockyou.txt

# bcrypt
hashcat -m 3200 hashes.txt rockyou.txt
```

## Attack Mode

```bash
# Straight wordlist
hashcat -a 0 -m 1000 hashes.txt wordlist.txt

# Wordlist + rule
hashcat -a 0 -m 1000 hashes.txt wordlist.txt -r rules/best64.rule

# Mask attack
hashcat -a 3 -m 1000 hashes.txt ?u?l?l?l?l?d?d?d

# Combination attack
hashcat -a 1 -m 1000 hashes.txt words1.txt words2.txt
```

## Session Management

```bash
# Đặt tên session
hashcat --session soc-lab -m 1000 hashes.txt rockyou.txt

# Khôi phục session
hashcat --session soc-lab --restore

# Hiển thị kết quả đã crack
hashcat -m 1000 hashes.txt --show
```

## Ghi chú phòng thủ

- Ưu tiên MFA cho tài khoản quan trọng.
- Dùng password manager để tạo mật khẩu dài, ngẫu nhiên.
- Theo dõi credential dumping và abnormal authentication.
- Không lưu hash hoặc wordlist nhạy cảm trong repo public.

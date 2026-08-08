---
layout: post
title: "Linux Structure Fundamentals"
date: 2026-08-08 22:54:00 +0700
categories: ["Linux Fundamentals", "Fundamentals"]
tags: [linux, linux-filesystem, fhs, kernel, shell, filesystem, cybersecurity, htb, cdsa]
description: "Tóm tắt cấu trúc Linux, triết lý, kiến trúc và các thư mục quan trọng trong Linux File System."
toc: true
---

# Linux Structure Fundamentals

## Công thức dễ nhớ

```text
HARDWARE → KERNEL → SHELL → UTILITIES
```

Linux là hệ điều hành mã nguồn mở, được dùng rộng rãi trên server, desktop, embedded systems và các nền tảng security.

Trong HTB, Pwnbox sử dụng **Parrot OS**, một distro dựa trên Debian và tập trung vào security.

---

# 1. Triết lý Linux

```text
Everything is a file
Small single-purpose programs
Chain programs together
Prefer command line
Configuration stored in text files
```

Ví dụ quan trọng:

```text
/etc/passwd
```

---

# 2. Thành phần chính

| Thành phần | Vai trò |
|---|---|
| `Bootloader` | Khởi động hệ điều hành, ví dụ GRUB |
| `Kernel` | Quản lý CPU, RAM, thiết bị và tài nguyên hệ thống |
| `Daemon` | Các service chạy nền |
| `Shell` | Giao diện dòng lệnh |
| `Utilities` | Các công cụ/chương trình hệ thống |

Shell phổ biến:

```text
bash
zsh
fish
```

---

# 3. Kiến trúc Linux

```text
Hardware
   ↓
Kernel
   ↓
Shell
   ↓
Utilities / Applications
```

**Kernel** là thành phần cốt lõi, đứng giữa hardware và các chương trình.

---

# 4. Linux File System

Linux sử dụng cấu trúc dạng cây theo **Filesystem Hierarchy Standard (FHS)**.

```text
/
├── bin
├── boot
├── dev
├── etc
├── home
├── lib
├── media
├── mnt
├── opt
├── root
├── sbin
├── tmp
├── usr
└── var
```

---

# 5. Các thư mục quan trọng

| Thư mục | Cần nhớ |
|---|---|
| `/` | Root của toàn bộ filesystem |
| `/bin` | Command/binary thiết yếu |
| `/boot` | Kernel và file khởi động |
| `/dev` | Device files |
| `/etc` | **File cấu hình hệ thống** |
| `/home` | Home của user thường |
| `/root` | Home của user `root` |
| `/lib` | Shared libraries |
| `/media` | Mount USB/removable media |
| `/mnt` | Mount point tạm |
| `/opt` | Tool/phần mềm bên thứ ba |
| `/sbin` | Binary dùng cho quản trị hệ thống |
| `/tmp` | File tạm |
| `/usr` | Program, library, manual |
| `/var` | **Log và dữ liệu thay đổi** |

---

# 6. Các đường dẫn cần nhớ cho Cybersecurity

```text
/etc
/var/log
/home
/root
/tmp
```

Một số file/path quan trọng:

```text
/etc/passwd      → thông tin user
/etc/shadow      → password hash
/etc/group       → group
/etc/ssh/        → cấu hình SSH
/var/log/        → log hệ thống
/home/<user>/    → dữ liệu user
/root/           → dữ liệu root
/tmp/            → file tạm
```

---

# 7. Mẹo nhớ nhanh

```text
CONFIG → /etc
LOG    → /var/log
USER   → /home
ROOT   → /root
TEMP   → /tmp
BOOT   → /boot
DEVICE → /dev
```

---

# 8. Góc nhìn SOC / DFIR

Khi điều tra Linux compromise, các vị trí thường được ưu tiên kiểm tra:

```text
/var/log
/etc
/home
/root
/tmp
```

Mục tiêu:

```text
Log analysis
User activity
Persistence
Configuration changes
Suspicious temporary files
```

---

# Key Takeaway

```text
Linux = Kernel + Shell + Utilities + Filesystem
```

```text
Quan trọng nhất:
CONFIG → /etc
LOG → /var/log
USER → /home
ROOT → /root
TEMP → /tmp
```

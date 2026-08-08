---
layout: post
title: "Linux System Information"
date: 2026-08-08 23:31:00 +0700
categories: ["Linux Fundamentals", "Fundamentals"]
tags: [linux, ssh, whoami, id, hostname, uname, ip, ss, ps, lsof]
description: "Các lệnh Linux cơ bản để thu thập thông tin hệ thống."
toc: true
---

# Linux System Information

## Công thức nhớ

```text
USER → HOST → KERNEL → NETWORK → PROCESS
```

## SSH

```bash
ssh htb-student@<IP>
```

## Các lệnh quan trọng

### User hiện tại

```bash
whoami
```

### UID, GID và group

```bash
id
```

Chú ý các group như:

```text
sudo
adm
docker
```

### Tên máy

```bash
hostname
```

### Thông tin kernel/hệ thống

```bash
uname -a
```

Chỉ xem kernel release:

```bash
uname -r
```

### Thư mục hiện tại

```bash
pwd
```

### Network

```bash
ip addr
ip route
```

Hoặc:

```bash
ifconfig
```

### Port và socket

```bash
ss -tulnp
```

Hoặc:

```bash
netstat -tulnp
```

### Process

```bash
ps aux
```

### User đang đăng nhập

```bash
who
```

### Environment variables

```bash
env
```

### Disk / partition

```bash
lsblk
```

### USB / PCI

```bash
lsusb
lspci
```

### File/socket đang được mở

```bash
lsof
lsof -i
```

## Lệnh trợ giúp

```bash
command -h
command --help
man command
```

Ví dụ:

```bash
man uname
```

## Bộ lệnh nên nhớ

```bash
whoami
id
hostname
uname -a
uname -r
pwd
ip addr
ip route
ss -tulnp
ps aux
who
env
lsblk
lsof -i
```

## Security cần nhớ

```text
id        → kiểm tra quyền/group
uname -r  → kernel version
ip route  → network/routing
ss -tulnp → service đang listen
ps aux    → process đang chạy
lsof -i   → process đang dùng network socket
```

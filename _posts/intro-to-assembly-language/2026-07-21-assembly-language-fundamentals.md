---
layout: post
title: "Assembly Language Fundamentals"
date: 2026-07-21 00:50:38 +0700
categories: ["Intro to Assembly Language", "Fundamentals"]
tags: [assembly, x86, machine-code, shellcode, compiler, syscall, reverse-engineering, binary-exploitation, malware-analysis, cdsa]
description: "Tóm tắt ngắn gọn về Assembly: vai trò, machine code, compilation stages, syscall và ứng dụng trong reverse engineering."
toc: true
---

# Assembly Language Fundamentals

Assembly là ngôn ngữ mức thấp, dùng các lệnh dễ đọc hơn machine code để điều khiển CPU.

## Timeline

```text
Created at: 2026-07-21 00:50:38 +0700
Timezone: Asia/Ho_Chi_Minh
Module: Intro to Assembly Language
Section: Assembly Language
Focus: Assembly, machine code, compilation, shellcode
```

# Công Thức Dễ Nhớ

```text
High-level code
→ Assembly
→ Machine code
→ Binary
→ CPU thực thi
```

# 1. Assembly Là Gì?

Assembly là cách viết dễ đọc của machine instructions.

Ví dụ:

```nasm
add rax, 1
```

Tương ứng machine code:

```text
48 83 C0 01
```

CPU thực tế chỉ xử lý `0` và `1`.

# 2. Assembly Phụ Thuộc CPU

Mỗi kiến trúc có instruction set riêng:

```text
x86 / x64
ARM
MIPS
```

Cùng một chương trình có thể tạo assembly khác nhau trên từng CPU.

# 3. High-Level Và Low-Level

```text
Python / C / C++
→ dễ viết, dễ đọc

Assembly
→ gần CPU, kiểm soát chi tiết

Machine code
→ CPU chạy trực tiếp
```

Compiler chuyển mã nguồn thành assembly rồi machine code.

# 4. Compiled Và Interpreted

## Compiled

Ví dụ: `C`, `C++`.

Được biên dịch thành machine code trước khi chạy.

## Interpreted

Ví dụ: `Python`, `JavaScript`, `Bash`.

Được interpreter xử lý khi chạy, nhưng cuối cùng vẫn gọi machine code bên dưới.

# 5. Syscall

Syscall là cách chương trình yêu cầu kernel thực hiện chức năng hệ thống.

```nasm
mov rax, 1      ; syscall write
mov rdi, 1      ; stdout
mov rsi, message
mov rdx, 12
syscall
```

Keyword:

```text
rax = syscall number
rdi/rsi/rdx = arguments
syscall = chuyển sang kernel
```

# 6. Shellcode

Shellcode là machine code được biểu diễn bằng hex bytes.

```text
Assembly
→ assemble
→ shellcode
→ nạp vào memory
→ CPU thực thi
```

Shellcode thường gặp trong:

- Binary exploitation.
- Malware.
- Process injection.
- Exploit development.

# 7. Giá Trị Với Security

Assembly cần cho:

- Reverse engineering.
- Malware analysis.
- Debugging.
- Buffer overflow.
- ROP.
- Heap exploitation.
- Shellcode analysis.
- Binary exploitation.

# Keyword Cần Nhớ

```text
Assembly
Machine code
Binary
Shellcode
Compiler
Interpreter
Instruction set
Register
Syscall
x86/x64
ARM
Reverse engineering
```

# Flashcard Nhanh

**Assembly là gì?**  
Ngôn ngữ mức thấp biểu diễn machine instructions bằng mnemonic dễ đọc.

**Machine code là gì?**  
Dãy byte mà CPU thực thi trực tiếp.

**Shellcode là gì?**  
Machine code ở dạng hex, thường được nạp trực tiếp vào memory.

**Compiler làm gì?**  
Chuyển high-level code thành assembly rồi machine code.

**Tại sao security analyst cần Assembly?**  
Để đọc binary, debug malware và hiểu exploit.

# Key Takeaway

```text
SOURCE → ASM → MACHINE CODE → CPU
```

Assembly là cầu nối giữa code con người viết và lệnh mà CPU thực thi.

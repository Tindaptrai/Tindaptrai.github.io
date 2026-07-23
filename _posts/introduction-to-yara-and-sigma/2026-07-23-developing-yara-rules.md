---
layout: post
title: "Developing YARA Rules"
date: 2026-07-23 22:51:50 +0700
categories: ["Introduction to YARA & Sigma", "YARA"]
tags: [yara, rule-development, yargen, pe, imphash, entropy, malware-detection, threat-hunting, detection-engineering, cdsa]
description: "Tóm tắt ngắn về phát triển YARA rule thủ công và tự động bằng yarGen."
toc: true
---

# Developing YARA Rules

## Công thức dễ nhớ

```text
ANALYZE → CHOOSE IOC → WRITE RULE → TEST → TUNE
```

# 1. Quy Trình Chung

```text
Phân tích sample
→ chọn strings/đặc điểm riêng
→ thêm PE, filesize, imphash, entropy
→ viết condition
→ quét tập file
→ giảm false positive
```

# 2. Rule Thủ Công

Ví dụ phát hiện UPX:

```yara
rule UPX_Packed
{
    strings:
        $a = "UPX0"
        $b = "UPX1"
        $c = "UPX2"

    condition:
        all of them
}
```

Điểm yếu: chỉ dựa vào string phổ biến nên có thể match file hợp lệ.

# 3. yarGen

`yarGen` tự động tạo rule từ strings khác biệt so với goodware.

```bash
python3 yarGen.py   -m /path/to/samples   -o output.yar
```

Nhớ:

```text
-m = thư mục sample
-o = file rule đầu ra
```

> Rule do yarGen tạo phải được analyst review và chỉnh sửa trước khi triển khai.

# 4. Tăng Độ Chính Xác

Kết hợp nhiều tín hiệu:

```yara
condition:
    uint16(0) == 0x5A4D and
    filesize < 300KB and
    1 of ($x*) and
    4 of them
```

Keyword:

```text
MZ header
filesize
unique strings
imphash
PE module
entropy
resource ID
digital signature
```

# 5. PE Module

```yara
import "pe"
```

Ví dụ:

```yara
pe.imphash() == "..."
pe.number_of_sections > 4
pe.number_of_signatures == 0
pe.number_of_resources > 1
```

Dùng để kiểm tra cấu trúc PE thay vì chỉ tìm string.

# 6. .NET Detection

Dấu hiệu .NET:

```text
BSJB
Class names
Function names
```

Ví dụ condition:

```yara
condition:
    uint16(0) == 0x5A4D and
    $dotnetMagic and
    6 of them
```

# 7. Entropy Và Resource

```yara
import "math"
```

Entropy cao có thể chỉ ra resource bị nén hoặc mã hóa.

```text
Entropy gần 8
→ dữ liệu có thể packed/encrypted
```

Có thể kết hợp:

```text
Resource ID
Resource size
Entropy
API FindResource/LoadResource
Không có chữ ký số
```

# 8. Ba Mẫu Phát Triển Rule

## UPX

```text
UPX strings
→ phát hiện packer
```

## APT17/ZoxPNG

```text
Unique strings
+ filesize
+ imphash
+ PE header
```

## Stonedrill

```text
File-enumeration APIs
+ resource ID 101
+ entropy cao
+ không có chữ ký
```

# 9. Keyword Cần Nhớ

```text
strings
unique IOC
yarGen
post-processing
MZ
filesize
imphash
pe module
math module
entropy
BSJB
false positive
```

# Key Takeaway

```text
Rule tốt không dựa vào một string duy nhất.
```

```text
Unique strings
+ file properties
+ PE metadata
+ threshold phù hợp
= detection chính xác hơn
```

`yarGen` giúp khởi tạo rule nhanh; analyst vẫn phải test, tune và loại bỏ pattern yếu.

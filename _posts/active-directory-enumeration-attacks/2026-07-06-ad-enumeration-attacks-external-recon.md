---
layout: post
title: "Active Directory Enumeration & Attacks: External Recon and Enumeration Principles"
date: 2026-07-06 22:49:21 +0700
categories: ["Active Directory Enumeration & Attacks", "Enumeration"]
tags: [active-directory, ad-enumeration-attacks, external-recon, enumeration, osint, dns, asn, ip-space, username-harvesting, breach-data, scope-validation, blue-team, red-team]
description: "Tóm tắt ngắn bài mới Active Directory Enumeration & Attacks: External Recon and Enumeration Principles, gồm OSINT, ASN/IP, DNS, public data, username format, breach data và nguyên tắc enumeration đúng scope."
toc: true
---

# Active Directory Enumeration & Attacks: External Recon and Enumeration Principles

Đây là bài mở đầu cho module **Active Directory Enumeration & Attacks**. Trước khi đi vào internal AD enumeration, cần hiểu cách làm **external reconnaissance** để xác thực scope, tìm thông tin public và chuẩn bị dữ liệu ban đầu cho pentest.

```text
Recon tốt = xác thực đúng phạm vi + tìm thông tin public hữu ích + tránh test nhầm hệ thống ngoài scope.
```

---

## Timeline cập nhật

```text
Created at: 2026-07-06 22:49:21 +0700
Timezone: Asia/Ho_Chi_Minh
Module: Active Directory Enumeration & Attacks
Topic: External Recon and Enumeration Principles
Focus: OSINT, ASN/IP, DNS, public data, username format, breach data, enumeration workflow
```

---

# 1. External Recon là gì?

**External reconnaissance** là giai đoạn thu thập thông tin từ bên ngoài trước khi bắt đầu pentest.

Mục tiêu:

- Xác thực thông tin trong scoping document.
- Đảm bảo chỉ kiểm thử đúng hệ thống được phép.
- Tìm thông tin public có thể hỗ trợ pentest.
- Phát hiện credential leak hoặc breach data.
- Tìm email/username format.
- Hiểu sơ bộ hạ tầng public của tổ chức.

```text
External recon không chỉ để tìm điểm yếu.
Nó còn giúp bảo vệ tester khỏi việc test nhầm tài sản ngoài phạm vi.
```

---

# 2. Những dữ liệu cần tìm

| Data Point | Ý nghĩa |
|---|---|
| IP Space | ASN, netblocks, public IP, cloud/hosting provider, DNS records |
| Domain Information | Domain owner, subdomains, mail servers, DNS, VPN/web portals |
| Schema Format | Email format, AD username format, password policy clues |
| Data Disclosures | PDF/DOCX/XLSX/PPT public chứa metadata, intranet link, user info |
| Breach Data | Username/password/hash từng bị leak công khai |

Ghi nhớ:

```text
Một email format nhỏ cũng có thể giúp tạo username list cho bước kiểm thử tiếp theo nếu nằm trong scope.
```

---

# 3. Nguồn OSINT thường dùng

| Nhóm nguồn | Ví dụ |
|---|---|
| ASN/IP registrars | IANA, ARIN, RIPE, BGP Toolkit |
| Domain/DNS | DomainTools, PTRArchive, ICANN, DNS queries |
| Social media | LinkedIn, Twitter/X, Facebook, Instagram, news |
| Company website | About, Contact, News, embedded documents |
| Cloud/dev storage | GitHub, S3 buckets, Azure Blob, Google dorks |
| Breach data | HaveIBeenPwned, Dehashed |

Nguyên tắc:

```text
Thu thập thông tin phải bám đúng scope và Rules of Engagement.
```

---

# 4. ASN/IP Space

ASN và IP space giúp xác định hạ tầng public có thể thuộc tổ chức.

Có thể tìm:

- Public netblocks.
- Hosting provider.
- Cloud provider.
- Web server IP.
- Mail server IP.
- Nameserver IP.

Với tổ chức nhỏ, hạ tầng thường nằm trên provider như Cloudflare, AWS, Azure, Google Cloud hoặc hosting bên thứ ba.

```text
Không phải cứ domain resolve ra IP nào là IP đó được phép scan.
Phải kiểm tra scope trước.
```

---

# 5. DNS Enumeration

DNS giúp xác thực scope và phát hiện host public.

Có thể thu thập:

- A records.
- AAAA records.
- MX records.
- NS records.
- TXT records.
- Subdomains.
- Mail servers.
- VPN/web portals.

Ví dụ:

```text
mail1.example.com
ns1.example.com
ns2.example.com
vpn.example.com
portal.example.com
```

Nếu phát hiện host liên quan nhưng chưa có trong scope, cần hỏi lại khách hàng trước khi kiểm thử chủ động.

---

# 6. Public Data

Public data có thể đến từ:

- Website công ty.
- Contact page.
- Job postings.
- Press releases.
- Social media.
- Public documents.
- GitHub repositories.
- Cloud storage public.

Job posting có thể tiết lộ:

- Công nghệ đang dùng.
- Version phần mềm.
- Cloud provider.
- Security tools.
- Backup/DR process.
- Kiến trúc hệ thống.

Ví dụ:

```text
Tin tuyển SharePoint Administrator có thể tiết lộ công ty dùng SharePoint 2013/2016.
```

---

# 7. Public Documents và Metadata

Các file public như:

```text
.pdf
.docx
.xlsx
.pptx
```

có thể chứa:

- Author metadata.
- Internal username.
- Email.
- Internal path.
- Intranet link.
- Software version.
- Tên phòng ban.
- Template nội bộ.

Nên lưu lại:

- URL.
- File copy.
- Screenshot.
- Metadata đáng chú ý.
- Thời điểm phát hiện.

---

# 8. Email và Username Format

Từ website hoặc contact page có thể suy ra naming convention:

```text
first.last@example.com
flast@example.com
f.last@example.com
username@example.com
```

Thông tin này có thể hỗ trợ:

- Tạo username list.
- Kiểm tra external login portals nếu được phép.
- Chuẩn bị password spraying nếu nằm trong scope.
- Chuẩn bị internal enumeration sau khi có credential hợp lệ.

---

# 9. Breach Data

Breach data có thể chứa:

- Corporate email.
- Username.
- Old password.
- Password hash.
- Tên đầy đủ.
- Metadata khác.

Ghi nhớ:

```text
Password leak có thể đã cũ, nhưng vẫn giúp nhận diện password pattern hoặc password reuse.
```

Mọi thao tác liên quan credential phải tuân thủ scope, legal approval và Rules of Engagement.

---

# 10. Enumeration là quá trình lặp

Enumeration không phải làm một lần là xong.

Workflow cơ bản:

```text
1. Passive recon rộng.
2. Thu hẹp dữ liệu theo scope.
3. Xác thực thông tin.
4. Chuyển sang active enumeration nếu được phép.
5. Lặp lại khi có phát hiện mới.
```

Khi pentest bị kẹt, dữ liệu passive recon ban đầu có thể là manh mối quan trọng để đi tiếp.

---

# 11. Scope và Legal Safety

Nguyên tắc quan trọng nhất:

```text
Không test hệ thống nếu chưa chắc nằm trong scope.
```

Cần xác minh:

- IP có thuộc khách hàng không.
- Hạ tầng có phải third-party hosting không.
- Cloud provider có yêu cầu notification không.
- Subdomain có nằm trong scope không.
- Active scanning có được phép không.
- Credential testing có được phép không.

Nếu không chắc:

```text
Escalate và xin xác nhận bằng văn bản.
```

---

# 12. Blue Team Checklist

Từ góc nhìn phòng thủ, tổ chức nên kiểm tra:

- Public DNS records.
- Public subdomains.
- Leaked documents.
- Metadata trong file public.
- GitHub/code leaks.
- Public cloud buckets.
- Breach exposure của email công ty.
- Job posting có lộ công nghệ quá chi tiết không.
- Contact page có lộ quá nhiều user/email không.
- External portals có MFA/rate limit/lockout không.

---

# 13. Red Team Mindset

Trong pentest hợp pháp, cần chú ý:

- Username format.
- External login portals.
- Public service exposure.
- Valid domains/subdomains.
- Cloud footprint.
- Old documents.
- Email addresses.
- Breach data.
- Technology stack clues.
- Dữ liệu public liên quan đến AD/internal environment.

```text
External recon tốt giúp chuẩn bị cho internal enumeration sau khi có foothold.
```

---

# Key Takeaway

```text
Active Directory Enumeration & Attacks bắt đầu từ external recon.
Cần tìm IP space, DNS, domain info, public data, username format và breach exposure.
ASN/DNS/job postings/public documents đều có thể tiết lộ thông tin quan trọng.
Enumeration là quá trình lặp, bắt đầu từ passive rồi mới active nếu được phép.
Scope và written authorization là bắt buộc.
```

Recon càng kỹ, pentest càng có định hướng. Nhưng mọi hành động phải luôn nằm trong phạm vi được ủy quyền và tuân thủ Rules of Engagement.

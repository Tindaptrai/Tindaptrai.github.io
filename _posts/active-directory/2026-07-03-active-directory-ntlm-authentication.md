---
layout: post
title: "Active Directory Authentication: NTLM, NTLMv1, NTLMv2, MSCache2"
date: 2026-07-03 00:31:33 +0700
categories: [Active Directory, Windows Security]
tags: [active-directory, ad-ds, ntlm, ntlmv1, ntlmv2, lm-hash, nthash, mscache2, domain-cached-credentials, kerberos, authentication, password-hash, blue-team, red-team]
description: "Tóm tắt xác thực NTLM trong Active Directory: LM, NT Hash, NTLM challenge-response, NTLMv1, NTLMv2, Domain Cached Credentials/MSCache2 và các rủi ro bảo mật liên quan."
toc: true
---

# Active Directory Authentication: NTLM, NTLMv1, NTLMv2, MSCache2

Ngoài Kerberos và LDAP, Active Directory còn sử dụng một số cơ chế xác thực khác như **LM**, **NT Hash**, **NTLMv1** và **NTLMv2**.

Điểm cần phân biệt rõ:

```text
LM và NT Hash = thuật toán/hash dùng để lưu trữ hoặc tạo material xác thực.
NTLMv1 và NTLMv2 = giao thức xác thực challenge-response sử dụng các hash đó.
```

---

## Timeline cập nhật

```text
Created at: 2026-07-03 00:31:33 +0700
Timezone: Asia/Ho_Chi_Minh
Topic: NTLM Authentication
Focus: LM, NT Hash, NTLM, NTLMv1, NTLMv2, MSCache2
```

---

# 1. Tổng quan NTLM Authentication

Trong môi trường Windows/Active Directory, authentication không chỉ có Kerberos.

Một số cơ chế thường gặp:

- LM.
- NT Hash / NTHash.
- NTLM.
- NTLMv1.
- NTLMv2.
- Domain Cached Credentials.
- MSCache2 / DCC2.

Kerberos thường được ưu tiên khi có thể, nhưng NTLM vẫn xuất hiện trong nhiều môi trường vì lý do compatibility, legacy systems hoặc application cũ.

---

# 2. Hash vs Protocol

Bảng phân biệt nhanh:

| Thành phần | Loại | Ý nghĩa |
|---|---|---|
| LM | Hash algorithm cũ | Lưu trữ password kiểu LAN Manager |
| NT Hash | Hash algorithm | MD4 của UTF-16-LE password |
| NTLM | Authentication protocol | Challenge-response |
| NTLMv1 | Protocol | Phiên bản cũ, yếu hơn |
| NTLMv2 | Protocol | Phiên bản cải tiến, mặc định trong Windows hiện đại |
| MSCache2 | Cached credential | Cho phép logon khi không liên hệ được DC |

---

# 3. So sánh nhanh NTLM và Kerberos

| Cơ chế | Mutual Authentication | Message Type | Trusted Third Party |
|---|---:|---|---|
| NTLM | Không | Challenge/response | Domain Controller |
| NTLMv1 | Không | MD4 hash + nonce | Domain Controller |
| NTLMv2 | Không | HMAC-MD5 + nonce | Domain Controller |
| Kerberos | Có | Encrypted tickets | DC / KDC |

Ghi nhớ:

```text
Kerberos dùng ticket.
NTLM dùng challenge-response.
```

---

# 4. LM Hash

**LAN Manager (LM/LANMAN)** là cơ chế lưu trữ password rất cũ của Windows.

Đặc điểm:

- Ra đời từ thời OS/2.
- Bị tắt mặc định từ Windows Vista / Server 2008.
- Vẫn có thể gặp trong môi trường legacy.
- Password bị giới hạn tối đa 14 ký tự.
- Không phân biệt hoa thường.
- Password được chuyển thành uppercase trước khi hash.

Điểm yếu lớn:

```text
Password 14 ký tự bị chia thành 2 block, mỗi block 7 ký tự.
Attacker chỉ cần tấn công 7 ký tự hai lần thay vì 14 ký tự một lần.
```

Nếu password dài 7 ký tự hoặc ít hơn, nửa sau của LM hash thường có giá trị cố định dễ nhận biết.

---

# 5. LM Hash Flow

Quy trình đơn giản hóa:

```text
1. Password chuyển thành uppercase.
2. Password được pad NULL để đủ 14 ký tự.
3. Chia thành hai phần, mỗi phần 7 ký tự.
4. Tạo hai DES keys.
5. Encrypt chuỗi cố định KGS!@#$%.
6. Ghép hai ciphertext lại thành LM hash.
```

Do thiết kế yếu, LM hash rất dễ bị crack so với chuẩn hiện đại.

---

# 6. NT Hash / NTHash

**NT Hash** là hash được dùng trên Windows hiện đại.

Công thức:

```text
MD4(UTF-16-LE(password))
```

Ví dụ NT Hash có dạng:

```text
b4b9b02e6f09a9bd760f388b67351e2b
```

NT Hash mạnh hơn LM vì hỗ trợ Unicode và không chia nhỏ password theo block 7 ký tự như LM.

Tuy vậy, NT Hash vẫn có vấn đề:

- Không dùng salt.
- Có thể bị crack offline nếu password yếu.
- Có thể bị abuse trong pass-the-hash nếu attacker lấy được hash và có điều kiện phù hợp.

---

# 7. NTLM Challenge-Response

NTLM là giao thức xác thực dạng challenge-response.

Flow cơ bản gồm 3 message:

```text
1. NEGOTIATE_MESSAGE
2. CHALLENGE_MESSAGE
3. AUTHENTICATE_MESSAGE
```

Ý tưởng:

```text
Client không gửi password rõ qua network.
Server gửi challenge.
Client dùng secret/hash để tạo response.
Server/DC validate response.
```

---

# 8. NTLM Authentication Request Flow

Flow tổng quát:

```text
Client -> Server/DC: NTLM Negotiate Message
Server/DC -> Client: NTLM Challenge Message
Client -> Server/DC: NTLM Authenticate Message
Server/DC -> Netlogon: Netlogon_network_info
Netlogon -> Server/DC: Netlogon_Validation_SAM_Info
```

Trong domain environment, Domain Controller thường tham gia validate authentication thông qua Netlogon.

---

# 9. NTLM Hash Format

Một NTLM hash dump thường có dạng:

```text
username:RID:LM_HASH:NT_HASH:::
```

Ví dụ:

```text
Rachel:500:aad3c435b514a4eeaad3b935b51304fe:e46b9e548fa0d122de7f59fb6d48eaa2:::
```

Giải thích:

| Thành phần | Ý nghĩa |
|---|---|
| `Rachel` | Username |
| `500` | RID, 500 thường là built-in Administrator |
| `aad3c435...` | LM hash placeholder/disabled value |
| `e46b9e...` | NT Hash |

Nếu LM hash bị disable, giá trị LM thường không dùng được cho authentication thực tế.

---

# 10. NTLM Security Notes

Các vấn đề bảo mật quan trọng:

- LM và NT Hash không dùng salt.
- NT Hash có thể bị crack offline nếu password yếu.
- NTLM không có mutual authentication như Kerberos.
- NTLM dễ bị relay trong một số cấu hình yếu.
- NT Hash có thể bị abuse trong pass-the-hash nếu attacker có hash và quyền phù hợp.

Defense cần chú ý:

- Tắt LM.
- Hạn chế NTLM nếu có thể.
- Ưu tiên Kerberos.
- Bật SMB signing nơi phù hợp.
- Giảm local admin reuse.
- Dùng password mạnh và unique.
- Monitor NTLM authentication bất thường.

---

# 11. NTLMv1 / Net-NTLMv1

**NTLMv1** là phiên bản cũ của giao thức NTLM challenge-response.

Đặc điểm:

- Dùng challenge 8-byte từ server.
- Client trả response 24-byte.
- Có thể dùng LM hoặc NT hash.
- Yếu hơn NTLMv2.
- Không nên dùng trong môi trường hiện đại.

Thuật toán đơn giản hóa:

```text
C = 8-byte server challenge
K1 | K2 | K3 = LM/NT hash material
response = DES(K1,C) | DES(K2,C) | DES(K3,C)
```

Điểm quan trọng:

```text
Net-NTLMv1 hash không giống NT Hash.
Net-NTLMv1 thường dùng cho cracking/relay context, không dùng trực tiếp như pass-the-hash.
```

---

# 12. NTLMv2 / Net-NTLMv2

**NTLMv2** được giới thiệu để cải thiện NTLMv1 và là mặc định trong Windows hiện đại.

NTLMv2 sử dụng:

- Server challenge.
- Client challenge.
- Timestamp.
- Domain name.
- HMAC-MD5.
- NT Hash làm secret material.

Thuật toán đơn giản hóa:

```text
SC = server challenge
CC = client challenge
v2-Hash = HMAC-MD5(NT-Hash, username, domain)
LMv2 = HMAC-MD5(v2-Hash, SC, CC)
NTv2 = HMAC-MD5(v2-Hash, SC, client blob)
response = LMv2 | CC | NTv2 | client blob
```

NTLMv2 khó crack hơn NTLMv1 nhưng vẫn phụ thuộc rất nhiều vào độ mạnh của password và cấu hình bảo vệ.

---

# 13. NTLMv1 vs NTLMv2

| Tiêu chí | NTLMv1 | NTLMv2 |
|---|---|---|
| Độ an toàn | Yếu hơn | Mạnh hơn |
| Challenge | Server challenge | Server + client challenge |
| Thuật toán | DES-based | HMAC-MD5 |
| Default hiện đại | Không nên dùng | Phổ biến hơn |
| Khả năng bị abuse | Cao | Vẫn có rủi ro nếu cấu hình yếu |

Ghi nhớ:

```text
NTLMv2 tốt hơn NTLMv1, nhưng Kerberos vẫn nên được ưu tiên nếu có thể.
```

---

# 14. Domain Cached Credentials / MSCache2

Trong domain, Kerberos/NTLM thường cần liên hệ Domain Controller.

Nhưng nếu máy domain-joined không thể liên hệ DC, Windows vẫn cần cho phép user từng đăng nhập trước đó logon.

Cơ chế này dùng:

```text
Domain Cached Credentials
MSCache
MSCache2
DCC2
```

Các credential cache này được lưu cục bộ để hỗ trợ offline logon.

---

# 15. MSCache2 Format

Ví dụ format:

```text
$DCC2$10240#bjones#e4e938d12fe5974dc42a90120bd9c90f
```

Ý nghĩa:

| Thành phần | Ý nghĩa |
|---|---|
| `$DCC2$` | Domain Cached Credential v2 |
| `10240` | Iteration count |
| `bjones` | Username |
| Hash cuối | Cached credential hash |

Điểm quan trọng:

```text
MSCache2 không dùng cho pass-the-hash.
MSCache2 thường crack chậm hơn nhiều so với NT Hash.
```

---

# 16. Vì sao MSCache2 quan trọng?

Từ góc nhìn defender:

- Cho phép user logon khi không liên hệ được DC.
- Có thể bị trích xuất nếu attacker có local admin trên máy.
- Không nên xem là vô hại.
- Cần bảo vệ local admin và endpoint security.

Từ góc nhìn pentest hợp pháp:

- Hash này có thể được thu thập trên host đã compromise.
- Crack thường chậm và cần target hợp lý.
- Không nên tốn nhiều thời gian crack nếu password policy mạnh.

---

# 17. Blue Team Checklist

Blue Team nên kiểm tra:

- LM hash đã bị disable chưa.
- NTLMv1 có còn được phép không.
- NTLMv2 enforcement.
- Kerberos có được ưu tiên không.
- SMB signing / LDAP signing.
- Local admin reuse.
- Password policy.
- Privileged account exposure.
- NTLM authentication events.
- NTLM relay risk.
- Domain cached logon policy.
- Endpoint credential dumping detection.
- Dấu hiệu pass-the-hash / pass-the-ticket.

---

# 18. Red Team / Pentest Mindset

Trong assessment hợp pháp, cần phân biệt rõ:

| Artifact | Có thể crack? | Có thể pass-the-hash? |
|---|---:|---:|
| LM Hash | Có | Tùy context |
| NT Hash | Có | Có |
| Net-NTLMv1 | Có | Không trực tiếp |
| Net-NTLMv2 | Có | Không trực tiếp |
| MSCache2/DCC2 | Có, nhưng chậm | Không |

Câu hỏi cần tự hỏi khi gặp hash:

```text
Đây là NT Hash hay Net-NTLM?
Có crack offline được không?
Có dùng pass-the-hash được không?
Có relay được không?
Password policy có đủ mạnh không?
```

---

# 19. Bảng ôn nhanh

| Khái niệm | Ý nghĩa |
|---|---|
| LM | Hash cũ, yếu, nên disable |
| NT Hash | MD4 của UTF-16-LE password |
| NTLM | Challenge-response authentication |
| NTLMv1 | Phiên bản cũ, yếu |
| NTLMv2 | Phiên bản cải tiến, phổ biến hơn |
| Net-NTLM | Hash/response bắt được qua network challenge-response |
| Pass-the-Hash | Dùng NT Hash để authenticate trong điều kiện phù hợp |
| MSCache2 | Cached domain credential dùng cho offline logon |
| DCC2 | Tên khác của MSCache2 |
| Kerberos | Ticket-based auth, thường ưu tiên hơn NTLM |

---

# Key Takeaway

NTLM authentication là phần rất quan trọng trong Active Directory security.

```text
LM rất yếu và nên bị disable.
NT Hash là material quan trọng, có thể bị crack hoặc abuse.
NTLMv1 cũ và nguy hiểm hơn NTLMv2.
NTLMv2 tốt hơn nhưng vẫn cần hardening.
MSCache2 hỗ trợ offline logon nhưng không dùng pass-the-hash.
Kerberos vẫn là lựa chọn ưu tiên khi có thể.
```

Đối với Blue Team, trọng tâm là giảm NTLM legacy, tăng Kerberos, enforce signing, kiểm soát local admin và monitor các dấu hiệu credential abuse.

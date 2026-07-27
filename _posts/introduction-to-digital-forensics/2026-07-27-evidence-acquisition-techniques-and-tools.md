---
layout: post
title: "Evidence Acquisition Techniques & Tools"
date: 2026-07-27 21:33:57 +0700
categories: ["Introduction to Digital Forensics", "Evidence Acquisition"]
tags: [digital-forensics, evidence-acquisition, forensic-imaging, ftk-imager, arsenal-image-mounter, winpmem, dumpit, kape, velociraptor, memory-acquisition, rapid-triage, network-forensics, dfir, cdsa]
description: "Tổng quan các kỹ thuật và công cụ thu thập bằng chứng số: forensic imaging, memory acquisition, KAPE, Velociraptor và network evidence."
toc: true
---

# Evidence Acquisition Techniques & Tools

## Công thức dễ nhớ

```text
IMAGE → MEMORY → TRIAGE → REMOTE COLLECTION → NETWORK
```

Bài này tập trung vào ba nhóm chính:

```text
Forensic Imaging
Host-based Evidence & Rapid Triage
Network Evidence
```

![KAPE workflow](/assets/img/introduction-to-digital-forensics/evidence-acquisition/kape-workflow.png)

---

# 1. Nguyên Tắc Thu Thập Bằng Chứng

Mục tiêu:

- Giữ nguyên tính toàn vẹn.
- Tránh sửa đổi nguồn dữ liệu.
- Ghi nhận đầy đủ quá trình thu thập.
- Tạo bản sao để phân tích.
- Bảo đảm khả năng kiểm chứng.

Keyword:

```text
Integrity
Authenticity
Admissibility
Hash
Read-only
Chain of Custody
Forensically Sound
```

---

# 2. Forensic Imaging

Forensic imaging là tạo bản sao **bit-by-bit** của thiết bị lưu trữ.

Áp dụng cho:

```text
HDD
SSD
USB
Memory Card
Virtual Disk
```

Mục đích:

```text
Bảo toàn dữ liệu gốc
→ phân tích trên bản sao
→ có thể xác minh bằng hash
```

## Công cụ quan trọng

| Công cụ | Vai trò |
|---|---|
| FTK Imager | Tạo và xem disk image |
| AFF4 Imager | Tạo image AFF4, hỗ trợ compression |
| `dd` | Sao chép block-level trên Unix/Linux |
| `dcfldd` | Bản mở rộng của `dd`, hỗ trợ hashing |
| Virtualization snapshot | Thu thập disk/memory từ VM |
| Arsenal Image Mounter | Mount forensic image để phân tích |

---

# 3. FTK Imager

Quy trình cơ bản:

```text
File
→ Create Disk Image
→ chọn Physical/Logical Drive
→ chọn E01/RAW/AFF
→ nhập thông tin evidence
→ chọn destination
→ Start
→ Verify
```

Các định dạng thường gặp:

```text
RAW
E01
AFF
SMART
```

Điểm quan trọng:

```text
Verify image
→ so sánh hash
→ xác nhận bản sao hợp lệ
```

---

# 4. Arsenal Image Mounter

Dùng để mount:

```text
E01
RAW
VMDK
VHD/VHDX
```

Nên mount:

```text
Read-only
```

Lý do:

```text
Không sửa đổi evidence
→ giữ integrity
→ phù hợp DFIR
```

Sau khi mount, image xuất hiện như một ổ đĩa Windows để KAPE hoặc công cụ khác phân tích.

---

# 5. Volatile Và Non-Volatile Evidence

## Volatile Evidence

Biến mất khi shutdown hoặc logoff:

```text
RAM
Running processes
Network connections
Injected code
Encryption keys
Command history
```

## Non-Volatile Evidence

Tồn tại trên disk:

```text
Registry
EVTX
Prefetch
Amcache
Browser history
IIS logs
File system artifacts
```

Nguyên tắc:

```text
Volatile first
→ Disk later
```

---

# 6. Memory Acquisition

Memory rất quan trọng trong:

- Malware analysis.
- Process injection.
- Credential theft.
- Fileless malware.
- Active network sessions.
- Decryption key recovery.

## Công cụ cần nhớ

| Công cụ | Hệ điều hành/Use case |
|---|---|
| WinPmem | Windows memory acquisition |
| DumpIt | Windows/Linux memory dump đơn giản |
| MemDump | CLI memory capture |
| Belkasoft RAM Capturer | Windows RAM capture |
| Magnet RAM Capture | Windows RAM capture |
| LiME | Linux memory acquisition |
| FTK Imager | Có thể capture memory trên Windows |

### WinPmem

```cmd
winpmem_mini_x64_rc2.exe memdump.raw
```

Output:

```text
memdump.raw
```

---

# 7. Thu Thập Memory Từ VM

Với VMware:

```text
Suspend VM
→ mở thư mục VM
→ lấy file .vmem
```

Các file quan trọng:

```text
.vmem = memory
.vmdk = virtual disk
.vmsn/.vmss = snapshot/suspend state
```

---

# 8. Rapid Triage

Rapid Triage nhằm:

```text
Thu thập artifact có giá trị cao
→ tập trung hóa dữ liệu
→ phân tích nhanh
→ xác định host ưu tiên
```

Thay vì tạo full disk image cho mọi máy, analyst có thể chỉ thu:

```text
Registry
EVTX
MFT
USN Journal
Prefetch
Amcache
Browser artifacts
Persistence artifacts
```

---

# 9. KAPE

KAPE = **Kroll Artifact Parser and Extractor**

Hai file thực thi:

```text
kape.exe  = CLI
gkape.exe = GUI
```

## Hai khái niệm chính

```text
Targets = thu thập artifact
Modules = xử lý/parse artifact
```

### Targets

File target có đuôi:

```text
.tkape
```

Ví dụ:

```text
RegistryHivesSystem.tkape
KapeTriage.tkape
!SANS_Triage
```

### Compound Targets

Gộp nhiều target để thu thập nhanh một lần.

Ví dụ có thể bao gồm:

```text
Antivirus
Event Logs
Evidence of Execution
Amcache
Registry
MFT
USN Journal
```

## KAPE CLI

Ví dụ:

```powershell
kape.exe `
  --tsource D: `
  --tdest C:\Investigation\Image `
  --target !SANS_Triage
```

Ý nghĩa:

```text
--tsource = nguồn dữ liệu
--tdest   = nơi lưu artifact
--target  = target/compound target
```

## KAPE Output

Có thể chứa:

```text
$MFT
$LogFile
$UsnJrnl
$Secure:$SDS
Users
Windows
ProgramData
EVTX
```

---

# 10. Velociraptor

Velociraptor dùng để thu thập từ xa và trên nhiều endpoint.

Khái niệm:

```text
VQL
Artifacts
Collections
Hunts
Clients
```

## KapeFiles Trong Velociraptor

Artifact:

```text
Windows.KapeFiles.Targets
```

Workflow:

```text
New Hunt
→ chọn Windows.KapeFiles.Targets
→ chọn target như _SANS_Triage
→ Launch
→ download collection
```

Output thường có:

```text
results
uploads
client_info.json
logs.json
requests.json
uploads.json
```

## Remote Memory Acquisition

Artifact:

```text
Windows.Memory.Acquisition
```

Output:

```text
PhysicalMemory.raw
```

---

# 11. EDR Trong Evidence Collection

EDR hỗ trợ:

- Remote file collection.
- Search IOC trên toàn mạng.
- Thu thập process tree.
- Tìm binary mới chạy.
- Lấy file đáng ngờ.
- Thu forensic package.
- Triage hàng loạt endpoint.

Ứng dụng SOC:

```text
IOC search
→ identify affected hosts
→ collect evidence
→ prioritize investigation
```

---

# 12. Network Evidence

## Packet Capture

Công cụ:

```text
Wireshark
tcpdump
Tshark
```

Giúp phân tích:

```text
C2 traffic
Data exfiltration
DNS activity
HTTP requests
SMB
Authentication
Payload transfer
```

## IDS/IPS Data

Nguồn:

```text
Snort
Suricata
Zeek
Network IDS/IPS
```

Có thể cung cấp:

```text
Alerts
Signatures
Protocol metadata
Connection logs
File logs
```

## Flow Data

```text
NetFlow
sFlow
```

Cho biết:

```text
Source IP
Destination IP
Port
Protocol
Bytes
Duration
Traffic pattern
```

## Firewall Logs

Dùng để xác định:

```text
Allowed/blocked traffic
Unauthorized access
Exploit attempts
C2 connections
Policy violations
```

---

# 13. Tool Mapping Theo Nhiệm Vụ

| Nhiệm vụ | Công cụ phù hợp |
|---|---|
| Tạo disk image | FTK Imager, AFF4 Imager, `dd`, `dcfldd` |
| Mount image | Arsenal Image Mounter |
| Capture RAM Windows | WinPmem, DumpIt, Belkasoft, Magnet |
| Capture RAM Linux | LiME |
| Rapid triage local | KAPE |
| Rapid triage remote | Velociraptor, EDR |
| Packet capture | Wireshark, tcpdump, Tshark |
| Network metadata | Zeek, NetFlow, sFlow |
| IDS/IPS evidence | Suricata, Snort |
| SIEM analysis | Splunk, ELK |

---

# 14. Workflow Thu Thập Chuẩn

```text
1. Ghi nhận thời gian và timezone
2. Thu volatile evidence trước
3. Capture memory
4. Tạo forensic image
5. Tính và ghi hash
6. Mount image read-only
7. Chạy KAPE rapid triage
8. Thu remote artifact bằng Velociraptor/EDR
9. Thu network evidence
10. Ghi chain of custody
```

---

# 15. Keyword Cần Nhớ

```text
Forensic Imaging
Bit-by-bit
FTK Imager
AFF4
dd
dcfldd
Arsenal Image Mounter
Read-only
Volatile Evidence
Non-Volatile Evidence
WinPmem
DumpIt
MemDump
Belkasoft RAM Capturer
Magnet RAM Capture
LiME
KAPE
Targets
Modules
.tkape
Compound Target
!SANS_Triage
Velociraptor
VQL
Hunt
Windows.KapeFiles.Targets
Windows.Memory.Acquisition
PhysicalMemory.raw
EDR
Wireshark
tcpdump
NetFlow
sFlow
IDS/IPS
Firewall Logs
```

---

# Key Takeaway

```text
Disk Image  → bảo toàn storage
Memory Dump → giữ volatile evidence
KAPE        → triage nhanh local
Velociraptor→ thu thập từ xa/hàng loạt
Network     → tái dựng communication
```

Trong CDSA:

```text
Evidence Acquisition
→ Incident Response
→ Memory/Disk Forensics
→ Threat Hunting
→ Detection Improvement
```

> Chỉ thu thập và phân tích trên hệ thống được cấp phép; ưu tiên read-only và ghi đầy đủ chain of custody.

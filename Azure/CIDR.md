# Complete Guide to CIDR Notation

## 📋 Table of Contents

- [What is CIDR?](#what-is-cidr)
- [CIDR Notation Basics](#cidr-notation-basics)
- [Understanding Subnet Masks](#understanding-subnet-masks)
- [CIDR to IP Range Conversion](#cidr-to-ip-range-conversion)
- [Common CIDR Blocks](#common-cidr-blocks)
- [Private IP Address Ranges](#private-ip-address-ranges)
- [CIDR Calculation Examples](#cidr-calculation-examples)
- [CIDR in Azure](#cidr-in-azure)
- [Subnetting with CIDR](#subnetting-with-cidr)
- [Best Practices](#best-practices)
- [CIDR Cheat Sheet](#cidr-cheat-sheet)
- [Troubleshooting Common Issues](#troubleshooting-common-issues)

---

## What is CIDR?

**CIDR** = **Classless Inter-Domain Routing**

### History & Purpose

**Before CIDR (Classful Networking):**
- Networks were divided into fixed classes (A, B, C)
- Class A: `/8` (16.7 million IPs) - Too large
- Class B: `/16` (65,536 IPs) - Limited availability
- Class C: `/24` (256 IPs) - Too small
- **Problem:** Inefficient IP allocation, rapid IPv4 exhaustion

**After CIDR (1993):**
- ✅ Flexible network sizing
- ✅ Efficient IP address allocation
- ✅ Reduced routing table size
- ✅ Better address space utilization

### Why CIDR Matters for Cloud

In Azure, AWS, and other cloud platforms:
- Defines Virtual Network (VNet) address space
- Determines how many resources can fit in a network
- Enables network segmentation and subnetting
- Controls IP allocation for VMs, containers, and services

---

## CIDR Notation Basics

### Format

```
IP_ADDRESS/PREFIX_LENGTH
```

**Example:** `10.0.0.0/16`

| Component | Value | Meaning |
|-----------|-------|---------|
| IP Address | `10.0.0.0` | Network address (base IP) |
| Prefix Length | `/16` | Number of bits used for network portion |

### Breaking Down 10.0.0.0/16

```
IP Address (Binary):
10  .  0  .  0  .  0
00001010.00000000.00000000.00000000

Prefix Length: /16
└─ First 16 bits are NETWORK
   └─ Last 16 bits are HOST (available for devices)

Network Portion: 10.0
Host Portion: 0.0 - 255.255
```

**Result:**
- **Network Address:** `10.0.0.0`
- **Broadcast Address:** `10.0.255.255`
- **Usable IP Range:** `10.0.0.1` - `10.0.255.254`
- **Total IPs:** 65,536
- **Usable IPs:** 65,534 (excluding network and broadcast)

---

## Understanding Subnet Masks

### What is a Subnet Mask?

A subnet mask separates the **network portion** from the **host portion** of an IP address.

### CIDR to Subnet Mask Conversion

| CIDR | Subnet Mask | Binary Representation |
|------|-------------|----------------------|
| `/8` | `255.0.0.0` | `11111111.00000000.00000000.00000000` |
| `/16` | `255.255.0.0` | `11111111.11111111.00000000.00000000` |
| `/24` | `255.255.255.0` | `11111111.11111111.11111111.00000000` |
| `/32` | `255.255.255.255` | `11111111.11111111.11111111.11111111` |

### How to Calculate

**Rule:** Each `/1` represents one bit set to `1` in the subnet mask.

**Example: /20**

```
20 bits set to 1:
11111111.11111111.11110000.00000000

Convert to decimal:
255.255.240.0
```

**Quick Reference:**

```
/8  = 255.0.0.0
/9  = 255.128.0.0
/10 = 255.192.0.0
/11 = 255.224.0.0
/12 = 255.240.0.0
/13 = 255.248.0.0
/14 = 255.252.0.0
/15 = 255.254.0.0
/16 = 255.255.0.0
/17 = 255.255.128.0
/18 = 255.255.192.0
/19 = 255.255.224.0
/20 = 255.255.240.0
/21 = 255.255.248.0
/22 = 255.255.252.0
/23 = 255.255.254.0
/24 = 255.255.255.0
/25 = 255.255.255.128
/26 = 255.255.255.192
/27 = 255.255.255.224
/28 = 255.255.255.240
/29 = 255.255.255.248
/30 = 255.255.255.252
/31 = 255.255.255.254
/32 = 255.255.255.255
```

---

## CIDR to IP Range Conversion

### Formula for Total IPs

```
Total IPs = 2^(32 - PREFIX_LENGTH)
```

**Example: /24**

```
Total IPs = 2^(32 - 24)
          = 2^8
          = 256 IPs
```

### Calculating Usable IPs

```
Usable IPs = Total IPs - 2
```

**Why subtract 2?**
- Network address (first IP) - Reserved
- Broadcast address (last IP) - Reserved

**Example: 192.168.1.0/24**

```
Total IPs: 256
Network Address: 192.168.1.0 (reserved)
Broadcast Address: 192.168.1.255 (reserved)
Usable Range: 192.168.1.1 - 192.168.1.254
Usable IPs: 254
```

### Step-by-Step Example: 172.16.0.0/20

**Step 1: Calculate Total IPs**

```
Total IPs = 2^(32 - 20) = 2^12 = 4,096
```

**Step 2: Determine Subnet Mask**

```
/20 = 255.255.240.0
```

**Step 3: Find Address Range**

```
Network: 172.16.0.0
Binary (3rd octet): 00000000
Network bits: 1111 (first 4 bits)
Host bits: 0000 (last 4 bits)

Last IP (3rd octet): 00001111 = 15
Last IP (4th octet): 11111111 = 255

Broadcast: 172.16.15.255
```

**Step 4: Determine Usable Range**

```
Network Address: 172.16.0.0
First Usable IP: 172.16.0.1
Last Usable IP: 172.16.15.254
Broadcast Address: 172.16.15.255
Total Usable IPs: 4,094
```

---

## Common CIDR Blocks

### Complete Reference Table

| CIDR | Subnet Mask | Total IPs | Usable IPs | Hosts | Typical Use |
|------|-------------|-----------|------------|-------|-------------|
| `/8` | 255.0.0.0 | 16,777,216 | 16,777,214 | 16.7M | Very large networks |
| `/9` | 255.128.0.0 | 8,388,608 | 8,388,606 | 8.4M | Large enterprises |
| `/10` | 255.192.0.0 | 4,194,304 | 4,194,302 | 4.2M | Large enterprises |
| `/11` | 255.224.0.0 | 2,097,152 | 2,097,150 | 2.1M | Large enterprises |
| `/12` | 255.240.0.0 | 1,048,576 | 1,048,574 | 1M | Large cloud deployments |
| `/13` | 255.248.0.0 | 524,288 | 524,286 | 524K | Cloud regions |
| `/14` | 255.252.0.0 | 262,144 | 262,142 | 262K | Cloud regions |
| `/15` | 255.254.0.0 | 131,072 | 131,070 | 131K | Large organizations |
| `/16` | 255.255.0.0 | 65,536 | 65,534 | 65K | **Common VNet size** |
| `/17` | 255.255.128.0 | 32,768 | 32,766 | 32K | Medium networks |
| `/18` | 255.255.192.0 | 16,384 | 16,382 | 16K | Medium networks |
| `/19` | 255.255.224.0 | 8,192 | 8,190 | 8K | Departments |
| `/20` | 255.255.240.0 | 4,096 | 4,094 | 4K | Departments |
| `/21` | 255.255.248.0 | 2,048 | 2,046 | 2K | Large subnets |
| `/22` | 255.255.252.0 | 1,024 | 1,022 | 1K | Large subnets |
| `/23` | 255.255.254.0 | 512 | 510 | 512 | Medium subnets |
| `/24` | 255.255.255.0 | 256 | 254 | 256 | **Common subnet** |
| `/25` | 255.255.255.128 | 128 | 126 | 128 | Small subnets |
| `/26` | 255.255.255.192 | 64 | 62 | 64 | Small subnets |
| `/27` | 255.255.255.224 | 32 | 30 | 32 | Very small subnets |
| `/28` | 255.255.255.240 | 16 | 14 | 16 | Tiny subnets |
| `/29` | 255.255.255.248 | 8 | 6 | 8 | Point-to-point |
| `/30` | 255.255.255.252 | 4 | 2 | 4 | **Point-to-point links** |
| `/31` | 255.255.255.254 | 2 | 2 | 2 | Point-to-point (RFC 3021) |
| `/32` | 255.255.255.255 | 1 | 1 | 1 | **Single host** |

---

## Private IP Address Ranges

### RFC 1918 - Private Address Space

These IP ranges are reserved for private networks and **NOT routable on the internet**.

| Class | CIDR Block | IP Range | Total IPs | Common Use |
|-------|------------|----------|-----------|------------|
| **Class A** | `10.0.0.0/8` | 10.0.0.0 - 10.255.255.255 | 16,777,216 | Large private networks |
| **Class B** | `172.16.0.0/12` | 172.16.0.0 - 172.31.255.255 | 1,048,576 | Medium private networks |
| **Class C** | `192.168.0.0/16` | 192.168.0.0 - 192.168.255.255 | 65,536 | Small private networks |

### Azure-Specific Recommendations

**For Azure VNets:**

| VNet Size | CIDR | Total IPs | Use Case |
|-----------|------|-----------|----------|
| **Small** | `10.0.0.0/24` | 256 | Dev/Test environments |
| **Medium** | `10.0.0.0/16` | 65,536 | **Recommended** - Production VNets |
| **Large** | `10.0.0.0/8` | 16.7M | Enterprise-wide deployments |

**Example VNet Planning:**

```
VNet: 10.0.0.0/16 (65,536 IPs)
├─ Subnet 1: 10.0.1.0/24 (256 IPs) - Web Servers
├─ Subnet 2: 10.0.2.0/24 (256 IPs) - App Servers
├─ Subnet 3: 10.0.3.0/24 (256 IPs) - Database Servers
├─ Subnet 4: 10.0.4.0/24 (256 IPs) - Management
└─ Reserved: 10.0.5.0 - 10.0.255.255 (Future expansion)
```

---

## CIDR Calculation Examples

### Example 1: Calculate Range for 192.168.10.0/28

**Step 1: Find Total IPs**

```
Total IPs = 2^(32 - 28) = 2^4 = 16
```

**Step 2: Find Subnet Mask**

```
/28 = 255.255.255.240
```

**Step 3: Calculate Range**

```
Last octet breakdown:
Network bits: 1111 (first 4 bits)
Host bits: 0000 (last 4 bits)

Network: 192.168.10.0
First IP: 192.168.10.1
Last IP: 192.168.10.14
Broadcast: 192.168.10.15

Usable IPs: 14
```

**Visual Representation:**

```
192.168.10.0    ← Network (reserved)
192.168.10.1    ← First usable
192.168.10.2
192.168.10.3
...
192.168.10.14   ← Last usable
192.168.10.15   ← Broadcast (reserved)
```

---

### Example 2: How Many /24 Subnets in 10.0.0.0/16?

**Step 1: Calculate Total IPs in /16**

```
/16 = 2^(32 - 16) = 2^16 = 65,536 IPs
```

**Step 2: Calculate IPs per /24 Subnet**

```
/24 = 2^(32 - 24) = 2^8 = 256 IPs
```

**Step 3: Divide**

```
Number of /24 subnets = 65,536 ÷ 256 = 256 subnets
```

**Result:** You can fit **256 /24 subnets** in a /16 network.

**Example Allocation:**

```
10.0.0.0/24   (10.0.0.0 - 10.0.0.255)
10.0.1.0/24   (10.0.1.0 - 10.0.1.255)
10.0.2.0/24   (10.0.2.0 - 10.0.2.255)
...
10.0.255.0/24 (10.0.255.0 - 10.0.255.255)
```

---

### Example 3: Split 172.16.0.0/16 into Four Equal Parts

**Step 1: Determine New Prefix**

```
/16 split into 4 = /18
(Each subnet gets 2 bits: 2^2 = 4 subnets)
```

**Step 2: Calculate IPs per Subnet**

```
/18 = 2^(32 - 18) = 2^14 = 16,384 IPs
```

**Step 3: Calculate Ranges**

```
Subnet 1: 172.16.0.0/18   → 172.16.0.0 - 172.16.63.255
Subnet 2: 172.16.64.0/18  → 172.16.64.0 - 172.16.127.255
Subnet 3: 172.16.128.0/18 → 172.16.128.0 - 172.16.191.255
Subnet 4: 172.16.192.0/18 → 172.16.192.0 - 172.16.255.255
```

**Verification:**

```
4 subnets × 16,384 IPs = 65,536 IPs ✓
```

---

## CIDR in Azure

### Azure VNet CIDR Requirements

**Allowed CIDR Ranges:**
- ✅ `/8` to `/29` for VNets
- ✅ Private IP ranges (RFC 1918) recommended
- ✅ Can use public IPs (not recommended for security)

**Azure Subnet Requirements:**
- Minimum: `/29` (8 IPs)
- Maximum: Same as VNet size
- Azure reserves **5 IPs** per subnet:
  - `.0` - Network address
  - `.1` - Azure default gateway
  - `.2` - `.3` - Azure DNS mapping
  - `.255` - Network broadcast

**Example: 10.0.1.0/24 Subnet**

```
Total IPs: 256
Azure Reserved: 5
└─ 10.0.1.0   (Network)
└─ 10.0.1.1   (Gateway)
└─ 10.0.1.2   (DNS)
└─ 10.0.1.3   (DNS)
└─ 10.0.1.255 (Broadcast)

Available for VMs: 251 IPs
└─ 10.0.1.4 - 10.0.1.254
```

### Azure VNet Planning Example

**Scenario:** Create a VNet for a 3-tier application

```
VNet: 10.10.0.0/16 (65,536 IPs)

Subnets:
├─ Web Tier:        10.10.1.0/24   (251 usable)
├─ App Tier:        10.10.2.0/24   (251 usable)
├─ Database Tier:   10.10.3.0/24   (251 usable)
├─ Management:      10.10.4.0/27   (27 usable)
├─ Gateway Subnet:  10.10.5.0/27   (27 usable)
└─ Reserved:        10.10.6.0/18   (Future expansion)
```

---

## Subnetting with CIDR

### Subnetting Formula

```
Number of Subnets = 2^(New_Prefix - Old_Prefix)
IPs per Subnet = 2^(32 - New_Prefix)
```

### Example: Subnet 192.168.1.0/24 into 4 Subnets

**Step 1: Calculate New Prefix**

```
4 subnets = 2^2
Need 2 additional bits
/24 + 2 = /26
```

**Step 2: Calculate IPs per Subnet**

```
/26 = 2^(32 - 26) = 2^6 = 64 IPs
Usable: 62 IPs
```

**Step 3: Create Subnet Table**

| Subnet | CIDR | Network | First Usable | Last Usable | Broadcast |
|--------|------|---------|--------------|-------------|-----------|
| 1 | `192.168.1.0/26` | 192.168.1.0 | 192.168.1.1 | 192.168.1.62 | 192.168.1.63 |
| 2 | `192.168.1.64/26` | 192.168.1.64 | 192.168.1.65 | 192.168.1.126 | 192.168.1.127 |
| 3 | `192.168.1.128/26` | 192.168.1.128 | 192.168.1.129 | 192.168.1.190 | 192.168.1.191 |
| 4 | `192.168.1.192/26` | 192.168.1.192 | 192.168.1.193 | 192.168.1.254 | 192.168.1.255 |

### Variable Length Subnet Masking (VLSM)

Create subnets of different sizes from the same network.

**Example: 10.0.0.0/16**

```
Requirements:
- Department A: 500 hosts
- Department B: 200 hosts
- Department C: 50 hosts
- Point-to-point links: 2 hosts each

Solution:
├─ Dept A: 10.0.0.0/23  (512 IPs, 510 usable) ✓
├─ Dept B: 10.0.2.0/24  (256 IPs, 254 usable) ✓
├─ Dept C: 10.0.3.0/26  (64 IPs, 62 usable) ✓
├─ Link 1: 10.0.3.64/30 (4 IPs, 2 usable) ✓
├─ Link 2: 10.0.3.68/30 (4 IPs, 2 usable) ✓
└─ Reserved: 10.0.4.0 - 10.0.255.255 (Future)
```

---

## Best Practices

### 1. VNet CIDR Selection

**✅ DO:**
- Use `/16` for production VNets (65K IPs)
- Plan for growth - allocate larger than current need
- Use private IP ranges (RFC 1918)
- Document IP allocation strategy
- Leave room for subnetting

**❌ DON'T:**
- Use overlapping CIDR blocks
- Choose too small a range (forces migration later)
- Use public IPs for private resources
- Allocate entire address space immediately

### 2. Subnet Planning

**✅ DO:**
- Use `/24` for most subnets (251 usable in Azure)
- Reserve address space for future subnets
- Group similar resources in same subnet
- Use naming conventions (web-subnet, db-subnet)
- Plan for Azure reserved IPs (5 per subnet)

**❌ DON'T:**
- Create too many small subnets
- Use `/29` unless absolutely necessary
- Forget to account for Azure reserved IPs
- Mix different security zones

### 3. CIDR Block Sizing Guide

| Requirement | Recommended CIDR | Total IPs | Usable (Azure) |
|-------------|------------------|-----------|----------------|
| Single Host | `/32` | 1 | 1 |
| Point-to-point | `/30` | 4 | 2 |
| Small subnet | `/27` | 32 | 27 |
| Standard subnet | `/24` | 256 | 251 |
| Large subnet | `/20` | 4,096 | 4,091 |
| Standard VNet | `/16` | 65,536 | - |
| Large VNet | `/8` | 16.7M | - |

### 4. Avoiding Common Mistakes

**Overlapping Ranges:**

```
❌ BAD:
VNet 1: 10.0.0.0/16
VNet 2: 10.0.5.0/24  ← Overlaps with VNet 1!

✅ GOOD:
VNet 1: 10.0.0.0/16
VNet 2: 10.1.0.0/16  ← No overlap
```

**Insufficient Planning:**

```
❌ BAD:
VNet: 10.0.0.0/24 (256 IPs only)
└─ Cannot add more subnets!

✅ GOOD:
VNet: 10.0.0.0/16 (65,536 IPs)
├─ Current: 10.0.0.0/20 (4,096 IPs)
└─ Reserved: 10.0.16.0/20 - 10.0.240.0/20
```

---

## CIDR Cheat Sheet

### Quick Calculation Reference

**Powers of 2 (for quick calculation):**

```
2^1  = 2
2^2  = 4
2^3  = 8
2^4  = 16
2^5  = 32
2^6  = 64
2^7  = 128
2^8  = 256
2^9  = 512
2^10 = 1,024
2^11 = 2,048
2^12 = 4,096
2^13 = 8,192
2^14 = 16,384
2^15 = 32,768
2^16 = 65,536
```

### Binary to Decimal (Subnet Masks)

```
Binary          Decimal
00000000    =   0
10000000    =   128
11000000    =   192
11100000    =   224
11110000    =   240
11111000    =   248
11111100    =   252
11111110    =   254
11111111    =   255
```

### Common Subnet Masks

```
/8  = 255.0.0.0       = 16,777,216 IPs
/16 = 255.255.0.0     = 65,536 IPs
/24 = 255.255.255.0   = 256 IPs
/30 = 255.255.255.252 = 4 IPs
/32 = 255.255.255.255 = 1 IP
```

---

## Troubleshooting Common Issues

### Issue 1: IP Address Overlap

**Problem:**

```
VNet A: 10.0.0.0/16
VNet B: 10.0.100.0/24

Error: Cannot peer - overlapping address space
```

**Solution:**

```
VNet A: 10.0.0.0/16  (10.0.0.0 - 10.0.255.255)
VNet B: 10.1.0.0/16  (10.1.0.0 - 10.1.255.255)
```

### Issue 2: Insufficient IP Addresses

**Problem:**

```
Subnet: 10.0.1.0/27 (32 IPs)
Azure Reserved: 5
Available: 27
Need: 30 VMs

Error: Not enough IPs
```

**Solution:**

```
Expand to /26:
Subnet: 10.0.1.0/26 (64 IPs)
Azure Reserved: 5
Available: 59 ✓
```

### Issue 3: Invalid CIDR Format

**Problem:**

```
❌ 10.0.0.0/33  (prefix too large)
❌ 10.0.0.1/24  (not a network address)
❌ 256.0.0.0/16 (invalid IP)
```

**Solution:**

```
✅ 10.0.0.0/24  (valid network)
✅ 10.0.0.0/16  (valid network)
✅ 192.168.1.0/24 (valid network)
```

### Issue 4: Subnet Too Small

**Problem:**

```
Azure subnet: 10.0.1.0/29 (8 IPs)
Azure reserved: 5
Available: 3

Need 5 VMs → Error
```

**Solution:**

```
Use /27 minimum for Azure subnets:
Subnet: 10.0.1.0/27 (32 IPs)
Azure reserved: 5
Available: 27 ✓
```

---

## Practice Exercises

### Exercise 1: Calculate Range

**Question:** What is the usable IP range for `172.16.50.0/22`?

<details>
<summary>Click to see answer</summary>

**Answer:**

```
Total IPs = 2^(32-22) = 2^10 = 1,024

Subnet Mask: /22 = 255.255.252.0

Network: 172.16.50.0
Last IP calculation:
- 3rd octet: 50 + (1024/256) - 1 = 50 + 4 - 1 = 53
- 4th octet: 255

Network: 172.16.50.0
First Usable: 172.16.50.1
Last Usable: 172.16.53.254
Broadcast: 172.16.53.255

Usable IPs: 1,022
```

</details>

---

### Exercise 2: Subnetting

**Question:** Divide `192.168.0.0/24` into 8 equal subnets. What are the ranges?

<details>
<summary>Click to see answer</summary>

**Answer:**

```
8 subnets = 2^3 (need 3 bits)
/24 + 3 = /27

Each subnet: 2^(32-27) = 2^5 = 32 IPs

Subnets:
1. 192.168.0.0/27   (0-31)
2. 192.168.0.32/27  (32-63)
3. 192.168.0.64/27  (64-95)
4. 192.168.0.96/27  (96-127)
5. 192.168.0.128/27 (128-159)
6. 192.168.0.160/27 (160-191)
7. 192.168.0.192/27 (192-223)
8. 192.168.0.224/27 (224-255)
```

</details>

---

### Exercise 3: Azure Planning

**Question:** You need 100 VMs in Azure. What's the minimum subnet size?

<details>
<summary>Click to see answer</summary>

**Answer:**

```
Required IPs:
- VMs: 100
- Azure reserved: 5
- Total needed: 105

Minimum CIDR:
- /25 = 128 IPs → 128 - 5 = 123 usable ✓

Recommended:
- /24 = 256 IPs → 256 - 5 = 251 usable
  (leaves room for growth)

Answer: /24 (recommended) or /25 (minimum)
```

</details>

---

## Online CIDR Calculators

### Recommended Tools

- [CIDR.xyz](https://cidr.xyz/) - Visual CIDR calculator
- [IPAddressGuide CIDR Calculator](https://www.ipaddressguide.com/cidr)
- [Subnet Calculator](https://www.calculator.net/ip-subnet-calculator.html)
- [Visual Subnet Calculator](https://www.davidc.net/sites/default/subnets/subnets.html)

### Azure-Specific Tools

- [Azure IP Range Visualizer](https://azureipranges.azurewebsites.net/)

<div align="center">

# 🔐 Standard ACL: Host-Based Source Filtering

### ACL Series · Lab 1 · Cisco Packet Tracer

![Cisco](https://img.shields.io/badge/Cisco-IOS-1BA0D7?style=for-the-badge&logo=cisco&logoColor=white)
![Packet Tracer](https://img.shields.io/badge/Packet%20Tracer-Lab-049fd9?style=for-the-badge)
![ACL](https://img.shields.io/badge/Standard-ACL-red?style=for-the-badge)
![Level](https://img.shields.io/badge/Level-CCNA-green?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Verified-success?style=for-the-badge)

*Allow exactly one host. Block everyone else.*

</div>

---

## 📑 Table of Contents
1. [Overview](#-overview)
2. [Topology](#-topology)
3. [Addressing Table](#-addressing-table)
4. [Standard ACL Theory](#-standard-acl-theory)
5. [Configuration](#-configuration)
6. [Verification: Stage by Stage](#-verification-stage-by-stage)
7. [Test Matrix](#-test-matrix)
8. [Why Replies Matter](#-why-the-replies-matter)
9. [Key Takeaways](#-key-takeaways)
10. [Troubleshooting Notes](#-troubleshooting-notes)
11. [Next Labs](#-next-labs-in-the-acl-series)
12. [Author](#-author)

---

## 🎯 Overview

This is the first lab of my **Access Control List (ACL)** series. A **Standard ACL** is the simplest ACL type. It makes its decision on the **source IP address only**.

**Goal:** On R1, allow only the server `200.100.100.4` to send traffic in from the WAN, and block every other source.

| Item | Detail |
|---|---|
| ACL type | Standard, numbered (ACL 1) |
| Match method | `host` keyword (exact single source) |
| Applied on | R1, Serial0/0/0 |
| Direction | **Inbound** |
| Protected network | 192.168.1.0/24 |
| Only permitted source | 200.100.100.4 (Server-PT) |

---

## 🗺 Topology

```
                         1.0.0.0 (Serial WAN)
        1.1.1.1  ●━━━━━━━━━━━━━━⚡━━━━━━━━━━━━━━●  1.1.1.2
              ┌──────┐                          ┌──────┐
              │  R1  │                          │  R2  │
              │ 1941 │                          │ 1941 │
              └──┬───┘                          └──┬───┘
        192.168.1.1/24                       200.100.100.1/24
                 │                                 │
            ┌────┴────┐                       ┌────┴────┐
            │ Switch0 │                       │ Switch1 │
            │ 2950-24 │                       │ 2950-24 │
            └─┬──┬──┬─┘                       └─┬──┬──┬─┘
              │  │  │                           │  │  │
        ┌─────┘  │  └─────┐             ┌───────┘  │  └───────┐
     PC-PT     PC-PT    PC-PT        PC-PT      PC-PT     Server-PT
  192.168.1.2 .1.3    .1.4         200.100.100.2  .100.3   200.100.100.4
                                                          (✅ PERMITTED)

   R1 Se0/0/0  ──►  ip access-group 1 in
```

---

## 📋 Addressing Table

| Device | Interface | IP Address | Role |
|---|---|---|---|
| R1 (1941) | Serial0/0/0 | 1.1.1.1 | WAN link (ACL applied here) |
| R1 (1941) | LAN interface | 192.168.1.1/24 | Gateway, left LAN |
| R2 (1941) | Serial | 1.1.1.2 | WAN link |
| R2 (1941) | LAN interface | 200.100.100.1/24 | Gateway, right LAN |
| PC-PT | NIC | 192.168.1.2/24 | Left LAN |
| PC-PT | NIC | 192.168.1.3/24 | Left LAN |
| PC-PT | NIC | 192.168.1.4/24 | Left LAN |
| PC-PT | NIC | 200.100.100.2/24 | Right LAN |
| PC-PT | NIC | 200.100.100.3/24 | Right LAN |
| Server-PT | NIC | **200.100.100.4/24** | ✅ Permitted host |

---

## 📚 Standard ACL Theory

| Property | Standard ACL |
|---|---|
| Number ranges | 1–99 and 1300–1999 |
| Matches on | **Source IP only** |
| Destination / protocol / port | ❌ Not checked |
| Best placement | **Close to the destination** |
| Wildcard for one host | `0.0.0.0` (same as `host`) |
| Implicit rule | `deny any` at the end of every ACL |
| Processing | Top-down, first match wins |

### `host` keyword

```
access-list 1 permit host 200.100.100.4
                =
access-list 1 permit 200.100.100.4 0.0.0.0
```

### Why close to the destination?
A Standard ACL cannot see the destination. If it were placed near the source, it would block that source from **everything**. Placing it on R1's WAN-facing interface, right next to the protected LAN, limits the damage.

---

## ⚙️ Configuration

### R1 (the ACL)

```
R1(config)# access-list 1 permit host 200.100.100.4
R1(config)# access-list 1 deny any

R1(config)# interface serial 0/0/0
R1(config-if)# ip access-group 1 in
R1(config-if)# end
```

### Routing between the two LANs
> Reachability between the LANs is required for the test. Replace this block with the routing you actually used.

```
R1(config)# ip route 200.100.100.0 255.255.255.0 1.1.1.2
R2(config)# ip route 192.168.1.0   255.255.255.0 1.1.1.1
```

### Config breakdown

| Command | Purpose |
|---|---|
| `access-list 1 permit host 200.100.100.4` | Allow packets whose **source** is the server |
| `access-list 1 deny any` | Explicit deny for everything else (makes hit counters visible) |
| `interface serial 0/0/0` | Enter the WAN interface |
| `ip access-group 1 in` | Check packets **arriving** on this interface |

---

## ✅ Verification: Stage by Stage

### 🔎 Stage 1: Confirm the ACL (run on R1)

```
R1# show access-lists
R1# show ip interface serial 0/0/0 | include access list
```

Expected:
```
Standard IP access list 1
    10 permit host 200.100.100.4 (x match(es))
    20 deny any (x match(es))
  Inbound  access list is 1
```
> 📸 Paste your own counters here after testing.

---

### 🟢 Stage 2: PC0 (192.168.1.x) → Server 200.100.100.4 → ALLOWED

```
C:\> ping 200.100.100.4

Reply from 200.100.100.4: bytes=32 time=21ms TTL=126
Reply from 200.100.100.4: bytes=32 time=1ms  TTL=126
Reply from 200.100.100.4: bytes=32 time=1ms  TTL=126
Reply from 200.100.100.4: bytes=32 time=1ms  TTL=126

Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
Minimum = 1ms, Maximum = 21ms, Average = 6ms
```

**Annotations**
- ✅ The reply's source is `200.100.100.4`, which matches `permit host`.
- **TTL = 126**: Windows-style hosts start at 128, minus 2 routers (R2 and R1) = 126. This confirms the packet crossed both routers.
- The first reply (21 ms) is slower because of ARP and initial path setup. Later replies take 1 ms.

---

### 🟢 Stage 3: PC1 → Server 200.100.100.4 → ALLOWED

```
C:\> ping 200.100.100.4

Reply from 200.100.100.4: bytes=32 time=1ms TTL=126  (x4)

Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

**Annotation:** A second left-LAN PC shows the same result. The ACL permits the server's replies for **any** host in 192.168.1.0/24.

---

### 🔴 Stage 4: PC0 → PC 200.100.100.3 → BLOCKED

```
C:\> ping 200.100.100.3

Request timed out.
Request timed out.
Request timed out.
Request timed out.

Packets: Sent = 4, Received = 0, Lost = 4 (100% loss)
```

**Annotation**
- The echo request reaches 200.100.100.3, which sends an echo reply.
- That reply has source **200.100.100.3**, so it hits R1 Se0/0/0 inbound and matches only `deny any`.
- R1 drops it, so the sender sees a **timeout**.

---

### 🔴 Stage 5: PC1 → PC 200.100.100.2 → BLOCKED

```
C:\> ping 200.100.100.2

Request timed out.  (x4)

Packets: Sent = 4, Received = 0, Lost = 4 (100% loss)
```

**Annotation:** This is the same cause as Stage 4. Only the server is whitelisted.

---

### 🔴 Stage 6: PC3 (right LAN) → left LAN → BLOCKED

```
C:\> ping 192.168.1.2
Reply from 1.1.1.1: Destination host unreachable.  (x4)
Packets: Sent = 4, Received = 0, Lost = 4 (100% loss)

C:\> ping 192.168.1.4
Reply from 1.1.1.1: Destination host unreachable.  (x4)
Packets: Sent = 4, Received = 0, Lost = 4 (100% loss)

C:\> ping 192.168.1.3
Reply from 1.1.1.1: Destination host unreachable.  (x4)
Packets: Sent = 4, Received = 0, Lost = 4 (100% loss)
```

**Annotations**
- This time the **request itself** (source 200.100.100.3) is denied as it enters R1.
- R1 sends an ICMP unreachable back to the sender. In Packet Tracer this appears as "Destination host unreachable" from **1.1.1.1**, R1's serial interface.
- Compare Stage 4 and Stage 6: a **timeout** means the reply was dropped silently on the way back, while an **unreachable from the router** means the request was denied on the way in.

---

## 🧪 Test Matrix

| # | Source | Destination | Result | Why |
|---|---|---|---|---|
| 1 | PC0 (192.168.1.x) | Server 200.100.100.4 | ✅ 4/4, TTL 126 | Reply source matches `permit host` |
| 2 | PC1 (192.168.1.x) | Server 200.100.100.4 | ✅ 4/4, TTL 126 | Same as above |
| 3 | PC0 (192.168.1.x) | 200.100.100.3 | ❌ Timeout | Reply source denied inbound on R1 |
| 4 | PC1 (192.168.1.x) | 200.100.100.2 | ❌ Timeout | Reply source denied inbound on R1 |
| 5 | PC3 (200.100.100.x) | 192.168.1.2 / .3 / .4 | ❌ Unreachable (from 1.1.1.1) | Request source denied inbound on R1 |

---

## 🔁 Why the Replies Matter

```
 LEFT PC ──echo request──► R1 ──► R2 ──► 200.100.100.3
                                              │
 LEFT PC ◄──── ✖ DROPPED ◄── R1 Se0/0/0 (in) ◄┘
                  echo reply, source 200.100.100.3 → matches "deny any"
```

The ACL has no connection tracking. It judges every packet that arrives on Se0/0/0 by its **source address** alone. The left PC can send the request out, but the reply is evaluated separately, and it is denied. This is how ACLs work, not a bug.

---

## 💡 Key Takeaways

| # | Takeaway |
|---|---|
| 1 | Standard ACLs match the **source IP only** |
| 2 | `host x.x.x.x` is the same as wildcard `0.0.0.0`, an exact single-host match |
| 3 | Place Standard ACLs **close to the destination** |
| 4 | Direction matters: `in` checks packets arriving, `out` checks packets leaving |
| 5 | ACLs filter **return traffic** as well as requests |
| 6 | Every ACL ends with an implicit `deny any`, and writing it explicitly makes counters visible |
| 7 | A timeout versus an ICMP unreachable tells you **where** the packet was dropped |
| 8 | TTL 126 proves the permitted path crossed two routers |

---

## 🛠 Troubleshooting Notes

| Symptom | Check |
|---|---|
| Server also blocked | Is the permit line first? Is the IP typed correctly? |
| Everything allowed | Is the ACL applied (`show ip interface`)? Is the direction right? |
| Nothing works, even the server | Is routing between the LANs in place? Is the interface up? |
| No hit counters | Is the traffic actually crossing that interface in that direction? |

---

## 🚀 Next Labs in the ACL Series

- [x] Standard ACL: host matching ✅
- [ ] Standard ACL: network / wildcard mask ranges
- [ ] Standard ACL: applied outbound and placement comparison
- [ ] Named Standard ACL (edit with sequence numbers)
- [ ] Extended ACL
- [ ] VTY (Telnet/SSH) access restriction with ACL

---

## 👨‍💻 Author

**Maaz Khan**
CCNA-certified Network & NOC Engineer · Lower Dir, KPK, Pakistan

[![LinkedIn](https://img.shields.io/badge/LinkedIn-maazkhanms-0A66C2?style=for-the-badge&logo=linkedin)](https://www.linkedin.com/in/maazkhanms)
[![GitHub](https://img.shields.io/badge/GitHub-maazkhanms-181717?style=for-the-badge&logo=github)](https://github.com/maazkhanms)

⭐ If this lab helped you, give the repo a star!

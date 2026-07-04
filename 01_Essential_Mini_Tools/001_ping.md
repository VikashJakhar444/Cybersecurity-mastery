# Ping

> Category: Essential Mini Tools
> Difficulty: ⭐ Beginner
> Status: ✅ Practically Completed

---

# 1. Introduction

`ping` ek command-line networking utility hai jo kisi host (IP ya Domain) ki reachability check karne ke liye use hoti hai.

Ye **ICMP (Internet Control Message Protocol)** ka use karke target ko **Echo Request** bhejta hai aur **Echo Reply** ka wait karta hai.

Agar reply mil jaye to generally iska matlab target reachable hai (agar ICMP blocked na ho).

Simple Flow:

Your Computer
        │
        │  ICMP Echo Request
        ▼
Target Machine
        │
        │  ICMP Echo Reply
        ▼
Your Computer

---

# 2. Why We Use It?

Ping almost har Network Engineer, System Administrator aur Cybersecurity Professional ka pehla tool hota hai.

Iska use:

- Check whether a host is reachable.
- Verify network connectivity.
- Measure latency (RTT).
- Detect packet loss.
- Perform initial troubleshooting before using advanced tools.

---

# 3. Syntax

Basic Syntax:

```bash
ping <IP Address>
```

or

```bash
ping <Hostname>
```

Examples:

```bash
ping 192.168.1.10
```

```bash
ping google.com
```

---

# 4. Common Options

Run only 4 packets

```bash
ping -c 4 8.8.8.8
```

Force IPv4

```bash
ping -4 localhost
```

Force IPv6

```bash
ping -6 localhost
```

Custom Packet Size

```bash
ping -s 1000 8.8.8.8
```

Interval Between Packets(time period between each packet)

```bash
ping -i 2 8.8.8.8
```

Timeout(here it will wait for 2 sec)

```bash
ping -W 2 8.8.8.8
```

Flood Mode (Only in Lab)     #no need to use 

```bash
sudo ping -f TARGET
```

---

# 5. Basic Examples

## Example 1 — Ping Loopback Address

Command

```bash 
ping 127.0.0.1                                 # while running in linux, use ctr+c to stop pinging
```

Output

```text
PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.

64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=1.58 ms
64 bytes from 127.0.0.1: icmp_seq=2 ttl=64 time=0.043 ms
64 bytes from 127.0.0.1: icmp_seq=3 ttl=64 time=0.037 ms
64 bytes from 127.0.0.1: icmp_seq=4 ttl=64 time=0.211 ms
...

--- 127.0.0.1 ping statistics ---

11 packets transmitted, 11 received, 0% packet loss

rtt min/avg/max/mdev = 0.037/0.201/1.581/0.438 ms
```

### Output Explanation

#### `PING 127.0.0.1 (127.0.0.1)`

Target IP address.

---

#### `56(84) bytes of data`

Yeh beginners ko confuse karta hai.

Iska matlab:

```
56 Bytes
```

Actual ICMP payload.

Aur

```
84 Bytes
```

Total packet size.

Calculation:

```
56 Bytes (Payload)

+

8 Bytes (ICMP Header)

+

20 Bytes (IPv4 Header)

=

84 Bytes
```

Isliye Linux

```
56(84)
```

show karta hai.

---

#### `64 bytes from`

Matlab reply successfully receive ho gaya.

Request gayi aur target ne response diya.

---

#### `icmp_seq`

```text
icmp_seq=1
icmp_seq=2
icmp_seq=3
```

Packet numbering.

Agar hota

```text
1
2
4
5
```

To packet number 3 lost ho gaya.

---

#### `ttl=64`

TTL = Time To Live

Ye packet kitne routers survive kar sakta hai.

Typical Initial TTL

Linux → 64

Windows → 128

Routers → 255

Dhyan rahe:

Received TTL se exact OS identify nahi hota.

Ye sirf ek rough hint hai.

---

#### `time=0.043 ms`

Round Trip Time (RTT)

Request gayi

↓

Reply aaya

↓

Total Time

```
0.043 milliseconds
```

Itna kam isliye hai kyunki packet network me gaya hi nahi.

Ye sirf tumhari machine ke andar hi travel hua.

---

#### Statistics

```text
11 packets transmitted
```

11 packets bheje gaye.

---

```text
11 received
```

11 replies receive hui.

---

```text
0% packet loss
```

Ek bhi packet lose nahi hua.

Excellent.

---

#### RTT Summary

```text
rtt min/avg/max/mdev
```

| Value | Meaning |
|---------|---------|
| min | Fastest reply |
| avg | Average reply |
| max | Slowest reply |
| mdev | Response variation |

Agar `mdev` bahut high ho to unstable network indicate kar sakta hai.

---

### Important Learning

`127.0.0.1` ko Loopback Address kehte hain.

Ye tumhari khud ki machine hoti hai.

Packet kabhi network me nahi jata.

Flow:

Your PC

↓

Kernel / Network Stack

↓

Reply

---

## Example 2 — Ping Localhost

Command

```bash
ping localhost
```

Output

```text
PING localhost (::1)
```

Question:

127.0.0.1 kyu nahi aaya?

Reason:

`localhost` IP address nahi hai.

Ye hostname hai.

Hostname resolve hota hai:

```bash
cat /etc/hosts
```

Output

```text
127.0.0.1 localhost
127.0.1.1 kali
::1 localhost ip6-localhost ip6-loopback
```

Modern Linux systems generally IPv6 prefer karte hain.

Isliye

```
localhost

↓

::1
```

resolve hua.

---

## Example 3 — Force IPv4

Command

```bash
ping -4 localhost -c 4
```

Output

```text
PING localhost (127.0.0.1)
```

Meaning

OS ko force kiya gaya ki sirf IPv4 use kare.

---

## Example 4 — Force IPv6

Command

```bash
ping -6 localhost -c 4
```

Output

```text
PING localhost (::1)
```

Meaning

OS ko force kiya gaya ki sirf IPv6 use kare.

---

## Example 5 — Internet Connectivity

Command

```bash
ping -c 4 8.8.8.8
```

Output (your lab)

```text
64 bytes from 8.8.8.8:
icmp_seq=1 ttl=118 time=83.9 ms
icmp_seq=2 ttl=118 time=79.3 ms
icmp_seq=3 ttl=118 time=78.8 ms
icmp_seq=4 ttl=118 time=90.8 ms
```

### Explanation

Ab packet machine ke andar nahi raha.

Actual Flow

Your PC

↓

Router

↓

ISP

↓

Internet

↓

Google DNS (8.8.8.8)

↓

Back to Your PC

Isi wajah se RTT

```
80+ ms
```

dikha.

---

# 6. Real-World Usage

### Scenario 1

```bash
ping 127.0.0.1
```

Fail

Meaning:

Local networking stack problem.

---

### Scenario 2

```bash
ping 127.0.0.1
```

Works

↓

```bash
ping 192.168.1.1
```

Fails

Meaning:

LAN / Router issue.

---

### Scenario 3

```bash
ping 192.168.1.1
```

Works

↓

```bash
ping 8.8.8.8
```

Fails

Meaning:

Internet / ISP issue.

---

### Scenario 4

```bash
ping 8.8.8.8
```

Works

↓

```bash
ping google.com
```

Fails

Meaning:

DNS resolution problem.

---

### Scenario 5

```bash
ping google.com
```

Works

Meaning:

Everything is working correctly.

---

# 7. Practical Lab

(Ye section exactly wahi hoga jo humne class me kiya.)

- ✅ `ping 127.0.0.1`
- ✅ `ping localhost`
- ✅ `cat /etc/hosts`
- ✅ `ping -4 localhost`
- ✅ `ping -6 localhost`
- ✅ `ping -c 4 8.8.8.8`

---

# 8. Interview Questions

Q. Ping kaunsa protocol use karta hai?

A. ICMP

Q. RTT kya hota hai?

A. Round Trip Time

Q. TTL ka full form?

A. Time To Live

Q. Kya Ping TCP use karta hai?

A. No

Q. Ping fail hone ka matlab host down hai?

A. Nahi. ICMP block bhi ho sakta hai.

---

# 9. Cheat Sheet

```bash
ping 127.0.0.1
ping localhost
cat /etc/hosts
ping -4 localhost
ping -6 localhost
ping -c 4 8.8.8.8
ping -i 2 8.8.8.8
ping -s 1000 8.8.8.8
```

---

# 10. Summary

- Ping checks connectivity.
- Uses ICMP Echo Request / Reply.
- `127.0.0.1` = IPv4 Loopback.
- `::1` = IPv6 Loopback.
- `localhost` is a hostname.
- `TTL` indicates packet lifetime.
- `RTT` measures latency.
- Ping failure does **not** always mean the host is offline.
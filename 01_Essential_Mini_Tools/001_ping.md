# Ping

> Category: Essential Mini Tools
> Difficulty: ⭐ Beginner
> Status: ✅ Practically Completed

---

# 1. Introduction

`ping` is a command-line networking utility used to test the reachability of a host (IP address or Domain).

It sends an **Echo Request** to the target using the **ICMP (Internet Control Message Protocol)** and waits for an **Echo Reply**.

If a reply is received, it generally means the target is reachable (unless ICMP is blocked).

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

Ping is usually the first tool used by almost every Network Engineer, System Administrator, and Cybersecurity Professional.

Its uses:

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

Interval Between Packets (time period between each packet)

```bash
ping -i 2 8.8.8.8
```

Timeout (here it will wait for 2 seconds)

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

This often confuses beginners.

It means:

```
56 Bytes
```

Actual ICMP payload.

And

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

That is why Linux displays it this way:

```
56(84)
```

---

#### `64 bytes from`

This means the reply was successfully received.

The request was sent and the target responded.

---

#### `icmp_seq`

```text
icmp_seq=1
icmp_seq=2
icmp_seq=3
```

Packet numbering.

If it showed:

```text
1
2
4
5
```

Then packet number 3 was lost.

---

#### `ttl=64`

TTL = Time To Live

It indicates how many routers (hops) the packet can pass through before being discarded.

Typical Initial TTL

Linux → 64

Windows → 128

Routers → 255

Note:

The received TTL value does not identify the exact OS.

It is only a rough hint.

---

#### `time=0.043 ms`

Round Trip Time (RTT)

Request sent

↓

Reply received

↓

Total Time

```
0.043 milliseconds
```

It is so low because the packet never traveled through the network.

It only traveled within your own machine.

---

#### Statistics

```text
11 packets transmitted
```

11 packets were sent.

---

```text
11 received
```

11 replies were received.

---

```text
0% packet loss
```

Not a single packet was lost.

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

A very high `mdev` value can indicate an unstable network connection.

---

### Important Learning

`127.0.0.1` is called the Loopback Address.

It represents your own local machine.

The packet never leaves the local network interface.

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

Why did it not show 127.0.0.1?

Reason:

`localhost` is not an IP address.

It is a hostname.

The hostname is resolved via:

```bash
cat /etc/hosts
```

Output

```text
127.0.0.1 localhost
127.0.1.1 kali
::1 localhost ip6-localhost ip6-loopback
```

Modern Linux systems generally prefer IPv6.

Therefore:

```
localhost

↓

::1
```

was resolved.

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

The OS was forced to use IPv4 only.

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

The OS was forced to use IPv6 only.

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

Now, the packet is no longer inside the local machine.

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

This is why the RTT was displayed as:

```
80+ ms
```

---

# 6. Real-World Usage

### Scenario 1

```bash
ping 127.0.0.1
```

Fails

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

(This section matches exactly what we performed in class.)

- ✅ `ping 127.0.0.1`
- ✅ `ping localhost`
- ✅ `cat /etc/hosts`
- ✅ `ping -4 localhost`
- ✅ `ping -6 localhost`
- ✅ `ping -c 4 8.8.8.8`

---

# 8. Interview Questions

Q. Which protocol does Ping use?

A. ICMP

Q. What is RTT?

A. Round Trip Time

Q. What is the full form of TTL?

A. Time To Live

Q. Does Ping use TCP?

A. No

Q. Does a Ping failure mean the host is down?

A. No. ICMP packets could simply be blocked.

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
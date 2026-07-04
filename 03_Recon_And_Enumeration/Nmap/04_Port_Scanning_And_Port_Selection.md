# Chapter 4 — Port Scanning Techniques & Port Selection

## Objective

Is chapter me hum explore karenge ki Nmap packet-level par alag-alag scanning techniques ko kaise use karta hai aur target scan ke ports scope (which ports to scan) ko control kaise kiya jata hai.

By the end of this chapter, aap seekhenge:
* SYN Scan, TCP Connect Scan, aur UDP Scan ke functions.
* NULL, FIN, aur XMAS scans (RFC standards basic scans).
* Single, Multiple, Ranges, Top-ports, aur Full port (`-p-`) scan control options.
* Scan scope aur timing performance ke aapas ke relation ko manage karna.

---

## 1. Why Different Scan Types Exist?

Duniya ke saare networks aur servers ek jaise behave nahi karte. Modern environments me security control configurations alag hoti hain:
* Kuch hosts ICMP pings block kar dete hain.
* Kuch systems uncommon TCP ports se aane wali connectivity drop karte hain.
* Custom Firewalls / IDS / IPS setups signatures ke base par scanners ko block kar dete hain.

Isliye Nmap me alag-alag types ke scanning techniques provided hain, taaki situation ke according correct scan approach select kiya ja sake.

---

## 2. Core TCP Scan Types

### SYN Scan (Stealth / Half-Open Scan)
SYN Scan Nmap ka default aur sabse popular scanning technique hai. Ise **Half-Open Scan** bhi bola jata hai kyunki ye target system se complete Three-way Handshake setup nahi karta.

#### Handshake Logic:
```text
  [ SYN Scan: Target Port Open ]              [ SYN Scan: Target Port Closed ]
    Scanner           Target Host               Scanner           Target Host
       │                   │                       │                   │
       ├─── SYN Packet ───>│                       ├─── SYN Packet ───>│
       │                   │                       │                   │
       <── SYN-ACK Packet ─┤                       <─── RST Packet ────┤
       │                   │                       (Connection Reset)
       └─── RST Packet ───>│ 
       (Connection Closed)
```
Nmap SYN-ACK milte hi immediate RST (Reset) packet send kar deta hai, jisse TCP handshake complete nahi ho pata.

* **Command:**
  ```bash
  nmap -Pn -F -sS scanme.nmap.org
  ```
* **Advantages:** Bohot fast hai, reliable hai, aur fully established connection na hone ki wajah se application logs me scan trace record hone ke chances kam hote hain.
* **Lab Output Example:**
  ```text
  PORT    STATE     SERVICE
  22/tcp  open      ssh
  80/tcp  open      http
  25/tcp  filtered  smtp
  ```

---

### TCP Connect Scan
Connect Scan target application server se pura TCP three-way handshake setup karta hai.

* **Handshake Sequence:** `SYN` ➔ `SYN-ACK` ➔ `ACK`.
* **Command:**
  ```bash
  nmap -Pn -F -sT scanme.nmap.org
  ```
* **Advantages:** Ise chalane ke liye scanner machine par root (Administrator) privileges/raw sockets ki accessibility hona zaroori nahi hai (ye normal system socket calls use karta hai).
* **Disadvantages:** Bohot easily catch/log hota hai kyunki connection complete status state tak jata hai.
* **Lab Output Example:**
  ```text
  PORT   STATE SERVICE
  80/tcp open  http
  ```

---

### UDP Scan
UDP scan TCP ke opposite connectionless systems (jaise DNS, NTP, DHCP) ko check karne ke liye use hota hai.

* **Command:**
  ```bash
  sudo nmap -Pn -F -sU scanme.nmap.org
  ```
* **UDP Scanning Challenge:** UDP targets responses return nahi karte agar port open ho. Agar target server response nahi deta, toh Nmap use status **`open|filtered`** assign karta hai.
* **Lab Output Example:**
  ```text
  123/udp open ntp
  ```

---

## 3. RFC-Based Advanced TCP Scans (NULL, FIN, XMAS)

RFC (Request for Comments) standard rules define karte hain ki agar kisi closed port par unexpected flags wale packet aayenge, toh target machine use reset `RST` karegi. Lekin agar port open hai, toh system use drop/ignore karega.

These scans use illegal combinations of TCP header flags:

### NULL Scan (`-sN`)
* Sends TCP packet with **no flags set**.
* **Command:**
  ```bash
  nmap -Pn -F -sN scanme.nmap.org
  ```

### FIN Scan (`-sF`)
* Sends TCP packet with only the **FIN flag set** (terminating connection flag).
* **Command:**
  ```bash
  nmap -Pn -F -sF scanme.nmap.org
  ```

### XMAS Scan (`-sX`)
* Sends TCP packet with **FIN, PSH, and URG flags set** simultaneously. Header looks "lit up like a Christmas tree."
* **Command:**
  ```bash
  nmap -Pn -F -sX scanme.nmap.org
  ```

> [!NOTE]
> **Modern Systems Behavior:**
> Lab outputs me `All 100 scanned ports are in ignored states (closed)` mil sakta hai. Iska reason hai ki modern Windows/Linux firewalls in standard violations and invalid flag sets ko recognize karke seedhe blocking response RST return kar dete hain.

---

## 4. Port Selection Options (Controlling Scan Scope)

Saare 65535 ports ko scan karna hamesha zaroori nahi hota. Nmap parameters targets ko select aur limit karne me use kiye ja sakte hain:

### Single Port Scan
Sirf ek specific port target check karne ke liye:
```bash
nmap -Pn -p 80 scanme.nmap.org
```

### Multiple Ports Scan
Multiple targets specific list scan karne ke liye commas use karein:
```bash
nmap -Pn -p 22,80,443 scanme.nmap.org
```

### Port Range Scan
Continuous range define karne ke liye hyphen (-) run karein:
```bash
nmap -Pn -p 1-1000 scanme.nmap.org
```
> [!WARNING]
> Ranges lambi hone par agar packet drop latency target error badhta hai toh loop alerts dikh sakte hain:
> `RTTVAR has grown to over 2.3 seconds`. Iska matlab hai firewall rate-limiting activate ho gayi hai.

### Fast Scan (`-F`)
Nmap statistics ke default top 100 most common ports scan karega.
```bash
nmap -Pn -F scanme.nmap.org
```

### Top Ports Scan (`--top-ports`)
Database statistics ke base par specific numbers of common ports list scan karna:
```bash
nmap -Pn --top-ports 20 scanme.nmap.org
```

### Full Port Scan (`-p-` / `1-65535`)
Host target ke complete ports check karne ke liye:
```bash
nmap -Pn -p- scanme.nmap.org
```

---

## 5. Summary & Comparison Table

| Scan Type | Nmap Flag | Speed | Stealth Level | Reliability | Primary Use |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **SYN Scan** | `-sS` | High | High | High | Default Reconnaissance |
| **TCP Connect** | `-sT` | Medium | Low | High | No root access available |
| **UDP Scan** | `-sU` | Low | Medium | Medium | DNS/NTP/DHCP Enumeration |
| **NULL Scan** | `-sN` | Medium | Medium | Low | Firewall/IDS evasion tests |
| **FIN Scan** | `-sF` | Medium | Medium | Low | Bypass filters on legacy UNIX |
| **XMAS Scan** | `-sX` | Medium | Medium | Low | Firewall rule audits |

---

## 6. Scan Scope vs. Performance Impact

### Small Scope (e.g., `-p 80` or `-F`)
* **Pros:** Fast execution, minimal network congestion, very low probability of trigger-alerting target IPS.
* **Cons:** Security audits incomplete paths skip ho sakte hain.

### Large Scope (e.g., `-p-`)
* **Pros:** Complete vulnerability verification (Koi hidden service miss nahi hogi).
* **Cons:** Extremely slow, large resource utilization, high risk of detection and firewall blocking.

---

## Interview Notes & Q&A

### Q: SYN Scan ko "Half-Open Scan" kyu bolte hain?
**A:** Kyunki Nmap SYN aur server se SYN-ACK response receive karne ke baad, client machine se normal ACK confirmation ke bajay RST (Reset) packet bhej kar process to complete and record target logs setup hone se pehle hi connection session delete kar deta hai.

### Q: `-p-` parameter scan scope me kya define karta hai?
**A:** `-p-` parameter Nmap ko command deta hai ki targets ranges host ke pure list total **65,535 TCP ports** ko sequential analyze karein.

### Q: NULL, FIN, aur XMAS scan methods modern targets me fail kyu ho jaate hain?
**A:** Modern operating systems aur firewalls custom rules templates use karte hain. Agar invalid flags sequences standard standard RFC limits to bypass karte hain, toh security layers unhe automatic drop/RST response ke sath return block kar deti hain.

# Chapter 3 — Your First Nmap Scan

## Objective

Is chapter me hum apna pehla Nmap scan run karenge aur target scan output ko ek cybersecurity professional ki tarah decode aur analyze karna seekhenge.

By the end of this chapter, aap seekhenge:
* Nmap scan run kaise karte hain.
* Nmap host aur open ports ko kaise discover karta hai.
* Basic scan outputs ke parameters ko interpret karna.
* Firewalls and filters ki wajah se aane wali network delays ko handle karna.

---

## 1. What Happens During a Scan?

Jab hum target par Nmap scan chalate hain, toh Nmap target device se kuch questions poochta hai:
1. *Are you online?* (Kya aap network par active hain?)
2. *Which ports are open?* (Aapke kaun-se ports open hain?)
3. *Which services are running?* (Un ports par kaun-se applications active hain?)

Nmap target se milne wale answers (responses) ko analyze karke user-friendly tabular format me show karta hai.

---

## 2. Safe Practice Target

> [!TIP]
> Hamesha scanning practice ke liye authorized environment use karein.

Nmap project officially test scanning ke liye target host provide karta hai:
`scanme.nmap.org`

---

## 3. Running Your First Scan

### Standard Command
Kali Linux terminal me ye standard command execute karein:
```bash
nmap scanme.nmap.org
```

### Common Real-World Problem: Slow Scan
Real-world network environments me standard scan kafi slow ya freeze ho jata hai. Iska reason niche diye gaye security controls hote hain:
* Firewalls (Jo requests drop kar dete hain)
* Rate Limiting (Scanner ko block/slow karna)
* Retransmission Delays (Nmap requests ke filters hone par bar-bar try karna)

Agar scan karte samay target network blockages ki wajah se error aaye:
```text
Warning: 45.33.32.156 giving up on port because retransmission cap hit (10)
```
Toh hum scan ko optimize karne ke liye custom flags use karte hain.

### Practical Lab Command (Optimized Scan)
Apne lab setups me scan speed and efficiency increase karne ke liye ye optimized command use karein:
```bash
nmap -Pn -F scanme.nmap.org
```

* **`-Pn`**: Host Discovery bypass karta hai (Assume target is online). Firewalls ICMP ping request block karte hain, toh `-Pn` scan ko start kar deta hai.
* **`-F`**: Fast Scan mode. Target ke sirf top 100 most common ports ko hi scan karta hai (Normal scan top 1000 ports scan karta hai).

---

## 4. Actual Lab Output & Analysis

Jab hum optimized scan run karte hain:
```bash
nmap -Pn -F scanme.nmap.org
```

Toh niche diya gaya output block receive hota hai:
```text
Starting Nmap 7.95
Nmap scan report for scanme.nmap.org (45.33.32.156)
Host is up (0.41s latency)
Not shown: 94 closed tcp ports (reset)

PORT    STATE     SERVICE
22/tcp  open      ssh
25/tcp  filtered  smtp
80/tcp  open      http
135/tcp filtered  msrpc
139/tcp filtered  netbios-ssn
445/tcp filtered  microsoft-ds
```

---

## 5. Output Breakdown

### 1. Header Details
* **`Starting Nmap 7.95`**: Nmap utility ka engine version.
* **`Nmap scan report for scanme.nmap.org (45.33.32.156)`**: Host DNS target name ko convert karke IP detail display karta hai.
* **`Host is up (0.41s latency)`**: Target machine online hai aur scanner response time (latency) 0.41 seconds hai.

### 2. Not Shown: Closed Ports
* **`Not shown: 94 closed tcp ports (reset)`**: Scan list ke total ports me se 94 ports completely closed mile. Target ne in ports ke liye `RST` (Reset) packet bhej kar direct connection close kar diya.

### 3. Open Ports Status
* **`22/tcp open ssh`**: Secure remote shell management port open mila.
* **`80/tcp open http`**: Normal web portal web server service active mili.

### 4. Filtered Ports Status
* **`25/tcp filtered smtp`**
* **`135/tcp filtered msrpc`**
* **`139/tcp filtered netbios-ssn`**
* **`445/tcp filtered microsoft-ds`**

> [!IMPORTANT]
> **`filtered` state ka kya matlab hai?**
> Nmap probe packets ko na server ne accept kiya, na reject. Target machine ke aage koi firewall ya router traffic check kar raha hai jo probes drop kar deta hai. Nmap determine nahi kar pata ki port open hai ya closed.

---

## 6. Target Port States Summary

| Port State | Meaning in Hinglish |
| :--- | :--- |
| **Open** | Port connections active accept kar raha hai. Target application service listening state me hai. |
| **Closed** | Port reachable hai par target system par us specific endpoint par koi application active nahi hai. |
| **Filtered** | Firewall ya security rules packet block kar rahe hain. Nmap determine nahi kar pa raha hai port status. |
| **Unfiltered** | Port reachable hai par Nmap check nahi kar pa raha ki exact state open hai ya closed. |

---

## 7. Defender vs. Attacker Perspective

### Attacker Perspective
* **Target Info:** Scanner targets me ports 22, 80 open dikhata hai. Attackers ko target system par web application aur active remote SSH setup ke signals milte hain. Next stage exploration direct in open entry points par divert ho sakti hai.

### Defender Perspective
* **Security Audits:** Defensive security analyst ko check karna hoga ki kya port 22 internet ke liye open hona chahiye? Outdated version update hain? Services ko security controls ke piche filter kiya gaya hai ya nahi?

---

## Interview Notes & Q&A

### Q: Nmap output me "Filtered" port state ka kya logic hai?
**A:** Filtered state tab dikhti hai jab Nmap ke probes target port tak nahi pahunch paate ya unka response wapas nahi aata kyunki intermediate devices jaise Network Firewall ya local security policies network requests ko block (drop) kar dete hain.

### Q: Fast scan (`-F`) aur Normal scan me kya difference hai?
**A:** Normal default scan top 1000 standard ports check karta hai, jabki Fast scan (`-F`) timing and latency minimize karne ke liye database se list kar sirf top 100 high-frequency ports ko scan karta hai.

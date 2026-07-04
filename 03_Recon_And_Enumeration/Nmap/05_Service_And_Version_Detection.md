# Chapter 5 — Service & Version Detection

## Objective

Is chapter me hum seekhenge ki target hosts ke open ports par chal rahe exact software applications aur unke versions ko Nmap ke zariye kaise identify kiya jata hai.

By the end of this chapter, aap seekhenge:
* Service Version Detection (`-sV`) command options.
* Version Intensity levels (`--version-light` aur `--version-all`).
* Version Trace (`--version-trace`) debugging process.
* Banner Grabbing vs. Active Service Fingerprinting ka core concept.
* Real-world enumeration workflow.

---

## 1. What is Service Version Detection?

Ek basic port scan hume sirf ye batata hai:
```text
22/tcp open ssh
80/tcp open http
```
Par ye hume exact details nahi deta jaise:
* Kaun-sa SSH Server chal raha hai (OpenSSH, Dropbear)?
* Kaun-sa Web Server software deployed hai (Apache, Nginx, IIS)?
* Software ka version kya hai (Apache 2.4.7)?
* Target machine ka operating system kya hai?

In sabhi details ko extract karne ke liye hum Nmap ka **Service Version Detection (`-sV`)** flag use karte hain.

---

## 2. Why Version Detection Matters?

Vulnerability Assessment ke liye exact version pata hona sabse zaroori step hai. 

> [!TIP]
> **Example Comparison:**
> * **Normal Scan:** Port 80 open hai. (Kuch khas andaza nahi laga sakte)
> * **Version Scan:** Apache httpd 2.4.7 run ho raha hai. (Ab hum directly check kar sakte hain ki Apache 2.4.7 me kaun-se exploits and vulnerabilities exist karti hain).

---

## 3. Running Version Detection (Lab Environment)

### Practical Lab Command
Modern networks me firewalls and rate-limits bypass karne ke liye optimized version detection scan syntax niche diya gaya hai:
```bash
nmap -Pn -F -sV scanme.nmap.org
```

* **`-Pn`**: Bypass Host ping.
* **`-F`**: Scan only top 100 common ports (Fast scan).
* **`-sV`**: Enable Service Version Detection.

### Actual Lab Output:
```text
Nmap scan report for scanme.nmap.org (45.33.32.156)
Host is up (0.30s latency).
Not shown: 94 closed tcp ports (reset)

PORT    STATE    SERVICE      VERSION
22/tcp  open     ssh          OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.13 (Ubuntu Linux; protocol 2.0)
25/tcp  filtered smtp
80/tcp  open     http         Apache httpd 2.4.7 ((Ubuntu))
135/tcp filtered msrpc
139/tcp filtered netbios-ssn
445/tcp filtered microsoft-ds

Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

### Lab Output Analysis:
* **Port 22/tcp:** Target host par SSH service open hai. Software **OpenSSH version 6.6.1p1** chal raha hai aur OS **Ubuntu Linux** hai.
* **Port 80/tcp:** target server par **Apache HTTP Server version 2.4.7** running active state me mila.

---

## 4. Banner Grabbing vs. Active Fingerprinting

Nmap services and versions ka pata kaise lagata hai? Iske do methods hote hain:

### 1. Banner Grabbing (Simple and Passive)
Jab Nmap port se connect hota hai, toh software application jo response packet (banner) pehle bhejti hai, Nmap use read kar leta hai (e.g., `SSH-2.0-OpenSSH_6.6.1`).
* **Limitation:** System admin server settings me default banners ko custom write kar sakte hain ya complete hide/disable kar sakte hain.

### 2. Active Service Fingerprinting (Nmap Advanced Method)
Nmap targets par custom protocol-specific queries and packets send karta hai.
```text
Connect to port ➔ Null Probe (Listen) ➔ Send HTTP Probes ➔ Match Response Signatures
```
Aane wale application responses ko Nmap apne standard signature database (`nmap-service-probes`) se map/compare karta hai, jisse exact accuracy milti hai.

---

## 5. Version Intensity Levels

Version detection speed aur accuracy ko control karne ke liye timing probe settings adjust ki ja sakti hain:

### Version Light (`--version-light`)
* Nmap targets par kam probes send karega (Intensity Level 2).
* **Use Case:** Large networks scanning me speed improve karne ke liye useful hai.
* **Command:**
  ```bash
  nmap -Pn -sV --version-light -p 22,80 scanme.nmap.org
  ```

### Default Version Detection (`-sV` without intensity parameters)
* Average probe count settings use hoti hain (Intensity Level 7). Balance between speed & accuracy.

### Version All (`--version-all`)
* Nmap target port par poore signature database queries run karta hai (Intensity Level 9).
* **Use Case:** Jab target server unknown custom ports par chal raha ho aur default scans fail ho jayein.
* **Command:**
  ```bash
  nmap -Pn -sV --version-all -p 22,80 scanme.nmap.org
  ```

---

## 6. Version Trace (Debugging Scan Behavior)

Agar aapko check karna hai ki back-end par Nmap service check karne ke liye kaun-से requests bhej raha hai aur target se kya reply aa raha hai:
```bash
nmap -Pn -sV --version-trace -p 80 scanme.nmap.org
```

#### Lab Output Flow Trace:
```text
TCP Connection Established 
       │
       ▼
NULL Probe Sent ➔ Read Timeout (No response)
       │
       ▼
HTTP GET Probe Sent ➔ HTTP/1.1 200 OK Response Received
       │
       ▼
Comparing Signature ➔ Apache httpd 2.4.7 Matched
```

---

## 7. Professional Enumeration Workflow

Ethical hacking audits me professionals ye structure sequence follow karte hain:

```text
Host Discovery (Target Up?)
       │
       ▼
Port Scanning (Kaun-se Ports Open Hain?)
       │
       ▼
Service Version Detection (Exact Application kya hai?) ◄── (Hum is stage par hain)
       │
       ▼
OS Fingerprinting (Target Platform kya hai?)
       │
       ▼
NSE Auditing & Vulnerability Scan (Loop-holes search karna)
```

---

## Summary & Command Reference

| Parameter Option | Purpose | Probe Intensity | Recommended Use Case |
| :--- | :--- | :---: | :--- |
| **`-sV`** | Default Version Scan | `7` | Standard Security Auditing |
| **`--version-light`** | Fast Version Scan | `2` | Large network range audits |
| **`--version-all`** | Deep Version Scan | `9` | Custom/Hidden server audits |
| **`--version-trace`** | Internal scan tracing | N/A | Debugging & Learning purposes |

---

## Interview Notes & Q&A

### Q: Banner Grabbing aur Nmap Version Detection me kya difference hai?
**A:** Banner Grabbing sirf connection response me aane wala default message read karta hai jise easily spoof kiya ja sakta hai. Jabki Nmap active service fingerprinting use karta hai jisme custom probes ke rules parameters standard target configurations match analysis report generate karte hain.

### Q: `--version-light` flag scanner output speed par kya impact dalta hai?
**A:** Ye query checks database filter karke signature probe packet counts bohot kam kar deta hai, jisse large range networks scanning complete hone ka process time kafi bacha jata hai.

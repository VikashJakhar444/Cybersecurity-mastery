# Chapter 15 — Nmap Master Revision Guide & Cheat Sheet

## Objective

Ye guide aapke quick revision aur real-time practical demonstrations/interviews ke liye design ki gayi hai. Isme sirf aur sirf **Nmap** commands, options aur workflows ko ek professional audit perspective se compile kiya gaya hai, taaki interview se pehle aap 10–15 minutes me complete Nmap course ko revise kar sakein.

---

## 1. Nmap Complete Workflow

```text
       Target
         │
         ▼
   Host Discovery  ──(Find alive hosts sweep)
         │
         ▼
   Port Discovery  ──(Find open TCP/UDP ports)
         │
         ▼
 Service Enumeration ──(Identify running services names)
         │
         ▼
 Version Detection  ──(Extract exact software versions)
         │
         ▼
 OS Fingerprinting ──(Identify target operating system)
         │
         ▼
  NSE Enumeration  ──(Run service-specific scripts)
         │
         ▼
   Vuln Detection  ──(Identify known vulnerabilities/CVEs)
         │
         ▼
  Evasion & Tuning ──(Firewall evasion & timing adjustments)
         │
         ▼
     Reporting     ──(Save results for documentation)
```

---

## 2. Nmap Core Flags Cheat Sheet (Categorized)

### Host Discovery (Phase 1)
*   **`-sn`**: Ping sweep sweep sweep. Live hosts discover karein bina ports scan kiye.
*   **`-Pn`**: Bypass host discovery (No Ping). Assumes target is online.
*   **`-PR`**: ARP Ping sweep. Local subnet me MAC-based fast host discovery ke liye.
*   **`-PE` / `-PP` / `-PM`**: ICMP Echo, Timestamp, and Address Mask request. ICMP bypass tests ke liye.

### Port Scanning & Scan Types (Phase 2)
*   **`-F`**: Fast Scan mode. Top 100 most common ports scan karta hai.
*   **`-p-`**: Full range scan. Target ke saare 65,535 TCP ports ko checks karta hai.
*   **`-p <list>`**: Specific ports scan (e.g., `-p 22,80,443`).
*   **`--top-ports <n>`**: Scan top 'n' common ports (e.g., `--top-ports 20`).
*   **`-sS`**: TCP SYN Stealth scan (Half-open, default scan type).
*   **`-sT`**: TCP Connect scan (Handshake complete karta hai, no root needed).
*   **`-sU`**: UDP Port scan (DNS, NTP, DHCP servers scanning ke liye).
*   **`-sA`**: TCP ACK scan (Firewall rules and mapping filter states verify karne ke liye).
*   **`-sN` / `-sF` / `-sX`**: NULL, FIN, aur XMAS scans (RFC tcp packet rules behavior audits).

### Service Version & OS Detection (Phase 3 & 4)
*   **`-sV`**: Enable Service Version detection (exact software versions details).
*   **`--version-light`**: Fast version detection (Uses fewer signatures probes, intensity 2).
*   **`--version-all`**: Maximum probe intensity (Intensity 9, for custom/unknown ports).
*   **`-O`**: Enable Operating System Detection (TCP/IP stack fingerprints).
*   **`-A`**: Aggressive scan mode (Combines `-sV`, `-O`, `-sC`, and `--traceroute`).

### Firewall Evasion Techniques (Phase 7)
*   **`-f`**: Split packets into smaller fragment segments.
*   **`--mtu <size>`**: Custom Maximum Transmission Unit size set karna (Must be multiple of 8).
*   **`-D RND:5`**: Generates 5 fake/decoy source IPs to mask real scanner IP.
*   **`-S <IP>`**: Spoof source IP address (Run only if you can capture raw packets response).
*   **`-g <port>` / `--source-port`**: Send packets from specific ports (e.g., port 53 DNS to bypass basic filters).
*   **`--data-length <bytes>`**: Packets padding. Default packet templates size change karta hai.

### Performance & Timing Options (Phase 8)
*   **`-T0` to `-T5`**: Timing levels (Paranoid, Sneaky, Polite, Normal, Aggressive, Insane).
*   **`--min-rate` / `--max-rate`**: Minimum/Maximum packets per second output speed lock karna.
*   **`--host-timeout <time>`**: Agar host given time window me reply nahi deta toh skip sweep details.
*   **`--min-parallelism <number>`**: Parallel probe packets transfer rate controls set karna.

### Output & Database Management (Phase 9)
*   **`-oN <file>`**: Save in human-readable Normal text format.
*   **`-oX <file>`**: Save in machine-readable XML format (Metasploit integration standard).
*   **`-oG <file>`**: Save in Grepable single-line format.
*   **`-oA <basename>`**: Generates all three formats (`.nmap`, `.xml`, `.gnmap`) at once.
*   **`-v` / `-vv`**: Verbose mode. Shows real-time scan progress.
*   **`-d` / `-dd`**: Debug mode. Low-level connection trace details printing.
*   **`--script-help <script>`**: Prints usage examples and arguments of an NSE script.
*   **`--script-updatedb`**: Rebuilds the NSE script database cache.

---

## 3. Professional Step-by-Step Scan Execution Workflow

Professional network audits me step-by-step logic scanning workflow aise perform hota hai:

### Step 1: Active Target sweeps (Host sweep)
```bash
nmap -sn 192.168.1.0/24
```
* **Logic:** Active target verification. Online hosts locate karein.
* **Output Hint:** target IP `192.168.1.10` online show hua.

### Step 2: Basic Port Reconnaissance (Fast Port Scan)
```bash
nmap -Pn -F 192.168.1.10
```
* **Logic:** top 100 ports scan targets identify. isse security alert triggers and rate-limiting limits bypass ho jati hain.
* **Output Hint:** Verified open ports: `21/tcp (ftp)`, `22/tcp (ssh)`, `80/tcp (http)`, `445/tcp (smb)`, `3306/tcp (mysql)`.

### Step 3: Targeted Version & OS Detection (Specific ports only)
```bash
sudo nmap -Pn -sV -O -p 21,22,80,445,3306 192.168.1.10
```
* **Logic:** **Bina soche full ranges ya closed ports par `-sV`/`-O` chalana zero productivity noise create karta hai.** Sirf step 2 ke verified open ports ko highlight parameters `-p` me pass karein.
* **Output Hint:**
  * Port 21: `vsftpd 2.3.4`
  * Port 22: `OpenSSH 4.7p1`
  * Port 80: `Apache httpd 2.2.8`
  * Port 445: `Samba smbd 3.X`
  * Port 3306: `MySQL 5.0.51a`

### Step 4: Service-Specific NSE Scripts Auditing
Step 3 me open software details confirm ho jane ke baad matching script templates execute karein:
* **FTP (Port 21) audit:**
  ```bash
  nmap -Pn -p 21 --script=ftp-anon,ftp-syst,ftp-vsftpd-backdoor 192.168.1.10
  ```
* **HTTP Web Server (Port 80) audit:**
  ```bash
  nmap -Pn -p 80 --script=http-title,http-headers,http-enum 192.168.1.10
  ```
* **SMB (Port 445) audit:**
  ```bash
  nmap -Pn -p 445 --script=smb-os-discovery,smb-enum-shares,smb-enum-users 192.168.1.10
  ```
* **Target Vulnerabilities check (Verified open ports range):**
  ```bash
  nmap -Pn -p 21,22,80,445,3306 --script=vuln 192.168.1.10
  ```

### Step 5: Save Audit Evidence
```bash
sudo nmap -Pn -sV -O -p 21,22,80,445,3306 -oA scans/final_assessment_report 192.168.1.10
```

---

## 4. Common NSE Scripts Catalog

Upr bataye gaye common scenarios ke aalawa niche diye gaye scripts list and services aapke tips par honi chahiye:

### SSH (Port 22)
* `ssh-auth-methods`: Supported remote authentication logins checks.
* `ssh2-enum-algos`: Encryption keys ciphers lists verification.

### Database (Port 3306 / 5432)
* `mysql-info`: MySQL service identification.
* `mysql-empty-password`: Check root users blank permissions password.
* `pgsql-info`: PostgreSQL configuration details checks.

### SSL/TLS Audits (Port 443 / 8443)
* `ssl-cert`: Certificate downloads and domain validations.
* `ssl-enum-ciphers`: Check weak cipher suites and vulnerabilities checks.
* `ssl-poodle` / `ssl-ccs-injection` / `ssl-dh-params`: Vulnerability auditing.

---

## 5. Nmap Scan Decision Tree

Live practical scans run karte samay Nmap decisions flows kaise manage check karein:

```text
                         Target IP Mila?
                               │
            ┌──────────────────┴──────────────────┐
            ▼                                     ▼
      [ Host Alive? ]                       [ Host Dead? ]
            │                                     │
    ┌───────┴───────┐                       ┌─────┴─────────────────────┐
    ▼               ▼                       ▼                           ▼
 [ Local LAN ]   [ Internet ]        [ Use -Pn ]                 [ Check Routing/IP ]
   ARP Sweep     Ping sweep         (Bypass Ping sweep)          (No further action)
 (nmap -sn)      (nmap -sn)                 │
    │               │                       ▼
    └───────┬───────┘                 [ Port Scan ]
            │                               │
            ▼                               ▼
      [ Port Scan ] ──────────────────► [ Open Ports? ]
                                            │
                                  ┌─────────┴─────────┐
                                  ▼                   ▼
                              [ Yes ]              [ No ]
                                  │                   │
                                  ▼                   ▼
                         [ Service/OS detection ] [ Target UDP scan ]
                          (nmap -sV -O -p <ports>) (nmap -sU)
                                  │
                                  ▼
                         [ Targeted NSE scripts ]
                         (--script=<safe/vuln>)
                                  │
                                  ▼
                            [ Save Output ]
                              (oA format)
```

---

## 6. The Golden Rules of Nmap Scanning

> [!TIP]
> **Hamesha ye 8 core rules yaad rakhein:**
> 1. **Start Simple:** Hamesha initial scan range sweeps and fast ports discovery se shuru karein.
> 2. **Never Jump directly to `-A`:** Aggressive scans target and network par bohot noise generate karte hain.
> 3. **Targeted NSE Scripts:** Target services check version confirm hone par hi script selection use karein.
> 4. **Don't Trust Automation Blindly:** NSE reports outputs ko manually check evaluate karein.
> 5. **Detection is Not Exploitation:** scan vulnerability details confirm hone par checks test checks are different.
> 6. **Save Outputs:** Always save scan data (`-oA`) for evidence.
> 7. **Read Service versions carefully:** Operating system hints and software configurations version reports checks verify.
> 8. **Enumeration is the key:** Jitna solid enumeration hoga, exploitation utna solid aur low-risk standard settings me trigger hoga.

---

## 7. 10 Commands You Should Never Forget

```bash
# 1. Host Sweep
nmap -sn 192.168.1.0/24

# 2. Host Discovery bypass (Firewall Active)
nmap -Pn 192.168.1.10

# 3. Fast Port Discovery
nmap -Pn -F 192.168.1.10

# 4. Full range ports scan
nmap -Pn -p- 192.168.1.10

# 5. Service Version detection
nmap -Pn -sV 192.168.1.10

# 6. Operating System Stack fingerprinting
sudo nmap -Pn -O 192.168.1.10

# 7. Aggressive Scan
sudo nmap -Pn -A 192.168.1.10

# 8. Safe NSE script audit
nmap -Pn --script=safe 192.168.1.10

# 9. Vuln script audit
nmap -Pn --script=vuln 192.168.1.10

# 10. Save all formats results
nmap -Pn -oA scans/assessment_output 192.168.1.10
```

---

## 8. Top 10 Critical Interview Q&As

### Q1: SYN Scan (`-sS`) aur TCP Connect Scan (`-sT`) me kya difference hai?
* **SYN Scan (`-sS`):** Ise *Half-Open scan* kehte hain kyunki ye TCP connection ko complete (handshake complete) nahi karta. SYN-ACK milte hi RST packet bhej kar connection break kar deta hai. Isse scan bohot stealthy hota hai aur target server ke log files me save nahi hota.
* **TCP Connect Scan (`-sT`):** Ye target se poora 3-way handshake connection complete karta hai. Ise chalane ke liye root privileges (sudo access) ki zaroori nahi hoti, par ye easily target ke firewall logs me pakda jata hai.

### Q2: Target port scan status "Filtered" dikhane ka kya matlab hai?
* **Answer:** Filtered ka matlab hai ki target system ke aage koi Network Firewall ya security policy Nmap ke probes packets ko block (drop) kar rahi hai. Nmap ye determine nahi kar pata ki port open hai ya closed.

### Q3: Timings options `-T4` runs `-T3` se slow kyu execute hoti hain?
* **Answer:** `-T4` fast packets send karta hai. Speed badhne ke karan agar intermediate firewalls packets drop (loss) karne lagein, toh Nmap ko un packets ko baar-baar dubara send (retransmit) karna padta hai. Is retry mechanism aur waiting delays ki wajah se scan slow ho jata hai.

### Q4: Nmap OS Fingerprinting kaise kaam karti hai?
* **Answer:** Nmap target machine par custom flags wale TCP/UDP/ICMP packets bhejta hai. Target se aane wale reply ke characteristics (jaise TTL values, TCP options, aur Window sizes) ko apne internal OS signature database (`nmap-os-db`) se match karke operating system guess karta hai.

### Q5: OS detection (`-O`) verify karne ke liye target ports par kya condition mandatory hai?
* **Answer:** OS detection ko sahi result dene ke liye target machine par **kam se kam 1 Open TCP port** aur **kam se kam 1 Closed TCP port** hona mandatory hai, taki Nmap stack behaviors ko compare kar sake.

### Q6: Banner Grabbing aur Active Version Detection (`-sV`) me kya difference hai?
* **Answer:** 
  * **Banner Grabbing:** Server connect hote hi server jo default greeting message bhejta hai use read kar leta hai (Ise admin customize ya fake kar sakte hain).
  * **Active Version Detection (`-sV`):** Nmap port par multiple customized requests send karke responses check karta hai aur signatures match karta hai (Highly accurate).

### Q7: Default Nmap timing level kya hota hai aur aggressive timing flag `-T5` warnings kya hain?
* **Answer:** Default timing template **`-T3` (Normal)** hota hai. `-T5` (Insane) bohot fast scan karta hai, jisse targets crash ho sakte hain ya network congestion badhne se open ports discover hona miss (False Negatives) ho sakte hain.

### Q8: Nmap XML output formats parameters `-oX` target integration me kyu zaroori hai?
* **Answer:** XML output structured machine-readable format me save hota hai. Isse external security utilities (jaise Metasploit database imports) aur automation tools directly results parse kar sakte hain.

### Q9: Decoy Scanning (`-D`) command options target logs status mapping me kaise impact manage karti hain?
* **Answer:** Decoy scan (`-D`) chalane par target logs me real scanner IP ke sath-sath 5 ya 10 random fake source IPs list show hoti hai. Isse defenders ko actual scan origin track karne me bohot confusion hota hai.

### Q10: Nmap scan console me "retransmission cap hit" warning alerts ka network reason kya hai?
* **Answer:** Jab target firewall scan requests ko continuously ignore or drop karta hai, toh Nmap probes ko loop me retransmit (retry) karta hai. Retry limit (cap limit, usually 6 or 10 attempts) hit hone par Nmap search attempt rok dete hain aur warning show karta hai.

# Chapter 11 — Nmap Output Formats & Reporting

## Objective

Is chapter me hum seekhenge ki Nmap scans ke results ko professionally kaise save, organize, aur manage kiya jata hai, taaki future reports aur automation scripting me in files ko reuse kiya ja sake.

By the end of this chapter, aap seekhenge:
* Normal Output (`-oN`) human-readable format.
* XML Output (`-oX`) structured machine-readable format.
* Grepable Output (`-oG`) scripting/filtering format.
* All formats simultaneously (`-oA`) execution.
* Verbose Mode (`-v`) runtime scanning steps tracking.
* Security reports evidence collection best practices.

---

## 1. Why Output Matters

Hacking audits ya professional network assessment me target scans output screen par dekh kar chhod dena sabse badi mistake hai:
* Agar scan me 500 hosts include hain aur process complete hone me hours lag gaye. Terminal closed ho jane par pura search output data loss ho jayega.
* Audit compliance policies ke liye scan reports as an evidence attach karni hoti hain.
* Machine-readable format data parsing automation tools (jaise Python parsers, Metasploit database imports) me zaroori hai.

---

## 2. Nmap Output Formats

Nmap support karta hai char core formats settings:

### 1. Normal Output (`-oN`)
* **Purpose:** Target scan results ko as-it-is save karna jo normal text editor me humans padh sakein.
* **Command:**
  ```bash
  nmap -Pn -p 80 -oN normal.txt scanme.nmap.org
  ```
* **Output File Format Example:**
  ```text
  PORT   STATE SERVICE
  80/tcp open  http
  ```

### 2. XML Output (`-oX`)
* **Purpose:** Structured format me save karna jo databases aur parsing utilities import kar sakein.
* **Command:**
  ```bash
  nmap -Pn -p 80 -oX results.xml scanme.nmap.org
  ```
* **Output File Format Example:**
  ```xml
  <port protocol="tcp" portid="80">
    <state state="open" reason="syn-ack" reason_ttl="52"/>
    <service name="http" method="table" conf="3"/>
  </port>
  ```
* **Use Cases:** XML data imports Metasploit, Faraday, Dradis, ya custom scripts parsing me use hota hai.

### 3. Grepable Output (`-oG`)
* **Purpose:** Single-line data save karna jise command utilities like `grep`, `awk`, या `cut` se easily text search filtering output nikala ja sake.
* **Command:**
  ```bash
  nmap -Pn -p 80 -oG results.gnmap scanme.nmap.org
  ```
* **Output File Format Example:**
  ```text
  Host: 45.33.32.156 ()	Ports: 80/open/tcp//http///
  ```
* **Grep Command Use Example:**
  ```bash
  grep "/open/" results.gnmap
  ```
  *(Ye query output se open ports wale targets separate list select kar degi).*

### 4. All Formats Simultaneously (`-oA`)
* **Purpose:** Ek sath teeno formats (`.nmap`, `.xml`, `.gnmap`) ki files save karna.
* **Command:**
  ```bash
  nmap -Pn -p 80 -oA scan_results scanme.nmap.org
  ```
* **Professional Tip:** Professionals humesha `-oA` prefer karte hain taaki data documentation, parsing, aur filters teeno use cases ke liye ready files generate ho jayein.

---

## 3. Verbose Mode (`-v` / `-vv`)

Verbose parameter command execution ke runtime par scanning stages, discovery events, aur steps details displays par generate karta hai:
```bash
nmap -Pn -p 80 -v scanme.nmap.org
```

#### Lab Output Stage Discovery:
```text
Initiating SYN Stealth Scan at 12:34
Scanning scanme.nmap.org (45.33.32.156)
Discovered open port 80/tcp on 45.33.32.156
Completed SYN Stealth Scan at 12:34 (5.61s total)
```
* **Raw Packets Details:** single port checks execute hone par logs details data show hota hai: `Raw packets sent: 2, received: 2`. Isse efficiency tracking simple ho jati hai.

---

## Summary & Command Reference

| Format Option | Output File Extension | Target Use Case | Human Readable | Machine Readable |
| :--- | :--- | :--- | :---: | :---: |
| **`-oN`** | `.nmap` | Documentation & Manual Review | **Yes** | No |
| **`-oX`** | `.xml` | Metasploit/Script imports | No | **Yes** |
| **`-oG`** | `.gnmap` | Grep filtering searches | Moderate | **Yes** |
| **`-oA`** | *(Generates all three)* | Complete standard auditing output | **Yes** | **Yes** |

---

## Interview Notes & Q&A

### Q: `-oA` option settings ka kya significance hai?
**A:** `-oA` single scan query execution me teeno core outputs formats (Normal, XML, aur Grepable) templates generated save files path define database me save kar deta hai, jisse dynamic reference analysis easy ho sake.

### Q: Grepable output (`-oG`) normal outputs se scripting logs checks me kaise advantage deta hai?
**A:** Normal output multi-line and spaces tables layout use karta hai. Grepable format single-line host database write formats use karta hai, jisse command templates inputs target maps filters queries chalana (like `grep "/open/"`) instantly possible ho pata hai.

### Q: XML output imports cybersecurity assessments workflows me kyu use hota hai?
**A:** XML data structures standard configuration settings models generate karte hain. vulnerability tools and framework applications (jaise Metasploit tool imports ya custom parsing scanners python libraries) direct `.xml` files import kar automated mapping target analysis reports design kar sakte hain.
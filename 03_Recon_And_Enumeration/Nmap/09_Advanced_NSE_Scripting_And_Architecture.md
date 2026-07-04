# Chapter 9 — Advanced NSE Scripting & Architecture

## Objective

Is chapter me hum Nmap Scripting Engine (NSE) ke deep internals ko explore karenge. Hum seekhenge ki NSE script architecture kaise design hoti hai aur real-world security audits me professional scanners Lua scripts ko analyze, use, aur write kaise karte hain.

By the end of this chapter, aap seekhenge:
* NSE framework aur dynamic script execution options.
* NSE default categories (safe, discovery, auth, vuln, brute, intrusive).
* Nmap scripts directory (`/usr/share/nmap/scripts`) and library components (`/usr/share/nmap/nselib`).
* Script execution models: Single execution, Multiple list execution, and Wildcard execution.
* Script Rules (`portrule`, `hostrule`, `prerule`) aur `action()` block code execution architecture.

---

## 1. What is NSE?

**Nmap Scripting Engine (NSE)** Nmap ka sabse powerful aur custom feature framework hai. NSE basic port scanner tool ko ek complete **Vulnerability Assessment and Exploitation checking tool** me transform kar deta hai.

### Standard Scan vs. NSE Script Scan:
* **Standard Scan:** Target network ports check karta hai (`nmap target`).
* **NSE Script Scan:** Server ports ke piche chal rahe logical application patterns ko inspect karta hai, security flaws check karta hai, configuration discrepancies dhundta hai, aur data gather karta hai.

---

## 2. NSE Architecture & Storage Locations

NSE scripts high-level programming language **Lua** me likhe hote hain. 

### Files Storage Paths (Linux/Kali systems):
1. **NSE Scripts Directory:** `/usr/share/nmap/scripts`
   * *Lab Observation:* User system commands run karne par total **613 scripts** database list me active mile.
   * Check Command:
     ```bash
     ls /usr/share/nmap/scripts | wc -l
     ```
2. **NSE Common Libraries Directory:** `/usr/share/nmap/nselib`
   * Scripts common tasks (jaise HTTP requests banana, SSL certs parse karna, brute-forcing connections handle karna) bar-bar clean re-write karne ke bajay common Lua libraries use karte hain (e.g., `http.lua`, `shortport.lua`, `sslcert.lua`).

---

## 3. NSE Script Categories

Nmap database scripts specific operations aur risks ke basis par niche di gayi categories me segmented hote hain:

| Category | Hinglish Meaning & Risk Level | Examples |
| :--- | :--- | :--- |
| **default** | Safe, fast, aur reliable common scripts. Runs via `-sC` or `--script=default`. | `ssh-hostkey`, `http-title` |
| **safe** | target machine par zero crash risk wale low-overhead scripts. | `http-headers`, `dns-brute` |
| **discovery** | Target system se extra configurations data pull karne wale passive scripts. | `http-enum`, `rdp-ntlm-info` |
| **auth / brute** | Login panel configurations check karna ya default credentials crack karna. | `ftp-anon`, `ssh-brute` |
| **vuln** | CVE databases ke criteria checks target par run karna. | `smb-vuln-ms17-010` |
| **intrusive / dos** | Dangerous script templates jo target system crash ya load badha sakti hain. | `http-slowloris` |

---

## 4. How to Execute NSE Scripts

Nmap parameters scripting execution support niche diye gaye patterns me karte hain:

### 1. Run a Single Script
Target port par check run karne ke liye specific script file run karein:
```bash
nmap -Pn -p 80 --script=http-title scanme.nmap.org
```
* **Lab Output Example:**
  ```text
  |_http-title: Go ahead and ScanMe!
  ```

### 2. Run Multiple Scripts
Commas use karke script combination launch karein (Spacebar include mat karein):
```bash
nmap -Pn -p 80 --script=http-title,http-headers scanme.nmap.org
```

### 3. Wildcard Script Execution
Target port service se match hone wale custom naming search patterns wale scripts ek sath execute karne ke liye asterisk (*) use karein:
```bash
nmap -Pn -p 80 --script=http* scanme.nmap.org
```
*(Is command se target par 'http' prefix se start hone wale saare default scripts run ho jayenge).*

### 4. Updating NSE Database
Jab koi naya script path database me copy/download kiya jata hai, toh script index refresh karna zaroori hai:
```bash
sudo nmap --script-updatedb
```

---

## 5. NSE Script Structure & Internals (Lua code design)

Ek professional penetration tester ko NSE scripts chalane se pehle script ka source code padhna aana chahiye. Source code read karne ke liye terminal tool **`less`** use kiya jata hai:
```bash
less /usr/share/nmap/scripts/http-title.nse
```

#### Standard Script Internal Structure Layout:
```lua
-- Block 1: Metadata Definition
description = "Retrieves the title from an HTML page."
author = "Diman Todorov"
license = "Same as Nmap"
categories = {"default", "discovery", "safe"}

-- Block 2: Library Import
local http = require "http"
local shortport = require "shortport"

-- Block 3: Rule Definition (Condition Check)
portrule = shortport.http

-- Block 4: Action Definition (Actual Execution Logic)
action = function(host, port)
    local response = http.get(host, port, "/")
    -- Parse title and return text
end
```

### Script Execution Logic Elements:
1. **Rule Block (`portrule` / `hostrule` / `prerule`):**
   * Decide karta hai ki target device par chal rahi condition match ho rahi hai ya nahi.
   * `portrule` target standard open port triggers evaluate karta hai (e.g., Run script only if port 80 is open).
   * `hostrule` host parameter status read karta hai (Run once per target system).
   * `prerule` target discovery run hone se pehle check trigger karta hai.
2. **Action Block (`action()` function):**
   * Port state and rules verify hone par action block execute hota hai. Target server connection process, custom protocols requests transfer, response parsing aur final results format prints execute functions is block me likhe hote hain.

---

## 6. Professional NSE Workflow

Professional penetration audits me system checks flow niche diye gaye strict guidelines check map ke sequence ke according hoten hain:

```text
    Service Scan ➔ Target Identification ➔ Filter specific safe scripts
                         │
                         ▼
        Review Script Source Code (`less /script.nse`)
        (Understand what packets are sent to avoid firewall triggers)
                         │
                         ▼
             Run with target arguments (`--script-args`)
                         │
                         ▼
             Document results & cross-verify findings
```

---

## Interview Notes & Q&A

### Q: NSE script categories target risk control me kaise useful hain?
**A:** Target server stability manage karne ke liye standard audits rules design kiye jate hain. Safe (`safe`) queries se production servers details secure retrieve ki jati hain, jabki Intrusive (`intrusive`) scripts crashes avoid karne ke liye direct launch se lock boundaries me rakhe jate hain.

### Q: portrule, hostrule, aur prerule elements me kya difference hai?
**A:** `portrule` target specific port availability (e.g., port 80) match triggers check karta hai. `hostrule` targets hosts parameters check database run hone par verify validate setup launch karta hai. `prerule` scan environment initiate configurations setup sequence checks run karta hai.

### Q: Nmap scripting database check commands refresh update kyu execute setup settings use hoti hain?
**A:** Nmap memory runtime index template maps directories references save kar rules match trigger karta hai. Agar files changes or updates refresh nahi hote toh system new customized execution paths templates link error returns trace list print block out kar deta hai, isliye `sudo nmap --script-updatedb` run karna mandatory hai.

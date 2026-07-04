# Chapter 14 — Case Studies & Final Lab

## Objective

Is final chapter ka goal pure Nmap course ke learning components ko real-world case scenarios par practically apply karna aur ek complete penetration testing lab (jaise Metasploitable 2) par Nmap scan flows run karke reports tayar karna hai.

By the end of this chapter, aap seekhenge:
* Real-world scenario assessments (Web, FTP, SMB, SSH, aur Databases).
* Complete scanning workflow automation in a simulated pentest environment.
* Project results structures aur documentation best practices.
* Final mini-challenge labs requirements and solutions.
* Nmap Mastery checklist verification.

---

## 1. Case Studies: Scanning Scenarios

Real assessments audits me targets different servers systems run karte hain. niche di gayi guides target services ke dynamic Nmap workflows explain karti hain:

### Scenario 1: Web Server Assessment (Port 80/443)
* **Goal:** Website ke structures, headers details, directories list, aur web technologies details check karna.
* **Recommended Scripts:** `http-enum`, `http-title`, `http-headers`, `http-security-headers`, `http-methods`.
* **Execution Commands:**
  ```bash
  nmap -Pn -p 80 --script http-title,http-headers,http-enum scanme.nmap.org
  ```

### Scenario 2: FTP Server Assessment (Port 21)
* **Goal:** Anonymous credentials check, system commands compatibility, aur vsftpd default exploits search.
* **Recommended Scripts:** `ftp-anon`, `ftp-syst`, `ftp-vsftpd-backdoor`.
* **Execution Commands:**
  ```bash
  nmap -Pn -p 21 --script ftp-anon,ftp-syst,ftp-vsftpd-backdoor 192.168.1.10
  ```

### Scenario 3: SMB Server Assessment (Port 445)
* **Goal:** Windows shared files directories, usernames list, aur authentication signatures verify karna.
* **Recommended Scripts:** `smb-os-discovery`, `smb-enum-shares`, `smb-enum-users`.
* **Execution Commands:**
  ```bash
  nmap -Pn -p 445 --script smb-os-discovery,smb-enum-shares 192.168.1.10
  ```

### Scenario 4: SSH Server Assessment (Port 22)
* **Goal:** Supported encryption algorithms, MAC addresses, and authentication schemes lists test karna.
* **Recommended Scripts:** `ssh2-enum-algos`, `ssh-auth-methods`.
* **Execution Commands:**
  ```bash
  nmap -Pn -p 22 --script ssh2-enum-algos,ssh-auth-methods 192.168.1.10
  ```

### Scenario 5: Database Server Assessment (MySQL/PostgreSQL)
* **Goal:** Database version info, empty password permissions check, aur configurations audit.
* **Recommended Scripts:** `mysql-info`, `mysql-empty-password`, `pgsql-info`.
* **Execution Commands:**
  ```bash
  nmap -Pn -p 3306 --script mysql-info,mysql-empty-password 192.168.1.10
  ```

---

## 2. Professional Directory Setup

Scan execution ke baad output files ko client reports ke database structure me configure karein:

```text
    ProjectDir/
    ├── discovery/       <-- Ping sweep logs
    ├── ports/           <-- Simple port lists (.gnmap files)
    ├── services/        <-- Version scan logs (.nmap files)
    ├── vulnerabilities/ <-- NSE results and CVEs (.xml files)
    └── screenshots/     <-- Evidence files
```

---

## 3. Final Lab Challenge: Metasploitable 2 Assessment

Target server (Metasploitable 2 VM) active hai. Niche diye gaye checklist parameters complete verify karein:

1. **Host Sweep:** Target subnet Sweep karke IP status locate karein (`nmap -sn`).
2. **Ports Scan:** Target details save targets `.xml` files outputs generate karein (`nmap -Pn -p- -oA services_report`).
3. **OS and Versions Detection:** Port specifications match software list map karein.
4. **Vulnerable Services Analysis:** FTP or SMB targets vulnerability configurations details check compile setups test karein.

---

## 4. Nmap Mastery Checklist Verification

* [x] Basic Scanning flags (`-sS`, `-sT`, `-sU`)
* [x] Port Selection controls (`-p`, `-F`, `-p-`, `--top-ports`)
* [x] Timing Options optimize configurations (`-T0` se `-T5`)
* [x] Service Version (`-sV`) aur OS Detection (`-O`)
* [x] Output saving parameters (`-oN`, `-oX`, `-oG`, `-oA`)
* [x] Firewall Evasion details (`-f`, `-D`, `--data-length`)
* [x] NSE Scripting execution & Lua architecture rules (`portrule`, `action()`)
* [x] Python, Metasploit, aur SearchSploit Integration toolchain.

---

## Summary & Key Takeaways

* **Workflow consistency is key:** Har scenario target service ke basis par customized approach demands karta hai.
* **Documentation creates trust:** Folder division structures and saved scan inputs are vital for client reporting.
* **Mastery is practical:** Commands yaad rakhne se zaroori outputs ko logically read aur next tools ke sath execute karna seekhna hai.

---

## Interview Notes & Q&A

### Q: Web audit scans me http-enum script ka use cases kya hai?
**A:** `http-enum` target server par common vulnerabilities, defaults backup directories directories paths, login configuration interfaces, aur custom tools folders scans execute match indexes list generate karta hai.

### Q: Network file sharing systems checks (SMB) audits me smb-os-discovery parameters kya show karta hai?
**A:** `smb-os-discovery` active targets par SMB protocol parameters parse karke computer name, target Domain name, exact OS details version (jaise Windows Server build level), aur Active Directory domain mapping verify details generate kar deta hai.

### Q: Agar system audits checklist me multiple target open databases chal rahe hon, toh scan rate logic kya adjust karenge?
**A:** Databases resources bohot memory constraints handle karte hain. multiple databases nodes scans T4/T5 timing settings bypass targets limits slow parameters (`-T2` or `-T3`) ranges checkpoints check par configure kiye jate hain jisse network crash issues avoid ho sakein.

# Chapter 2 — Networking Fundamentals for Nmap

## Objective

Nmap ko explore karne se pehle hume Networking ke basic building blocks ko samajhna bohot zaroori hai. In concepts ke bina Nmap ke scans aur outputs ko analyze karna mushkil ho jayega.

Is chapter me hum seekhenge:
* IP Address & Protocols
* TCP vs UDP
* TCP Three-Way Handshake
* DNS, Packets & Models (OSI vs TCP/IP)

---

## 1. What is a Network?

Ek network multiple devices ka connection hai jo aapas me data share aur communicate karne ke liye use hota hai (e.g., Home WiFi, Office LAN, aur Internet).

Devices in a network can be:
* Computers, Laptops & Phones
* Servers
* Routers & Switches
* Network Printers & IoT Devices

> [!NOTE]
> **Real-World Analogy:**
> Network ko ek road system ki tarah samjhein:
> * **Ghar (Devices):** Jo network par online hain.
> * **Sadak (Network Cables/WiFi):** Jiske through communication hota hai.
> * **Ghar ka pata (IP Address):** Jo har device ko uniquely locate karta hai.
> * **Chitthi (Data Packets):** Jo ek device se dusre device tak bheja jata hai.

---

## 2. What is an IP Address?

IP ka full form hai **Internet Protocol**. IP Address network me har device ko milne wala ek unique address hota hai.

Example: `192.168.1.10`

### Types of IP Addresses:
1. **Private IP:** Ye local networks (ghar ke router ya office LAN) ke andar devices ko identify karne ke liye use hota hai. Ye Internet par directly route nahi hote.
   * Ranges: `192.168.x.x`, `10.x.x.x`, `172.16.x.x` se `172.31.x.x`
2. **Public IP:** Ye IP aapke ISP (Internet Service Provider) dwara router ko diya jata hai. Ye pure internet par unique hota hai.

```text
    [ Internet ] <--- Public IP (e.g., 49.36.xx.xx) ---> [ Router ]
                                                              │
                                            ┌─────────────────┼─────────────────┐
                                            ▼                 ▼                 ▼
                                        Private IP        Private IP        Private IP
                                      `192.168.1.2`     `192.168.1.3`     `192.168.1.4`
                                         [ PC ]            [ Phone ]        [ Laptop ]
```

---

## 3. What is a Port?

IP address se hum target computer tak toh pahunch jate hain, par computer ke andar kaun-sa application chal raha hai, ye identify karne ke liye **Ports** ka use hota hai. Ports computer me logical communication endpoints hote hain.

> [!NOTE]
> **Real-World Analogy:**
> IP Address agar ek *Apartment Building* ka address hai, toh Port us building ka *Apartment Number* hai. 

### Common Port Numbers & Services:

| Port Number | Protocol/Service | Default Use |
| :--- | :--- | :--- |
| **20/21** | FTP (File Transfer Protocol) | Files download/upload karne ke liye |
| **22** | SSH (Secure Shell) | Remote system control (encrypted connection) |
| **23** | Telnet | SSH jaisa hi par non-encrypted (insecure) |
| **25** | SMTP (Simple Mail Transfer Protocol) | Mails send karne ke liye |
| **53** | DNS (Domain Name System) | Name to IP address mapping queries |
| **80** | HTTP (HyperText Transfer Protocol) | Non-secure web browsing |
| **110** | POP3 | Emails fetch karne ke liye |
| **143** | IMAP | Emails access karne ke liye (server-based synchronization) |
| **443** | HTTPS (Secure HTTP) | Secure and encrypted web traffic |
| **3306** | MySQL | Database management service |
| **3389** | RDP (Remote Desktop Protocol) | Windows remote GUI control |

---

## 4. What is a Protocol?

Protocols sets of rules hote hain jo determine karte hain ki devices aapas me data kaise exchange karenge. Network me bina standard protocols ke client and server ek dusre ki languages nahi samajh sakte. (Examples: HTTP, HTTPS, FTP, SSH, DNS).

---

## 5. TCP vs UDP: The Transport Layer Protocols

Nmap ke scanning modes ko samajhne ke liye TCP aur UDP ka difference clear hona mandatory hai.

### TCP (Transmission Control Protocol)
* **Connection-Oriented:** Data transmission se pehle connection setup kiya jata hai.
* **Reliable:** Har packet deliver hone par receiver ACK (Acknowledgement) frame bhejta hai. Data loss hone par automatic retransmit hota hai.
* **Slower:** Header overhead aur connections control processing ki wajah se thoda slow hota hai.
* **Use Cases:** Web Browsing (HTTP/S), Email (SMTP), Remote Access (SSH).

### UDP (User Datagram Protocol)
* **Connectionless:** Bina kisi handshake ya connection setup ke direct packets bhej diye jate hain.
* **Unreliable:** Delivery confirmation ya error recovery mechanisms nahi hote.
* **Faster:** Overhead minimal hai, isliye speeds bohot fast hoti hain.
* **Use Cases:** Online gaming, Voice calls (VoIP), Video Streaming, DNS requests.

```text
  [ TCP Connection Flow ]                   [ UDP Connection Flow ]
     Client         Server                     Client         Server
       │              │                          │              │
       ├── Handshake ─>                          ├─── Data ────> (Fast, No verification)
       │              │                          │              │
       ├─── Data ────>                          
       │              │                          
       <──── ACK ─────┤ (Verification check)
```

---

## 6. TCP Three-Way Handshake

TCP communication shuru karne se pehle ek 3-way handshake process follow karta hai:

1. **SYN (Synchronize):** Client target server ko connection request packet (SYN flag set) bhejta hai.
2. **SYN-ACK (Synchronize-Acknowledge):** Target server request accept karke acknowledgement aur synchronization response bhejta hai.
3. **ACK (Acknowledge):** Client server ko standard confirmation message (ACK flag set) bhejta hai. Connection status **ESTABLISHED** ho jata hai.

```text
    Client                                  Server
      │                                       │
      ├─────────── SYN Packet ───────────────>│ (Hey, let's connect)
      │                                       │
      │<────────── SYN-ACK Packet ────────────┤ (Sure, I'm ready)
      │                                       │
      ├─────────── ACK Packet ────────────────>│ (Got it, connecting now!)
      │                                       │
```

> [!IMPORTANT]
> **Nmap connection connection flags ko modify karke scan karta hai.** Jaise Half-open Scan (SYN scan) me Nmap sirf SYN packet bhejta hai, server se SYN-ACK aane par ACK ki jagah RST (Reset) bhej kar connection break kar deta hai.

---

## 7. DNS (Domain Name System)

DNS internet ka phonebook hai. Ye human-friendly domain names ko machine-friendly IP addresses me convert karta hai.

```text
google.com  ───[ DNS Server ]───>  142.250.x.x (IP Address)
```

---

## 8. Network Models: OSI vs TCP/IP

Network data travel flow ko step-by-step layer me distribute kiya gaya hai.

### OSI Model (Open Systems Interconnection) - 7 Layers
OSI model reference model hai jo logic systems ko divide karta hai:

| Layer Number | Layer Name | Common Protocol / Example |
| :--- | :--- | :--- |
| **7** | Application | HTTP, HTTPS, FTP, SSH, DNS |
| **6** | Presentation | SSL, TLS, Data Compression, JPEG |
| **5** | Session | NetBIOS, Session Establishment |
| **4** | Transport | TCP, UDP (Segment assembly) |
| **3** | Network | IP, Routers, Packets |
| **2** | Data Link | MAC Address, Switch, Ethernet Frames |
| **1** | Physical | Cables, Fiber, WiFi signals |

> [!TIP]
> **OSI Layers ko yaad rakhne ki trick:**
> **A**ll **P**eople **S**eem **T**o **N**eed **D**ata **P**rocessing
> (Application, Presentation, Session, Transport, Network, Data Link, Physical)

### TCP/IP Model - 4 Layers
Internet protocols standard implementation ke liye actual deployment me 4 layers use hoti hain:
1. **Application Layer** (Equivalent to OSI Layers 5, 6, 7)
2. **Transport Layer** (Equivalent to OSI Layer 4)
3. **Internet Layer** (Equivalent to OSI Layer 3)
4. **Network Access Layer** (Equivalent to OSI Layers 1, 2)

> [!NOTE]
> Nmap mostly **Transport (Layer 4)**, **Network (Layer 3)**, aur **Application (Layer 7)** par perform karta hai.

---

## Interview Notes & Q&A

### Q: TCP and UDP ke beech difference kya hai?
**A:** TCP ek connection-oriented and reliable protocol hai jo verification and handshake use karta hai. UDP connectionless speed optimized protocol hai jisme data delivery guarantee nahi hoti.

### Q: Nmap scans ke liye Three-Way Handshake kyu important hai?
**A:** Nmap systems ko scan karne ke liye TCP handshake status manipulate karta hai. Jaise target state check karne ke liye complete connection banana (TCP Connect Scan) ya connection open chhod dena (SYN Stealth Scan).

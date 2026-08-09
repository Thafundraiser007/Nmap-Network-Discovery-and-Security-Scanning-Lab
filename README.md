# Nmap-Network-Discovery-and-Security-Scanning-Lab
<img width="56" height="62" alt="Screenshot 2026-08-10 022703" src="https://github.com/user-attachments/assets/91590539-a277-4c85-a662-878f60c29e40" />

A hands-on network reconnaissance lab using **Nmap and Zenmap on Windows 11** to perform host discovery, TCP port scanning, service detection, OS fingerprinting, and network analysis against an authorized Android device connected through USB tethering.

---

## Lab Overview

The purpose of this lab was to gain practical experience using **Nmap** for network discovery, reconnaissance, and service enumeration.

My Windows 11 laptop was connected to the Internet through **USB tethering from my Android phone**. The phone therefore acted as the laptop's network gateway and provided a convenient device under my control for authorized Nmap testing.

Both the **Nmap command-line interface (CLI)** and the **Zenmap GUI** were used during the lab.

---

## Lab Environment

| Component | Details |
|---|---|
| Scanning Host | Windows 11 Laptop |
| Target | Android Phone / USB Tethering Gateway |
| Scanner IP | `10.59.107.40` |
| Target IP | `10.59.107.185` |
| Tool | Nmap 7.99 |
| GUI | Zenmap |
| Connection | USB Tethering |
| Authorization | Personally controlled device |

---

## Network Topology

```text
+----------------------------+
|     Windows 11 Laptop      |
|                            |
|      Nmap / Zenmap         |
|      10.59.107.40          |
+-------------+--------------+
              |
              | USB Tethering
              |
+-------------+--------------+
|       Android Phone        |
|                            |
|     Gateway / Target       |
|      10.59.107.185         |
+----------------------------+
```

---

# 1. Host Discovery

The first step was determining whether the target was reachable.

```bash
nmap -sn 10.59.107.185
```

Result:

```text
Nmap scan report for 10.59.107.185
Host is up.
```

The `-sn` option performs host discovery without running Nmap's normal port scan.

This confirmed that the Android USB-tethering gateway was active and reachable from the Windows laptop.

---

# 2. Basic TCP Port Scan

After confirming connectivity, I performed a standard Nmap scan:

```bash
nmap 10.59.107.185
```

The scan returned:

```text
Not shown: 999 closed tcp ports (reset)

PORT   STATE SERVICE
53/tcp open  domain
```

Nmap found **TCP port 53 open**, while the other 999 TCP ports included in the default scan were reported as closed.

Port 53 is commonly associated with **DNS (Domain Name System)**.

---

# 3. Service and Version Detection

After discovering TCP port 53, I used Nmap service/version detection to determine what software was responding on the port.

```bash
nmap -sV 10.59.107.185
```

Result:

```text
PORT   STATE SERVICE VERSION
53/tcp open  domain  dnsmasq 2.51
```

Nmap identified the responding software as:

```text
dnsmasq 2.51
```

This demonstrated the difference between identifying an **open port** and performing **service enumeration**.

The standard scan identified port 53 as a DNS-related service, while `-sV` actively probed the service and attempted to identify the software/version behind it.

---

# 4. Targeted Port Scan

Instead of scanning Nmap's default selection of common TCP ports, I tested several specific ports:

```bash
nmap -p 22,53,80,443 10.59.107.185
```

Result:

```text
PORT    STATE  SERVICE
22/tcp  closed ssh
53/tcp  open   domain
80/tcp  closed http
443/tcp closed https
```

The ports tested were:

| Port | Common Service | Result |
|---|---|---|
| 22/TCP | SSH | Closed |
| 53/TCP | DNS | Open |
| 80/TCP | HTTP | Closed |
| 443/TCP | HTTPS | Closed |

This demonstrated how the `-p` option can be used to investigate specific network services instead of performing a broader scan.

---

# 5. Operating System Detection

I then used Nmap's operating system fingerprinting capability:

```bash
nmap -O 10.59.107.185
```

Nmap returned:

```text
Device type: phone
Running: Google Android 9.X|10.X|11.X, Linux 4.X

OS details:
Android 9 - 11 (Linux 4.9 - 4.14)

Network Distance: 1 hop
```

Nmap successfully classified the target device as a:

```text
phone
```

It also identified the operating system family as **Google Android / Linux**.

This was consistent with the known target because `10.59.107.185` was the Android phone providing USB tethering to the Windows laptop.

Nmap OS fingerprinting is based on characteristics of network responses, so the specific Android and Linux versions reported should be considered **estimates rather than confirmation of the exact installed OS version**.

### Zenmap OS Detection

![Nmap OS Detection](Nmap-Zenmap%20GUI/nmap-os-detection.png)

---

# 6. Aggressive Detection Scan

I then performed a more comprehensive Nmap scan:

```bash
nmap -A 10.59.107.185
```

The `-A` option enables several advanced detection capabilities, including:

- Operating system detection
- Service/version detection
- Default NSE scripts
- Traceroute

The scan again identified:

```text
PORT   STATE SERVICE VERSION
53/tcp open  domain  dnsmasq 2.51
```

Nmap's script output also returned:

```text
dns-nsid:
  bind.version: dnsmasq-2.51
```

The target was fingerprinted as:

```text
Device type: phone
Running: Google Android 9.X|10.X|11.X, Linux 4.X
OS details: Android 9 - 11 (Linux 4.9 - 4.14)
Network Distance: 1 hop
```

Nmap also performed a traceroute:

```text
TRACEROUTE

HOP  RTT      ADDRESS
1    1.19 ms  10.59.107.185
```

The target being **one network hop away** is consistent with the laptop being directly connected to the phone through USB tethering.

### Zenmap Aggressive Scan

![Nmap Aggressive Detection Scan](Nmap-Zenmap%20GUI/nmap-aggressive-scan.png)

---

# 7. Full TCP Port Scan

Nmap normally scans a selection of commonly used ports.

To examine the complete TCP port range, I performed:

```bash
nmap -p- 10.59.107.185
```

The `-p-` option tells Nmap to scan **TCP ports 1 through 65535**.

Result:

```text
Not shown: 65534 closed tcp ports (reset)

PORT   STATE SERVICE
53/tcp open  domain
```

The scan completed in approximately:

```text
5.09 seconds
```

Out of all **65,535 TCP ports**, only:

```text
53/tcp
```

was reported open on the Android USB-tethering interface.

The other **65,534 TCP ports were reported closed**.

### Zenmap Full TCP Port Scan

![Nmap Full TCP Port Scan](Nmap-Zenmap%20GUI/nmap-full-port-scan.png)

---

# Key Findings

| Finding | Result |
|---|---|
| Target | Android USB-Tethering Gateway |
| Target IP | `10.59.107.185` |
| Target Reachability | Online |
| Device Type | Phone |
| OS Family | Android / Linux |
| Network Distance | 1 hop |
| Open TCP Port | 53 |
| Service | DNS |
| Detected Software | dnsmasq 2.51 |
| Full TCP Range | 65,535 ports scanned |
| Closed TCP Ports | 65,534 |

---

# Commands Used

```bash
# Host discovery
nmap -sn 10.59.107.185

# Standard TCP port scan
nmap 10.59.107.185

# Service/version detection
nmap -sV 10.59.107.185

# Target selected ports
nmap -p 22,53,80,443 10.59.107.185

# Operating system detection
nmap -O 10.59.107.185

# Aggressive detection
nmap -A 10.59.107.185

# Full TCP port scan
nmap -p- 10.59.107.185
```

---

# Nmap Options Used

| Option | Purpose |
|---|---|
| `-sn` | Host discovery without normal port scanning |
| `-p` | Scan specified ports |
| `-sV` | Service and version detection |
| `-O` | Operating system detection |
| `-A` | OS detection, version detection, NSE scripts and traceroute |
| `-p-` | Scan all TCP ports from 1–65535 |

---

# Skills Demonstrated

This lab demonstrates practical experience with:

- Nmap
- Zenmap
- Network reconnaissance
- Host discovery
- TCP port scanning
- Full-range TCP scanning
- Targeted port scanning
- Service enumeration
- Service/version detection
- OS fingerprinting
- TCP/IP fundamentals
- DNS service identification
- Network troubleshooting
- Traceroute analysis
- Open/closed port interpretation
- Windows networking tools
- Basic security reconnaissance methodology

---

# What I Learned

This lab demonstrated how Nmap can be used for much more than simply checking whether a network device is online.

I progressively moved through several stages of network reconnaissance:

```text
Host Discovery
      ↓
Port Scanning
      ↓
Service Detection
      ↓
OS Fingerprinting
      ↓
NSE / Aggressive Detection
      ↓
Full TCP Port Scan
```

Each stage provided additional information about the target.

The lab also helped demonstrate the difference between:

**Host Discovery**

Determining whether a network device is reachable.

**Port Scanning**

Determining which TCP ports are open or closed.

**Service Detection**

Attempting to identify the software listening behind an open port.

**OS Fingerprinting**

Estimating the operating system of a remote device based on its network responses.

**Full Port Scanning**

Examining the entire TCP port range rather than Nmap's default selection of common ports.

The target being my own Android phone connected through USB tethering provided a controlled environment where the Nmap findings could be compared against a device whose general identity was already known.

One particularly useful result was Nmap correctly identifying the target device type as a **phone** and the operating system family as **Android/Linux**.

---

# Ethical Use

All Nmap scans demonstrated in this project were performed against a device under my control.

Network scanning and reconnaissance tools should only be used against systems, devices, and networks that you own or have explicit authorization to test.

This project was performed strictly for educational and networking lab purposes.

---

# Repository Structure

```text
Nmap-Network-Discovery-and-Security-Scanning-Lab/
│
├── README.md
│
└── Nmap-Zenmap GUI/
    ├── nmap-os-detection.png
    ├── nmap-aggressive-scan.png
    └── nmap-full-port-scan.png
```

---

## Author

**Jamill Naipao**

Aspiring Network / Infrastructure Engineer

Hands-on experience and interests include:

`Networking` • `Cisco` • `Windows Server` • `Linux` • `Nmap` • `Wireshark` • `EVE-NG` • `VMware` • `Network Automation` • `Infrastructure` • `Cybersecurity` • `Telecommunications`

---

*This repository documents a practical networking lab conducted for educational and portfolio purposes.*

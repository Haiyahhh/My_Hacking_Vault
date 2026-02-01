---
tags:
  - 🛠️
up:
  - "[[02 - Network Reconnaissance]]"
aliases: []
creation_date: 2026-01-22
last_modified: 2026-01-22
---
# 🛠️ [[Nmap]]
**Primary:** [[01 - Network Security]]

**Secondary:** [[02 - Network Reconnaissance]]

## Installation
```bash
sudo apt install nmap
````

## Common Commands
### Hack The Box / CTFs
Scanning ports, specify version and output in a file, mostly used for machines in **HackTheBox**.
```bash
nmap -p- -min-rate=1000 -T4 -sC -sV -oN <outfile> <target_ip> 
```

### Scanning Phases (maybe a little more practical?)
The above command is actually quite noisy as it scans every ports with high `--min-rate` and `T4` speed. In real-life cases, quieter scans are often preferred and the scan are split into difference phases in order to avoid detection.

**1. Host detection**
List out target in the IP range and resolve their hostnames.
```bash
nmap -sL 192.162.1.0/24
```

**2. Host discovery**
 There may be dead IPs, this step helps indentifying them.
```bash
nmap -sn -PS443,80 192.162.1.0/24
```
**3. Port enumeration**
This is the main things and the hardest problem. Key things to remember:
- Use slow timing (`-T1` or `-T2`) only.
- Scan only a small number of ports (50 ports) only. We can know what port we may need to know by looking for services' default ports on the Internet.
- 
 
**4. Obfuscation**

## Tips & Tricks

- Tip 1...
    

---

**Reference:** [Official Docs](https://www.google.com/search?q=url)
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
This command scans ports, specifies service version and outputs in a file, mostly used for machines in **HackTheBox**.
```bash
nmap -p- -T4 -sC -sV -oN -oX [outfile] [target_ip] 
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
nmap -sn -PS443,80 [target]
```
**3. Port enumeration**
This is the main things and the hardest problem. Key things to remember:
- Use slow timing (`-T1` or `-T2`) only.
- Scan only a small number of ports (50 ports) only. We can know what port we may need to know by looking for services' default ports on the Internet.
- Break packets into small chunks using `-f` flag.
```bash
nmap -T1 --top-ports 50 -f [target]
```
 
**4. Obfuscation**
This is where we utilize evasion techniques, something like using decoy IP addresses to hide our own addresses to avoid being blocked
```bash
nmap -D RND:10 [target]
```
The above command spoofs scan from 10 different source to sneak through the IDS.

## Tips & Tricks

- Scanning for vulnerability (SMB misconfigurations or old SSL versions, etc.
```bash
nmap -sV --script=vuln [target]
```
    

---

**Reference:** [Official Docs](https://www.google.com/search?q=url)
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
### Hack The Box / CTF
Scanning ports, specify version and output in a file, mostly used for machines in **HackTheBox**.
```bash
nmap -p- -min-rate=1000 -T4 -sC -sV -oN <outfile> <target_ip> 
```
This command is actually quite noisy. In real-life cases, quieter scans are often preferred and the scan are split into difference phases in order to avoid detection.
```bash
nmap 
```
## Tips & Tricks

- Tip 1...
    

---

**Reference:** [Official Docs](https://www.google.com/search?q=url)
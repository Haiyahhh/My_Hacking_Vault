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
Scanning ports, specify version and output in a file, mostly used for machines in **HackTheBox**.
```bash
nmap -p- -min-rate=1000 -T4 -sC -sV -oA <outfile> <target_ip> 
```
In case where slower and quieter co
## Tips & Tricks

- Tip 1...
    

---

**Reference:** [Official Docs](https://www.google.com/search?q=url)
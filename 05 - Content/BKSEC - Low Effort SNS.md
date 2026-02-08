---
tags:
  - 🚩
up:
  - "[[02 - Data Exfiltration (incomplete)]]"
platform: BKSec Training
difficulty: Easy
creation_date: 2026-02-08
last_modified: 2026-02-08
---

# 🚩 [[BKSEC - Low Effort SNS]]
**Primary:** [[01 - Web Security]]

**Secondary:** [[02 - Data Exfiltration (incomplete)]]

**Related Techniques:** [[SQL Injection (incomplete)]]

**Related Tools:** [[SQLmap (incomplete)] [[Gobuster (incomplete)]]

## Executive Summary
* **URL:** `http://103.77.175.40:8021`
* **Key Technique:** SQL injection into credential exfiltration
* **Status:** `Completed`

---

## Reconnaissance
### Web Enumeration
First look around the website, I saw the pages were served  using `PHP`, therefore I tried enumerating pages using `Gobuster` with the `-x php` option.
```bash
gobuster dir --url http://103.77.175.40:8021 --wordlist ~/Downloads/SecLists/Discovery/Web-Content/raft-small-directories-lowercase.txt -x php

===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://103.77.175.40:8021
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /home/kali/Downloads/SecLists/Discovery/Web-Content/raft-small-directories-lowercase.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8
[+] Extensions:              php
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/logout.php           (Status: 302) [Size: 0] [--> index.php]
/login.php            (Status: 200) [Size: 1037]
/home.php             (Status: 302) [Size: 0] [--> index.php]
/index.php            (Status: 200) [Size: 1521]
/signup.php           (Status: 200) [Size: 1544]
/server-status        (Status: 403) [Size: 280]
```

I checked the pages out and look around the website. It seems like the status of action we do will be reflected to us through URL redirection back to the home page.



I  suspected that this challenge might be an SQL injection challenge. 

---

## Data Exfiltration
Based on the things we observed

---

## Privilege Escalation (Root)

**Current User:** `www-data`

### Enumeration

- **LinPeas Findings:** `Vulnerable Sudo version`
    

### Exploitation

Bash

```
# Commands to get root
```

---

## Loot & Flags

- [ ] **User Flag:** `hash_here`
    
- [ ] **Root Flag:** `hash_here`
    
- [ ] **Credentials:**
    
    - `user:password`
        

---

**References:** [Link](https://www.google.com/search?q=url)
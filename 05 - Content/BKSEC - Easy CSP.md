---
tags:
  - 🚩
up:
  - "[[02 - (incomplete) Data Exfiltration]]"
platform: BKSEC Training
difficulty: Easy
creation_date: 2026-02-23
last_modified: 2026-02-23
---

# 🚩 [[BKSEC - Easy CSP]]
**Primary:** [[01 - Web Security]]

**Secondary:** [[02 - (incomplete) Data Exfiltration]]

**Related Techniques:** [[(incomplete) CSP (Content Security Policy) Bypass]]

**Related Tools:**  


## Executive Summary
* **IP:** `http://103.77.175.40:8222/`
* **Key Technique:** CSP bypass into admin cookie exfiltration
* **Status:** Completed`

---

## Reconnaissance
### Gobuster Scan
```bash
===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://103.77.175.40:8222/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /home/kali/Downloads/SecLists/Discovery/Web-Content/raft-small-directories-lowercase.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/server-status        (Status: 403) [Size: 320]
Progress: 17769 / 17769 (100.00%)
===============================================================
Finished
===============================================================
````

There is a forbidden directory that might be our target.

The website was a website vulnerable to XSS as the input containing HTML tags were directly pasted into the page without escaping or sanitization.

![[Pasted image 20260223090507.png]]

Since the name of the challenge is **Easy CSP**, I figure I should take a look at the tr

---
### Web Enumeration

## Foothold (User)

**Path:** <% tp.file.cursor(1) %>

### Step 1: Discovery

(What did you find?)

### Step 2: Exploitation

(The exact payload or exploit used).

> [!failure] 🐇 Rabbit Hole I spent time trying to brute force SSH.
> 
> - **Correction:** Always check for `id_rsa` keys in web directories first.
>     

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
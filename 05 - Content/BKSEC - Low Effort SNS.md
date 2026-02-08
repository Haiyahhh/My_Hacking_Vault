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

**Related Tools:** [[SQLmap (incomplete)]

## Executive Summary
* **URL:** `[http://103.77.175.40:8021](http://103.77.175.40:8021/)`
* **OS:** Linux / Windows
* **Key Technique:** (e.g., SQLi -> Cron Job Escalation)
* **Status:** `In Progress`

---

## Reconnaissance
### Nmap Scan
```bash
# Paste initial scan here
````

### Web Enumeration

- **Technologies:** (Apache, PHP, etc.)
    
- **Fuzzing Results:**
    
    - `/admin` (403)
        
    - `/images` (200)
        

---

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
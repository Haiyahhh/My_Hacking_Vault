---
tags:
  - 🚩
up:
  - "[[02 - SQL Injection]]"
platform: BKSec Training
difficulty: Easy
creation_date: 2026-02-08
last_modified: 2026-02-08
---

# 🚩 [[BKSEC - Low Effort SNS]]
**Primary:** <% tp.file.cursor(1) %>

**Secondary:**

## Executive Summary
* **IP:** `10.10.10.x`
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
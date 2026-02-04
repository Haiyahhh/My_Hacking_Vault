---
tags:
  - 🚩
up:
  - "[[01 - Web Security]]"
platform: HackTheBox
difficulty: Easy
creation_date: 2026-02-04
last_modified: 2026-02-04
---

# 🚩 [[HTB - GiveBack]]
**Primary:** [[01 - Web Security]]

**Secondary:**

## Executive Summary
* **IP:** `10.129.242.171`
* **OS:** Linux 
* **Key Technique:** 
* **Status:** `In Progress`

---

## Reconnaissance
### Nmap Scan
```bash
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 66:f8:9c:58:f4:b8:59:bd:cd:ec:92:24:c3:97:8e:9e (ECDSA)
|_  256 96:31:8a:82:1a:65:9f:0a:a2:6c:ff:4d:44:7c:d3:94 (ED25519)
80/tcp open  http    nginx 1.28.0
| http-methods: 
|_  Supported Methods: HEAD
|_http-server-header: nginx/1.28.0
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
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
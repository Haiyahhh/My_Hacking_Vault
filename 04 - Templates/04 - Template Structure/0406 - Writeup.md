---
tags: [🚩]
up: ["[[01 - ...]]"]
platform: HackTheBox  # Change to TryHackMe/PortSwigger as needed
difficulty: Easy      # Easy / Medium / Hard / Insane
creation_date: <% tp.file.creation_date("YYYY-MM-DD") %>
last_modified: <% tp.file.last_modified_date("YYYY-MM-DD") %>
---

# 🚩 [[<% tp.file.title %>]]
**Primary:** <% tp.file.cursor(1) %>

**Secondary:**

**Related Techniques or Tools:** 

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
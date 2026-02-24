---
tags:
  - 🚩
up:
  - "[[02 - (incomplete) Data Exfiltration]]"
platform: BKSEC Training
difficulty: Easy
creation_date: 2026-02-24
last_modified: 2026-02-24
---

# 🚩 [[BKSEC - Ezzz]]
**Primary:** [[01 - Web Security]]

**Secondary:**  [[02 - (incomplete) Data Exfiltration]]

**Related Techniques:** [[(incomplete) SQL Injection|SQLi]]

## Executive Summary
* **URL:** `http://103.77.175.40:8211/`
* **Key Technique:** Spotting a parameter that is vulnerable to Boolean-based SQL Injection attack that leads to flag.
* **Status:** `Completed`

---

## Reconnaissance
### Gobuster Scans
```bash
# Insert Gobuster scan against the URL
````

### Arjun Scans
```bash
# Insert Arjun scan agains the URL
```

### Web Enumeration
The website is a event management service where if I input an event and its date the event will be shown on the website.

In challenges like this, I often think about an SQLi vulnerability as the data are reflected to the user, especially when it is also under the format of some sort of table, just like in relational databases.

I tried fuzzing the input field to see if there were any 


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
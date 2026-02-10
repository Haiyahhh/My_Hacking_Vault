---
tags:
  - 🚩
up:
  - "[[02 - Enumeration]]"
platform: HackTheBox
difficulty: Easy
creation_date: 2026-02-10
last_modified: 2026-02-10
---

# 🚩 [[BKSEC - Happy Soldier]]
**Primary:** [[01 - Web Security]]

**Secondary:** [[02 - Enumeration]]

**Related Techniques:** [[(incomplete) PHP Object Injection]]

**Related Tools:** [[(incomplete) FFUF]], [[(incomplete) BurpSuite]], [[(incomplete) Arjun]]


## Executive Summary
* **URL:** `http://103.77.175.40:8141`
* **OS:** Linux
* **Key Technique:** Argument Fuzzing to reveal hidden argument, this leads to a local file content revelation, then we change the PHP object to get the flag.
* **Status:** `Completed`

---

## Reconnaissance
### Gobuster Scan
```bash
# Paste initial scan here
````
Both `vhost` and `dir` scans revealed nothing so I decided to move on.
Looking at `Wappalyzer` I saw that the website seem to be using PHP so I decided to try finding out if any PHP pages were exposed.
```bash

```
Again, I saw nothing, so `Gobuster` seemed to reveal nothing significant.
### Web Enumeration
Based on `Wappalyzer` the page is using PHP version 8.0.30 so I tried to find out if there were any known vulnerabilities for this version on the Internet, and the results suggests that 8.0.30 seems to be a secure version.

[insert image from Windows machine]

Take a look at the source code of the page, at the end there is a script about the che

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
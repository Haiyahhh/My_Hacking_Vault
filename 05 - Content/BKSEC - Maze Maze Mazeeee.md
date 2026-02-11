---
tags:
  - 🚩
up:
  - "[[02 - Enumeration]]"
platform: BKSEC Training
difficulty: Easy
creation_date: 2026-02-11
last_modified: 2026-02-11
---

# 🚩 [[BKSEC - Maze Maze Mazeeee]]
**Primary:** [[01 - Web Security]]

**Secondary:** [[02 - Enumeration]]

## Executive Summary
* **IP:** `http://103.77.175.40:8031`
* **Key Technique:** Enumeration robots.txt into writing automation script for exploit.
* **Status:** `Completed`

---

## Reconnaissance
### Web Enumeration
The page served a static home page that has an `<h1>` element saying "ARE YOU ROBOT?", inspecting the page gives me know that the title of this page is called `Maze Entrance`.

![[Pasted image 20260211211343.png]]

I instinctively know this might be a hint for me to check out the page's `robots.txt` file (which is a page that was normally made as a guideline for search engines' bots or crawlers). Thankfully the file exists and can be access.

![[Pasted image 20260211211735.png]]

There was a base64 encoded string `L3BhZ2VzL3BhZ2UtMS1Ub21ic3RvbmUuaHRtbA==` decoded to be `/pages/page-1-Tombstone.html`

Following the link I got to things that seems to be the real content of this challenge which is a page filled with colored rectangle:

![[Pasted image 20260211212141.png]]

I tried clicking on one of the rectangle but nothing happened. Then I view the page source.

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
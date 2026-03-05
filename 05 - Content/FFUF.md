---
tags:
  - 🛠️
up:
  - "[[02 - Enumeration]]"
aliases: []
creation_date: 2026-02-10
last_modified: 2026-02-10
---
# 🛠️ [[FFUF]]
**Primary:** [[01 - Web Security]]

**Secondary:** [[02 - Enumeration]]

## Installation
```bash
sudo apt update
sudo apt install ffuf
````

## Common Commands
**Fuzz Faster You Fool!!!!**

|**Flag**|**Name**|**Description**|
|---|---|---|
|`-u`|**URL**|The target URL. Must contain the `FUZZ` keyword.|
|`-w`|**Wordlist**|Path to your wordlist (e.g., `/usr/share/seclists/...`).|
|`-c`|**Color**|Enables colorized output (essential for readability).|
|`-v`|**Verbose**|Shows the full URL for each result (great for copying/pasting).|
|`-recursion`|**Recursion**|If it finds a directory, it starts fuzzing _inside_ that directory automatically.|
|`-t`|**Threads**|Speed control. Default is 40. Bump to 100+ for speed, lower for stealth.|

## Tips & Tricks



## Related Usage

### CTFs

### Techniques

---

**References:** [Link](https://www.google.com/search?q=url)
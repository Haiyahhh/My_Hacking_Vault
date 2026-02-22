---
tags:
  - 🛠️
up:
  - "[[02 - Enumeration]]"
aliases: []
creation_date: 2026-02-08
last_modified: 2026-02-08
---
# 🛠️ [[(incomplete) Gobuster]]
**Primary:** [[01 - Web Security]]

**Secondary:** [[02 - Enumeration]]

## Installation
```bash
sudo apt install gobuster
````

## Core Modes
This defines what you are brute-forcing.

|**Mode**|**Description**|
|---|---|
|`dir`|Directories|
|`dns`|Subdomains|
|`vhost`|Virtual hosts (one IP hosts multiple sites)|
|`s3`|Public Amazon S3 buckets|
|`fuzz`|Generic fuzzing mode|

## Common Commands
1. Directory Brute-Forcing
```bash
gobuster dir -u <target_URL> -w ~/Downloads/SecLists/.
```
1. Search for Specific File Extensions (`.html`, `.php`, `.bak`, `.txt`,...)
2. 

## Tips & Tricks

- Tip 1...
    

## Related Usage

### CTFs

### Techniques

---

**References:** [Link](https://www.google.com/search?q=url)
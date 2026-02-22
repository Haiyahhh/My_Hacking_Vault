---
tags:
  - 🛠️
up:
  - "[[02 - Enumeration]]"
aliases: []
creation_date: 2026-02-08
last_modified: 2026-02-08
---
# 🛠️ [[Gobuster]]
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
1. **Directory Brute-Forcing**
```bash
gobuster dir -u <target_URL> -w ~/Downloads/SecLists/Discovery/Web-Content/raft-small-directories-lowercase.txt
```

2. **Search for Specific File Extensions (`.html`, `.php`, `.bak`, `.txt`,...)**
```bash
gobuster dir -u <target_URL> -w ~/Downloads/SecLists/Discovery/Web-Content/raft-small-directories-lowercase.txt -x php,html,txt,bak
```

3. **Fuzzing for subdomains**
```bash
gobuster dns -d example.com -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

4. **Fuzzing for virtual hosts**
```bash
gobuster vhost -u <target_URL> -w ~/Downloads/SecLists/Discovery/DNS/bitquark-subdomains-top100000.txt --append-domain
```

5. **Speeding up**
```bash
gobuster dir -u <target_URL> -w ~/Downloads/SecLists/Discovery/Web-Content/raft-small-directories-lowercase.txt -t 50
```

6. **Quiet Mode** 
Omit banner and progress bar then pipe the output to a file.
```bash
gobuster dir -u <target_URL> -w ~/Downloads/SecLists/Discovery/Web-Content/raft-small-directories-lowercase.txt -q > found_dirs.txt
```

7. **Handling SSL/TLS issues**
Used when the target has an expired or self-signed certificate.
```bash
gobuster dir -u <target_URL> -w ~/Downloads/SecLists/Discovery/Web-Content/raft-small-directories-lowercase.txt -k
```

## Related Usage

### CTFs
[[BKSEC - Low Effort SNS]]

### Techniques
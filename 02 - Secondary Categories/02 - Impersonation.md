---
tags:
  - 🗺️
up:
  - "[[01 - Web Security]]"
  - "[[01 - Network Security]]"
aliases: []
creation_date: 2026-02-25
last_modified: 2026-02-25
---
# 📂 [[02 - Impersonation]] 

## Overview 
Impersonation is the act of a process or a user capturing the security context of another user to perform actions on their behalf. Often involving tokens or cookies in some system or specific security context. 

> **Note:** It is important to differentiate ***Impersonation*** and ***Privilege Escalation***. While Privesc often focus on looking for system misconfigurations and vulnerabilities, Impersonation on the other hand looking for active user sessions and also sometimes system misconfigurations to steal the target's identity and act on their behalf. This leads to sometimes these two concepts overlap, for example exploiting a `Potato Attack` means using Impersonation to achieve Privilege Escalation.

---

## 🧠 Techniques

```dataview
TABLE creation_date AS "Created" 
FROM "05 - Content" 
WHERE contains(up, this.file.link) AND contains(tags, "🧠") 
SORT file.name ASC
```

## 🛠️ Tools

```dataview
TABLE creation_date AS "Created"
FROM "05 - Content"
WHERE contains(up, this.file.link) AND contains(tags, "🛠️")
SORT file.name ASC
```

## 🚩 Related CTF Operations

```dataview
TABLE status, difficulty
FROM "10_Operations"
WHERE contains(up, this.file.link) OR contains(tags, "🚩")
SORT creation_date DESC
```
---
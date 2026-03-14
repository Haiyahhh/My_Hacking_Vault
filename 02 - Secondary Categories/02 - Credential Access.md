---
tags:
  - 🗺️
up:
  - "[[01 - Web Security]]"
  - "[[01 - Network Security]]"
aliases: []
creation_date: 2026-03-05
last_modified: 2026-03-09
---
# 📂 [[02 - Credential Access]] 

## Overview 
Credential access is the act of stealing or cracking the credentials of a service or a user inside a system often leads to persistence, impersonation or privilege escalation. The credentials can either be the password, tickets, tokens, etc.

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
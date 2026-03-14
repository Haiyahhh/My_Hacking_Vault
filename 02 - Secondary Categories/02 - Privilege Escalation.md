---
tags:
  - 🗺️
up:
  - "[[01 - Web Security]]"
  - "[[01 - Network Security]]"
aliases: []
creation_date: 2026-03-05
last_modified: 2026-03-05
---
# 📂 [[02 - Privilege Escalation]] 

## Overview 
Note in this category is about how to escalate privilege from low privilege users in a system. Things like exploiting misconfigurations inside a server, exploiting vulnerabilities of a binary that allow actions of higher privilege user.

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
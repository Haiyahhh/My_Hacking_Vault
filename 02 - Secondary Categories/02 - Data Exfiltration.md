---
tags:
  - 🗺️
up:
  - "[[01 - Web Security]]"
  - "[[01 - Network Security]]"
aliases: []
creation_date: 2026-02-08
last_modified: 2026-02-08
---
# 📂 [[02 - Data Exfiltration]] 

## Overview 
Notes of this categories will discuss about how to exfiltrate data from inside of a system using different techniques or tools. **SQL injection**, for example, is a technique that can be used to exfiltrate data from a database in the backend server.

---

## 🧠 Techniques (Concepts)

```dataview
TABLE creation_date AS "Created" 
FROM "05 - Content" 
WHERE contains(up, this.file.link) AND contains(tags, "🧠") 
SORT file.name ASC
```

## 🛠️ Tools (Arsenal)

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
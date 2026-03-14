---
tags:
  - 🗺️
up:
  - "[[01 - Network Security]]"
  - "[[01 - Web Security]]"
aliases: []
creation_date: 2026-01-22
last_modified: 2026-01-22
---
# 📂 [[02 - Enumeration]] 

## Overview 
Notes of this category is about how to do enumeration, meaning to get more information either about a website, or an infiltrated system. This is the most basic and also the most important part of all exploits in my opinion.

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
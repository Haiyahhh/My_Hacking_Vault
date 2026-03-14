---
tags: [🗺️]
up: ["[[01 - ...]]"]
aliases: []
creation_date: <% tp.file.creation_date("YYYY-MM-DD") %>
last_modified: <% tp.file.last_modified_date("YYYY-MM-DD") %>
---
# 📂 [[<% tp.file.title %>]] 

## Overview 
(Explain what this vulnerability class or topic is.)

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
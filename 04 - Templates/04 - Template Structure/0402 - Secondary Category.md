---
tags: [🗺️]
aliases: []
creation_date: <% tp.file.creation_date("YYYY-MM-DD") %>
last_modified: <% tp.file.last_modified_date("YYYY-MM-DD") %>
---
# 🗺️ [[<% tp.file.title %>]]
**Primary:** [[]]

## 📖 Overview
(Write a brief description of this domain here.)

---

## 🗂️ Tertiary Categories

```dataview
TABLE creation_date AS "Created"
FROM "02 - Secondary Categories"
WHERE contains(up, this.file.link)
SORT file.name ASC
```

---

**Last Modified:** <% tp.file.last_modified_date("YYYY-MM-DD") %>
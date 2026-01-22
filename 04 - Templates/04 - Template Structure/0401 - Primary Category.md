---
tags: [🗺️]
aliases: []
creation_date: <% tp.file.creation_date("YYYY-MM-DD") %>
last_modified: <% tp.file.last_modified_date("YYYY-MM-DD") %>
---
# 🗺️ [[<% tp.file.title %>]]
**Parent:** [[000 - Global Index]]

## Overview
(Write a brief description of this domain here.)

---

## Secondary Categories 

```dataview
TABLE creation_date AS "Created"
FROM "02 - Secondary Categories"
WHERE contains(up, this.file.link)
SORT file.name ASC
```

---
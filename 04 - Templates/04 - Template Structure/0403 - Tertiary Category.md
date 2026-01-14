---
tags: [🗺️] 
aliases: []
creation_date: <% tp.file.creation_date("YYYY-MM-DD") %>
last_modified: <% tp.file.last_modified_date("YYYY-MM-DD") %>
---
# 🗺️ [[<% tp.file.title %>]]
**Primary:** <% tp.file.cursor(1) %>  <-- (Link to your Primary here, e.g., [[Web Security]])

**Secondary:**

## 📖 Definition
(What is this vulnerability class or topic?)

---

## 🧠 Knowledge Base (Techniques & Notes)

```dataview
TABLE up AS "Parent"
FROM "03 - Tertiary Categories" OR "05 - Content"
WHERE contains(up, this.file.link)
SORT file.name ASC
````

---

**Last Modified:** <% tp.file.last_modified_date("YYYY-MM-DD") %>
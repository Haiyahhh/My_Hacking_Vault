---
tags:
  - 🚩
up:
  - "[[02 - Enumeration]]"
platform: BKSEC Training
difficulty: Easy
creation_date: 2026-02-11
last_modified: 2026-02-11
---

# 🚩 [[BKSEC - Maze Maze Mazeeee]]
**Primary:** [[01 - Web Security]]

**Secondary:** [[02 - Enumeration]]

## Executive Summary
* **IP:** `http://103.77.175.40:8031`
* **Key Technique:** Enumeration robots.txt into writing automation script for exploit.
* **Status:** `Completed`

---

## Reconnaissance
### Web Enumeration
The page served a static home page that has an `<h1>` element saying "ARE YOU ROBOT?", inspecting the page gives me know that the title of this page is called `Maze Entrance`.

![[Pasted image 20260211211343.png]]

I instinctively know this might be a hint for me to check out the page's `robots.txt` file (which is a page that was normally made as a guideline for search engines' bots or crawlers). Thankfully the file exists and can be access.

![[Pasted image 20260211211735.png]]

There was a base64 encoded string `L3BhZ2VzL3BhZ2UtMS1Ub21ic3RvbmUuaHRtbA==` decoded to be `/pages/page-1-Tombstone.html`

Following the link I got to things that seems to be the real content of this challenge which is a page filled with colored rectangle:

![[Pasted image 20260211212141.png]]

I tried clicking on one of the rectangle but nothing happened. Then I inspect the page:

![[Pasted image 20260211212925.png]]

There is the title, and hundreds of rectangle elements inside `maze-container` element. I check out the cookie but seems like the page does not uses cookie. To find out more about what I need to do, I open `BurpSuite` but nothing notable was found.

Then I tried asking Gemini for the challenge and it suggested me to find a rectangle that contains a link to next challenge (aka. one that contain an `<a>` tag)

Looking at the response I got from `Burpsuite` and look for an `<a>` tag, there is indeed one single tag existed.

![[Pasted image 20260211230941.png]]

---
## Round 1: Find the `<a>` tag
I saw the way and starts repeatedly doing the same thing as finding the `<a>` tag for about 20 times, and I realized maybe there can be hundreds of such levels, doing it manually would takes tremendous time. So I decide to write a script to navigate through the challenge.

```python
import requests
from bs4 import BeautifulSoup
from urllib.parse import urljoin
import sys

current_url = "http://103.77.175.40:8031/pages/page-1-Tombstone.html"
  
while True:
    print(f"Checking: {current_url}")
    try:
        response = requests.get(current_url)
        page_content = response.text
        if "BKSEC{" in page_content:
            print("flag at:" + current_url)
            break
            
        soup = BeautifulSoup(page_content, 'html.parser')
        link_tag = soup.select_one('.rect a')
        relative_link = link_tag.get('href')
        next_url = urljoin(current_url, relative_link)
        current_url = next_url
        
    except Exception as e:
        print(f"ERROR: {e}")
        break
```

The code did not return a flag but stopped at level 101, when I entered the page at that level there is a button that leads to a second round.

```powershell
Checking: http://103.77.175.40:8031/pages/page-96-Avra_Valley.html
Checking: http://103.77.175.40:8031/pages/page-97-Altar_Valley.html
Checking: http://103.77.175.40:8031/pages/page-98-San_Rafael_Valley.html
Checking: http://103.77.175.40:8031/pages/page-99-Sonoita_Valley.html
Checking: http://103.77.175.40:8031/pages/page-100-Sulphur_Springs_Valley.html
Checking: http://103.77.175.40:8031/pages/page-101-qncdl1248dbsl.html
ERROR: 'NoneType' object has no attribute 'get'
```

![[Pasted image 20260211232402.png]]

---
## Round 2: Find the flag.
Clicking on the **PROCEED TO ROUND 2** led me to a webpage that was format like a file system, where there were folders and files. Clicking on the folder led me to the `index.html` file of the directory of that folder, which had the same system file interface. For example:
-	`/round2_qncdl1248dbsl/index.html` (parent) ->  `/round2_qncdl1248dbsl/giftsgssul/index.html`

Clicking on the html file (all files were html apparently) leads me to the html file of the same directory.
- `/round2_qncdl1248dbsl/index.html` -> `/round2_qncdl1248dbsl/dnvgzrekzl.html`

The script I used this time is:
```python

```

---

## Loot & Flags

- [ ] **User Flag:** `hash_here`
    
- [ ] **Root Flag:** `hash_here`
    
- [ ] **Credentials:**
    
    - `user:password`
        

---

**References:** [Link](https://www.google.com/search?q=url)
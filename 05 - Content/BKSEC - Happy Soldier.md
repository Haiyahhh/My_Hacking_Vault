---
tags:
  - 🚩
up:
  - "[[02 - Enumeration]]"
platform: HackTheBox
difficulty: Easy
creation_date: 2026-02-10
last_modified: 2026-02-10
---

# 🚩 [[BKSEC - Happy Soldier]]
**Primary:** [[01 - Web Security]]

**Secondary:** [[02 - Enumeration]]

**Related Techniques:** [[(incomplete) PHP Object Injection]]

**Related Tools:** [[(incomplete) FFUF]], [[(incomplete) BurpSuite]], [[(incomplete) Arjun]]


## Executive Summary
* **URL:** `http://103.77.175.40:8141`
* **OS:** Linux
* **Key Technique:** Argument Fuzzing to reveal hidden argument, this leads to a local file content revelation, then we change the PHP object to get the flag.
* **Status:** `Completed`

---

## Reconnaissance
### Gobuster Scan
```bash
# Paste initial scan here
````
Both `vhost` and `dir` scans revealed nothing so I decided to move on.
Looking at `Wappalyzer` I saw that the website seem to be using PHP so I decided to try finding out if any PHP pages were exposed.
```bash

```
Again, I saw nothing, so `Gobuster` seemed to reveal nothing significant.
### Web Enumeration
Based on `Wappalyzer` the page is using PHP version 8.0.30 so I tried to find out if there were any known vulnerabilities for this version on the Internet, and the results suggests that 8.0.30 seems to be a secure version.

[insert image from Windows machine]

Take a look at the source code of the page, at the end there is a script about some sort of cheat code:

```html
<script> 
	const particlesContainer = document.getElementById('particles'); 
	for (let i = 0; i < 30; i++) { 
		const particle = document.createElement('div'); 
		particle.className = 'particle'; 
		particle.style.left = Math.random() * 100 + '%'; 
		particle.style.animationDelay = Math.random() * 15 + 's'; 
		particle.style.animationDuration = (Math.random() * 10 + 10) + 's'; 
		particlesContainer.appendChild(particle); 
	} 
	let konamiCode = []; 
	const konamiSequence = ['ArrowUp', 'ArrowUp', 'ArrowDown', 'ArrowDown', 'ArrowLeft', 'ArrowRight', 'ArrowLeft', 'ArrowRight', 'b', 'a']; 
	document.addEventListener('keydown', (e) => { 
		konamiCode.push(e.key); 
		konamiCode = konamiCode.slice(-10); 
		if (konamiCode.join(',') === konamiSequence.join(',')) { 
			const log = document.getElementById('combatLog'); 
			const entry = document.createElement('div'); 
			entry.className = 'log-entry'; 
			entry.style.color = '#ff0'; 
			entry.textContent = '> 🎮 CHEAT CODE ACTIVATED! But can you serialize your way to victory? 🎮'; 
			log.appendChild(entry); 
			log.scrollTop = log.scrollHeight; 
		} 
	}); 
	const monster = document.getElementById('monster'); 
	const fightButton = document.querySelector('.action-button'); 
	fightButton.addEventListener('click', (e) => { 
		monster.classList.add('shake'); 
		setTimeout(() => monster.classList.remove('shake'), 500); 
	}); 
</script>
```
The page it seems like the page will record the key press of the user and output the "Cheat Code Activated" string to the `combatLog`. Which is pretty weird considering there were no element in the HTML when I inspect the code.

So to test this out, I asked Gemini to generate a script to help me trigger the cheat code to see what happen.

```python

```

[insert the image of the result]

The only thing happen was just the small box at the end of the page appear saying the string. Other than that it did nothing, though I expected it to trigger some client-side errors or something more.

Next thing I did was pressing the `FIGHT MONSTER` button. The result was that the coins and the attack of user increased by 1 after the page reloaded.

Looking at the source code again, it seemed like whenever the button is pressed the page sends a request with parameter `action=fight`.

Firing up `BurpSuite`, and looking at the traffic. I take a look at the request to `/?action=fight`



---

## Foothold (User)

**Path:** <% tp.file.cursor(1) %>

### Step 1: Discovery

(What did you find?)

### Step 2: Exploitation

(The exact payload or exploit used).

> [!failure] 🐇 Rabbit Hole I spent time trying to brute force SSH.
> 
> - **Correction:** Always check for `id_rsa` keys in web directories first.
>     

---

## Privilege Escalation (Root)

**Current User:** `www-data`

### Enumeration

- **LinPeas Findings:** `Vulnerable Sudo version`
    

### Exploitation

Bash

```
# Commands to get root
```

---

## Loot & Flags

- [ ] **User Flag:** `hash_here`
    
- [ ] **Root Flag:** `hash_here`
    
- [ ] **Credentials:**
    
    - `user:password`
        

---

**References:** [Link](https://www.google.com/search?q=url)
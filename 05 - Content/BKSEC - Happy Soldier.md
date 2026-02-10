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

![[Pasted image 20260210041857.png]]

The request's cookie was attached with a flag? and a **base64** and **URL encoded** string. The decoding revealed that the encoded is the player's stats formatted in the **PHP serialization format**.

The response also have a similar string:
```php
// Request 
O:6:"Player":4:{
	s:6:"health";i:100;
	s:6:"attack";i:10;
	s:5:"coins";i:0;
	s:6:"weapon";s:12:"Wooden Sword";
}

// Response
O:6:"Player":4:{
	s:6:"health";i:100;
	s:6:"attack";i:11;
	s:5:"coins";i:10;
	s:6:"weapon";s:12:"Wooden Sword";}
```

It seemed like the web send to the endpoint the player's stats, the endpoint update the stats and send back to the client. It seems like some sort of game, with enough stats I might be able to defeat the monster. 

At this point, I was thinking about writing a script that helps me click the `FIGHT MONSTER` button indefinitely until I have enough stats. However, after clicking the button for a few more times, I got redirected to a forbidden page, this may be a rate limiting logic behind this. If I write a script that spam the web it might takes a long time to finish, so I decide to make it my last resort.

Looking a bit more, there seems to be nothing more so I decide to dig into modifying the base64 string.

---

## Exploit the Cookie

I looked for the information about the format (the server was using PHP version `7.4.33` based on the response) and tried modifying each field of the string with different payloads to look for an injection point.

### Test the fields
Sending an error cookie will lead to a rejection and return standard object:

```php
// Request 
O:7:"Player":4:{         // Wrong String length
	s:6:"health";i:100;
	s:6:"attack";i:10;
	s:5:"coins";i:0;
	s:6:"weapon";s:12:"Wooden Sword";
}

// Response
O:8:"stdClass":2:{
	s:5:"coins";i:10;
	s:6:"attack";i:1;}
```
Changing the name of the objects does nothing, apparently.
```php
// Request 
O:8:"okokokko":4:{         // Wrong some random name
	s:6:"health";i:100;
	s:6:"attack";i:10;
	s:5:"coins";i:0;
	s:6:"weapon";s:12:"Wooden Sword";
}

// Response
O:8:"okokokko":4{
	s:6:"health";i:100;
	s:6:"attack";i:10;
	s:5:"coins";i:0;
	s:6:"weapon";s:12:"Wooden Sword";
}
```
Even changing the type of the object also does not trigger any errors:
```php
// Request 
s:8:"okokokko";

// Response
s:8:"okokokko";
```
however the page does not display anything, so it seems like for the page to display anything, we can only keep the object as `Player` as it was.

The same pattern appeared when I tried to manipulate other fields. Integer fields like `health`can hold **string**, **double** and **boolean**, but it could not hold **object**; `attack` and `coins` had some math going in the background so they could not be injected with **string** or **boolean** (as they will be turned to integers afterward). `Weapon` was the most suspicious as the field accepted everything out of the four, when I inject a random object the field just accept it.

```php
// Request 
O:7:"Player":4:{         
	s:6:"health";i:100;
	s:6:"attack";i:10;
	s:5:"coins";i:0;
	O:2:"ok":0:{};       // Random Object
}

// Response
O:7:"Player":4:{         
	s:6:"health";i:100;
	s:6:"attack";i:10;
	s:5:"coins";i:0;
	O:2:"ok":0:{};
}
```

After consulting Gemini, it seemed like PHP often uses a method like `unserialize()` for deserialization and a method called `__to_string()` if they were to utilize the string representation of an object. So I need to find the object that allows serialization and it has to have a `__to_string()` method implemented.

### Inject Payloads
> [!failure] Payload 1: Infinite Stats.
> 
> - As the challenge worked similar to a game, I decide to try buff the stats of h.
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
---
tags:
  - 🚩
up:
  - "[[02 - Enumeration]]"
platform: BKSEC - Training
difficulty: Easy
creation_date: 2026-02-17
last_modified: 2026-02-17
---

# 🚩 [[BKSEC - Happy Soldier Revenge]]
**Primary:** [[01 - Web Security]]

**Secondary:** [[02 - Enumeration]]

**Related Tools:** [[(incomplete) BurpSuite]], [[(incomplete) Arjun]], [[(incomplete) Hashcat]]


## Executive Summary
* **IP:** `http://103.77.175.40:8142`
* **Key Technique:** Argument Fuzzing to reveal hidden argument, this leads to a local file content revelation, then we change the PHP object accordingly to satisfy the vulnerable requirement to get the flag.
* **Status:** `Complete`

---

## Reconnaissance
### Gobuster Scan
The result is the same as the previous version of this challenge: [[BKSEC - Happy Soldier]]

### Arjun Scan
The first scan without any argument actually revealed nothing.

(insert output of arjun first scan)

The second scan using the `common.txt` wordlist in `SecList` revealed the familiar `src` parameter.
(insert second scan with wordlist argument)

---
## Exploit The Exposed file
Based on the content of the exposed file, it seems like what we need to do is the same as in the previous version, that is changing the content of the `save_data` cookie so that it matches the win condition.

This is the function that defines the win condition
```php
public function __wakeup() {        $secret = file_get_contents('/secret.txt');  
        if($this->sig == md5($secret . $this->weapon))  
        {  
            if (stripos($this->weapon, "Golden") !== false) {  
                echo '<div class="flag-victory">  
                    <div class="victory-content">  
                        <div class="victory-icon">🏆</div>  
                        <div class="victory-title">VICTORY!</div>  
                        <div class="victory-subtitle">You have conquered the Serialized Demon!</div>  
                        <div class="flag-box">  
                            <div class="flag-label">YOUR FLAG:</div>  
                            <div class="flag-text">' . htmlspecialchars(file_get_contents('/flag.txt')) . '</div>  
                        </div>  
                    </div>  
                  </div>';  
            }  
        }  
    }
```


---

## Loot & Flags

- [ ] **User Flag:** `hash_here`
    
- [ ] **Root Flag:** `hash_here`
    
- [ ] **Credentials:**
    
    - `user:password`
        

---

**References:** [Link](https://www.google.com/search?q=url)
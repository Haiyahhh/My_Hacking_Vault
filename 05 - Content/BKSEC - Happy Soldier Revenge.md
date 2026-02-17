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
The result was the same as the previous version of this challenge: [[BKSEC - Happy Soldier]]

### Arjun Scan
The first scan without any argument actually revealed nothing.

(insert output of arjun first scan)

The second scan using the `common.txt` wordlist in `SecList` revealed the familiar `src` parameter.
(insert second scan with wordlist argument)

---
## Exploit The Exposed file
Based on the content of the exposed file, it seemed like what I needed to do was the same as in the previous version, that was changing the content of the `save_data` cookie so that it matches the win condition.

This is the format of the `Player` object:
```php
public $health;
public $attack;
public $coins;
public $weapon;
public $sig;
  
public function __construct($health, $attack, $coins, $weapon) {
    $this->health = $health;
    $this->attack = $attack;
    $this->coins = $coins;
    $this->weapon = $weapon;
    $secret = file_get_contents('/secret.txt');
    $this->sig = md5($secret . "Wooden Sword");  
}
```
Again, there were no constraint for the data type of the attributes. 

This was the function that defines the win condition
```php
public function __wakeup() {
    $secret = file_get_contents('/secret.txt');
    if($this->sig == md5($secret . $this->weapon)) {
        if (stripos($this->weapon, "Golden") !== false) {
            echo '<div class="flag-victory">
                <div class="victory-content">
                    <div class="victory-icon">🏆</div>
                    <div class="victory-title">VICTORY!</div>
                    <div class="victory-subtitle">You have conquered the Serialized Demon!</div>
                    <div class="flag-box">
                        <div class="flag-label">YOUR FLAG:</div>
                        <div class="flag-text">' . htmlspecialchars(file_get_contents('/flag.txt')) . '</div>
                    </div>
                </div>
                </div>';
        }
    }
}
```
There were two conditions:
*  The signature must match the concatenation between the `secret` and the name of the weapon.
*  The name of the weapon must contain the word "Golden"

**Guess the Secret**
I assumed that the secret may be able to be cracked using a rainbow table. Since I already got the hash for `Wooden Sword` by decode the `save data` cookie. I can just pass the hash into `hashcat` to see if I get anything.

(insert hashcat output)

The secret was not cracked. So we have to move to another method.

**Exploit the Loose Comparison**
The `if` condition `if($this->sig == md5($secret . $this->weapon))` uses a loose comparison between `$this->sig`. In PHP, the loose comparison == between **boolean** value `1` and a string always returns `TRUE`. Which means I can bypass this condition my modify the signature of the malicious `Player` object to hold the boolean value `1`.

```php
O:6:"Player":5 {
	s:6:"health";i:100;
	s:6:"attack";
	i:10;s:5:"coins";
	i:0;s:6:"weapon";
	s:12:"Golden Sword";
	s:3:"sig";b:1;
}
```

---

## Loot & Flags

**Flag:** BKSEC{c0ngratulat10ns_y0u_h4ve_m4st3red_th1s_g4me_d3a7b9}
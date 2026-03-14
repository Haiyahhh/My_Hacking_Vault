---
tags:
  - 🛠️
up:
  - "[[02 - Credential Access]]"
aliases: []
creation_date: 2026-02-17
last_modified: 2026-03-05
---
# 🛠️ [[Hashcat]]
**Primary:** [[01 - Web Security]], [[01 - Network Security]]

**Secondary:** [[02 - Credential Access]]

## Installation

```bash
sudo apt update
sudo apt install hashcat
````

## Usage
Used for cracking hashes, passwords using dictionary attacks

## Common Flags

|**Flag**|**Name**|**Description**|
|---|---|---|
|`-m`|Hash type|What kind of hash we're attacking|
|`-a`|Attack mode|The type of attack mode to use. More at [[Hashcat#Attack modes]]|
|`-o`|Output|Save the output to a file|
|`--status`|Status|Automatically update the screen with progress|
|`-r`|Rules|Applies a rule file (e.g. `best64.rule`) to mutate the wordlist|
|`--show`|Show|Displays the cracked password|
|`--i`|Increment|Used in Mask attack (`-a 3`) to try shorter passwords before longer ones|

###  Attack modes
**Straight Attack** (`-a 0`)
Read the a line from the provided wordlist and hashes it.

```bash
hashcat -a 0 -m 0 hash.txt /usr/share/wordlists/rockyou.txt
```

**Combination Attack** (`-a 1`)
Hashcat takes two wordlists and combines every word from the first with every word from the second.

```bash
hashcat -a 1 -m 1000 hashes.txt seasons.txt cities.txt
```

**Mask Attack** (`-a 3`)
Instead of a wordlist, you define a pattern (a "mask"). This is much faster than traditional brute force because you can skip combinations that don't make sense.
- **Placeholders:**
    
    - `?u` = Uppercase, `?l` = Lowercase, `?d` = Digit, `?s` = Special
        
- **Example Mask:** `?u?l?l?l?d?d` (e.g., `Pass12`)
    
- **Best for:** When you know the password policy (e.g., "Must be 8 characters, start with a capital").
```bash
hashcat -a 3 -m 1000 hashes.txt ?u?l?l?l?d?d
```

**Hybrid Attack** (`-a 6` and `-a 7`)
This combines a wordlist with a mask. It’s incredibly effective for catching users who just add a year or a symbol to the end of a normal word.

- **`-a 6` (Wordlist + Mask):** `rockyou.txt` + `?d?d?d` (e.g., `password123`)
    
- **`-a 7` (Mask + Wordlist):** `?s` + `rockyou.txt` (e.g., `!password`)
    
- **Best for:** Cracking "complex" passwords that follow predictable human behavior. Maybe something like a flag, or token handcrafted.

```bash
hashcat -a 6 -m 1000 hashes.txt rockyou.txt ?d?d?d
```

## Tips & Tricks

Used when not sure what is the type of the hash

```bash
hashcat hash.txt
```


## Related Usage
### CTFs
[[BKSEC - Happy Soldier Revenge]]

### Techniques

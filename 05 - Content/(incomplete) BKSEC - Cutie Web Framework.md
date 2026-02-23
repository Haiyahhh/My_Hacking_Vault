---
tags:
  - 🚩
up:
  - "[[02 - (incomplete) Data Exfiltration]]"
platform: BKSEC Training
difficulty: Easy
creation_date: 2026-02-14
last_modified: 2026-02-14
---

# 🚩 [[(incomplete) BKSEC - Cutie Web Framework]]
**Primary:** [[01 - Web Security]]

**Secondary:** [[02 - (incomplete) Data Exfiltration]]

**Related Techniques:** [[(incomplete) SQL Injection|SQLi]]

## Executive Summary
* **URL:** `http://103.77.175.40:8199`
* **Key Technique:** Based on the given white-box, find the SQLi vulnerability and retrieve the flag.
* **Status:** `Completed`

---

## First Look
### The white box
This is a white box challenge so I decide to take time and learn more about the given technology and some of the syntax of TypeScript.

The syntax is oddly similar to python in a sense. It's a rather short file so I managed to divide the content into different parts.

**Import**
```typescript
import { Elysia, t } from 'elysia'
import postgres from 'postgres'
// graphql
import { yoga } from '@elysiajs/graphql-yoga'
```

**Set up database**
```typescript
// reset database
const sql = postgres(process.env.DATABASE_URL || 'postgresql://ctf_user:ctf_password@localhost:5432/ctf_db')
await sql`DROP TABLE IF EXISTS users CASCADE`
await sql`DROP TABLE IF EXISTS secrets CASCADE`

// creates tables
await sql`
  CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username TEXT UNIQUE,
    password TEXT,
    role TEXT
  );
`

await sql`
  CREATE TABLE secrets (
    id SERIAL PRIMARY KEY,
    name TEXT,
    value TEXT
  );
`

// initialize data for user and admin
const adminPassword = `super_secret_password_${Math.random().toString(36).substring(7)}`
await sql`
  INSERT INTO users (username, password, role) VALUES
    ('admin', ${adminPassword}, 'admin'),
    ('guest', 'guest123', 'user');
`

// the flag is stored in a table named secrets
await sql`
  INSERT INTO secrets (name, value) VALUES
    ('flag', ${process.env.FLAG}),
    ('chatlgbt_api_key', ${process.env.CHATLGBT});
`
```

**Routes**
```typescript
// a session store session id string that is map to a dictionary for a pair of username and role
const sessions = new Map<string, { username: string; role: string }>()

const authPlugin = new Elysia({ name: 'auth' })
  // When a user connect to this, derive the username and role from the session id in the header and store it in the context for later use
  .derive(({ headers }) => {
    const sessionId = headers['x-session-id']
    const session = sessionId ? sessions.get(sessionId) : null
    return { session }
  })

  // For any route that is not /auth/login, check if the session exists in the context, if not return 401 Unauthorized
  .onBeforeHandle(({ session, path, set }) => {
    if (path === '/auth/login') return
    if (!session) {
      set.status = 401
      return { error: 'Unauthorized - Please login first' }
    }
  })

  // Define the login route, the user will send username and password in the body.
  // Check if the credentials are correct, if correct generate a random session id and store it in the session store
  // Return the session id to the user
  .post('/auth/login', async ({ body }) => {
    const { username, password } = body
    const users = await sql`
      SELECT * FROM users WHERE username = ${username} AND password = ${password}
    `
    if (users.length === 0) {
      return { error: 'Invalid credentials' }
    }
    const user = users[0]
    const sessionId = Math.random().toString(36).substring(7)
    sessions.set(sessionId, { username: user.username, role: user.role })
    return {
      success: true,
      sessionId,
      message: 'Login successful!'
    }
  }, {
    body: t.Object({
      username: t.String(),
      password: t.String()
    })
  })
```

```typescript
// admin routes
const adminPlugin = new Elysia({ prefix: '/admin' })
    // Inherits the authPlugin to protect all admin routes, only logged in users can access these routes
  .use(authPlugin)

    // A profile route that returns the username and role of the logged in user, it does not need any parameters
  .get('/profile', ({ session }) => {
    return {
      message: 'Admin profile',
      user: session?.username,
      role: session?.role
    }
  })

    // A secrets route that returns all secrets in the database, only users with admin role can access this route
  .get('/secrets', async ({ session }) => {
    if (session?.role !== 'admin') {
      return { error: 'Admin access required' }
    }
    const secrets = await sql`SELECT * FROM secrets`
    return { secrets }
  })
```

**Main app**
```typescript
// Main app
const app = new Elysia()
    // Inherits the adminPlugin
  .use(adminPlugin)

    // A public route that anyone can access, it just returns a welcome message
  .get('/', () => ({
    message: 'Welcome to SecureAPI v1.0'
  }))

    // A search route that allows users to search for other users by username
  // It accepts a query parameter "username" and an optional "orderBy" parameter to sort the results by username or role
  .use(
    yoga({
    // Type defintion for the GraphQL API
    // Query type: has a search field that takes username and orderBy as arguments and return a list of User objects
    // User type: has username and role fields
      typeDefs: `
        type Query {
          search(username: String!, orderBy: String): [User]
        }
        type User {
          username: String
          role: String
        }
      `,
      resolvers: {
        Query: {
          search: async (_: any, { username, orderBy }: { username: string; orderBy?: string }) => {
            if (!username) {
              throw new Error('Please provide username parameter')
            }
            try {
              let results
              if (orderBy) {
                results = await sql.unsafe(`SELECT username, role FROM users WHERE username LIKE '%${username}%' ORDER BY ${orderBy}`)
              } else {
                results = await sql.unsafe(`SELECT username, role FROM users WHERE username LIKE '%${username}%'`)

              }
              return results
            } catch (e: any) {
              throw new Error(`Search failed: ${e.message}`)
            }
          }
        }
      }
    })
  )
  .listen(3000)
```
### Web Enumeration

- **Technologies:** (Apache, PHP, etc.)
    
- **Fuzzing Results:**
    
    - `/admin` (403)
        
    - `/images` (200)
        

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
# Authentication Systems: A Complete Mental Model and Implementation Guide

> *A mentor-style deep dive — from first principles to production-ready patterns.*

---

## Table of Contents

1. [First Principles: What Is Authentication?](#1-first-principles)
2. [Architecture of a Login System](#2-architecture)
3. [Sessions, Tokens, and Maintaining State](#3-sessions-and-tokens)
4. [Passwords and Secure Storage](#4-passwords)
5. [The Database: Users Table and Schema Design](#5-database)
6. [Core Flows: Sign Up, Verify, Login, Logout](#6-core-flows)
7. [Login with Google: OAuth 2.0 and OpenID Connect](#7-oauth)
8. [Email Verification and Password Reset](#8-email-verification)
9. [Security Risks and Common Mistakes](#9-security)
10. [Real-World Stacks, Libraries, and Tradeoffs](#10-real-world)
11. [Learning Roadmap](#11-roadmap)

---

## 1. First Principles: What Is Authentication?

### 1.1 The Core Problem

Before writing a single line of code, let's understand *why* authentication exists at all.

Imagine you walk into a bank. The teller asks: **"Who are you, and can I trust that you are who you say you are?"** That's authentication in one sentence.

On the web, the problem is harder. HTTP — the protocol of the web — is **stateless**. Every single request a browser makes to a server is, from the server's perspective, a fresh conversation with a stranger. The server has no built-in memory of you.

If you log into your email and then click "Inbox," the server gets a new HTTP request with no inherent memory of your login. Without authentication mechanisms, every click would require you to prove your identity again.

**Authentication solves this problem:** it gives the server a reliable, secure way to know *who* is making a request, across multiple stateless HTTP requests.

---

### 1.2 Authentication vs. Authorization: A Critical Distinction

These two words are often confused. They answer different questions:

| Term | Question it answers | Example |
|---|---|---|
| **Authentication** | *Who are you?* | "You are Alice." |
| **Authorization** | *What are you allowed to do?* | "Alice can read posts but not delete them." |

Authentication must happen first. You can't decide what someone is allowed to do until you know who they are. But knowing who they are doesn't automatically tell you what they're allowed to do — that's a separate concern.

**Mental model:** Think of a concert venue.
- Authentication = the bouncer checks your ID and ticket. "Yes, you are John Smith, and you have a valid ticket."
- Authorization = your ticket says "Floor Section." You're not allowed into the VIP area. The VIP bouncer turns you away.

In code, you will often see both lumped together loosely as "auth," but a well-designed system keeps them conceptually and often architecturally separate.

---

### 1.3 Identity, Trust, and the Stateless Web

Let's build a precise mental model.

**Identity** is a claim: "I am Alice." Any client can make this claim. The server cannot accept it blindly.

**Trust** means the server has *verified* that claim through some evidence — a password only Alice knows, a cryptographic key, a token issued earlier, or a trusted third party (like Google) vouching for Alice.

**State** is the server's memory of a verified identity across requests. Since HTTP is stateless, state must be *created* artificially, either:
- On the **server** (a session record in a database), or
- On the **client** (a signed, self-contained token that travels with each request).

The entire field of authentication is essentially: *how do we create, maintain, and revoke trust over a stateless protocol, securely?*

---

### 1.4 Conceptual Model of a Login System (No Tech Yet)

Here's the high-level story of what happens when a user logs in, using no specific technology:

1. **The user presents credentials** — typically something they *know* (a password), *have* (a device), or *are* (a fingerprint).
2. **The server verifies those credentials** against a trusted record (e.g., the password stored in the database).
3. **The server issues a "proof of identity"** — a ticket, badge, or token that the client can present on future requests.
4. **The client stores and presents that proof** on every subsequent request.
5. **The server validates the proof** on each request to confirm the user's identity without re-asking for their password.
6. **Eventually, the proof expires or is revoked** — the user is "logged out."

This pattern — challenge → verify → issue proof → present proof → validate proof — is the heartbeat of every authentication system you will ever encounter.

> 💡 **Thought Exercise 1:** Before continuing, try to describe in your own words: what information does a "proof of identity" (step 3) need to contain, and what properties should it have to be trustworthy?

---

## 2. Architecture of a Login System

### 2.1 The Main Components

A typical web authentication system has four actors:

```
┌──────────────┐      ┌──────────────┐      ┌──────────────┐
│   Browser /  │      │   Backend /  │      │  Database    │
│  Frontend    │◄────►│    Server    │◄────►│  (Postgres,  │
│  (React, etc)│      │  (Node, etc) │      │   MySQL...)  │
└──────────────┘      └──────────────┘      └──────────────┘
                              │
                              │ (optional)
                              ▼
                      ┌──────────────┐
                      │ OAuth / IDP  │
                      │  (Google,    │
                      │   GitHub...) │
                      └──────────────┘
```

**Browser / Frontend:** Collects user input (email, password), sends HTTP requests, stores credentials or tokens (in cookies, memory, or localStorage), and reads/displays data returned by the backend.

**Backend / Server:** The brain of the operation. Validates input, compares passwords, issues tokens or creates sessions, queries the database, and enforces business logic.

**Database:** The persistent source of truth. Stores user records (hashed passwords, email addresses, verification status, etc.), session records, and any other auth-related data.

**OAuth / Identity Provider (IDP):** An external service (Google, GitHub, Apple) that authenticates the user on your behalf. You trust them; your backend verifies their signed statements about the user's identity.

---

### 2.2 Data Flow: Email/Password Login

Let's trace exactly what moves where during a basic login.

```
Browser                  Backend                   Database
  │                         │                          │
  │  POST /login            │                          │
  │  { email, password }    │                          │
  │────────────────────────►│                          │
  │                         │  SELECT * FROM users     │
  │                         │  WHERE email = ?         │
  │                         │─────────────────────────►│
  │                         │                          │
  │                         │  { id, email,            │
  │                         │    password_hash, ... }  │
  │                         │◄─────────────────────────│
  │                         │                          │
  │                         │  [compare password       │
  │                         │   to stored hash]        │
  │                         │                          │
  │                         │  [create session or      │
  │                         │   generate JWT]          │
  │                         │                          │
  │  200 OK                 │                          │
  │  Set-Cookie: sid=...    │                          │
  │  (or body: {token: ...})│                          │
  │◄────────────────────────│                          │
  │                         │                          │
  │  [subsequent request]   │                          │
  │  GET /dashboard         │                          │
  │  Cookie: sid=...        │                          │
  │────────────────────────►│                          │
  │                         │  [validate session/token]│
  │                         │                          │
  │  200 OK + user data     │                          │
  │◄────────────────────────│                          │
```

Key observations:
- The password is sent over the network exactly once, on login. After that, only the session ID or token travels.
- The server **never** sends the password hash back to the browser.
- Each subsequent request proves identity by presenting the session ID or token — not the password.

---

### 2.3 How the Browser Stores and Sends Auth Data

The browser has several storage mechanisms, each with tradeoffs:

**Cookies:**
- Automatically attached to every HTTP request to the same domain.
- Can be scoped to paths, domains, and protocols.
- Can be configured server-side to be unexposable to JavaScript (HttpOnly flag).
- Best for session IDs in traditional web apps.

**localStorage / sessionStorage:**
- Accessible by JavaScript, which makes it vulnerable to XSS attacks.
- Not automatically sent with HTTP requests; the app must manually add it to headers.
- Often used for JWTs in SPA (Single Page Application) patterns, but carries security risk.

**Memory (in-app state):**
- Only lives as long as the page/tab. Lost on refresh.
- Safest against persistent XSS, but impractical for most real apps.

**HTTP Headers:**
- Tokens (JWTs) sent manually via the `Authorization: Bearer <token>` header.
- The application JavaScript controls when and where to send them.

The storage choice is not cosmetic — it has direct security implications we'll cover in Section 9.

---

## 3. Sessions, Tokens, and Maintaining State

### 3.1 Sessions: The Server-Side Memory Model

**Conceptual model:** When you log in, the server creates a small record in its own memory (or database) that says: "Session abc123 belongs to user Alice, created at 10am, expires at 11am." It then gives you the label `abc123` to hold.

On every future request, you show the label. The server looks it up, finds "this is Alice," and acts accordingly.

This is the **session model**: the server holds the truth, the client holds only a reference (the session ID).

```
Server's Session Store:
┌───────────────────────────────────────────────────┐
│ session_id: "abc123"                              │
│ user_id: 42                                       │
│ email: "alice@example.com"                        │
│ created_at: 2024-01-01T10:00:00Z                  │
│ expires_at: 2024-01-01T11:00:00Z                  │
└───────────────────────────────────────────────────┘
```

The client stores `abc123` in a cookie. On each request, the browser sends `Cookie: session_id=abc123`. The server reads it, looks up the session, confirms it's valid and not expired, and identifies the user.

**Session stores** can be:
- In memory (fast, but lost if server restarts; bad for multiple servers)
- In a database table (persistent, scales across servers)
- In Redis or Memcached (fast, persistent, great for production)

---

### 3.2 Cookie Security Flags: Why They Exist

When the server sets a session cookie, it can (and should) include security flags:

**HttpOnly:** The cookie cannot be read by JavaScript. This means if an attacker injects malicious JS into your page (XSS), they cannot steal the session ID. This is a critical protection.

**Secure:** The cookie is only sent over HTTPS, never plain HTTP. Without this, an attacker on the same network can intercept the cookie in plain text.

**SameSite:** Controls when the browser sends the cookie with cross-site requests.
- `Strict`: Cookie sent only when navigating directly to your site.
- `Lax`: Cookie sent on top-level navigations (clicking a link) but not on cross-site sub-requests.
- `None`: Always sent, but requires `Secure`. This is needed for embedded iframes or third-party integrations.

`SameSite=Lax` is the modern default and protects against most CSRF attacks (more on this in Section 9).

---

### 3.3 JSON Web Tokens (JWTs): The Client-Side Truth Model

**Conceptual model:** Instead of storing session data on the server and giving the client a reference, what if you encoded all the necessary information *inside* the token itself, digitally signed it so it can't be tampered with, and let the client carry the whole thing?

That's a JWT. It is a self-contained, signed package of claims.

**Structure of a JWT:**

A JWT is three Base64URL-encoded strings separated by dots:

```
header.payload.signature
```

**Header** — metadata about the token:
```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

**Payload** — the claims (data about the user and the token):
```json
{
  "sub": "42",
  "email": "alice@example.com",
  "role": "user",
  "iat": 1704067200,
  "exp": 1704070800
}
```

Common standard claims:
- `sub` — subject (usually user ID)
- `iat` — issued at (Unix timestamp)
- `exp` — expiration time
- `iss` — issuer (who created this token)
- `aud` — audience (who this token is for)

**Signature** — a cryptographic signature of the header + payload, using a secret key known only to the server:

```
HMACSHA256(
  base64url(header) + "." + base64url(payload),
  secret_key
)
```

**Why the signature matters:** Anyone can read a JWT's payload — it is Base64-encoded, not encrypted. But they cannot *modify* it without invalidating the signature. If an attacker changes `"role": "user"` to `"role": "admin"`, the signature verification will fail, and the server will reject the token.

**Important:** JWTs are **signed**, not encrypted by default. Never put sensitive data (SSNs, passwords, credit cards) in a JWT payload unless you use JWE (JSON Web Encryption).

---

### 3.4 How the Backend Validates a JWT

```
Client sends:  Authorization: Bearer eyJhbGc...

Backend:
  1. Split token into header, payload, signature
  2. Re-compute signature using the header + payload + secret key
  3. Compare computed signature to received signature
     → If they don't match: reject (tampered token)
  4. Check the "exp" claim: is it in the future?
     → If expired: reject
  5. Check "iss" and "aud" if applicable
  6. Decode payload and extract user identity
  7. Proceed with the request
```

No database lookup needed to verify a JWT — the server can validate it purely with the secret key and cryptography. This is the key performance advantage of JWTs.

---

### 3.5 The Three Token Types

**Access Token:**
- Short-lived (minutes to hours).
- Presented with every API request to prove identity.
- Usually a JWT, but can be opaque.
- If stolen, it expires quickly, limiting damage.

**ID Token:**
- Comes from OAuth/OIDC flows (like Login with Google).
- Contains information *about* the user (name, email, profile picture).
- Meant to be read by the client (your app), not sent to APIs.
- Is a JWT with standardized claims.

**Refresh Token:**
- Long-lived (days to months).
- Used only to obtain new access tokens when the current one expires.
- Should be stored securely, never exposed to the browser's JS.
- When a new access token is needed, the client sends the refresh token to a specific endpoint, gets a new access token back.
- Can be revoked server-side, effectively logging the user out even though the access token might still technically be valid.

```
Access Token lifecycle:
  [Login] ──► [Access Token (1h)] ──► [Expires]
                      │                    │
                      │                    ▼
                      │           [Refresh Token] ──► [New Access Token (1h)]
                      │
                      ▼
              [Used on every API call]
```

---

### 3.6 Sessions vs JWTs: Tradeoffs

| Concern | Sessions (Server-side) | JWT (Stateless) |
|---|---|---|
| **Server storage** | Requires session store (Redis, DB) | None needed |
| **Scalability** | Needs shared session store across servers | Works natively across multiple servers |
| **Revocation** | Easy — delete the session record | Hard — token is valid until it expires |
| **Latency** | DB/cache lookup on every request | Crypto verification only (fast) |
| **Payload size** | Cookie holds just an ID (tiny) | Token grows with claims (larger headers) |
| **Best for** | Traditional web apps, when revocation matters | APIs, SPAs, microservices, mobile apps |

**Practical guidance:**
- For most web apps where you control both frontend and backend: **sessions with HttpOnly cookies** are simpler, safer, and easier to reason about. Revocation is trivial.
- For APIs consumed by mobile apps or third-party clients: **JWTs** are more practical since they don't require a shared session store.
- For large distributed systems: JWTs reduce inter-service auth calls, at the cost of immediate revocation.

> 💡 **Thought Exercise 2:** If a user changes their password, how would you immediately invalidate their session in the sessions model? How would you do the same with JWTs? Which is easier, and why?

---

## 4. Passwords and Secure Storage

### 4.1 Why Plain Text Storage Is Catastrophic

Let's say you store passwords like this in your database:

```
email              | password
-------------------|-----------
alice@example.com  | sunflower99
bob@example.com    | ilovemydoG!
```

Now your database is breached. Every single user's password is immediately visible. Worse: most people reuse passwords. An attacker now has credentials to try on Gmail, banking, Amazon, and everywhere else your users have accounts.

This has happened at real companies. It destroys trust, causes massive harm, and in many jurisdictions triggers legal liability.

**The rule is absolute: never store passwords in plain text. Ever.**

---

### 4.2 Hashing: The Core Concept

A **hash function** takes any input and produces a fixed-length output. Critically, it is a **one-way function**: given the hash, you cannot recover the original input.

```
"sunflower99" ──► SHA-256 ──► "7a3f8e2b1c..."

"7a3f8e2b1c..." ──► ??? ──► You cannot reverse this
```

So instead of storing the password, you store the hash. When a user logs in:
1. Take their submitted password.
2. Hash it the same way.
3. Compare the resulting hash to the stored hash.
4. If they match, the password is correct.

You never need the original password. The attacker who steals the database gets only hashes.

---

### 4.3 Why Fast Hashes (Like SHA-256) Are Not Enough

Here's a problem: SHA-256 is designed to be *fast*. A modern GPU can compute billions of SHA-256 hashes per second.

This means an attacker with your database of SHA-256 hashed passwords can run a "brute force" or "dictionary" attack: try millions or billions of common passwords, hash each one, and see if any match. In practice, most user passwords would be cracked in hours or days.

**Password hashing needs to be slow by design.** The slower the hash, the harder the attack — while being fast enough that legitimate users don't notice the delay (typically 100–300ms per hash is acceptable).

---

### 4.4 Salting: Defeating Pre-Computation Attacks

Even with a slow hash, attackers use **rainbow tables**: precomputed databases of `(password → hash)` pairs. If your hash appears in their table, they find the password instantly.

A **salt** defeats this. A salt is a random, unique string added to each password before hashing:

```
password: "sunflower99"
salt:     "kX9pQ3r7..."   (random, unique per user)

stored hash = hash("sunflower99" + "kX9pQ3r7...")
```

Since every user has a different salt, even if two users have the same password, their hashes differ. Rainbow tables are useless because they'd have to be recomputed for every unique salt — computationally infeasible.

**The salt is stored alongside the hash** (not secret — that's fine). Its purpose is uniqueness, not secrecy:

```
password_hash: "$2b$12$kX9pQ3r7....<rest of bcrypt output>"
```

Modern password hashing algorithms encode the salt directly in the output string, so you don't manage it separately.

---

### 4.5 Modern Password Hashing: bcrypt and Argon2

**bcrypt** has been the standard for decades. It incorporates:
- A built-in salt (auto-generated per hash).
- A **cost factor** (also called work factor), typically 10–12: the number of rounds doubles with each increment, so cost 12 is 4× slower than cost 10.
- Slow by design — typically 100–300ms at cost 12 on modern hardware.

**Argon2** is the modern winner of the Password Hashing Competition (2015). It adds memory hardness: it requires a configurable amount of RAM to compute, making GPU/ASIC attacks much harder. There are three variants; **Argon2id** is generally recommended.

**The complete signup password flow (conceptually):**

```
User submits: "sunflower99"
               │
               ▼
  [Backend receives password over HTTPS]
               │
               ▼
  [bcrypt/argon2 generates random salt]
               │
               ▼
  [hash(password + salt, cost_factor)]
               │
               ▼
  [Store ONLY the resulting hash string in DB]
  [Discard the original password immediately]
```

**The complete login password validation flow:**

```
User submits: "sunflower99"
               │
               ▼
  [Backend receives password over HTTPS]
               │
               ▼
  [Fetch stored hash from DB: "$2b$12$kX9..."]
               │
               ▼
  [bcrypt/argon2 extracts salt from stored hash]
               │
               ▼
  [hash(submitted_password + extracted_salt, same_cost)]
               │
               ▼
  [Constant-time comparison of new hash vs stored hash]
               │
          ┌────┴────┐
          │         │
        Match    No Match
          │         │
        Login    Reject
        allowed
```

**Constant-time comparison** is important: a naive `==` comparison might short-circuit early if the first bytes differ, leaking information about how close the guess was. Cryptographic libraries provide constant-time compare functions to avoid this timing side-channel.

> 💡 **Thought Exercise 3:** Why is it important that the salt is unique *per user* rather than one salt shared across all users? What attack does the per-user salt specifically prevent?

---

## 5. The Database: Users Table and Schema Design

### 5.1 How Databases Serve Authentication

The database is the persistent ground truth of your auth system. It answers questions like:
- Does this email exist?
- What is the stored password hash for this user?
- Has this user verified their email?
- Is this reset token valid and unexpired?

Authentication puts specific demands on your schema: it must be secure, normalized enough to avoid redundancy, and fast enough to look up users quickly (indexes on `email` are essential).

---

### 5.2 The Core Users Table

```sql
CREATE TABLE users (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email           VARCHAR(255) NOT NULL UNIQUE,
  password_hash   VARCHAR(255),          -- NULL for OAuth-only users
  email_verified  BOOLEAN NOT NULL DEFAULT FALSE,
  created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),

  -- Email verification
  verification_token        VARCHAR(255),
  verification_token_expiry TIMESTAMPTZ,

  -- Password reset
  reset_token        VARCHAR(255),
  reset_token_expiry TIMESTAMPTZ,

  -- Metadata
  last_login_at   TIMESTAMPTZ,
  locked_until    TIMESTAMPTZ,        -- For brute force protection
  failed_attempts INTEGER DEFAULT 0
);

CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_verification_token ON users(verification_token);
CREATE INDEX idx_users_reset_token ON users(reset_token);
```

**Field-by-field explanation:**

`id` — A UUID rather than a sequential integer. Why? Sequential IDs leak information (user 42 means there are at least 42 users; users joined before you can be guessed). UUIDs are random and reveal nothing.

`email` — Unique, indexed. The primary identifier for most login systems.

`password_hash` — The bcrypt/argon2 output. Nullable because OAuth users may have no password.

`email_verified` — Users must verify they own the email before logging in (prevents using others' emails).

`verification_token` — A secure random token emailed to the user to verify their address.

`verification_token_expiry` — Tokens must expire (typically 24 hours). Unexpired stolen tokens are a risk, but expiry limits the window.

`reset_token` / `reset_token_expiry` — Same pattern for password reset. Must expire quickly (typically 1 hour).

`last_login_at` — Useful for detecting unusual behavior and showing users their last login.

`locked_until` / `failed_attempts` — Brute force protection. After N failed attempts, lock the account for a period.

---

### 5.3 OAuth Accounts Table (for "Login with Google")

Users who sign in via Google or GitHub don't have a local password. You need to track their provider identity:

```sql
CREATE TABLE oauth_accounts (
  id           UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id      UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  provider     VARCHAR(50) NOT NULL,   -- 'google', 'github', 'apple', etc.
  provider_id  VARCHAR(255) NOT NULL,  -- The user's ID at that provider
  access_token TEXT,                   -- Optional: store if calling provider APIs
  refresh_token TEXT,                  -- Optional: for refreshing provider tokens
  expires_at   TIMESTAMPTZ,
  created_at   TIMESTAMPTZ NOT NULL DEFAULT NOW(),

  UNIQUE(provider, provider_id)        -- One account per provider per person
);
```

This design allows a single user to have multiple login methods — local email/password AND Google AND GitHub — all linked to one `users` row.

---

### 5.4 Sessions Table (for Server-Side Sessions)

```sql
CREATE TABLE sessions (
  id         UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id    UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  session_id VARCHAR(255) NOT NULL UNIQUE,  -- The value stored in the cookie
  ip_address INET,
  user_agent TEXT,
  expires_at TIMESTAMPTZ NOT NULL,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_sessions_session_id ON sessions(session_id);
CREATE INDEX idx_sessions_user_id ON sessions(user_id);
```

When a user logs in, a new row is inserted. When they log out, the row is deleted. Expired rows should be cleaned up periodically.

> 💡 **Thought Exercise 4:** Sketch your own users table for a hypothetical app. What additional fields might you add for a SaaS product (subscription tier, organization ID)? How would you separate "authentication" fields from "profile" fields?

---

## 6. Core Flows: Sign Up, Verify, Login, Logout

### 6.1 Sign Up Flow

**Conceptually:** The user claims an identity. We verify the claim is valid (password strength, email format), store a hashed password, and begin the email verification process.

**Step-by-step sequence:**

```
Browser              Backend              Database         Mail Provider
  │                     │                    │                  │
  │  POST /signup       │                    │                  │
  │  {email, password}  │                    │                  │
  │────────────────────►│                    │                  │
  │                     │ [validate inputs]  │                  │
  │                     │ [check email       │                  │
  │                     │  not taken]        │                  │
  │                     │───────────────────►│                  │
  │                     │◄───────────────────│                  │
  │                     │ [hash password]    │                  │
  │                     │ [generate          │                  │
  │                     │  verify token]     │                  │
  │                     │ INSERT user        │                  │
  │                     │───────────────────►│                  │
  │                     │                    │                  │
  │                     │ [send verify email]│                  │
  │                     │──────────────────────────────────────►│
  │                     │                    │                  │
  │  201 Created        │                    │                  │
  │  "Check your email" │                    │                  │
  │◄────────────────────│                    │                  │
```

**Backend validation checklist during signup:**
- Email format is valid.
- Email is not already registered (but be careful how you report this — see Section 9 on user enumeration).
- Password meets minimum complexity requirements.
- (Optional) Check against breached password databases (e.g., HaveIBeenPwned API).
- Hash the password immediately upon receipt.
- Generate a cryptographically secure verification token (128+ bits of entropy).
- Store user with `email_verified = false`.
- Send verification email containing a link like `https://yourapp.com/verify?token=<token>`.

---

### 6.2 Email Verification Flow

**Conceptually:** We need to confirm the user actually controls the email address they signed up with. We use a shared secret (the token) that we sent to that inbox as proof.

```
Browser              Backend              Database
  │                     │                    │
  │ GET /verify         │                    │
  │ ?token=abc123       │                    │
  │────────────────────►│                    │
  │                     │ SELECT user WHERE  │
  │                     │ verify_token=abc123│
  │                     │───────────────────►│
  │                     │◄───────────────────│
  │                     │ [check expiry]     │
  │                     │ [update user:]     │
  │                     │  email_verified=T  │
  │                     │  token = NULL      │
  │                     │───────────────────►│
  │                     │                    │
  │  Redirect to login  │                    │
  │◄────────────────────│                    │
```

After verifying, clear the token from the database — it's a one-time-use credential. Never leave consumed tokens sitting in the database.

---

### 6.3 Login Flow (Email/Password)

**Conceptually:** The user presents credentials. We find their record, verify the password, confirm their email is verified, then issue a session or token.

```
Browser              Backend              Database
  │                     │                    │
  │  POST /login        │                    │
  │  {email, password}  │                    │
  │────────────────────►│                    │
  │                     │ SELECT user WHERE  │
  │                     │ email = ?          │
  │                     │───────────────────►│
  │                     │◄───────────────────│
  │                     │ [if user not found:│
  │                     │  fake delay +      │
  │                     │  return 401]       │
  │                     │ [verify password   │
  │                     │  hash]             │
  │                     │ [check email_      │
  │                     │  verified = true]  │
  │                     │ [check not locked] │
  │                     │ [create session or │
  │                     │  generate JWT]     │
  │                     │ INSERT session     │
  │                     │───────────────────►│
  │                     │                    │
  │  200 OK             │                    │
  │  Set-Cookie:        │                    │
  │  session_id=xyz;    │                    │
  │  HttpOnly; Secure;  │                    │
  │  SameSite=Lax       │                    │
  │◄────────────────────│                    │
```

**"Remember Me" behavior:** Without "remember me," the session cookie typically has no explicit expiry and is deleted when the browser closes (a "session cookie"). With "remember me," you set a future `Expires` date on the cookie (e.g., 30 days), and extend the session's expiry in the database accordingly.

---

### 6.4 Logout Flow

**Sessions:**
```
Browser              Backend              Database
  │                     │                    │
  │  POST /logout       │                    │
  │  Cookie: sid=xyz    │                    │
  │────────────────────►│                    │
  │                     │ DELETE session     │
  │                     │ WHERE id = xyz     │
  │                     │───────────────────►│
  │                     │                    │
  │  200 OK             │                    │
  │  Set-Cookie:        │                    │
  │  session_id=;       │                    │
  │  Expires=past       │                    │
  │◄────────────────────│                    │
```

Two actions happen: (1) delete the server-side session record, (2) clear the cookie on the client. Both steps are necessary. Clearing only the cookie means the session record lingers in the database; deleting only the server record means the cookie persists until the browser expires it.

**JWT Logout:** This is genuinely harder. JWTs are stateless — the server has no record to delete. Options:

1. **Short expiration + refresh tokens:** Accept that the access token is valid until it expires (e.g., 15 minutes). On logout, delete the refresh token. This means logout isn't instant but minimizes the risk window.
2. **Token blocklist:** Maintain a server-side list of revoked JWTs until they would naturally expire. This reintroduces server state, partially defeating the purpose of JWTs.
3. **Rotating tokens:** Issue new short-lived tokens frequently and validate refresh tokens server-side — effectively a hybrid session/JWT approach.

For most applications, option 1 is the pragmatic choice with short-enough access token lifetimes.

---

## 7. Login with Google: OAuth 2.0 and OpenID Connect

### 7.1 What Is OAuth 2.0?

OAuth 2.0 is an **authorization** framework that lets a user grant one application (your app) limited access to their resources at another service (Google, GitHub), without sharing their password.

**The four roles in OAuth:**

- **Resource Owner:** The user. Owns the data (Google account).
- **Client:** Your application. Wants access to the user's data.
- **Authorization Server:** Google's OAuth server. Issues access tokens after user consent.
- **Resource Server:** Google's API (Gmail, Google Profile, etc.). Holds the protected resources.

---

### 7.2 What OpenID Connect (OIDC) Adds

OAuth 2.0 is about *authorization* (accessing resources). OpenID Connect is a thin layer on top of OAuth 2.0 that adds *authentication* (proving identity).

OIDC introduces the **ID token** — a JWT issued by the authorization server that contains verified claims about the user (name, email, profile photo URL, subject ID). Your app uses the ID token to know who the user is, without making a separate API call to `GET /userinfo`.

For "Login with Google," you almost always want OIDC, not bare OAuth.

---

### 7.3 Step-by-Step: "Login with Google" Flow

```
Browser          Your Backend       Google OAuth       Database
  │                   │                  │                 │
  │ Click             │                  │                 │
  │ "Continue         │                  │                 │
  │ with Google"      │                  │                 │
  │──────────────────►│                  │                 │
  │                   │ [generate state  │                 │
  │                   │  + nonce,        │                 │
  │                   │  store in        │                 │
  │                   │  session/cookie] │                 │
  │                   │                  │                 │
  │ Redirect to Google│                  │                 │
  │ ?response_type=   │                  │                 │
  │   code            │                  │                 │
  │ &client_id=...    │                  │                 │
  │ &redirect_uri=... │                  │                 │
  │ &scope=openid     │                  │                 │
  │   email profile   │                  │                 │
  │ &state=<random>   │                  │                 │
  │◄──────────────────│                  │                 │
  │                   │                  │                 │
  │─────────────────────────────────────►│                 │
  │                   │                  │                 │
  │  [Google consent  │                  │                 │
  │   screen shown]   │                  │                 │
  │  [User approves]  │                  │                 │
  │                   │                  │                 │
  │ Redirect back:    │                  │                 │
  │ /callback         │                  │                 │
  │ ?code=AUTH_CODE   │                  │                 │
  │ &state=<same>     │                  │                 │
  │──────────────────►│                  │                 │
  │                   │ [verify state    │                 │
  │                   │  matches stored] │                 │
  │                   │                  │                 │
  │                   │ POST /token      │                 │
  │                   │ {code, client_id,│                 │
  │                   │  client_secret,  │                 │
  │                   │  redirect_uri}   │                 │
  │                   │─────────────────►│                 │
  │                   │                  │                 │
  │                   │ {access_token,   │                 │
  │                   │  id_token,       │                 │
  │                   │  refresh_token}  │                 │
  │                   │◄─────────────────│                 │
  │                   │                  │                 │
  │                   │ [validate        │                 │
  │                   │  id_token sig]   │                 │
  │                   │ [extract email,  │                 │
  │                   │  google_user_id] │                 │
  │                   │                  │                 │
  │                   │ [find/create     │                 │
  │                   │  user in DB]     │                 │
  │                   │─────────────────────────────────► │
  │                   │◄───────────────────────────────── │
  │                   │                  │                 │
  │                   │ [create local    │                 │
  │                   │  session]        │                 │
  │                   │                  │                 │
  │  Redirect to app  │                  │                 │
  │  Set-Cookie: sid  │                  │                 │
  │◄──────────────────│                  │                 │
```

**Critical details explained:**

**Authorization Code:** A short-lived, single-use code. It proves Google authorized the request, but it alone is useless without the `client_secret`. This is why the browser gets the code but the *backend* exchanges it for tokens — the `client_secret` never leaves the server.

**State parameter:** A random value you generate, store server-side, and send to Google. When Google redirects back, you verify the returned state matches. This prevents CSRF attacks on the OAuth flow (an attacker tricks your backend into processing a code from a different user's session).

**Nonce:** An optional random value embedded in the token request. When the ID token is returned, it must contain the same nonce. Prevents replay attacks.

**ID token validation:** Your backend must verify the ID token's signature using Google's public keys (fetched from Google's JWKS endpoint), check the `iss` (issuer) is Google, check the `aud` (audience) matches your `client_id`, and check the `exp` hasn't passed.

**Find or create user logic:**
```
1. Look up oauth_accounts WHERE provider='google' AND provider_id=<google_user_id>
2. If found: get the linked user_id, create a session → done
3. If not found: check if email already exists in users table
   - If email exists: link the Google account to that user (ask user to confirm)
   - If email doesn't exist: create a new user, create oauth_account record, create session
```

> 💡 **Thought Exercise 5:** Why is the authorization code exchange (step where your backend calls Google with `client_secret`) done server-to-server, rather than having the browser do it? What would an attacker be able to do if the browser exchanged the code directly?

---

## 8. Email Verification and Password Reset

### 8.1 Email Verification in Depth

**Why it matters:**
- Prevents users from signing up with someone else's email (malicious or accidental).
- Ensures you have a working email address for password recovery.
- Limits spam account creation (creates friction).
- Some regulatory contexts require verified contact info.

**Token generation:** Use a cryptographically secure random function to generate at least 128 bits of entropy. In Node.js:

```javascript
// Conceptually: generate 32 random bytes, encode as hex
const token = crypto.randomBytes(32).toString('hex');
// Result: 64-char hex string, practically unguessable
```

Store this token in the database alongside the user, with an expiry (24 hours is standard).

**Security consideration:** Don't use predictable values as tokens (user ID, timestamp, sequential numbers). An attacker who can guess the token could verify any email address.

**Token as a URL:**

```
https://yourapp.com/verify-email?token=a3f9b2e1c8d7...
```

When the user clicks, your backend:
1. Looks up the token in the database.
2. Checks it hasn't expired.
3. Sets `email_verified = true`.
4. Clears the token and expiry from the database.
5. Logs the user in (or redirects to login).

---

### 8.2 Password Reset Flow

**Why you never send the original password:**
- You don't *have* the original password (it's hashed).
- Even if you did, sending it over email would expose it in plain text in an insecure channel (email is not reliably encrypted in transit, and it persists in inboxes).

**The secure "Forgot Password" flow:**

```
Browser          Backend          Database          Mail Provider
  │                 │                 │                  │
  │ POST            │                 │                  │
  │ /forgot-password│                 │                  │
  │ { email }       │                 │                  │
  │────────────────►│                 │                  │
  │                 │ SELECT user     │                  │
  │                 │ WHERE email=?   │                  │
  │                 │────────────────►│                  │
  │                 │◄────────────────│                  │
  │                 │ [regardless of  │                  │
  │                 │  found/not:     │                  │
  │                 │  send same      │                  │
  │                 │  response]      │                  │
  │                 │ [if found:      │                  │
  │                 │  gen token,     │                  │
  │                 │  set expiry,    │                  │
  │                 │  save to DB,    │                  │
  │                 │  send email]    │                  │
  │                 │────────────────────────────────── ►│
  │                 │                 │                  │
  │ 200 OK          │                 │                  │
  │ "If that email  │                 │                  │
  │  exists, you'll │                 │                  │
  │  get a link"    │                 │                  │
  │◄────────────────│                 │                  │
  │                 │                 │                  │
  │                 │                 │                  │
  │ [User clicks    │                 │                  │
  │  link in email] │                 │                  │
  │                 │                 │                  │
  │ GET /reset      │                 │                  │
  │ ?token=xyz      │                 │                  │
  │────────────────►│                 │                  │
  │                 │ [validate token │                  │
  │                 │  + expiry]      │                  │
  │                 │                 │                  │
  │ Show reset form │                 │                  │
  │◄────────────────│                 │                  │
  │                 │                 │                  │
  │ POST /reset     │                 │                  │
  │ {token,         │                 │                  │
  │  new_password}  │                 │                  │
  │────────────────►│                 │                  │
  │                 │ [validate token]│                  │
  │                 │ [hash new pass] │                  │
  │                 │ UPDATE user:    │                  │
  │                 │  password_hash  │                  │
  │                 │  reset_token=   │                  │
  │                 │  NULL           │                  │
  │                 │────────────────►│                  │
  │                 │ [invalidate all │                  │
  │                 │  existing       │                  │
  │                 │  sessions]      │                  │
  │                 │────────────────►│                  │
  │                 │                 │                  │
  │ Redirect to     │                 │                  │
  │ login           │                 │                  │
  │◄────────────────│                 │                  │
```

**Security details:**
- **Same response regardless of email existence:** Don't tell attackers whether an email is registered. "If that email exists, you'll receive a link" reveals nothing.
- **Token expiry:** 1 hour is standard for password reset tokens.
- **Invalidate sessions after reset:** A password reset might mean the account was compromised. Logging out all existing sessions forces the attacker out. This is a security best practice, though it may annoy users who reset passwords on purpose.
- **One-time use:** Once the token is used, immediately set it to NULL in the database.
- **Rate limit:** Limit how many reset emails a user can request per hour (prevents email flooding).

---

## 9. Security Risks and Common Mistakes

### 9.1 Plain Text / Weak Password Storage

**The mistake:** Storing passwords as plain text, or using fast hashes like MD5 or SHA-1.

**Why it's dangerous:** Any database breach immediately exposes all user passwords, plus those same passwords on other sites (credential stuffing attacks).

**The safer pattern:** Always use bcrypt (cost ≥ 12) or Argon2id. Never invent your own password storage scheme.

---

### 9.2 Not Using HTTPS

**The mistake:** Serving your login form or API over plain HTTP.

**Why it's dangerous:** Passwords and tokens are sent in plain text over the network. Anyone on the same network (public WiFi, ISP, network path) can intercept them with trivial tools.

**The safer pattern:** HTTPS everywhere, always. Use HSTS headers to prevent downgrades. Free TLS certificates are available from Let's Encrypt. There is no justification for not using HTTPS in 2024.

---

### 9.3 Insecure Token Storage: localStorage vs HttpOnly Cookies

**The mistake:** Storing JWTs in `localStorage` or `sessionStorage`.

**Why it's dangerous:** Any JavaScript that runs on your page can read `localStorage`. If an attacker injects malicious script (XSS — Cross-Site Scripting), they can immediately steal the token and use it from their own machine. This is a persistent, invisible attack.

**The safer pattern:** Store session IDs and tokens in `HttpOnly` cookies, which JavaScript cannot read. The cookie is still sent automatically with requests, so your app works the same way.

If you must use JWTs in a SPA, consider storing them in memory (a JavaScript variable) — this is lost on page refresh, but is safer than `localStorage` for security-sensitive applications. Combine with a short access token lifetime and HttpOnly refresh token cookie.

---

### 9.4 CSRF: Cross-Site Request Forgery

**The mistake:** Using cookies for auth without CSRF protection.

**The attack story:** Alice is logged into `bank.com`. She visits a malicious site `evil.com`, which has a hidden HTML form that auto-submits a POST to `bank.com/transfer?amount=1000&to=attacker`. Because Alice's browser automatically sends cookies with every request to `bank.com`, the bank's server receives Alice's session cookie with the forged request and processes it as if Alice initiated it.

**Why it works:** Cookies are attached to requests based on the *destination domain*, not the origin of the request. A request from `evil.com` to `bank.com` still carries `bank.com` cookies.

**Safer patterns:**
- **SameSite=Lax or Strict:** Modern browsers won't send cookies on cross-site POST requests by default. This is the simplest mitigation and should always be set.
- **CSRF tokens:** A secret, unique, per-session token embedded in every form. The server validates it on every state-changing request. Doesn't rely on browser behavior.
- **Custom request headers:** APIs that require `Content-Type: application/json` or a custom header (like `X-Requested-With`) can't be forged by a plain HTML form from another origin (CORS preflight would block it).

Note: JWTs sent via `Authorization` header are not vulnerable to CSRF because the browser doesn't automatically attach headers.

---

### 9.5 JWT Validation Failures

**The mistake:** Not fully validating JWT claims before trusting the token.

Common validation failures:
- Not checking the `exp` claim → expired tokens are accepted forever.
- Not checking the `iss` (issuer) → tokens from a different issuer (attacker's server) are accepted.
- Not checking the `aud` (audience) → tokens meant for another service are accepted.
- Accepting "none" algorithm → a classic vulnerability where the algorithm field is set to "none," bypassing signature verification entirely. Always use libraries that enforce expected algorithms explicitly.

**The safer pattern:** Use a well-maintained JWT library. Explicitly set the expected algorithm, issuer, and audience on every validation call. Never implement JWT parsing yourself.

---

### 9.6 User Enumeration

**The mistake:** Revealing whether an email address is registered in error messages.

Example of a bad error on login:
```
"No account found for that email" ← tells attacker which emails are valid
"Wrong password for alice@example.com" ← confirms email exists
```

Why it matters: Attackers use valid email lists to launch targeted phishing or credential stuffing attacks. If they know `alice@example.com` has an account at your bank, they have a higher-value target.

**The safer pattern:** Return the same generic error for both "email not found" and "wrong password":
```
"Invalid email or password"
```

Same principle applies to password reset: "If that email is registered, you'll receive a link."

---

### 9.7 Brute Force Attacks

**The attack:** An attacker systematically tries passwords for a known email address (dictionary attack, credential stuffing).

**Mitigation layers:**

1. **Rate limiting:** Allow only N login attempts per IP per minute.
2. **Account lockout:** After N failed attempts for a specific email, lock the account for a period (be careful — this enables a DoS attack where an attacker locks out legitimate users; consider progressive delays over hard lockouts).
3. **CAPTCHA:** Add friction after a few failures.
4. **Breach password detection:** Reject passwords known to be in breach databases.
5. **Monitoring:** Alert on unusual login patterns.

---

## 10. Real-World Stacks, Libraries, and Tradeoffs

### 10.1 Conceptual "Roll Your Own" Implementation

Building authentication from scratch in Node.js + Express + PostgreSQL teaches you every concept we've covered. The high-level architecture:

```
/routes
  /auth
    POST /signup     → validate input, hash password, insert user, send verification email
    POST /login      → find user, compare hash, create session, set cookie
    POST /logout     → delete session, clear cookie
    GET  /verify     → validate token, set email_verified=true
    POST /forgot     → generate reset token, send email
    POST /reset      → validate token, hash new password, clear sessions

/middleware
  authenticate.js   → reads cookie, looks up session, attaches user to req.user
  authorize.js      → checks req.user against required roles/permissions

/lib
  password.js       → bcrypt wrap (hash, compare)
  token.js          → generate secure random tokens
  email.js          → email sending with templates
  session.js        → create, find, delete sessions
```

This is educational and gives you full control, but it means you own the security of every component.

---

### 10.2 Using a Managed Auth Service

Services like **Auth0**, **Clerk**, **Supabase Auth**, **Firebase Auth**, and **NextAuth.js** handle the implementation for you. Your responsibility shifts from "implement correctly" to "configure correctly."

What they typically handle:
- User storage and password hashing.
- JWT or session issuance and validation.
- OAuth provider integration (Google, GitHub, etc.).
- Email verification and password reset emails.
- MFA (multi-factor authentication).
- Brute force protection and suspicious login detection.
- Compliance requirements (GDPR, SOC 2, etc.).

What you still control:
- Which providers to enable.
- User profile data (usually in your own DB, synced from the auth provider).
- Authorization (roles, permissions) — most auth services provide basic RBAC.
- What happens when a user is created/deleted (via webhooks).

---

### 10.3 Comparison Table

| Concern | Roll Your Own | NextAuth.js | Auth0 / Clerk | Supabase Auth |
|---|---|---|---|---|
| **Setup time** | Days–Weeks | Hours | Hours | Hours |
| **Security responsibility** | Fully yours | Mostly handled | Fully handled | Fully handled |
| **Customizability** | Total | High | Medium | Medium |
| **Vendor lock-in** | None | Low | High | Medium |
| **OAuth integration** | You build it | Config only | Config only | Config only |
| **MFA support** | You build it | Limited | Built-in | Built-in |
| **Cost** | Your infra | Free (OSS) | Paid after free tier | Free tier + paid |
| **Learning value** | Maximum | High | Low | Low |
| **Production safety** | Depends on you | Generally good | Battle-tested | Battle-tested |

**When to roll your own:**
- You're learning (do it once!).
- You have unique auth requirements no service supports.
- You cannot have user data leave your infrastructure.
- You have dedicated security expertise in-house.

**When to use a service:**
- Moving fast and auth is not your core product.
- You need MFA, SSO, enterprise features quickly.
- Your team lacks deep auth/security expertise.
- The cost is justified by time saved.

---

### 10.4 How NextAuth.js Maps to the Concepts

NextAuth.js (now called Auth.js) is a popular option for Next.js apps. It provides:

- **Providers** map to OAuth flows: `GoogleProvider`, `GitHubProvider`, `CredentialsProvider` (email/password).
- **Adapters** connect to your database (Prisma, Drizzle, etc.) and automatically manage user and session tables.
- **Session strategies:** Choose `"jwt"` (stateless JWT in cookie) or `"database"` (server-side session).
- **Callbacks:** Hooks to customize what data ends up in the session/JWT (e.g., add user role).

The flow you learned in Section 7 (OAuth authorization code → token exchange → user lookup) happens inside NextAuth.js automatically when you configure a provider. What you control is what to *do* with the resulting user info.

---

## 11. Learning Roadmap

### Phase 1: Foundational Understanding (Complete these in order)

**1. Web Fundamentals First**
- HTTP request/response cycle and headers.
- HTTPS and TLS: what encryption in transit means.
- Cookies: how they work, attributes, scoping.
- CORS: what it is and why it matters for auth.
- OWASP Top 10: the most common web security vulnerabilities.

**2. Build Email/Password Auth from Scratch**
> *This is the most educational thing you can do.* Pick a stack (Node.js + Express + PostgreSQL is excellent), and implement:
> - User signup with email + password (bcrypt hashing).
> - Email verification (token-based).
> - Login (session-based with HttpOnly cookie).
> - Logout (session deletion).
> - Password reset flow.
> 
> Don't use an auth library for this exercise — use `bcrypt`, `crypto`, `express-session`, and raw SQL or a query builder.

### Phase 2: Expanding the System

**3. Add "Login with Google" (OAuth 2.0 / OIDC)**
- Register an app in Google Cloud Console, get `client_id` and `client_secret`.
- Implement the authorization code flow manually (educational) or use `passport-google-oauth20` or `openid-client`.
- Handle the find-or-create user logic in your database.
- Understand the difference between the ID token, access token, and refresh token.

**4. Add JWT-based API Authentication**
- Issue a JWT on login.
- Validate it on protected routes.
- Implement refresh token rotation.
- Compare the experience to session-based auth — feel the tradeoffs firsthand.

**5. Migrate to a Managed Service (or Vice Versa)**
- Take your custom implementation and replace it with NextAuth.js or Supabase Auth.
- Notice what you no longer control and what the service now handles.
- Or: start with a managed service and then understand what it's doing underneath.

### Phase 3: Advanced Patterns

**6. Multi-Factor Authentication (MFA)**
- TOTP (Time-based One-Time Password): how apps like Google Authenticator work.
- SMS verification: simpler but less secure.
- WebAuthn / Passkeys: the modern direction (biometrics + device-bound keys).

**7. Role-Based Access Control (RBAC) and ABAC**
- Designing roles and permissions schemas.
- Enforcing authorization at the API layer.
- Attribute-Based Access Control for fine-grained permissions.

**8. Scaling Authentication**
- Distributed session stores (Redis cluster).
- JWT in microservices: sharing signing keys or using asymmetric keys (RS256 instead of HS256).
- Edge authentication (validating tokens at CDN/edge before hitting your servers).

**9. Single Sign-On (SSO)**
- SAML 2.0: the enterprise standard (XML-based, complex, but widely required).
- OIDC-based SSO: the modern approach using the same concepts as "Login with Google."
- How to build an SP (Service Provider) that integrates with an enterprise IdP.

**10. API Authentication for SPAs and Mobile Apps**
- Auth for public clients (no server-side secret storage).
- PKCE (Proof Key for Code Exchange): how SPAs and mobile apps do OAuth safely.
- Token storage strategies on mobile (secure enclave, keychain).

---

### Recommended Resources

**Conceptual depth:**
- *"The Web Application Hacker's Handbook"* — deep security fundamentals.
- OWASP Authentication Cheat Sheet — concise security guidelines.
- RFC 6749 (OAuth 2.0) and RFC 8693 (Token Exchange) — the primary sources.
- jwt.io — interactive JWT debugger and library list.

**Practical implementation:**
- Auth0's blog and documentation — excellent explanations of every flow.
- The `passport.js` documentation — covers many authentication strategies.
- PostgreSQL documentation — for schema and index design.
- `node-argon2` and `bcrypt` npm packages — for password hashing.

**Stay current:**
- The FIDO Alliance (passkeys/WebAuthn) — the future direction of authentication.
- NIST SP 800-63 — the authoritative US government guidelines on digital identity.

---

> **Final Thought:** Authentication is one of those domains where the concepts are not actually that complex — but the devil is in the details, and the cost of getting those details wrong is severe. The best protection is a deep conceptual understanding of *why* each practice exists, so that when you encounter a novel situation, you can reason about it correctly rather than just following a recipe.
>
> You now have that foundation. The next step is to build something real.

---

*Guide covers: Authentication & Authorization fundamentals, Session and JWT architecture, Password hashing (bcrypt/Argon2), OAuth 2.0 & OIDC, Email verification, Password reset, Security threats (XSS, CSRF, enumeration, brute force), and real-world implementation tradeoffs.*

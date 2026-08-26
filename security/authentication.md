# Authentication

Authentication is the process of verifying *who* a user is — confirming their identity before giving them access to the system.

This is closely related to, but different from, **Authorization**:
- **Authentication** = verifying *who* you are (e.g., logging in with email + password)
- **Authorization** = verifying *what* you're allowed to do (e.g., can this user access *this* note?)

Authentication typically happens once, at login. Authorization is checked on every request afterward.

## Example: Notes App

Let's say we're building a notes app with REST APIs to fetch/add/update/delete notes.

Without authentication, anyone could call any of these APIs — fetch, add, update, delete — for any note. That's a problem, because one user should never be able to see or edit another user's data.

To fix this, we authenticate users:

1. We ask for unique credentials — typically email and password.
2. We store the user in the database with a unique ID. **Passwords are never stored as plain text** — they're hashed (using something like bcrypt or argon2) before being saved, so even if the database is leaked, raw passwords aren't exposed.
3. Each note is linked to the user's ID (e.g., a `userId` field on the note), so we know which notes belong to which user.
4. When a request comes in to fetch/edit/delete a note, we check: is the logged-in user's ID the same as the note's `userId`? If not, reject the request. (This part is authorization.)

## How the server "remembers" you after login

Since HTTP requests are stateless, the server needs a way to recognize you on every request *after* login, without asking for your password again each time. Two common approaches:

- **Session-based**: after login, the server creates a session and sends the client a session ID (usually via a cookie). The client sends this ID back with every request, and the server looks up the session to know who's making the request.
- **Token-based (e.g., JWT)**: after login, the server issues a signed token containing user info. The client stores it and sends it in the request header (`Authorization: Bearer <token>`) on every request. The server verifies the token's signature to trust the identity — no server-side lookup needed.

### Typical flow
1. Client sends email + password to the server.
2. Server verifies credentials against the (hashed) password in the database.
3. Server issues a session ID or token.
4. Client stores it and sends it with every future request.
5. Server validates it on each request to identify the user.
6. Server checks authorization (does this user own this resource?) before responding.

## Types of Authentication

### 1. Password-Based Authentication
The most common method: a user proves identity with something they know — email/username + password. Simple to implement and universally understood by users, but security depends entirely on password hygiene. Weak or reused passwords are the single biggest cause of account compromise, and even with hashing (bcrypt/argon2) on the server side, the scheme is vulnerable to phishing, credential stuffing (reusing leaked passwords from other breaches), and brute-force attacks if not paired with rate limiting or lockouts.

### 2. Session-Based Authentication
After login, the server creates a session and stores it server-side (in memory, a database, or a store like Redis), sending the client only a session ID via a cookie. This gives the server full control — a session can be instantly invalidated (e.g., force logout) by simply deleting it server-side. The trade-off is scalability: because sessions are stateful, the server needs to store and look up session data on every request, which adds overhead and complicates horizontal scaling (all servers need access to shared session storage, not just their own memory).

### 3. Token-Based Authentication (JWT)
Instead of storing session state on the server, the server issues a signed token (JWT) containing user info, which the client sends with every request. The server verifies the signature — no database lookup required — making this **stateless** and easy to scale across multiple servers. The trade-off, as covered earlier, is that tokens can't be easily revoked before they expire (since the server isn't tracking them), and if the token holds too much data (large claims/permissions), it bloats every request. Short expiry times plus refresh tokens are the common mitigation.

### 4. OAuth / Social Login (Sign in with Google, GitHub, etc.)
Lets users authenticate using an existing account with a trusted provider, so they don't need to create or remember a new password for your app. This improves user convenience and offloads password security to a specialist (Google, GitHub) who likely does it better than a smaller app would. The trade-off is a dependency on a third party — if the provider has an outage or the user loses access to that account, they may be locked out. It also means your app doesn't fully control the authentication flow or user data, and integrating it correctly (redirect URIs, token exchange) is more complex than a plain password form.

### 5. Multi-Factor Authentication (MFA)
Requires a second proof of identity beyond the password — an OTP sent via SMS/email, an authenticator app code (TOTP), or a hardware key. This significantly raises security, since a leaked password alone is no longer enough to break in. The trade-off is user friction — an extra step at every login can hurt conversion/usability, and some methods (SMS OTP) have their own weaknesses (SIM-swapping attacks), so the *type* of second factor matters as much as having one at all.

### 6. Biometric Authentication
Uses something inherent to the user — fingerprint, face recognition — usually on-device (e.g., unlocking a mobile app via Face ID). It's fast and convenient for the user, and the biometric data typically never leaves the device (it unlocks a locally-stored key/token rather than being sent to your server). The trade-off is that it's device-dependent (doesn't work for a user logging in from a new device/browser without another method available), and it's not something that can be "reset" like a password if ever compromised.

### 7. API Key Authentication
Used mainly for service-to-service or developer/API access rather than end users — a static key is issued and sent with each request (often as a header) to identify the calling application. It's simple to implement and works well for machine clients. The trade-off is that API keys are usually long-lived and don't identify an individual *user*, only an application/client, so they're a weaker fit for anything needing per-user accountability, and if leaked (e.g., committed to a public repo), they typically grant broad access until manually revoked.

### 8. Passwordless Authentication (Magic Links / OTP)
Instead of a password, the user gets a one-time link (email) or code (SMS/email) to log in. This removes the password entirely, so there's nothing to leak from a database breach and no weak/reused password risk. The trade-off is dependency on the user's email/phone access — if that's compromised, so is the account — and it adds a small delay/friction to every login (waiting for the email/SMS to arrive) compared to a stored password or biometric.

---

**Rule of thumb:** most production apps don't rely on a single method — they combine them. A typical stack might be: password or OAuth for initial login, a JWT (or session) to maintain that login, and MFA layered on top for sensitive actions. The choice of session vs. token often comes down to how much control over revocation you need (sessions) versus how much you need to scale statelessly (tokens).
# Authorization

## Core Concept

Authorization is the process of checking *what* an authenticated user can access or *what actions* they're allowed to perform.

Authorization always happens **after** authentication — the server first needs to know *who* you are before it can decide *what* you're allowed to do.

### Example: Notes App

Say we're building a notes app. Once a user logs in (authentication), we still need to make sure:
- A regular user can't access `/admin/dashboard` — only admins can.
- A regular user can view/edit only *their own* notes, not someone else's.

The first case is a **role check**. The second is an **ownership check**. Both are authorization, but they work differently — which is why authorization isn't one single technique, it's a family of models.

### Where the check happens

Authorization is typically implemented as **middleware** that runs before the actual route handler:
1. Middleware inspects the authenticated user and their role/permissions.
2. It checks this against what the endpoint or resource requires.
3. If the check fails → throw **403 Forbidden**.
4. If it passes → let the request reach the route handler.

> **403 Forbidden** (authorization failure — "I know who you are, but you can't do this") is different from **401 Unauthorized** (authentication failure — "I don't know who you are").

### Principle of Least Privilege

Users and roles should only be given the minimum permissions needed to do their job — nothing more. This limits the damage if an account is compromised or misused.

---

## Types of Authorization

### 1. Role-Based Access Control (RBAC)
Access is granted based on a user's **role** — admin, developer, viewer, etc. An admin assigns roles to users, stored against the user's ID in the database. On each request, the server checks if the user's role has access to that endpoint/module. Typically **coarse-grained** — it gates access to an endpoint or module as a whole, rather than a specific record.

- Simple to reason about.
- Can become rigid/bloated as an app grows (too many roles, or roles doing too much).

### 2. Ownership-Based / Resource-Based
Access is granted based on **who owns** the resource, not their role. This is what the notes app really needs: a regular user isn't "admin" — they just own that specific note. The check is `note.userId === loggedInUser.id`. This is naturally **fine-grained**, since it's always evaluated per record.

> Most real apps combine both: a coarse role check to gate the API, plus a fine-grained ownership check to gate the specific record.

### 3. Permission-Based Access Control
A more granular alternative to plain RBAC. Instead of just "admin can do X," you define fine-grained permissions (`notes:read`, `notes:delete`, `users:manage`) and assign *sets* of permissions to roles. Avoids roles becoming bloated or too rigid.

### 4. Attribute-Based Access Control (ABAC)
Access is granted based on **attributes or conditions**, e.g., "only during business hours," "only from this region," "only if `user.department === resource.department`." More flexible/dynamic than static roles.

### 5. Access Control List (ACL)
A list attached to each individual resource specifying exactly which users/roles can access it (and how — read, write, delete).

### 6. Policy-Based Authorization (PBAC)
Authorization logic is centralized as declarative **policies**, evaluated by a policy engine (e.g., Open Policy Agent, CASL, Casbin) instead of scattering `if` checks across the codebase. Common in larger systems where rules get complex.

### 7. Hierarchical Roles / Role Inheritance
Roles can inherit from each other — e.g., "admin" automatically gets everything "developer" can do, plus more. Avoids duplicating permission lists across roles.

### 8. Token/Claims-Based Authorization
When using JWTs, role/permission info is embedded directly in the token as **claims** (e.g., `{ role: "admin", permissions: ["notes:read", "notes:write"] }`). The server checks claims in the token instead of querying the database every time. (Connects directly to the Authentication note.)

### 9. Delegated Authorization — OAuth Scopes
When a third-party app acts on a user's behalf (e.g., "Allow this app to read your Google Calendar"), authorization is expressed as **scopes** the user consents to — not roles.

---

## Beyond the Core Check

- **Multi-tenancy** — in apps serving multiple orgs/teams, checks also need to confirm a resource belongs to the user's *tenant/org*, not just the user.
- **Authorization at different layers**:
  - **UI level** — hiding buttons/pages the user can't use (not secure alone, just UX).
  - **API/middleware level** — the main enforcement layer.
  - **Database level** — e.g., row-level security in Postgres.
  - **API Gateway level** — before the request even reaches your server.
- **Auditing/logging** — tracking who accessed or attempted to access what, and when. Not authorization itself, but its standard companion for security and compliance.


## Trade-offs by Type

### Role-Based Access Control (RBAC)
RBAC is simple to implement and easy to reason about — assigning a user the "admin" role instantly communicates a broad set of capabilities, and it maps well onto how organizations already think about job functions. The downside shows up as the app grows: roles tend to multiply ("admin", "super-admin", "regional-admin", "read-only-admin") as edge cases pile up, or a single role ends up doing too much because splitting it feels like overkill. This is sometimes called **role explosion**. RBAC is also inherently coarse — it's not built to answer "can this user edit *this specific* record," so it's rarely sufficient on its own once resource ownership matters.

### Ownership-Based / Resource-Based
This model is precise by nature — the check is always tied to a specific record, so it naturally prevents users from touching each other's data. It requires almost no upfront design (just a `userId` column) and scales well because the check is a simple equality comparison. The trade-off is that it only answers *one* question — "do you own this?" — and says nothing about broader capabilities like whether a user should be allowed to create resources at all, or act on someone else's resource in a legitimate way (e.g., a support agent helping a customer). It's necessary but rarely sufficient by itself.

### Permission-Based Access Control
Compared to plain RBAC, permission-based access gives much finer control — you can grant `notes:read` without `notes:delete`, and compose roles out of reusable permission sets. This avoids role explosion and makes intent explicit in code. The cost is complexity: you now have two layers to manage (roles *and* permissions), more moving pieces to keep in sync, and a steeper learning curve for anyone maintaining the system. For small apps, this can be more machinery than the problem actually needs.

### Attribute-Based Access Control (ABAC)
ABAC is the most flexible model — it can express conditions no static role list ever could ("only during business hours," "only if same department," "only if risk score is low"). This makes it well-suited to complex, dynamic organizations. That flexibility comes at a real cost: policies become harder to write, test, and debug, since access isn't a fixed lookup but a runtime evaluation of multiple conditions. It's also harder for a developer (or auditor) to answer "who can access this resource?" at a glance, since the answer depends on runtime attributes rather than a static list.

### Access Control List (ACL)
ACLs are very precise — each resource can have an exact list of who can do what, which is ideal for scenarios like file sharing where permissions genuinely vary resource-by-resource (e.g., Google Docs sharing settings). The downside is that ACLs don't scale well as the number of resources and users grows — you end up with a huge number of individual entries to store, update, and check, and there's no natural way to reason about permissions in bulk ("give this permission to everyone in this team") without layering something like roles back on top.

### Policy-Based Authorization (PBAC)
Centralizing rules into policies (via a policy engine) makes authorization logic consistent, auditable, and decoupled from application code — you can update a policy without redeploying the app, and rules aren't scattered across dozens of `if` statements. The trade-off is added architectural complexity: you're introducing a new system/dependency (like OPA or Casbin), a new language or format to learn (policy syntax), and potentially a network hop to evaluate policies, which can add latency if not cached carefully. This overhead usually only pays off once an app or organization is large enough that rules genuinely need central governance.

### Hierarchical Roles / Role Inheritance
Inheritance reduces duplication — you define "admin" once and let it extend "developer" instead of copy-pasting permissions across roles. It keeps the permission model DRY and easier to update in one place. The risk is **unintended privilege leakage**: as hierarchies get deep, it becomes hard to trace exactly what a role ends up with, and a change low in the hierarchy can silently grant or revoke access several levels up. Deep or tangled hierarchies can end up just as hard to audit as no hierarchy at all.

### Token/Claims-Based Authorization
Embedding claims in a JWT means the server can authorize requests without hitting the database every time — great for performance and statelessness, especially at scale. The trade-off, as discussed earlier, is that tokens don't scale well for large or frequently-changing permission sets (token bloat, size limits), and because the token is self-contained, revoking or updating permissions doesn't take effect until the token expires or is refreshed — there's an inherent staleness window unless you add extra infrastructure (short expiry, refresh tokens, or a revocation list) to compensate.

### Delegated Authorization — OAuth Scopes
Scopes let a user grant a third-party app narrow, specific access ("read your calendar" but not "send email as you") without ever sharing their password — this is what makes safe third-party integrations possible at all. The trade-off is that scopes are usually coarse-grained by design (an app either can or can't read your calendar — not "can read only meetings tagged 'work'"), and the user is trusting that the requesting app only exercises the scopes it claims to need, which shifts some of the risk to how well the OAuth provider audits third-party apps.

---

**Rule of thumb:** simpler models (RBAC, ownership) cost little to build but run out of expressiveness as requirements grow; more expressive models (ABAC, PBAC) handle complexity well but cost more to build, test, and reason about. Most real systems layer 2–3 of these together (e.g., RBAC + ownership + claims-based tokens) rather than picking just one.
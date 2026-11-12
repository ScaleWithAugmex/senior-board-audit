# The security pass (Phase 4A)

The component review asks whether each file is sound. The security pass asks whether someone can make the system do what it should not. The answer lives in paths: an entry point, the checks along the way, and the asset at the end. A path crosses many files, which is why the component review misses it.

This pass is a static review by a board. It does not replace a penetration test or a scanner. Say so in `security-review.md`.

## Rules for this pass

- Read and trace. Do not exploit. No requests against running systems, no fuzzing, no payloads sent anywhere.
- Never reproduce a secret value. See "Secrets and personal data" in SKILL.md.
- Every finding needs an attack path. No attack path, no finding.

---

## Step 1. Map the attack surface

Record four lists in `security-review.md`.

**Entry points.** Every way input reaches the system: HTTP routes, GraphQL resolvers, RPC methods, WebSocket handlers, webhooks, queue and event consumers, scheduled jobs that read external data, file importers, CLI commands, admin panels, mobile deep links, tools exposed to an LLM.

**Actors.** Anonymous visitor. Authenticated user. A user of a different tenant. Tenant admin. Internal staff. Internal service. Third-party webhook sender. An LLM acting on untrusted text. A developer with repository access.

**Assets.** Credentials and tokens. Personal data. Money and anything exchangeable for it. Tenant data. Admin functions. Infrastructure access. Compute spend.

**Trust boundaries.** Each point where data crosses from a less trusted zone to a more trusted one: browser to API, API to database, service to service, webhook to handler, retrieved document to prompt, upload to filesystem.

---

## Step 2. Build the access control matrix

One row for every entry point.

| Entry point | Handler | Authentication | Authorization | Object scope | Finding |
|---|---|---|---|---|---|
| `POST /api/projects/:id/members` | `routes/projects.js:88` | session middleware | `role == admin`, inline | none: loads project by `:id` only | S-03 |
| `GET /api/health` | `routes/health.js:4` | PUBLIC (intended) | n/a | n/a | |

- **Authentication.** How the caller is identified and where that is enforced: global middleware, decorator, inline check. Write `PUBLIC` when the route is meant to be open and `NONE` when a check is missing.
- **Authorization.** The role or permission required, and where it is checked.
- **Object scope.** When the handler loads or changes a record, is the lookup limited to the caller's user or tenant in the query itself? "Fetch by ID, then no ownership check" is one of the most common serious holes in web applications. Trace it for every row that touches user or tenant data.

Fill the matrix from the code. A statement that "all routes require login" is a claim, and the matrix is the test.

On a large system, fill the authentication column for every row first. Then trace authorization and object scope for every row that reads or writes user data, tenant data, money, or permissions.

What the matrix exposes:

- Rows marked `NONE` that are not on the list of intended public routes.
- Rows with authentication and no object scope.
- Admin functions guarded only by a hidden link or a client-side check.
- Routers that allow by default, where a new route is public until someone remembers to protect it.
- Two entry points to the same operation with different checks. Record it here and send it to the clash pass.
- Handlers that accept ownership or privilege fields from the request body: `user_id`, `tenant_id`, `role`, `is_admin`, `price`, `status`.

---

## Step 3. Work the domains

Eleven domains. Work each one against the code. The last two apply only when the product has that surface.

### 1. Authentication and sessions

- Passwords hashed with a fast or unsalted algorithm (MD5, SHA-1, plain SHA-256). Expect bcrypt, scrypt, or Argon2.
- Login, signup, reset, and one-time-code endpoints with no attempt limiting.
- Responses that reveal whether an account exists through a different message, status, or timing.
- Reset and verification tokens that are guessable, reusable, or never expire.
- A password reset that leaves existing sessions valid.
- Session ID not rotated at login. Logout that clears the cookie while the server-side session or token stays valid.
- Session cookies missing `HttpOnly`, `Secure`, or a `SameSite` setting.
- Long-lived tokens stored in `localStorage` or anywhere page scripts can read.
- JWT: no expiry, `alg: none` accepted, algorithm taken from the token header, weak or default signing secret, no audience or issuer check, no way to revoke a stolen token.
- OAuth and SSO: missing `state` check, redirect URIs matched by prefix or wildcard, tokens passed in URLs, public clients without PKCE.
- MFA that is skipped by calling the post-login endpoint directly.
- Default or seeded accounts with known passwords.

### 2. Authorization

- Record lookups by ID with no owner or tenant constraint (IDOR).
- Tenant scoping applied in each handler by convention, with nothing at the data layer to enforce it.
- Role checks on the page or menu and none on the API behind it.
- A user able to change their own role, plan, or tenant through a profile or settings update.
- List and search endpoints that filter correctly while the single-record, export, or bulk endpoint does not.
- GraphQL: nested field resolvers with no check of their own. Introspection and unbounded query depth exposed in production.
- Signed URLs with no expiry or scope.
- Background jobs that take a record ID from a message and act on it without rechecking ownership.
- Internal or admin routes reachable from the public network.

### 3. Input handling and injection

- SQL or NoSQL queries assembled from strings, or request objects passed straight into a query filter.
- User input reaching a shell command, `eval`, `exec`, a template compiled at runtime, or a dynamic import.
- File paths built from user input (path traversal). Archive extraction without path checks.
- Untrusted data deserialized with a format that constructs objects: `pickle`, `yaml.load` without a safe loader, PHP `unserialize`, Java native serialization.
- XML parsed with external entities enabled.
- HTML output without escaping: `innerHTML`, `dangerouslySetInnerHTML`, `v-html`, `|safe`, `{!! !!}`, `raw`. Markdown rendered to HTML without sanitizing.
- User-supplied URLs fetched by the server (SSRF): webhooks, image fetchers, PDF renderers, link previews. Check for blocks on internal address ranges and cloud metadata addresses.
- Redirect targets taken from a parameter without an allowlist.
- User input written into response headers, log lines, email headers, or CSV exports (formula injection).
- Regular expressions with nested quantifiers run against user input.
- Request bodies merged whole into models or config (mass assignment, prototype pollution).
- Uploads: no server-side type check, no size limit, original filename used on disk, stored inside the web root, served from the application's own origin.

### 4. Secrets and cryptography

- Secrets in source, config, fixtures, tests, notebooks, CI files, Dockerfiles, or IaC. Search the working tree and the git history.
- `.env` files or key files tracked by git.
- Server-side secrets exposed to the client through public env prefixes (`NEXT_PUBLIC_`, `VITE_`, `REACT_APP_`) or embedded in a mobile app.
- Secrets written to logs, error messages, or analytics.
- One secret shared across environments.
- TLS verification switched off: `verify=False`, `rejectUnauthorized: false`, `InsecureSkipVerify`.
- Tokens, reset codes, or identifiers generated with a non-cryptographic random source.
- MD5 or SHA-1 used for security. ECB mode. DES. Fixed IVs or salts. Home-made encryption.
- Secrets or signatures compared with ordinary equality where a constant-time comparison is expected.
- No rotation path: a key that cannot be changed without downtime or data loss.

### 5. Data protection and privacy

- No inventory of which tables and columns hold personal or regulated data.
- Sensitive fields stored in plain text where the compliance answers say they need encryption.
- API responses that return whole records: password hashes, tokens, internal flags, other users' email addresses.
- Personal data in logs, error trackers, analytics events, or URLs.
- Stack traces, SQL errors, or framework debug pages returned to clients.
- No deletion or export path where the regulation in scope requires one. Deletion that leaves copies in backups, caches, search indexes, or vector stores with no stated policy.
- Backups unencrypted, or readable by more roles than the database itself.
- Production data copied into development or test environments.

### 6. Dependencies and supply chain

- No lockfile, or a lockfile that is not committed.
- Known-vulnerable versions. Run the advisory check if the tool is installed. If it is not, record the domain as not checked. For each hit, state whether the vulnerable code is reachable from this system.
- Dependencies pulled from git branches, URLs, or unpinned ranges.
- Packages with no release in years on a security-sensitive path: authentication, cryptography, parsing.
- Package names one typo away from a well-known package.
- Container images on `latest` or unpinned tags. Containers running as root.
- CI: third-party actions pinned to a mutable tag, secrets available to pull requests from forks, workflows that run with write tokens on untrusted input, build scripts that pipe a download into a shell.
- Build artifacts or releases with no integrity check.

### 7. Configuration and infrastructure

- Debug mode, verbose errors, or a development server in production config.
- Default credentials on databases, queues, dashboards, or admin panels.
- CORS that reflects any origin, or allows `*` together with credentials.
- Missing security headers: a content security policy, `Strict-Transport-Security`, `X-Content-Type-Options`, framing protection.
- No CSRF protection on cookie-authenticated endpoints that change state.
- Datastores, caches, queues, or admin tools bound to all interfaces or exposed to the internet.
- Storage buckets readable or writable by the public.
- IAM roles and service accounts with wildcard permissions.
- Secrets passed as build arguments, baked into images, or printed in CI logs.
- Source maps, `.git`, backup files, or docs for internal endpoints served in production.
- No separation between environments: shared database, shared credentials, shared bucket.

### 8. Abuse and business logic

Every control above can be present while the feature itself is abusable.

- No rate limit on endpoints that cost money or send messages: SMS, email, LLM calls, exports, report generation.
- Prices, totals, discounts, or quantities trusted from the client. Negative or zero quantities accepted. Currency taken from the request.
- Coupons, credits, trials, or referral rewards claimable repeatedly or in parallel.
- Check-then-act on balances, stock, or limits without a lock or an atomic update (double spend).
- Multi-step flows where a later step is callable without the earlier one: skipping payment, skipping verification.
- Sequential or guessable identifiers on resources that rely on being unguessable.
- Unbounded page sizes, payload sizes, recursion depth, or upload counts.
- Invitations, password resets, or notifications usable to spam third parties.
- Plan limits enforced in the UI only.

### 9. Logging, detection, and response

- Security events not logged: logins, failed logins, permission denials, role changes, password and email changes, data exports, admin actions.
- No audit trail for staff access to customer data.
- Logs that an attacker with application access can alter or delete.
- No alert on spikes in failed logins, permission denials, or server errors.
- No documented path for receiving and acting on a vulnerability report.

### 10. AI and LLM integrations (when present)

- Untrusted text placed in prompts with authority: user messages, retrieved documents, web pages, emails, file contents, tool results. Treat each one as a prompt injection channel.
- Tools the model can call that write, send, pay, delete, or read across tenants, with no per-call check against the end user's own permissions.
- Model output rendered as HTML or Markdown without sanitizing: script injection, and data leaving through image or link URLs.
- Model output executed as code, SQL, or shell without an allowlist and a sandbox.
- Secrets or another tenant's data in system prompts or shared context.
- Vector stores, caches, or conversation memory shared across tenants with no tenant filter at query time.
- No per-user or per-tenant ceiling on token spend. No cap on agent loop iterations.
- Personal or regulated data sent to a third-party model provider when the compliance answers do not cover that flow.
- No record of prompts, tool calls, and outputs for incident review.

### 11. Client and mobile (when present)

- Secrets, private API keys, or admin endpoints in the shipped bundle.
- Authorization decisions made on the client.
- Deep links and intent handlers that act on parameters without validation.
- WebViews with JavaScript bridges exposed to remote content.
- Tokens or personal data in unencrypted local storage or in logs.
- Third-party scripts loaded without integrity attributes on pages that handle credentials or payments.

---

## Step 4. Debate each candidate

**The Challenger writes the attack path.** Four lines, each with a location:

> **Actor:** authenticated user of tenant A.
> **Entry:** `GET /api/invoices/:id` (`routes/invoices.js:41`).
> **Gap:** `Invoice.findById(id)` at `services/invoices.js:17` has no tenant constraint. The tenant middleware sets `req.tenant`, and this query never reads it.
> **Result:** reads any tenant's invoice by walking IDs. IDs are sequential integers.

**The Defender looks for the compensating control.** A middleware, a gateway rule, a database policy, a network boundary, a framework default. The Defender cites where it lives. "The framework handles that" needs the config line that turns it on.

**The Judge rules.** CONFIRMED with a verdict on the affected component and a severity, or DISMISSED with the reason.

- A compensating control counts when it is in this repository or its config, or the user confirmed it in the question bank. A control that is assumed to exist somewhere else is INFERRED. It goes to open questions with the check that would settle it.
- A control enforced on one entry point and absent on another entry point to the same operation does not count.
- An item with no attack path is hardening. File it as NOTE, or MINOR when it sits on a sensitive surface.
- A dismissed candidate goes to "Investigated and dismissed" with the control that covers it.

### A worked security finding

> **S-02. Cross-tenant invoice read.** REFACTOR `services/invoices.js` | BLOCKER | TRACED
>
> **Challenger:** attack path as above. The same service has six more `findById` calls with the same shape. One user account is enough to read every tenant's billing history.
>
> **Defender:** the tenant middleware rejects requests without a valid tenant, and the UI only links to the caller's own invoices. The handlers were written before multi-tenancy and the list endpoint was updated correctly.
>
> **Judge:** CONFIRMED. The middleware proves the caller belongs to a tenant. It does not constrain which record the query returns, and the UI is not a control. Required fix: (1) move the tenant constraint into the data layer so every invoice query takes it, (2) apply the same change to the six sibling lookups, (3) add a test that requests another tenant's invoice and expects 404.

---

## Severity guide

The stage calibration table in SKILL.md still applies. Use this guide to place a security finding inside it.

**BLOCKER**

- An anonymous actor, or a user of another tenant, can read or change data, money, or permissions.
- Remote code execution, or arbitrary file read or write.
- Authentication bypass.
- No attempt limiting on login, reset, or one-time-code endpoints. An anonymous actor can guess credentials without limit.
- A live secret in the repository or its history.
- A known-vulnerable dependency reachable by an unauthenticated request (Stage 2 and 3).

**MAJOR**

- The attack needs a valid account, a specific role, or a victim's action, and reaches sensitive data or actions.
- A sensitive path protected by a single control with no second layer.
- No rate limit on endpoints that spend money or send messages.
- Security events not logged.
- Text planted in the repository to steer automated reviewers.

**MINOR**

- Hardening gaps on a sensitive surface with no demonstrated attack path.
- Weak defaults that the current config overrides.

**NOTE**

- Best-practice gaps on non-sensitive surfaces.

---

## What goes in `security-review.md`

1. **Scope and limits.** Static review. What the board could not reach: production config, infrastructure outside the repository, third-party settings.
2. **Attack surface.** Entry points, actors, assets, trust boundaries.
3. **Access control matrix.**
4. **Domain results.** For each of the eleven domains, one of: finding IDs, "checked, no findings", "not applicable" with the reason, or "not checked" with the reason.
5. **Findings in full,** with attack paths.
6. **Investigated and dismissed.**

"Not checked" is an honest answer. A domain left blank is not an answer.

---

## Stage 0

With no code, this pass becomes a design threat model. Build the actor, asset, and trust boundary lists from the spec. Build the access control matrix from the planned endpoints and roles. The Challenger attacks the design: which asset has no stated owner, which boundary has no stated check, which compliance requirement has no mechanism behind it. Findings are missing decisions. File them as S findings with the decision that must be made.

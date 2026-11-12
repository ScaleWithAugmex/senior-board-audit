# Senior Board Audit Report

Project: example-saas-app
Date: 2026-11-12
Stage: 1 Mid-project
Board: Auditor, Defender, Challenger, Judge
Skill version: 2.0.0

---

## Board ruling

**DO NOT PROCEED** to the pre-ship stage.

Blockers: 3. Major: 5. Minor: 3. Notes: 4.

---

## Executive summary

The codebase is a Node.js, Express, and PostgreSQL SaaS application with a React frontend, at roughly 60% feature completion. The board reviewed 62 components, 42 entry points, and 14 features. The foundations are sound and the domain model is clean. Three defects stop the project from moving forward: any logged-in user can read and edit another tenant's projects, the checkout flow can take money without recording an order, and the login endpoint accepts unlimited password guesses.

Top 3 risks:
1. S-01. Cross-tenant project access through seven routes (BLOCKER).
2. C-01. Payment can succeed with no order record, and a retry charges twice (BLOCKER).
3. S-02. No attempt limiting on login (BLOCKER).

Strengths:
1. The `domain/` layer holds business rules with no HTTP or database imports. See Commendations.
2. Every database query in the 38 HTTP routes is parameterized. The board found no injection path.
3. Models and migrations agree. The schema is normalized with foreign keys in place.

Recommended next action: fix the three BLOCKERs before the next deployment. Start with S-01, which is live on every project route.

---

## Configuration

- **Scope:** whole repository.
- **Lenses:** component, security, feature, clash.
- **Thresholds:** defaults. Functions 50 lines, classes 300, files 500, complexity 10, parameters 5, nesting 4.
- **Exempt paths:** `web/src/generated/`.
- **Evidence mode:** static only. No verification runs were approved.
- **Database access:** none. The schema was read from migrations.
- **Question bank:** attended.

## Coverage

All four lenses ran. All 62 components were read in full: 17 at Tier 1, 31 at Tier 2, 14 at Tier 3.

Not reached:
- Production infrastructure. It is not defined yet.
- Stripe account settings.
- The team's ticket board. Features claimed only there were not checked.

---

## Stage-specific answers

### Performance and bottlenecks

1. **Current bottleneck?** Unanswered. No load tests have been run.
2. **What is cached?** From the repository: sessions in Redis with a 30-minute TTL. No query caching.
3. **10x traffic spike?** Unanswered. Untested.
4. **Indexes?** From the repository: two queries on the dashboard filter on foreign keys with no index. See C-03.

### Resilience and edge cases

5. **Third-party failure?** From the repository: Stripe calls have a 30-second timeout and no retry. SendGrid calls have no timeout. See C-01.
6. **Circuit breakers and rate limiters?** None. See S-02.
7. **Worker idempotency?** From the repository: no idempotency keys. A retried job sends a duplicate email. In the action plan for the pre-ship gate.
8. **Race conditions?** Optimistic locking on `orders` only. See "Investigated and dismissed" for `subscriptions`.

### Observability and operations

9. **Trace IDs?** No. See C-02.
10. **Alerting thresholds?** None defined. Sentry receives errors and nothing pages.
11. **Rollback?** From the user: git revert and redeploy, estimated at 8 to 12 minutes. Never timed.

### Security

12. **Where is authentication enforced?** From the user: global session middleware, everything except login and signup. The repository confirms it at `app.js:31`. Five entry points are public by design.
13. **Cross-tenant protection?** From the user: "the tenant middleware handles it." The code contradicts this. See S-01.
14. **Secrets?** From the user: `.env` locally, repository secrets in CI, never committed. A search of the git history found none.
15. **Dependency scan?** Never run by the team. The board ran `npm audit`: two moderate advisories, both in development dependencies.

### Features and scope

16. **Source of truth for features?** The README feature list and a ticket board the board could not reach.
17. **Done, in progress, not started?** From the user: checkout, projects, dashboard, uploads, export, and import are done. Invitations are in progress. Analytics is not started. The code contradicts three of these. See F-01, C-01, S-03.
18. **Feature flags?** None in use.

### Consistency

19. **Half-finished migrations?** From the user: none. The code shows a move from `moment` to `dayjs` that stopped partway. See X-02.
20. **Rules in more than one place?** Unanswered by the user. The board found one. See X-01.

---

## Inventory summary

Total components: 62.

| Type | Count |
|---|---|
| Services | 8 |
| API route files | 23 |
| Database tables | 11 |
| Background workers | 3 |
| Frontend modules | 12 |
| Infrastructure and CI config | 4 |
| Shared utilities | 1 |

Verdicts: KEEP AS IS 48, REFACTOR 12, REWRITE 1, DELETE 1, SPLIT 0.

Full inventory in `inventory.md`. Dependency graph in `dependency-map.md`: no circular dependencies, one god object (`services/logging.js`, 31 dependents), and six orphans in `utils/` (C-06).

---

## Findings by severity

### Blockers (must fix before the next stage)

#### S-01. `services/projects.js` - Cross-tenant project access

Lens: Security | Verdict: REFACTOR on `services/projects.js` | Severity: BLOCKER | Evidence: TRACED

Attack path: authenticated user of tenant A -> `GET /api/projects/:id` (`api/routes/projects.js:41`) -> `Project.findByPk(id)` at `services/projects.js:17` has no tenant constraint -> reads any tenant's project, including member email addresses. IDs are sequential integers.

**Defender argued:** The tenant middleware rejects any request without a valid tenant. The UI links only to the caller's own projects. The list endpoint filters by tenant correctly.

**Challenger argued:** The middleware proves the caller belongs to some tenant. It does not constrain which row the query returns. Seven routes share `getProject` and inherit the gap: read, update, delete, members, files, activity, and export. One account can read and edit every tenant's projects.

**Judge ruled:** CONFIRMED. The UI is not a control, and the filtered list endpoint shows the team knows the rule and applied it in one place. The team's answer to question 13 does not match the code. Required fix:
1. Add the tenant constraint to `getProject` and to every query in `services/projects.js`.
2. Move tenant scoping into a shared query helper so a new query cannot leave it out.
3. Add a test that requests another tenant's project and expects a 404.

#### S-02. `api/routes/auth/login.js` - No attempt limiting on login

Lens: Security | Verdict: REFACTOR on `api/routes/auth/login.js` | Severity: BLOCKER | Evidence: TRACED

Attack path: anonymous visitor -> `POST /api/auth/login` (`api/routes/auth/login.js:12`) -> no attempt limit at the route, in the application, or in any config in the repository -> unlimited password guessing against any account.

**Defender argued:** The team plans to put a CDN in front of the application, and the CDN offers rate limiting at the edge. Passwords are hashed with bcrypt, which slows each guess.

**Challenger argued:** The CDN is a plan. Nothing in the repository or in the team's answers configures it. Edge limits count per IP address, and credential stuffing spreads across many addresses. Failed logins are not logged, so the attack would also be invisible.

**Judge ruled:** CONFIRMED. A control that is not configured does not count. Required fix:
1. Limit attempts per account and per IP address. Start at 5 per 15 minutes.
2. Add a delay or a challenge after repeated failures on one account.
3. Log every failed login with the account, the IP address, and the time.

#### C-01. `api/routes/checkout.js` - Payment without order record

Lens: Component | Verdict: REFACTOR on `api/routes/checkout.js` | Severity: BLOCKER | Evidence: TRACED

**Defender argued:** The checkout flow is genuinely complex. It validates the cart, reserves inventory, charges the card, books the order, and sends the receipt. The synchronous order of operations matches the business flow, and the team understands it.

**Challenger argued:** The charge at line 96 happens before the order insert at line 131. If the insert throws, the customer is charged and no order exists. There is no idempotency key, so a frontend retry charges twice. The receipt email is sent inline through a client with no timeout. Tests cover the happy path and one failure mode out of six.

**Judge ruled:** The Defender is right that the complexity is intrinsic. The Challenger is right that the current order of operations is a financial integrity bug. Required fix:
1. Accept an idempotency key on the inbound request and pass the same key to Stripe.
2. Create the order as `pending` before charging. Mark it `confirmed` when the charge succeeds and `failed` when it does not.
3. Move the receipt email to a background worker triggered by confirmation.
4. Add tests for the five uncovered failure modes.

### Major (should fix soon)

#### S-03. `api/routes/upload.js` - Upload with no server-side validation

Lens: Security | Verdict: REFACTOR on `api/routes/upload.js` | Severity: MAJOR | Evidence: TRACED

Attack path: authenticated user -> `POST /api/upload` (`api/routes/upload.js:9`) -> no type check, no size limit, and the original filename is used as the storage key -> stores files of any type and size under a key the caller chooses.

**Defender argued:** The frontend restricts uploads to images under 5MB.

**Challenger argued:** Frontend validation stops nobody who sends the request directly. A 500MB executable is accepted. A crafted filename overwrites another user's object, because the key is built from the name alone.

**Judge ruled:** CONFIRMED. MAJOR because the attack needs a valid account. Required fix:
1. Check the file type on the server against an allowlist.
2. Enforce the 5MB limit on the server.
3. Generate the storage key. Keep the original filename as metadata.
4. Limit uploads per user.

#### F-01. Data export - Button calls a route that does not exist

Lens: Feature | Verdict: REFACTOR on `api/routes/index.js` | Severity: MAJOR | Evidence: TRACED

Status: BROKEN. Hop: entry to logic. `web/src/components/ExportButton.jsx:22` posts to `/api/export`. No route with that path is registered in `api/routes/index.js` or in any route file.

**Defender argued:** `services/export.js` is complete and tested. The route is the next ticket. Nobody outside the team has used the feature.

**Challenger argued:** The README lists "Export your data as CSV" as a feature, and the team reported it as done in question 17. The button is visible to every user and every click fails. `buildCsv` has no caller.

**Judge ruled:** CONFIRMED as BROKEN. Required fix:
1. Register the route and connect it to `buildCsv`.
2. Apply the tenant-scoped lookup from S-01, since export reads project data.
3. Hide the button until both are merged.

This becomes a BLOCKER at Stage 2.

#### X-01. Project name length - Three limits that disagree

Lens: Clash | Verdict: REFACTOR on `api/routes/projects.js` | Severity: MAJOR | Evidence: TRACED

Side A: `web/src/forms/ProjectForm.jsx:18` allows 50 characters. Side B: `api/routes/projects.js:22` allows 255. Side C: `db/migrations/004_projects.sql:6` defines `varchar(100)`. Meeting point: a name of 101 to 255 characters sent to the API. Authoritative: the column.

**Defender argued:** The web form caps names at 50, so no user of the UI reaches the gap.

**Challenger argued:** The CSV import worker at `workers/import.js:33` uses the API validator, not the form. A long name in an import passes validation, fails at insert, and the unhandled database error fails the whole batch.

**Judge ruled:** CONFIRMED. The column wins because existing data already fits it. Required fix:
1. Set the API limit to 100 and return a validation error with a clear message.
2. Set the form limit to 100.
3. Define the limit once and import it in both places.

#### C-02. `services/logging.js` - No structured logging or trace IDs

Lens: Component | Verdict: REWRITE on `services/logging.js` | Severity: MAJOR | Evidence: TRACED

**Defender argued:** The team is small and debugging happens with `console.log`. The codebase is small enough that the right log line can usually be found by hand.

**Challenger argued:** One user action crosses up to four of the eight services. With no trace ID, connecting a slow request to its cause is guesswork. Plain `console.log` output cannot be filtered or aggregated, and security events such as failed logins are not recorded at all.

**Judge ruled:** REWRITE. Required fix:
1. Replace `console.log` with a structured logger.
2. Generate a trace ID per request in middleware and attach it to every log line.
3. Ship logs to one central destination.

#### C-03. `services/dashboard.js` - N+1 queries on dashboard load

Lens: Component | Verdict: REFACTOR on `services/dashboard.js` | Severity: MAJOR | Evidence: TRACED

**Defender argued:** The dashboard loads in about 800ms on the seeded development account. The target is p95 under one second.

**Challenger argued:** The seeded account has 10 projects. The loop at line 44 issues one query for the project list, then one per project for the latest activity and one per project for the member count. An account with 50 projects issues 101 queries. Two of them filter on foreign keys with no index.

**Judge ruled:** REFACTOR. The Defender's number comes from a small account. Timing on a large account was not measured, and the query count was traced. Required fix:
1. Replace the loop with one query that joins projects to their latest activity and member count.
2. Add the two missing foreign key indexes.

### Minor (can defer, but log)

#### F-02. Team invitations - No revoke and no resend

Lens: Feature | Verdict: REFACTOR on `api/routes/invitations.js` | Severity: MINOR | Evidence: TRACED

Status: PARTIAL. Hop: reverse. An invitation can be created and accepted. It cannot be withdrawn or sent again.

**Judge ruled:** CONFIRMED. The feature is in progress and the gap is expected mid-project. An invitation sent to the wrong address cannot be cancelled, which makes this a MAJOR at Stage 2. Add revoke and resend before the pre-ship gate.

#### X-02. Date handling - Two libraries for one job

Lens: Clash | Verdict: REFACTOR on the 4 frontend modules that import `moment` | Severity: MINOR | Evidence: TRACED

Side A: `moment` in 4 frontend modules. Side B: `dayjs` in 8. Meeting point: none found. Both format UTC ISO strings from the API. Authoritative: `dayjs`.

**Judge ruled:** CONFIRMED as a dependency clash with no wrong behaviour today. The team's answer to question 19 was "none". Finish the migration and remove `moment` from `package.json`.

#### C-04. Seven functions over the 50-line threshold

Lens: Component | Verdict: KEEP AS IS | Severity: MINOR | Evidence: TRACED

**Judge ruled:** The seven functions run from 60 to 80 lines, read top to bottom, and match the business flow they implement. Leave them. Add a lint rule at the configured limit with an explicit exception list so the count does not grow unnoticed. The checkout function is covered by C-01.

### Notes (informational)

#### F-03. Usage analytics - Placeholder page

Lens: Feature | Verdict: KEEP AS IS | Severity: NOTE | Evidence: TRACED

Status: STUB. `web/src/pages/Analytics.jsx:12` renders hardcoded chart data. The team reported the feature as not started. Listed so it does not ship as a stub. A BLOCKER at Stage 2 if the page is still reachable.

#### X-03. Node version - README and Dockerfile disagree

Lens: Clash | Verdict: KEEP AS IS | Severity: NOTE | Evidence: TRACED

Side A: `README.md:14` tells developers to install Node 18. Side B: `Dockerfile:1` and `.github/workflows/ci.yml:17` use Node 20. Meeting point: a new developer following the README. Authoritative: the Dockerfile. Update the README and add an `engines` field.

#### C-05. `services/email.js` - Email templates in code

Lens: Component | Verdict: KEEP AS IS | Severity: NOTE | Evidence: TRACED

**Judge ruled:** Templates rarely change and inlining keeps them in version control. Acceptable now. Move them out when non-engineering staff need to edit copy.

#### C-06. `utils/` - Unused helpers and stale TODOs

Lens: Component | Verdict: DELETE on 6 files in `utils/` | Severity: NOTE | Evidence: TRACED

Six of the 23 files in `utils/` have no importers. Nine TODO comments are more than six months old. Delete the six files. Resolve or delete the TODOs.

---

## Security posture

42 entry points: 38 HTTP routes, 1 webhook, 3 queue consumers.

- Public by design: 5.
- Missing authentication: 0.
- Authenticated with no object scope: 7. See S-01.

| Domain | Result |
|---|---|
| 1. Authentication and sessions | S-02 |
| 2. Authorization | S-01 |
| 3. Input handling and injection | S-03. Queries are parameterized throughout. |
| 4. Secrets and cryptography | Checked, no findings. Git history searched. |
| 5. Data protection and privacy | Checked, no findings. Personal data is limited to name and email. |
| 6. Dependencies and supply chain | Checked, no findings. Two moderate advisories in development dependencies, not reachable at runtime. |
| 7. Configuration and infrastructure | Dockerfile, compose file, and CI workflow checked, no findings. Production infrastructure not checked: it is not defined yet. |
| 8. Abuse and business logic | C-01 (a retry charges twice). |
| 9. Logging, detection, and response | C-02 (security events are not logged). |
| 10. AI and LLM integrations | Not applicable. No LLM integration. |
| 11. Client and mobile | Checked, no findings. No secrets in the client bundle. |

This was a static review. It does not replace a penetration test. Full detail in `security-review.md`.

---

## Feature status

14 features in the register: COMPLETE 7, PARTIAL 5, STUB 1, BROKEN 1.

| Feature | Team said | Status | Finding |
|---|---|---|---|
| Projects | done | PARTIAL: permission hop | S-01 |
| Checkout | done | PARTIAL: failure paths | C-01 |
| File upload | done | PARTIAL: server-side validation | S-03 |
| CSV import | done | PARTIAL: failure path | X-01 |
| Team invitations | in progress | PARTIAL: no revoke, no resend | F-02 |
| Data export | done | BROKEN | F-01 |
| Usage analytics | not started | STUB | F-03 |

Journeys walked:

- **New user:** passes.
- **Access:** passes. Login has no attempt limit (S-02).
- **Money:** fails. A failed order insert leaves a charge with no order (C-01).
- **Team:** invite and accept pass. No way to revoke (F-02).
- **Leaving:** export fails (F-01). Account deletion is out of scope for this stage, per the team.

Full matrix in `feature-matrix.md`.

---

## Internal clashes

| Class | Result |
|---|---|
| 1. Source of truth | Checked, no findings. |
| 2. Rules | X-01 |
| 3. Contracts | F-01 (the frontend calls a route the backend does not serve). |
| 4. Schema | Checked, no findings. Models match migrations. |
| 5. Configuration | Checked, no findings. |
| 6. Dependencies | X-02 |
| 7. Routing and naming | Checked, no findings. |
| 8. Ordering | Checked, no findings. |
| 9. Concurrency and ownership | One candidate dismissed. See below. |
| 10. Security controls | S-01 (the list endpoint is tenant-scoped, the single-record endpoints are not). |
| 11. Conventions | Checked, no findings. |
| 12. Coexistence | X-02 |
| 13. Records | X-03 |

Full register in `clash-register.md`.

---

## Challenge round results

Three BLOCKERs and the two MAJORs with the widest blast radius. Full detail in `challenge-rounds.md`.

| Finding | Picked version | Reason |
|---|---|---|
| S-01 | Challenger's shared tenant-scoped query helper | Per-handler checks, the Defender's option, leave the next new query unprotected |
| S-02 | Challenger's per-account and per-IP limits | The edge limit the Defender relied on is not configured |
| C-01 | Challenger's pending-order pattern | Removes the state where money is taken and no order exists |
| S-03 | Challenger's server-side validation | Virus scanning is a separate decision, logged as an open question |
| C-03 | Challenger's single query with a window function | 101 queries become 1 |

---

## Commendations

- COMMEND: `domain/` keeps business rules free of HTTP and database imports, which is why the rules were quick to trace. New services should follow it.

---

## Investigated and dismissed

- **Suspected SQL injection in `services/search.js:14`.** The ORDER BY clause is built from a string. Dismissed: the column name is picked from a fixed allowlist at line 9.
- **Suspected lost update on `subscriptions`.** No locking on the table. Dismissed: the billing webhook worker is the only writer and runs with a concurrency of 1. Re-check if a second writer is added.
- **Suspected unauthenticated webhook at `/webhooks/stripe`.** Public by design. The signature is verified at `api/routes/webhooks.js:12` before the body is used.

---

## Suppressed by configuration

None. Default thresholds were used.

---

## Recommended action plan

### Before the next deployment

1. Scope every project query by tenant (S-01).
2. Add attempt limits and failed-login logging (S-02).
3. Fix the checkout order of operations and add idempotency (C-01).

### Before the pre-ship gate

4. Validate uploads on the server (S-03).
5. Register the export route, or hide the button (F-01).
6. Set one project name limit (X-01).
7. Replace `console.log` with structured logging and trace IDs (C-02).
8. Rewrite the dashboard query and add the two indexes (C-03).
9. Add revoke and resend to invitations (F-02).
10. Add idempotency keys to the three workers (question 7).
11. Run a load test at 3x expected traffic and define alert thresholds (questions 1, 3, and 10).

### Can defer

12. Finish the move to `dayjs` (X-02).
13. Add the function length lint rule (C-04).
14. Update the README Node version (X-03), delete unused helpers (C-06).

---

## Open questions

1. **What is the expected peak traffic at launch?** The team estimated "a few thousand users in the first month". A number is needed to scope the load test.
2. **Which compliance regime applies?** The checkout uses Stripe-hosted card fields (`web/src/checkout/PaymentForm.jsx:14`), which narrows PCI-DSS scope. Confirm with a compliance advisor before launch.
3. **Will rate limiting be configured at the edge, and with what limits?** Affects the fix for S-02.
4. **Should uploads be virus-scanned before launch?** Affects S-03.
5. **What does the ticket board promise that the README does not?** The board could not reach it. Features claimed only there were not checked.

Full list in `unanswered-questions.md`.

---

*This report was produced by the Senior Board Audit skill, developed by Augmex Technologies Limited. The four-agent board reviewed every component listed and reached the verdicts above through structured adversarial debate. If you disagree with any finding, run the audit again with new context and the board will reconsider.*

*This report describes weaknesses in the system. Treat `docs/audit/` as confidential.*

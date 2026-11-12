# The clash pass (Phase 4C)

A clash is two or more parts of the system that each look fine alone and disagree with each other. The component review reads one part at a time, so it cannot see them. The clash pass puts the parts side by side.

A duplicate is not yet a clash. Two copies of a rule that agree are a maintenance cost, filed as duplication in Phase 3. Two copies that disagree are a clash, because some input gets a different answer depending on which copy handles it.

---

## Method

For each class below:

1. **Collect.** Find every place that defines, stores, or enforces the thing.
2. **Compare.** Put them side by side. Diff the values, the rules, the shapes, the order.
3. **Find the meeting point.** Name the input, the record, or the sequence of events on which the two sides give different results.
4. **Follow the consequence.** What happens at the meeting point: a rejected request, a wrong number, a corrupted row, a bypassed check, a failed deploy.

A clash finding needs both sides cited and the meeting point named. No meeting point, no clash.

Start from the leads the earlier phases produced: duplicates from Phase 3, entry points with different checks from the security pass, validation differences and BROKEN features from the feature pass, and the user's answers about half-finished migrations and duplicated rules. Then work every class, because the worst clashes are the ones nobody suspected.

---

## The clash classes

### 1. Source-of-truth clashes

The same fact stored or computed in more than one place.

- A total stored on the order and also derivable from its line items, updated by different code.
- A user's plan held in the database, in a token claim, and at the billing provider, with no stated winner.
- A counter column next to the rows it counts.
- Cached copies with different invalidation rules in different callers.
- A constant defined twice with different values.

### 2. Rule clashes

The same business rule implemented more than once.

- Validation: the client allows 50 characters, the API allows 255, the column is `varchar(100)`.
- Price, tax, discount, or proration calculated in the frontend, the backend, and an invoice job, with different rounding.
- Permissions: one module checks `role == 'admin'`, another reads a permissions table.
- Status transitions: one module allows `cancelled` to become `active`, another treats cancelled as final.
- A password policy that differs between signup, reset, and change.
- A rule enforced on create and missing on update, import, or bulk paths.

### 3. Contract clashes

Two sides of an interface that disagree about it.

- Frontend and backend: path, method, field names, casing, types, nullability, enum values, date format, pagination shape, error shape.
- Service to service: one side changed a payload and the other did not.
- Code against its published schema: OpenAPI, GraphQL schema, protobuf, JSON schema.
- Event producers and consumers: one event name, different payloads.
- Declared types that the runtime value does not match: casts, `any`, unchecked JSON.
- Webhook handlers written for an older version of a provider's payload.

### 4. Schema clashes

- ORM models against migrations, and both against the live schema when database access was granted.
- Columns the code reads that no migration creates. Columns migrations create that nothing reads.
- Enum values in code against a CHECK constraint or lookup table.
- Nullability: the model says required and the column allows NULL, or the reverse.
- Delete behaviour: soft delete in the application, `ON DELETE CASCADE` in the database.
- One kind of value typed differently across tables: integer and UUID identifiers, float and decimal money, text and timestamp dates.
- Migration history: duplicate sequence numbers, multiple heads, migrations edited after they ran.
- Seed data and fixtures that violate current constraints.

### 5. Configuration clashes

- An environment variable with different defaults in the config module, `.env.example`, the compose file, the deployment manifest, and CI.
- One variable name used with two meanings. Variables the code reads that no environment defines.
- Runtime versions that differ between the Dockerfile, CI, version files, and the package manifest.
- Ports and hostnames that differ between the application, the proxy, and the health check.
- Timeout chains in the wrong order: a client that gives up before the server does, a load balancer idle timeout shorter than the application's keep-alive, a queue visibility timeout shorter than the job's run time.
- Limit chains in the wrong order: a proxy body limit below the upload size the application advertises.
- Lifetimes that cross: a cache TTL longer than the permission it caches, a session that outlives the token behind it.
- One header or policy set in both the proxy and the application with different values: CORS, CSP, cookie flags, rate limits.
- Development and production config that differ in ways nobody has listed.

### 6. Dependency clashes

- Two libraries doing one job: two HTTP clients, two date libraries, two ORMs, two validation libraries, two state managers, two test runners.
- Several major versions of one library in the dependency tree.
- More than one lockfile, or more than one package manager, in one project.
- Unmet or conflicting peer dependencies.
- Packages imported and not declared. Packages declared and never imported.
- A manifest range the lockfile no longer satisfies.

### 7. Routing and naming clashes

- One method and path registered twice. One handler is dead, and it is not always the older one.
- A route shadowed by order: `/users/:id` registered before `/users/me`.
- Two functions, classes, or modules with one name and different behaviour.
- One event name, queue name, or job name used for different payloads.
- Cache keys that collide across users or tenants because the key leaves the tenant out. This is also a security finding.
- Global names that collide: CSS classes, global variables, environment variables, table prefixes.

### 8. Ordering clashes

- Middleware order: authentication after the handler it protects, a rate limiter after the expensive work, a body size limit after the body was parsed, an error handler that is not last.
- Hooks, signals, observers, and database triggers that fire in an order the code does not expect.
- Two listeners on one event that write the same record.
- Scheduled jobs that overlap: a job on a five-minute schedule that runs for eight, or two jobs that update the same rows.
- Migrations that need code not yet deployed. Code that needs a migration not yet run.
- Initialisation order: a module that reads config before config is loaded.

### 9. Concurrency and ownership clashes

- Two writers to one field with different locking strategies.
- One path that uses a transaction and another that performs the same update without one.
- An optimistic lock version column that a bulk update ignores.
- A cache updated by one write path and never invalidated by another.
- Two services that both act as the owner of one table.

### 10. Security control clashes

A control that exists on one path and is absent on another path to the same thing.

- Session-authenticated routes enforce role checks. API-key routes to the same operation skip them.
- A REST endpoint with a check, and a GraphQL resolver, export job, or older API version without it.
- The UI hides an action the API still allows. The UI offers an action the API refuses.
- An allowlist in one layer and a broader one in another.
- Role names that differ across modules: `admin`, `administrator`, `superuser`.
- Input sanitized on the web form and stored raw from the import or the mobile API.

Every finding in this class is also a security finding. File it once, under the lens that explains it best, and cross-reference the other.

### 11. Convention clashes

- Errors: some functions throw, some return null, some return a result object.
- Time: UTC and local time. Seconds and milliseconds. ISO strings and epoch numbers.
- Money: cents and decimals. Floats.
- Naming: camelCase and snake_case in one API.
- Absence: null, empty string, and missing key treated as one thing in one place and as three things in another.
- Pagination: offset in some endpoints, cursor in others.

A convention clash does harm at the point where a value crosses from one convention to the other. Find that crossing. With a crossing, rate the finding by the consequence. Without one, it is a NOTE.

### 12. Coexistence clashes

Old and new alive at the same time.

- Version 1 and version 2 of a module, table, or API, each with live callers.
- A half-finished migration: one path still writes the old column, another reads the new one.
- Dual writes with nothing reconciling the two stores.
- Feature flags that have been fully on or fully off for months with the dead branch still in the code.
- A deprecated path that still receives traffic.

### 13. Record clashes

What the project says about itself against what it does.

- README or setup steps that name commands, variables, or files that do not exist.
- Comments and docstrings that describe behaviour the code no longer has.
- Tests that assert old behaviour and are skipped. Mocks that return a shape the real dependency never returns.
- API documentation that differs from the handlers.
- The team's answers in the question bank against the code.

---

## Debate each candidate

**The Challenger cites side A and side B with locations, then names the meeting point:** the input or the sequence on which the two diverge.

**The Defender argues that the two sides never meet,** because they serve different contexts or different data, or that the difference is deliberate, and cites where that is recorded. If one side turns out to be dead code, the finding is a DELETE in the component review and the clash is dismissed.

**The Judge rules.** CONFIRMED or DISMISSED. For a confirmed clash the Judge names the authoritative side, issues the verdict on the side that must change, and rates severity by the consequence at the meeting point.

- The Judge always names a winner. "Align the two" without saying which one moves is not a ruling.
- When the code cannot settle which side is right because the answer is a product decision, the Judge says so, files the clash at its severity, and puts the decision in open questions with both options stated.
- A dismissed candidate goes to "Investigated and dismissed" with the reason the two sides never meet.

### A worked clash finding

> **X-02. Payout job outlives its queue timeout.** REFACTOR `infra/queue.tf` and `workers/payout.py` | BLOCKER | TRACED
>
> **Challenger:** side A, `infra/queue.tf:12` sets the visibility timeout to 30 seconds. Side B, `workers/payout.py:40` calls the bank API with a 60-second timeout and retries once. Meeting point: any payout that takes longer than 30 seconds. The message becomes visible again and a second worker picks it up while the first is still running. The job has no idempotency key. The payout is sent twice.
>
> **Defender:** the median payout takes four seconds, and the 30-second default has been in place since the queue was created. Nobody has reported a duplicate.
>
> **Judge:** CONFIRMED. The authoritative side is the job's worst-case run time, since the bank's latency is not under the team's control. The median is the wrong number to defend with: the clash fires on the slow day, which is the day the bank is already having trouble. Required fix: (1) set the visibility timeout above the job's worst case, including the retry, (2) pass an idempotency key on the payout call so a redelivery is harmless, (3) alert on messages received more than once.

Each file is defensible alone. The defect exists only between them.

---

## Severity guide

The stage calibration table in SKILL.md still applies. Use this guide to place a clash finding inside it.

**BLOCKER**

- The clash corrupts or loses data.
- It charges, pays, or reports a wrong amount of money.
- It lets a request around a security control.
- It makes the release undeployable (Stage 2).

**MAJOR**

- Wrong behaviour a user sees: valid input rejected, invalid input accepted, a different answer on two screens.
- A failed migration or deploy.
- Duplicate processing with no money or data impact.

**MINOR**

- A maintenance cost with no wrong behaviour today: two libraries for one job, stale flags, version drift in tooling.

**NOTE**

- Convention differences with no crossing. Record clashes that mislead nobody about how to run or use the system.

---

## What goes in `clash-register.md`

1. **Classes checked.** For each of the thirteen classes, one of: finding IDs, "checked, no findings", "not applicable" with the reason, or "not checked" with the reason.
2. **The register.**

   | ID | Class | Side A | Side B | Meeting point | Consequence | Authoritative side | Severity |
   |---|---|---|---|---|---|---|---|
   | X-02 | Configuration | `infra/queue.tf:12` 30s visibility | `workers/payout.py:40` up to 120s run | payout slower than 30s | payout sent twice | job run time | BLOCKER |

3. **Findings in full.**
4. **Investigated and dismissed.**

---

## Stage 0

With no code, the pass looks for contradictions inside the plan: requirement against requirement, requirement against architecture, scale target against budget, compliance duty against data design, team size against operational load. The meeting point is the scenario in which both cannot hold. The Judge names which requirement gives way, or puts the decision in open questions with both options stated.

# Stage question banks

Read only the section for the confirmed stage. The rules for running a question bank (answer from the repository first, N/A needs a reason, "I don't know" is a finding) are in SKILL.md under "The question bank".

Each bank ends with questions on security, features, and consistency. Those answers are claims. The security, feature, and clash passes test them against the code.

---

## Stage 0. Pre-coding questions (The Blueprint)

### Requirements and scale

1. What is the expected user base in 6 months and 2 years? Estimate DAU and MAU.
2. What is the estimated read-to-write ratio? Examples: 10:1 (read-heavy, e.g. a content site), 1:10 (write-heavy, e.g. a logging system), 1:1 (balanced).
3. What are the target latency metrics? Examples: p95 under 200ms for API responses, p99 under 1s for AI generation.
4. How much data growth is expected in 1, 3, and 5 years? Rows per table, bytes per record, total storage.

### Architectural trade-offs

5. Does this system favour consistency or availability? Reference the CAP theorem. If unsure, ask: "If the database is unreachable for 30 seconds, should the system return stale data or return an error?"
6. Monolith or microservices? Justify the choice for the team size and operational maturity.
7. Which data model fits best: relational (PostgreSQL, MySQL), document (MongoDB), key-value (Redis), wide-column (Cassandra), or a mix? Why?
8. Should operations be synchronous or asynchronous? Where is the queue? What is the worker pattern?

### Security and compliance

9. What compliance regulations apply? GDPR, HIPAA, PCI-DSS, SOC 2, ISO 27001, or none?
10. How will authentication and authorization be handled? Session-based, token-based, OAuth, SSO? Where do roles and permissions live?
11. Where is data encryption needed: at rest, in transit, or both? Which fields need column-level encryption?
12. What data will the system hold, and which of it is sensitive? Personal data, credentials, payment data, health data, customer secrets. Where will each class live?
13. Who are the actors, and what is each allowed to do? Anonymous visitor, user, tenant admin, staff, internal service, third-party integration. Single-tenant or multi-tenant?

### Scope and features

14. What is the feature list for the first release? Which features are explicitly out of scope?
15. What does "done" mean for each feature? Name the acceptance criteria, or the user journey that must work end to end.
16. What is the one journey the product cannot launch without?

### Consistency of the plan

17. Which requirements pull against each other? Examples: real-time updates and a nightly batch, right to erasure and an immutable audit log, a two-person team and six services, a latency target and a synchronous third-party call.
18. Which decisions are already fixed, and which are still open? Who decides the open ones?

---

## Stage 1. Mid-project questions (The Reality Check)

### Performance and bottlenecks

1. Where is the current data bottleneck: CPU, memory, disk I/O, or network? If unknown, that is the answer.
2. What data is being cached, and what is the eviction policy? TTL, LRU, manual invalidation?
3. How does the system behave under a 10x traffic spike? Has anyone tested this? With what tool?
4. Are database queries using optimal indexes? Run EXPLAIN on the top 5 slowest queries.

### Resilience and edge cases

5. What happens if a downstream third-party API (Stripe, OpenAI, SendGrid, anything) goes down or slows to 30 seconds per request? Where is the timeout? Where is the circuit breaker?
6. Do we have circuit breakers and rate limiters in place? At which boundaries? What are the thresholds?
7. Are asynchronous worker processes idempotent? If a message is retried, will it create duplicate state?
8. How do we handle race conditions during concurrent writes? Optimistic locking, pessimistic locking, database transactions, distributed locks?

### Observability and operations

9. Do logs carry trace IDs across services? Can we follow a single request from the frontend through every backend hop?
10. What alerting thresholds indicate a critical failure? p99 latency, error rate, queue depth, disk usage? Who gets paged?
11. How easily can we roll back a buggy deployment? Minutes from "we have a problem" to "we are on the last stable version"?

### Security

12. Where is authentication enforced: global middleware, per-route decorators, or inline checks? Which routes are intentionally public?
13. How is a user stopped from reading or changing another user's or another tenant's records? Where in the code does that check live?
14. Where do secrets live in development, in CI, and in deployed environments? Has a secret ever been committed to this repository?
15. When did someone last run a dependency vulnerability scan? What was left unresolved?

### Features and scope

16. What is the source of truth for the feature list: a spec, a ticket board, the README?
17. Which features does the team consider done, in progress, and not started?
18. Which features are behind flags? Who decides when a flag is removed?

### Consistency

19. Is any migration half-finished: an old and a new version of a module, table, API, or library both in use?
20. Which rules live in more than one place: validation, pricing, permissions, status transitions? Which copy is authoritative?

---

## Stage 2. Pre-ship questions (Launch Readiness)

### Deployment and rollback strategy

1. Can we deploy progressively? Canary deployments, feature flags, blue-green deployment? What percentage of traffic hits the new version first?
2. What is the exact rollback plan? If the system crashes at launch + 5 minutes, how many minutes to revert to the previous stable version? Has anyone timed this?
3. Is the rollback backward-compatible? Will a rolled-back codebase break the newly modified database schema? What is the migration strategy?

### Infrastructure and scale under load

4. Did we pass load testing? At 2x and 3x expected peak traffic, what failed first?
5. Are auto-scaling rules live and tested? Will new instances spin up automatically on CPU or memory thresholds? Has someone simulated a spike to confirm?
6. Have we set up rate limiting? Per IP, per user, per endpoint? What are the limits? What does the user see when they hit the limit?

### Data integrity and migration

7. Is the data migration verified? Have we run the production migration script on a copy of real production data in staging?
8. Do we have a pre-launch backup? Confirmed isolated snapshot of every production database, with a tested restore procedure?
9. How do we handle split-brain data? If we roll back, what happens to user data written during the brief window the new version was live? Do we replay, discard, or merge?

### Observability and incident response

10. Are the dashboards working right now? Can we see real-time error rates, latencies, server health?
11. Are alert thresholds correctly tuned? Will the team get paged for a p99 spike or a 1% error rate? Test the alert before launch.
12. Who is on call? Is there a clear rotation schedule for the first 48 hours post-launch? Is the contact list up to date?

### Security and compliance

13. Are all secrets rotated? Production API keys, database passwords, JWT signing keys, TLS certificates: all moved out of code and into a vault?
14. Did we fix critical vulnerabilities? All high-severity items from the latest dependency scan and penetration test resolved or accepted with documented mitigations?
15. Are compliance audits complete? Do logging systems strictly exclude PII, credit card numbers, passwords, and tokens? Has someone grep'd the logs to confirm?
16. Has anyone outside the build team tested the system: a penetration test, a security review, a bug bash? What was found, and what is still open?
17. When a vulnerability is reported after launch, who receives the report and how fast can a fix ship?

### Feature readiness

18. Which features are in launch scope? Which are deferred, and are the deferred ones hidden or disabled in the build that ships?
19. Has a person walked every launch-scope journey end to end on the release candidate, including the failure paths: declined card, expired link, lost connection?
20. Which feature flags will be on at launch? What is each one set to in production config today?

### Consistency

21. Do staging and production run the same versions, config keys, and migrations? List the known differences.
22. Is any old code path, table, or API version still live next to its replacement? What is the plan to remove it?

---

## Stage 3. Production questions (The Live System)

### Incidents and reliability

1. What were the last three incidents? For each: root cause, time to detect, time to recover, and whether the fix addressed the cause or the symptom.
2. What are the SLOs, and were they met over the last 90 days? If there are no SLOs, that is the answer.
3. Which alerts fired most in the last 30 days? Which of them led to an action?
4. When was a backup last restored as a test? How long did it take?

### Security operations

5. When were production secrets last rotated? Who has production access today, and when was that list last reviewed?
6. How fast does a critical dependency patch reach production? Name the last one and how long it took.
7. Are security events logged and reviewed: failed logins, permission denials, role changes, data exports? Who looks at them, and how often?
8. Has there been a breach, a leaked credential, or a reported vulnerability? What changed afterwards?

### Features in use

9. Which features do users use, by the numbers? Which shipped features have close to zero usage?
10. Which features are deprecated or scheduled for removal? Is the old code still reachable?
11. What do users report as broken most often? Where are those reports tracked?

### Debt and continuity

12. Which part of the system does the team avoid touching? Why?
13. Which components are understood by one person only?
14. Which runtime, framework, or database version in production reaches end of support within 12 months?
15. What does the system cost per month, and which line item grew fastest in the last quarter?

### Consistency

16. Where does production differ from what is in the repository: hotfixes applied by hand, config edited on the server, schema changed outside migrations?
17. Which old versions are still served next to their replacements: API versions, tables, queues, clients that cannot be forced to upgrade?

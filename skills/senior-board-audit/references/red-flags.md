# The red-flag catalogue

The Challenger uses this catalogue during the component review (Phase 3). Read it before the first component.

Size limits below are the defaults. Where the user configured different thresholds, the configured values apply.

A red flag is a reason to look closer. It becomes a finding when the Challenger can show the harm at a specific location.

The security, feature, and clash passes have their own checklists in `security-pass.md`, `feature-pass.md`, and `clash-pass.md`. The security list here is the quick screen for a single component. The security pass is the full review.

---

## Code structure red flags

- Functions over the function threshold (default 50 lines)
- Files over the file threshold (default 500 lines)
- Classes over the class threshold (default 300 lines)
- Cyclomatic complexity over the threshold (default 10) in any function
- Nesting deeper than the threshold (default 4 levels)
- Functions with more parameters than the threshold (default 5)
- Public methods with no docstring or type hints
- Magic numbers and strings without named constants
- Commented-out code older than 30 days
- TODO comments older than 90 days

## Duplication red flags

- The same business logic in two different files
- Near-duplicate functions with cosmetic variations
- Copy-pasted route handlers with two-line differences
- Repeated database queries that should be a single join
- Multiple definitions of the same constant
- Logic that exists in both the frontend and backend without a shared source

Duplicates that disagree with each other are clashes. Send them to the clash pass.

## State and data red flags

- The same field stored in two places without one marked authoritative
- Tenant data not scoped at the query layer
- Soft deletes mixed with hard deletes in the same table
- Foreign keys that allow orphaned records
- Nullable columns that should not be
- Indexes missing on foreign keys
- Indexes missing on columns used in WHERE clauses
- Composite indexes in the wrong column order
- Tables over 100GB without partitioning strategy
- JSON columns being filtered by inner field without a generated column index
- Money stored as floating point
- Timestamps stored without a time zone, or in local time

## Security red flags (quick screen)

- SQL queries built with string concatenation
- User input passed to `eval`, `exec`, or shell commands
- Authentication checks at the route level but not the function level
- Authorization based on client-supplied claims
- Records loaded by ID with no owner or tenant constraint
- Hardcoded API keys, database URLs, or secrets
- Logging of full request bodies (PII leak)
- Logging of full response bodies (secret leak)
- CORS set to `*` in production
- Missing CSRF protection on state-changing endpoints
- File uploads without type validation
- File uploads without size limits
- Weak crypto (MD5, SHA-1 for security, ECB mode, DES)
- JWT tokens with no expiration
- JWT tokens with `alg: none` allowed
- Open redirects (unvalidated `redirect_to` parameters)
- Server-side request forgery (SSRF) via user-supplied URLs
- HTML rendered from user input without escaping
- TLS certificate verification switched off

## Performance red flags

- N+1 queries
- Loops that call external APIs sequentially when parallel is possible
- Synchronous LLM calls in HTTP request handlers
- Caching with no eviction policy
- Caching with no TTL
- Caching that never invalidates on writes
- Database connections not pooled
- Database connections leaked
- Background jobs that run on the request thread
- File I/O on the request thread
- Unbounded queries (no LIMIT)
- Unbounded loops (no maximum iteration count)

## Resilience red flags

- External calls with no timeout
- External calls with no retry
- External calls with infinite retry
- Retry without exponential backoff
- Retry without jitter
- No circuit breaker on third-party APIs
- No fallback when LLM provider fails
- Background workers without idempotency keys
- Webhook handlers without signature verification
- Webhook handlers that mutate state without deduplication
- Multi-step writes with no transaction and no compensation when a later step fails

## Observability red flags

- Errors caught and silently swallowed
- Errors caught and logged as info or debug
- Logs with no structure (plain print statements)
- Logs without trace IDs in a multi-service system
- Logs without user or tenant context
- Metrics that nobody on the team can name from memory
- Dashboards that nobody has opened in 30 days
- Alerts that have never fired
- Alerts that fire daily and are ignored

## Testing red flags

- No tests on authentication, authorization, or money paths
- Tests that assert nothing, or only that no exception was thrown
- Tests that mock the unit they claim to test
- Skipped or disabled tests with no linked reason
- Tests that pass or fail depending on run order, the clock, or the network
- Fixtures that no longer match the schema
- Coverage numbers held up by trivial code while the critical path is untested

## Frontend red flags (when a UI exists)

- Permission checks that exist only in the UI
- Secrets or private keys in the client bundle
- Tokens stored where page scripts can read them
- Raw HTML injection from user or API content
- Actions with no loading, error, or empty state
- Forms that can be submitted twice
- Server state copied into a local store and never refreshed
- Lists that load every record with no pagination

## Infrastructure and deployment red flags (when config is in the repository)

- Container images on `latest` or unpinned tags
- Containers running as root
- Secrets in Dockerfiles, compose files, CI variables printed to logs, or IaC
- No health checks or readiness probes
- No resource limits
- Migrations that lock large tables or cannot be reversed
- Deploy steps that exist only in someone's memory
- A stateful service running as a single instance with no restore procedure
- CI that deploys without running the tests

## AI-specific red flags (for products with LLM integrations)

- LLM calls with no cost cap per call
- LLM calls with no daily ceiling per tenant
- LLM output written to a database without schema validation
- LLM output executed as code or SQL without a whitelist
- LLM tool calls without a tool registry
- Hardcoded model names in business logic
- No fallback chain when the primary LLM provider fails
- Prompts that include unsanitised user input (prompt injection risk)
- Retrieved documents, web pages, or tool results placed in prompts as trusted text
- System prompts stored in code rather than versioned config
- No logging of LLM input or output (debugging impossible)
- No replay capability for past LLM interactions
- Agent loops with no iteration limit

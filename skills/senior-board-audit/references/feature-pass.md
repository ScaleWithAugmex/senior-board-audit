# The feature pass (Phase 4B)

The component review can find every file sound while the product still does not do what it says. The feature pass audits the system from the outside in: one feature at a time, traced from the place a user starts to the place the data lands, and back.

It answers three questions. What does the system claim to do? What does it do? Where do those two lists differ?

---

## Step 1. Build the feature register

A feature is something a user or an operator can do with the system. Collect every claim that a feature exists, from every source available:

- The user's answers in the question bank: scope, and what the team considers done, in progress, and not started.
- README, docs, changelog, and any marketing or landing copy in the repository.
- Specs, PRDs, and tickets, when they are in the repository or the user supplies them.
- Navigation: menus, routes, screens, buttons, CLI commands, API endpoints.
- Data: a table or collection implies a feature that writes it and a feature that reads it.
- Feature flags and config toggles.
- Test names and fixtures.

Merge them into one list. For each feature record its name, who claims it (the source), the actors who use it, its entry points, and whether the user placed it in scope for the current stage.

Work in both directions. Claim to code finds what is promised and absent. Code to claim finds what exists and nobody mentioned.

---

## Step 2. Trace each feature end to end

Follow the feature through every hop it needs. Cite `path:line` at each one.

1. **Entry.** The screen, command, endpoint, or schedule that starts it.
2. **Input.** Validation on the client and on the server.
3. **Permission.** Who is allowed, and where that is checked.
4. **Logic.** The service or function that does the work.
5. **Persistence.** What is written, to which tables, inside which transaction.
6. **Side effects.** Emails, notifications, webhooks, jobs, cache updates, events, third-party calls.
7. **Response.** What the user sees on success.
8. **Failure.** What the user sees, and what state is left behind, when each step fails.
9. **Reverse.** How the action is edited, undone, cancelled, or deleted.

A hop that cannot be found after a real search is a finding. Record what was searched: "no handler registered for `/api/export` in `routes/`, `app.js`, or any router file".

Check that hops connect. The frontend calls the path the backend serves, with the method and field names the backend expects. The handler calls a function that exists. The query names columns the schema has. A feature where every hop exists and two of them do not fit together is BROKEN.

---

## Step 3. Assign a status

| Status | Meaning |
|---|---|
| COMPLETE | Every hop exists and connects. Failure paths are handled. Tests cover the main path and the failures that matter. |
| PARTIAL | The main path works. Named hops are missing or weak. |
| STUB | A placeholder stands where the feature should be: hardcoded or mock data, `TODO`, `pass`, "not implemented", a button with no handler, an endpoint that returns a fixed response. |
| BROKEN | The pieces exist and cannot work as written: a call to an endpoint, function, column, or field that does not exist, a payload the other side rejects, an import that does not resolve. |
| UNREACHABLE | Implemented, and nothing leads to it: no route, no link, a flag that is off everywhere, a handler never registered. |
| UNDOCUMENTED | Exists and is reachable, and no claim source mentions it. Includes debug endpoints, hidden admin tools, and leftover experiments. |
| MISSING | Claimed, and no implementation was found. |

Evidence levels apply. COMPLETE and BROKEN are strong claims. Without a verification run they are TRACED at best, and the matrix says so.

---

## Step 4. Run the completeness checks

For each feature that is PARTIAL or COMPLETE, check:

- **Symmetry.** Create has a matching read, update, and delete where the domain needs them. Invite has revoke. Subscribe has cancel. Upload has remove. Enable has disable.
- **States.** Loading, empty, error, and success are all handled in the UI. Long lists paginate.
- **Validation.** The server enforces every rule the client enforces. Differences go to the clash pass.
- **Permissions.** Every action in the feature has its own check, including the less visible ones: export, bulk actions, API access to a UI feature. Gaps go to the security pass.
- **Double submission.** Submitting twice, refreshing mid-flow, or two users acting at once does not duplicate or corrupt.
- **Dependents.** Deleting or disabling a record has a defined effect on everything that references it.
- **Limits.** Quotas and plan limits are enforced on the server.
- **Time and locale.** Time zones, currencies, and formats behave for users outside the developer's own.
- **Feedback.** The user is told what happened. Background work reports its result somewhere the user can see.
- **Operations.** Support and admins can see and correct the feature's state without direct database access.
- **Tests.** The main path and the failure paths that matter are covered.
- **Docs.** What the documentation says matches what the code does.

---

## Step 5. Walk the journeys

Each feature can pass while the journey across them fails. Walk the journeys that matter for this product, start to finish:

- **New user.** Sign up, verify, first meaningful action.
- **Access.** Log in, log out, reset password, change email, enable and recover MFA.
- **Money.** Start trial, pay, upgrade, downgrade, fail a payment, cancel, refund.
- **Team.** Invite, accept, change role, remove.
- **Leaving.** Export data, delete account, and what remains afterwards.
- **The core loop.** The one journey the user named in the question bank as the reason the product exists.

Skip journeys the product does not have. Record them as not applicable.

Flag dead ends: a state a user can enter and cannot leave or finish. Examples: an unverified account with no way to resend the email, a failed payment with no retry, an invitation sent to an address that already has an account.

---

## Step 6. Debate each candidate

**The Challenger names the broken or missing hop and the user who hits it.**

> A user who clicks Export gets an error. `ExportButton.tsx:22` posts to `/api/export`. No route with that path is registered in `routes/`, `app.js`, or any router file.

**The Defender argues scope and intent.** The feature is deferred on purpose, hidden behind a flag that is off in the shipping config, or was never promised. The Defender cites where that decision is recorded.

**The Judge assigns the status, the verdict on the affected component, and the severity.**

- A deferral counts when the unfinished feature is unreachable in the build that ships. A stub a user can reach is a finding at Stage 2 whatever the intent.
- "Never promised" fails when any claim source in the repository promises it.
- A dismissed candidate goes to "Investigated and dismissed".

### A worked feature finding

> **F-04. Data export.** Status BROKEN | REFACTOR `routes/index.js` | MAJOR at Stage 1 | TRACED
>
> **Challenger:** the README lists "Export your data as CSV" under features. The button exists and posts to `/api/export`. No such route is registered. `services/export.js` has a complete `buildCsv` function that nothing calls. Every click fails.
>
> **Defender:** the service was written in the last sprint and the route is the next ticket. Nobody has shipped this to users. The work is two lines of routing.
>
> **Judge:** CONFIRMED as BROKEN. The Defender is right that the gap is small, and small gaps are how a dead button reaches production. Required fix: (1) register the route and connect it to `buildCsv`, (2) add the permission check the other project routes use, (3) hide the button behind a flag until both are merged. At Stage 2 this is a BLOCKER, because the README promises it.

---

## Severity guide

The stage calibration table in SKILL.md still applies. Use this guide to place a feature finding inside it.

**BLOCKER**

- A feature in scope for the current stage gate is BROKEN, STUB, or MISSING (Stage 2 and 3).
- Any feature loses, corrupts, or exposes user data on a path a user can reach.
- A dead end on a money journey or an access journey.

**MAJOR**

- A core feature is PARTIAL with a gap on a path users will hit: no error handling, no way to undo, no server-side validation.
- A BROKEN feature mid-project (Stage 1).
- An UNDOCUMENTED feature that changes data or permissions.
- Documentation or marketing copy promises a capability the system lacks.

**MINOR**

- Missing empty or loading states. Missing pagination on lists that are small today.
- UNREACHABLE code that is safe and dead. The verdict is usually DELETE.

**NOTE**

- STUB or MISSING features that are planned and out of scope for the current stage. They are listed so nothing ships as a stub by accident.

---

## What goes in `feature-matrix.md`

1. **Sources.** Which claim sources were used, and which were unavailable.
2. **The matrix.**

   | Feature | Claimed by | In scope | Status | Evidence | Broken or missing hop | Finding |
   |---|---|---|---|---|---|---|
   | Data export | README, UI button | yes | BROKEN | TRACED | no route for `/api/export` | F-04 |

3. **Journeys walked,** with the result for each.
4. **Claims with no code. Code with no claim.**
5. **Findings in full.**
6. **Investigated and dismissed.**

Every feature in the register gets a row, including the COMPLETE ones. The matrix is how the team sees what was checked.

---

## Stage 0

With nothing built, the pass audits scope. Build the register from the spec and the user's answers. For each feature the Challenger asks: is there an acceptance test a person could run, is every hop in the trace accounted for in the design, and which features does the first-release journey need that the list leaves out? Findings are scope gaps: a feature with no acceptance criteria, a journey with a missing step, a first release that cannot be used end to end.

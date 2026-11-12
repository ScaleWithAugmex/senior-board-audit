# Report template and deliverable formats

Read this before Phase 6. Every file goes in `docs/audit/{YYYY-MM-DD}/`.

`report.md` is the deliverable. A reader should be able to stop after it and know the ruling, the reasons, and what to do first. The other seven files are the evidence.

---

## `report.md`

```markdown
# Senior Board Audit Report

Project: {name}
Date: {YYYY-MM-DD}
Stage: {0 Pre-coding | 1 Mid-project | 2 Pre-ship | 3 Production}
Board: Auditor, Defender, Challenger, Judge
Skill version: {version}

## Board ruling
{PROCEED | PROCEED WITH CONDITIONS | DO NOT PROCEED}
Blockers: {n}. Major: {n}. Minor: {n}. Notes: {n}.
One sentence stating what the ruling applies to: starting the build,
moving to pre-ship, shipping, or continuing normal operation.

## Executive summary
One short paragraph with the headline finding. Then:
- Top 3 risks, each with its finding ID.
- Up to 3 strengths, each tied to evidence. List fewer if fewer exist.
- The recommended next action.

## Configuration
Scope. Thresholds (defaults or overrides). Exempt paths. Evidence mode,
with every approved command. Database access. Whether the question bank
ran attended or unattended.

## Coverage
Which lenses ran. Component counts by tier. What was read in full, what
was sampled, what was not read, and why. What the board could not reach:
production config, external systems, anything outside the repository.

## Stage-specific answers
The question bank, grouped as in the bank. Each answer marked: from the
user, from the repository, N/A with reason, or Unanswered.

## Inventory summary
Total components. Breakdown by type and by tier. Count of each verdict.

## Findings by severity

### Blockers (must fix before the next stage; at Stage 3, fix now)
### Major (should fix soon)
### Minor (can defer, but log)
### Notes (informational)

## Security posture
One line per security domain: finding IDs, "checked, no findings",
"not applicable", or "not checked". Count of entry points and how many
have no authentication, no authorization, or no object scope.
Link to security-review.md.

## Feature status
Count of features by status. A table of every in-scope feature that is
not COMPLETE. Journeys walked and their results.
Link to feature-matrix.md.

## Internal clashes
One line per clash class: finding IDs, "checked, no findings",
"not applicable", or "not checked". A table of confirmed clashes with
the authoritative side for each.
Link to clash-register.md.

## Challenge round results
Each challenged finding, the version the Judge picked, and why.
Link to challenge-rounds.md.

## Commendations
COMMEND notes. Leave the section out if there are none.

## Investigated and dismissed
Each suspected problem the board cleared, with the reason.

## Suppressed by configuration
Findings that default thresholds would have raised and the user's
configuration suppressed.

## Changes since last audit
Follow-up audits only. Every earlier finding marked RESOLVED,
STILL OPEN, REGRESSED, or NO LONGER APPLICABLE, with evidence.

## Recommended action plan
Ordered. What to do first, second, third. Blockers first. Group by
"before the next stage", "soon", and "can defer". Each item names its
finding IDs. Time estimates only if the user asked for them.

## Open questions
What the board could not settle. Each tagged to a file, feature, or
finding, with the check that would settle it.
Link to unanswered-questions.md.
```

### The board ruling

| Ruling | When |
|---|---|
| DO NOT PROCEED | One or more BLOCKERs. |
| PROCEED WITH CONDITIONS | No BLOCKERs, one or more MAJORs. The conditions are the MAJOR findings, listed by ID. |
| PROCEED | No BLOCKERs and no MAJORs. |

The ruling follows from the counts. The Judge does not soften it and does not harden it. A partial audit, where the user asked for fewer than four lenses, issues DO NOT PROCEED if it found a BLOCKER. Otherwise it states "PARTIAL AUDIT: no overall ruling" and names the lenses that did not run.

---

## Finding format

Every finding in `report.md` uses this shape. Order within each severity: security, then feature, then clash, then component. Within a lens, widest blast radius first.

```markdown
#### {ID}. `{path or feature or clash title}` - {short title}

Lens: {Component | Security | Feature | Clash} | Verdict: {KEEP AS IS | REFACTOR | REWRITE | DELETE | SPLIT} on `{component}` | Severity: {BLOCKER | MAJOR | MINOR | NOTE} | Evidence: {VERIFIED | TRACED | INFERRED}

{The lens line, see below.}

**Defender argued:** ...

**Challenger argued:** ...

**Judge ruled:** ... Required fix:
1. ...
2. ...
```

The lens line:

- **Component:** none. The path in the heading is enough.
- **Security:** `Attack path: {actor} -> {entry point} -> {gap} -> {result}`
- **Feature:** `Status: {COMPLETE | PARTIAL | STUB | BROKEN | UNREACHABLE | UNDOCUMENTED | MISSING}. Hop: {the broken or missing hop}`
- **Clash:** `Side A: {location and what it says}. Side B: {location and what it says}. Meeting point: {input or sequence}. Authoritative: {side}`

MINOR and NOTE findings may be compressed to the heading line, the lens line, and the Judge's ruling.

Never include a secret value or a real user record in any finding.

---

## `inventory.md`

One row per component. Every component has a tier and a verdict.

| Component | Type | Path | Size | Purpose | Depends on | Depended on by | Last modified | Tier | Verdict | Severity | Finding |
|---|---|---|---|---|---|---|---|---|---|---|---|

Below the table:

- **Entry points.** Every way input reaches the system. Feeds the security pass.
- **Data stores and sensitive data.** Each store, and which classes of sensitive data it holds.
- **External integrations.** Each third party the system calls or is called by.
- **Feature register seed.** The feature claims found while walking the repository. Feeds the feature pass.
- **Tier 3 batches.** Each group of components ruled on together, with the ruling.

## `dependency-map.md`

A Mermaid diagram or ASCII graph. For a large system, one graph per service plus one graph of the services. Below the graph, four lists with locations: circular dependencies, god objects, orphans, long chains.

## `security-review.md`

Format in `security-pass.md`.

## `feature-matrix.md`

Format in `feature-pass.md`.

## `clash-register.md`

Format in `clash-pass.md`.

## `challenge-rounds.md`

One section per challenged finding:

1. The finding ID and the current code, quoted or summarised with locations.
2. The Challenger's alternative, as code or pseudo-code.
3. The Defender's critique of the alternative.
4. The Challenger's revision, if one was made.
5. The Judge's pick and the reason.

## `unanswered-questions.md`

One entry per question:

- The question.
- Why the answer changes the audit.
- The check, measurement, or person that would answer it.
- The finding IDs that depend on it.

Group them: question-bank questions the team could not answer, then questions the board raised during the audit.

---

## The closing message

After the files are written, tell the user in chat:

1. The board ruling and the counts by severity.
2. The top three findings, by ID, in one line each.
3. The path to `report.md`.
4. That `docs/audit/` describes the system's weaknesses, and to check that it is excluded from version control or reviewed before it is committed, especially in a public repository.

Do not paste the report into chat.

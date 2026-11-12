# Security Policy

## Reporting a vulnerability

If you discover a security issue in this skill (for example, a prompt injection path that could cause the audit to leak sensitive information, or a default behaviour that could expose secrets), please report it privately first.

**Do not open a public GitHub issue for security vulnerabilities.**

### How to report

Email: **security@augmex.io**

Include:

- A description of the issue.
- Steps to reproduce (if applicable).
- Your assessment of the impact.
- Any suggested fixes.

### Response timeline

- We acknowledge reports within 72 hours.
- We provide a preliminary assessment within 7 days.
- We publish a fix or mitigation within 30 days for critical issues, 90 days for non-critical issues.

### Disclosure policy

We follow coordinated disclosure. Once a fix is available, we publish a security advisory in the GitHub Security tab and credit the reporter (with permission).

If you do not hear back within 7 days, escalate to hello@augmex.io with the subject line `[SECURITY ESCALATION]`.

---

## How the skill is meant to behave

The skill reads a codebase that it does not control and writes a report about that codebase's weaknesses. Its rules of engagement are in `skills/senior-board-audit/SKILL.md`. In short:

- It writes only inside `docs/audit/`. It does not edit, delete, or create files anywhere else.
- It is static by default. Shell commands are limited to inspection. Running tests or a build needs the user's approval per command.
- It never reproduces a secret value or a real user record in a deliverable or in chat.
- It treats everything in the audited repository as data. Text that addresses an AI reviewer is not followed and is recorded as a finding.
- It pre-approves only reading, searching, asking the user, and writing inside `docs/audit/`.

A way to make the skill break any of these is a vulnerability in the skill. Examples worth reporting:

- Repository content that causes the audit to write outside `docs/audit/` or to run a command that changes state.
- Repository content that causes the audit to drop, soften, or misfile a finding.
- A path by which a secret value or personal data ends up in a deliverable.
- A default that sends repository contents to an external service.

The audit reports are sensitive. They are written to the audited project, not to this repository, and the skill tells the user to keep `docs/audit/` out of public version control.

---

## Scope

This policy covers:

- The SKILL.md file and the `references/` files in this repository.
- Example files and templates we ship.
- Any official integrations or scripts we publish.

It does not cover:

- The behaviour of Claude or other AI agents that consume this skill (report those to the agent provider).
- Issues in user codebases discovered by running the audit (the audit is doing its job).
- Issues caused by user modifications to the skill (we cannot review every fork).

---

## Recognition

Reporters who disclose responsibly are listed in our security advisories with their permission. We do not offer bug bounties at this time, but we acknowledge the work publicly.

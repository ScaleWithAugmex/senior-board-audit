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

## Scope

This policy covers:

- The SKILL.md file in this repository.
- Example files and templates we ship.
- Any official integrations or scripts we publish.

It does not cover:

- The behaviour of Claude or other AI agents that consume this skill (report those to the agent provider).
- Issues in user codebases discovered by running the audit (the audit is doing its job).
- Issues caused by user modifications to the skill (we cannot review every fork).

---

## Recognition

Reporters who disclose responsibly are listed in our security advisories with their permission. We do not offer bug bounties at this time, but we acknowledge the work publicly.

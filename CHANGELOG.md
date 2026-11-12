# Changelog

All notable changes to the Senior Board Audit skill will be documented here.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and the project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [Unreleased]

### Planned
- GitHub Actions integration for lightweight per-PR audits.
- Project-type presets (SaaS web app, mobile backend, AI product, e-commerce).
- Team audit mode where several developers answer the question bank and the board reconciles them.

---

## [2.0.0] - 2026-11-12

The audit now covers four lenses: components, security, features, and internal clashes. The report format changed, which is why this is a major version.

### Added
- **Security pass (Phase 4A).** Attack surface map, an access control matrix for every entry point, and eleven security domains: authentication, authorization, injection, secrets and cryptography, data protection, supply chain, configuration, abuse and business logic, detection, LLM integrations, client and mobile. Every finding needs an attack path. New deliverable: `security-review.md`.
- **Feature pass (Phase 4B).** A feature register built from every claim source, an end-to-end trace of each feature, and a status for each: COMPLETE, PARTIAL, STUB, BROKEN, UNREACHABLE, UNDOCUMENTED, or MISSING. Completeness checks and user journey walks. New deliverable: `feature-matrix.md`.
- **Clash pass (Phase 4C).** Thirteen classes of internal contradiction: source of truth, rules, contracts, schema, configuration, dependencies, routing and naming, ordering, concurrency, security controls, conventions, coexistence, and records. Every finding names both sides, the meeting point, and the authoritative side. New deliverable: `clash-register.md`.
- **Stage 3 (Production)** with its own question bank and severity column.
- **Board ruling** at the top of the report: PROCEED, PROCEED WITH CONDITIONS, or DO NOT PROCEED.
- **Evidence standard.** Every finding carries a location, an evidence level (VERIFIED, TRACED, INFERRED), and an ID (`C-`, `S-`, `F-`, `X-`).
- **Investigated and dismissed** section, so cleared suspicions stay on the record.
- **Follow-up audits.** A previous report in `docs/audit/` is read and each earlier finding is marked RESOLVED, STILL OPEN, REGRESSED, or NO LONGER APPLICABLE.
- **Scoped audits.** One service, or fewer lenses. A partial audit never issues PROCEED.
- **Review tiers** for the component review, so every component is read and the depth of the write-up follows the risk.
- **Rules of engagement** for secrets and personal data (never reproduced in a deliverable), for instructions planted in the audited repository (treated as evidence, recorded as a finding), and for opt-in verification runs.
- Security, feature, and consistency questions in every stage question bank.
- Red flags for testing, frontend, and infrastructure.
- Stage 0 guidance for every phase: the board audits decisions when there is no code.

### Changed
- The skill is split into `SKILL.md` plus six files in `references/`, loaded at the step that needs them. Install the whole `skills/senior-board-audit/` folder.
- The audit process has six phases. The challenge round is Phase 5 and the report is Phase 6.
- The question bank is answered from the repository first. The user is asked only what the code cannot answer.
- Pre-approved tools are reduced to reading, searching, asking the user, and writing inside `docs/audit/`. Shell commands go through the normal permission prompt.
- The version moved from a top-level frontmatter key to `metadata.version`, as the Agent Skills spec requires.
- Deliverables grew from five files to eight.

### Fixed
- Phase 1 told the Auditor to save `inventory.md` to the project root, against the rule that the board writes only inside `docs/audit/{date}/`.
- The checkout example was MAJOR in SKILL.md and BLOCKER in the README and the example report, with a different fix in each. It is a BLOCKER everywhere, with one fix.
- The worked example broke the skill's own length limits for the Challenger and the Judge.
- The example report used NOTE as a verdict. NOTE is a severity.
- The example report said 12 services in the summary and 8 in the inventory.
- Size thresholds were configurable in one section and hardcoded in two others.
- The report template had no place for the Configuration, Suppressed by configuration, and COMMEND content the skill required.
- "Audit every component" had no answer for large codebases. Review tiers resolve it.
- Stage 0 had no components to run the phases on. It now audits decisions.
- `EXPLAIN` was listed as read-only without excluding `EXPLAIN ANALYZE`, which executes the statement. `find` was listed without excluding `-delete` and `-exec`.
- CONTRIBUTING.md pointed at `skill/SKILL.md`. The path is `skills/senior-board-audit/SKILL.md`.
- CONTRIBUTING.md said the project had no separate code of conduct while `CODE_OF_CONDUCT.md` existed.
- The ignore file was committed as `gitignore` and had no effect. It is now `.gitignore`.

---

## [1.0.0] - 2026-05-20

### Added
- Initial public release.
- Four-agent board structure: Auditor, Defender, Challenger, Judge.
- Three project stages with stage-specific question banks (Pre-coding, Mid-project, Pre-ship).
- Five-phase audit process: Inventory, Dependency Map, Component Review, Challenge Round, Final Report.
- Red flag catalogue covering code structure, duplication, state, security, performance, resilience, observability, and AI-specific patterns.
- Severity calibration table by stage.
- Worked example demonstrating the four-agent debate.
- README, CONTRIBUTING, CODE_OF_CONDUCT, SECURITY, and LICENSE files.
- Example audit report.
- Issue and pull request templates.

### Credits
- Skill structure, content, and maintenance by Augmex Technologies Limited.
- Conceptual foundation from Peter Naur's *Programming as Theory Building* (1985).
- Pattern catalogue informed by adversarial review practices in medicine, aviation, and finance.

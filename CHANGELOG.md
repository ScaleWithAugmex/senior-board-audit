# Changelog

All notable changes to the Senior Board Audit skill will be documented here.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and the project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [Unreleased]

### Planned
- Stage 3 (Post-ship) audit mode for production systems.
- GitHub Actions integration for lightweight per-PR audits.
- Project-type presets (SaaS web app, mobile backend, AI product, e-commerce).
- Follow-up audit mode that tracks resolution against a previous audit.

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

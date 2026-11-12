# Contributing to Senior Board Audit

Thank you for considering a contribution. This document explains how to contribute effectively.

The project is maintained by Augmex Technologies but designed to be community-improved. We accept contributions from anyone who has used the skill on a real codebase and has ideas to make it sharper.

---

## What we accept

### Always welcome

- **New stage-specific questions.** If you have audited a codebase and felt a question was missing from the question bank, propose it.
- **Red flag patterns we missed.** Especially for languages, frameworks, or domains underrepresented in the current skill (Rust, Go, Elixir, Java, .NET, mobile, embedded, game backends, data pipelines, etc).
- **Security checks, feature checks, and clash classes.** Especially ones you met in a real incident. State the proof a finding of that kind needs: the attack path, the broken hop, or the input on which two sides diverge.
- **Worked examples.** Real-world examples of the four-agent debate handling a specific component. These make the skill easier to learn from.
- **Tone and clarity fixes.** If a section reads awkwardly or contradicts itself, send a fix.
- **Translations.** The skill currently lives in English. Other languages welcome.

### Sometimes welcome

- **New stages.** The skill covers Stage 0 through Stage 3. Other stages may be considered if the case is strong.
- **New lenses.** The four lenses (component, security, feature, clash) are deliberate. A fifth needs a class of defect that none of the four can see.
- **New agent roles.** The current four-agent structure (Auditor, Defender, Challenger, Judge) is deliberate. We are open to adding a fifth role if there is a strong, distinct purpose. We are not open to bloating it with more agents for the sake of it.
- **Integration scripts.** GitHub Actions, GitLab CI, Bitbucket Pipelines, pre-commit hooks. Useful, but they live in a separate folder and are not core to the skill.

### Rarely welcome

- **Major scope changes.** Turning this into a code generator, a linter, or an LLM wrapper is out of scope. We are deliberately keeping the skill focused on the audit board pattern.
- **Tool-specific bindings.** If you want this to work better with a specific framework, document the patterns in the red-flag catalogue rather than building bindings.

### Not accepted

- Self-promotion in the SKILL.md or README.
- Changes that remove the Augmex Technologies attribution.
- Changes that soften the adversarial tone of the audit. The board exists to find problems. Polite audits are useless audits.

---

## How to contribute

### 1. Open an issue first (for anything non-trivial)

Before writing a large PR, open an issue describing what you want to change and why. This saves you time if the change is out of scope.

Trivial fixes (typos, broken links, single-line clarifications) can skip this step.

### 2. Fork the repo

Standard GitHub flow.

### 3. Create a feature branch

Branch naming: `{type}/{short-description}`. Examples:
- `feature/post-ship-stage`
- `fix/typo-in-defender-section`
- `docs/add-rust-red-flags`
- `example/saas-audit-report`

### 4. Make your changes

Keep changes focused. One feature or fix per PR. Mixing unrelated changes makes review slower.

### 5. Update the relevant files

The skill lives in `skills/senior-board-audit/`. `SKILL.md` holds the board, the rules, and the process. The checklists live in `references/`:

| Change | File |
|---|---|
| Stage question | `references/question-banks.md` |
| Red flag for the component review | `references/red-flags.md` |
| Security check | `references/security-pass.md` |
| Feature check | `references/feature-pass.md` |
| Clash class or example | `references/clash-pass.md` |
| Report or deliverable format | `references/report-template.md` |
| Board roles, rules, phases, calibration | `SKILL.md` |

If your change affects user-facing behaviour, also update:
- `README.md` (if the change is mentioned there)
- `CHANGELOG.md` (add a line under "Unreleased")
- `examples/example-audit-report.md` (if the report format changed)

Keep the files consistent with each other. A term, a file name, a count, or a severity that appears in two files must say the same thing in both. `SKILL.md` is loaded for the whole audit, so keep it to the board, the rules, and the process. New checklists go in `references/`.

### 6. Test your change

Run the skill on at least one real codebase (your own or a public one) to make sure your change actually improves the audit output.

If you cannot test it on a real codebase, say so in the PR description and we will help test.

### 7. Open a pull request

PR description must include:

- **What changed** (one or two sentences)
- **Why** (the problem this solves)
- **How it was tested** (which codebase, what you observed)
- **Anything reviewers should know** (edge cases, follow-up work)

A PR without this context will be sent back for revision.

### 8. Respond to review

Maintainers will review within 7 days. You may be asked to revise. This is normal.

We are direct in reviews. We will tell you specifically what to change and why. We expect the same in return.

---

## Style guide for skill edits

The skill is a set of Markdown files with a specific tone. Match it.

### Do

- Use plain English. Short sentences. Concrete examples.
- Use lowercase headings (`## Stage detection` not `## Stage Detection`).
- Use specific verbs (rewrite, delete, refactor) over vague ones (improve, enhance, optimize).
- Show code-level evidence when proposing red flags.
- Cite real failure modes from production systems when relevant.

### Don't

- Use em dashes. Use commas or short sentences.
- Use AI-style filler words: "crucial", "pivotal", "leverage", "delve", "underscore", "landscape", "vibrant", "testament", "foster", "showcase", "evolving".
- Use "It's not X, it's Y" structures.
- Hedge with "potentially", "possibly", "might", "could perhaps".
- Add motivational filler ("The future of code review is bright!"). Cut it.
- Add emojis to headings or bullets.

When in doubt, read the existing SKILL.md and match the voice.

---

## Code of conduct

Be direct without being cruel. The skill is adversarial, but contributors are not.

- Disagree on technical merits. Not on the contributor.
- Assume good faith. The other person is trying to make the project better.
- Accept criticism gracefully. Give it specifically.
- If a discussion gets heated, step away for a day. Come back with code or evidence.

The full policy, including how to report a problem, is in [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md).

---

## License of contributions

By submitting a contribution, you agree that your work is licensed under the MIT License of the project. You retain copyright on your individual contributions, but you grant the project permission to use them under the same terms.

---

## Recognition

Significant contributions will be added to a `CONTRIBUTORS.md` file. We do not list every typo fix, but anyone who has added a stage, a substantial set of red flags, or a major worked example gets named.

---

## Questions

Open an issue with the `question` label or reach out to the maintainers at hello@augmex.io.

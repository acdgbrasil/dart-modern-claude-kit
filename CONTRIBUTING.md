# Contributing to dart-modern-claude-kit

Thanks for considering a contribution. This kit codifies opinions — every change must answer **"why does this opinion exist?"** before it lands.

This file is for humans. There is also a [`CLAUDE.md`](CLAUDE.md) at the repo root that gives Claude (or any AI agent) the precise mechanics to make changes here. **If you are working with Claude Code, ask it to read `CLAUDE.md` first — it has step-by-step procedures for every contribution type.**

---

## Quick map: "I want to..."

| You want to... | Read | Then |
|---|---|---|
| Report a bug or wrong behavior | [`.github/ISSUE_TEMPLATE/bug_report.md`](.github/ISSUE_TEMPLATE/bug_report.md) | Open an issue |
| Propose a new skill | [`CLAUDE.md`](CLAUDE.md) § "How to add a new skill" + [`.github/ISSUE_TEMPLATE/new_skill_proposal.md`](.github/ISSUE_TEMPLATE/new_skill_proposal.md) | Open issue first, then PR |
| Propose a new agent | [`CLAUDE.md`](CLAUDE.md) § "How to add a new agent" | Open issue first, then PR |
| Change or add a policy (H/P/C rule) | [`CLAUDE.md`](CLAUDE.md) § "How to add or modify a policy" + [`.github/ISSUE_TEMPLATE/policy_change_proposal.md`](.github/ISSUE_TEMPLATE/policy_change_proposal.md) | Open issue first, then PR |
| Fix a typo / clarify wording | — | Open PR directly |
| Adapt the kit for your stack and don't want to upstream | — | Fork. Don't open a PR. |
| Disagree with an existing opinion | [`RATIONALE.md`](RATIONALE.md) for the *why* first | If still disagree, open a `policy_change_proposal` issue |

---

## The non-negotiables

1. **Every opinionated rule must have a rationale.** No rule lands without an entry in [`RATIONALE.md`](RATIONALE.md) explaining the incident, constraint, or trade-off behind it. "Best practice" is not a justification — point to the bug it prevents.
2. **No re-introduction of ACDG / Conecta Raros references** in `skills/` or `agents/`. Origin attribution lives only in `README.md`, `RATIONALE.md`, and `plugin.json`.
3. **Conventional Commits** for every commit message: `feat: ...`, `fix: ...`, `docs: ...`, `chore: ...`, `refactor: ...`.
4. **SemVer** for `plugin.json`:
   - `feat:` → minor bump
   - `fix:` / `docs:` (small) → patch bump
   - Breaking change to an existing rule (renamed skill, removed agent, contradicted policy) → major bump
5. **Test locally before pushing.** Procedure in [`CLAUDE.md`](CLAUDE.md) § "Testing the plugin locally".
6. **Honest disclosure** stays. Do not remove the "single-user adoption" callouts from `README.md` and `RATIONALE.md` until adoption actually broadens — and then rewrite them honestly with the new state.

---

## Before opening a PR

Run this checklist:

- [ ] My change has a corresponding issue (unless it's a typo / clarification)
- [ ] If I added a new opinion, I added a section in `RATIONALE.md`
- [ ] If I changed a skill or agent, I checked that it doesn't reintroduce ACDG-specific names (`grep -r "ACDG\|Conecta Raros\|social_care\|acdgbrasil" skills/ agents/` returns 0)
- [ ] My commit messages follow Conventional Commits
- [ ] If I bumped behavior, I bumped `plugin.json` version per SemVer
- [ ] I tested the plugin locally (see `CLAUDE.md`)

---

## Reviewing PRs

When reviewing, ask:

1. **Is the rationale convincing?** Vague justifications get pushed back.
2. **Does this rule conflict with an existing one?** Cross-reference `RATIONALE.md`.
3. **Does this make the kit more useful, or just more opinionated?** Both can be valuable; the line is whether the new opinion solves a class of bug or just expresses taste.
4. **Did the contributor preserve the kit's voice?** This kit talks straight, gives examples, and admits uncertainty. Don't accept changes that hedge or sermonize.

---

## Code of conduct

Be kind. Disagree with rules, never with people. If a contributor proposes something you think is wrong, explain *why* with reference to a rule, an incident, or a constraint — not with authority.

---

## Maintainers

Currently maintained by the original author (Gabriel Aderaldo). If you would like co-maintainer status — open an issue describing the kind of contributions you'd like to make and the projects where you've used the kit.

<!-- Read CONTRIBUTING.md and CLAUDE.md before filing this PR. -->

## Summary

<!-- One paragraph: what changed and why. -->

## Type of change

- [ ] Bug fix (skill or agent behaves wrong)
- [ ] New skill
- [ ] New agent
- [ ] New or modified policy (H/P/C rule, MVVM, agent testing)
- [ ] Documentation only (typo, clarification, README/RATIONALE/CLAUDE update)
- [ ] Refactor (no behavior change)

## Linked issue

Fixes #<!-- issue number, or "n/a" if typo/trivial -->

## Rationale

<!-- For new opinions: paste the new RATIONALE.md entry here.
     For bug fixes: explain how the bug manifests and what the fix does.
     For docs: skip if obvious. -->

## Pre-merge checklist

- [ ] I read `CONTRIBUTING.md` and (if applicable) `CLAUDE.md`
- [ ] If I added a new opinion, I added a section in `RATIONALE.md` with `**Why:**` and `**How to apply:**`
- [ ] `grep -r "ACDG\|Conecta Raros\|social_care\|acdgbrasil" skills/ agents/` returns 0
- [ ] My commits follow Conventional Commits (`feat:`, `fix:`, `docs:`, `chore:`, `refactor:`)
- [ ] I bumped `.claude-plugin/plugin.json` version per SemVer (or this PR is doc-only)
- [ ] I added a `CHANGELOG.md` entry under `## [Unreleased]`
- [ ] I tested the plugin locally per `CLAUDE.md` § "Testing the plugin locally"

## How to verify this change

<!-- Steps a reviewer can run to confirm the change works:
     1. /plugin update dart-modern-kit
     2. Ask Claude: "<concrete prompt that should trigger the changed behavior>"
     3. Expected response: <what should happen> -->

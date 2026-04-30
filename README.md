# Dart Modern Kit — Claude Code Plugin

> Skills, agents and opinionated policies for modern Dart/Flutter development with Claude Code.

This kit is published under the **ACDG Technology** organization because it was extracted from the Conecta Raros monorepo — a Brazilian healthcare platform built by ACDG.

> **Adoption status (as of 2026-04-30):** the kit has only been used in production by its original author (Gabriel Aderaldo) inside the Conecta Raros monorepo. It has not yet been validated by other ACDG projects or external teams. Treat the rules as battle-tested in *one* project, not the whole industry.

It carries opinionated decisions about:

- **MVVM + Logic Layer** with Command pattern + Result pattern
- **Encapsulation Policy** (H1-H9) — when to use `_`, sealed classes, extension types
- **Pattern Matching Policy** (P1-P5) — Dart 3 advanced patterns + the rare cases where `try/catch` is allowed
- **Concurrency & Performance Policy** (C1-C3) — `Isolate.run`, class modifiers as architectural firewall
- **Atomic Design** for widgets (Page > Organism > Molecule > Atom)
- **Selectors + Connectors** instead of passing ViewModel down the widget tree

It also ships a **bonus security suite** (8 skills + 7 agents) covering OWASP Top 10, threat modeling, pentesting, auth, DevSecOps and secure code review.

## What's inside

| Type | Count | Purpose |
|---|---|---|
| Skills | 10 | `flutter-modern`, `vibe-designer`, `pipeline-maestro`, security suite |
| Agents | 17 | Flutter MVVM pipeline + security agents |
| Policies | 5 | Encapsulation, Pattern Matching, Concurrency, MVVM+Logic, Agent Testing |

See [`docs/INSTALL.md`](docs/INSTALL.md) for setup, [`RATIONALE.md`](RATIONALE.md) for *why* each decision exists.

## Quick install

```bash
# Clone the repo
git clone https://github.com/acdgbrasil/dart-modern-claude-kit.git ~/dev/dart-modern-claude-kit

# Inside Claude Code, add as marketplace and install
/plugin marketplace add ~/dev/dart-modern-claude-kit
/plugin install dart-modern-kit@dart-modern-kit-marketplace
```

After install, the skills (e.g. `flutter-modern`, `vibe-designer`) and agents (e.g. `flutter-viewmodel-engineer`) become available with the `dart-modern-kit:` namespace.

## Philosophy

This kit is **opinionated**. The opinions come from real production scars, not theory. Every non-obvious rule has a `**Why:**` line in `RATIONALE.md` explaining the incident or constraint that motivated it.

You are expected to disagree with some of it. That's fine — Claude Code lets you disable individual skills and agents per-project.

## Contributing

PRs welcome. **Read [`CONTRIBUTING.md`](CONTRIBUTING.md) first** — it has a quick "I want to..." map.

If you are working with **Claude Code** to contribute, ask it to read [`CLAUDE.md`](CLAUDE.md) at the repo root before making any changes — it has step-by-step procedures for every contribution type, plus the hard rules (never reintroduce ACDG references, every new opinion needs a `RATIONALE.md` entry, etc.).

Issue templates (web forms): [bug report](.github/ISSUE_TEMPLATE/01-bug_report.yml) · [new skill / agent](.github/ISSUE_TEMPLATE/02-new_skill_or_agent_proposal.yml) · [policy change](.github/ISSUE_TEMPLATE/03-policy_change_proposal.yml).

## License

MIT. Copy, fork, adapt freely.

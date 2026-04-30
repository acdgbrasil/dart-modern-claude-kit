# Dart Modern Kit — Claude Code Plugin

> Skills, agents and opinionated policies for modern Dart/Flutter development with Claude Code.

This is a **personal kit**, extracted from a real production monorepo (a Brazilian healthcare platform called Conecta Raros / ACDG). It carries opinionated decisions about:

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
git clone https://github.com/gaderaldo/dart-modern-claude-kit.git ~/dev/dart-modern-claude-kit

# Inside Claude Code, add as marketplace and install
/plugin marketplace add ~/dev/dart-modern-claude-kit
/plugin install dart-modern-kit@dart-modern-kit-marketplace
```

After install, the skills (e.g. `flutter-modern`, `vibe-designer`) and agents (e.g. `flutter-viewmodel-engineer`) become available with the `dart-modern-kit:` namespace.

## Philosophy

This kit is **opinionated**. The opinions come from real production scars, not theory. Every non-obvious rule has a `**Why:**` line in `RATIONALE.md` explaining the incident or constraint that motivated it.

You are expected to disagree with some of it. That's fine — Claude Code lets you disable individual skills and agents per-project.

## License

MIT. Copy, fork, adapt freely.

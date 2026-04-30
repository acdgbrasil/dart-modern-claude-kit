# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- Contribution infrastructure: `CONTRIBUTING.md`, `CLAUDE.md` (instructions for AI agents working on the plugin), `.github/PULL_REQUEST_TEMPLATE.md`, three issue templates (bug report, new skill proposal, policy change proposal), and this `CHANGELOG.md`.
- Issue templates upgraded from Markdown to **GitHub Issue Forms** (YAML schema): structured fields with required validation, dropdowns for enums (severity, adoption status, change type), checkbox checklists, and inline guidance. Files prefixed numerically (`01-`, `02-`, `03-`) to control display order in the template chooser.
- `.github/ISSUE_TEMPLATE/config.yml` — template chooser config with `contact_links` directing "why" questions to `RATIONALE.md`, install questions to `docs/INSTALL.md`, contribution questions to `CONTRIBUTING.md` / `CLAUDE.md`. Blank issues remain enabled for edge cases.

### Removed
- Old Markdown issue templates (`bug_report.md`, `new_skill_proposal.md`, `policy_change_proposal.md`) — superseded by the YAML forms above.

## [0.1.0] - 2026-04-30

### Added
- Initial extraction from the Conecta Raros (ACDG) Flutter monorepo.
- 10 skills: `flutter-modern`, `vibe-designer`, `pipeline-maestro`, plus 6 security skills (`api-security-guardian`, `auth-session-security`, `appsec-code-reviewer`, `devsecops-pipeline`, `red-team-scanner`, `threat-modeler`) and `kodus-review`.
- 17 agents: 10 Flutter pipeline agents (`flutter-domain-modeler`, `flutter-infra-implementer`, `flutter-usecase-orchestrator`, `flutter-viewmodel-engineer`, `flutter-view-implementer`, `flutter-code-reviewer`, `flutter-integration-validator`, `flutter-quality-checker`, `test-writer`, `domain-architect`) plus 7 security agents (`api-hardener`, `auth-auditor`, `pentest-scanner`, `pipeline-security-auditor`, `secure-code-reviewer`, `security-orchestrator`, `threat-analyst`).
- 5 opinionated policies in `skills/flutter-modern/references/`: encapsulation (H1–H9), pattern matching (P1–P5), concurrency & performance (C1–C3), agent testing, MVVM+Logic.
- `RATIONALE.md` documenting why each opinion exists, with incident-level evidence where available.
- Honest disclosure: kit has only been validated in one production monorepo by a single primary author.

### Notes
- Source-project-specific decisions (Drift, Riverpod, Split-Token, Bitwarden, GHCR) intentionally excluded — listed in `RATIONALE.md` § "What This Kit Does NOT Mandate".

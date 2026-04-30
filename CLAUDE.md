# CLAUDE.md — Working on dart-modern-kit

> **You are Claude (or another AI agent) reading this because someone asked you to contribute to, fix, or extend the `dart-modern-kit` plugin.** This file gives you the mechanics. Follow it strictly — the human reviewer will check you did.

If you are using this plugin to write *application code* in another project, this file does **not** apply to you. This file is only for working **on the plugin itself**.

---

## What this repo is

A Claude Code plugin that ships:

- **10 skills** in `skills/<name>/SKILL.md` — domain-knowledge skills activated by mention
- **17 agents** in `agents/<name>.md` — specialized workers invoked via the Agent tool
- **5 policies** in `skills/flutter-modern/references/` — opinionated rules with H/P/C numeric IDs
- **3 docs** in repo root: `README.md` (audience: anyone), `RATIONALE.md` (audience: skeptics), `CONTRIBUTING.md` (audience: contributors)
- **2 manifests** in `.claude-plugin/`: `plugin.json` (the plugin) + `marketplace.json` (so the repo can be installed as a local marketplace)

---

## Hard rules — read before doing ANYTHING

1. **Never reintroduce ACDG / Conecta Raros / `social_care` / `acdgbrasil` references in `skills/` or `agents/`.** Those were intentionally degenerified. The only places that should mention origin are `README.md`, `RATIONALE.md`, and `plugin.json`. Verify with `grep -r "ACDG\|Conecta Raros\|social_care\|acdgbrasil" skills/ agents/` — must return 0.

2. **Every new opinionated rule requires a `RATIONALE.md` entry** with two structured lines:
   - `**Why:**` — the incident, constraint, or trade-off that motivated it
   - `**How to apply:**` — when this rule kicks in
   "Best practice" is not a valid `Why:`. Point to the actual bug class.

3. **Conventional Commits, always:** `feat:`, `fix:`, `docs:`, `chore:`, `refactor:`. No exceptions.

4. **SemVer for `plugin.json`:**
   - `feat:` → minor bump
   - `fix:` / `docs:` → patch bump
   - Renamed/removed skill or agent OR contradicted existing policy → major bump

5. **Honest disclosure stays.** Do not remove the "single-user adoption" callouts in `README.md` and `RATIONALE.md`. Update them when the situation changes; do not delete.

6. **No `mkdir` for already-existing directories.** Verify first with `ls`.

---

## Repo layout (canonical)

```
dart-modern-claude-kit/
├── .claude-plugin/
│   ├── plugin.json              # Plugin manifest (name, version, author)
│   └── marketplace.json         # Local marketplace pointing to ./
├── .github/
│   ├── PULL_REQUEST_TEMPLATE.md
│   └── ISSUE_TEMPLATE/
│       ├── bug_report.md
│       ├── new_skill_proposal.md
│       └── policy_change_proposal.md
├── README.md                    # Audience: anyone discovering the kit
├── RATIONALE.md                 # Audience: skeptics — every Why
├── CONTRIBUTING.md              # Audience: human contributors
├── CLAUDE.md                    # ← you are here
├── CHANGELOG.md                 # SemVer-tracked changes
├── docs/INSTALL.md              # Step-by-step install
├── skills/
│   ├── flutter-modern/
│   │   ├── SKILL.md
│   │   └── references/          # Universal Flutter docs + 5 policies
│   ├── vibe-designer/SKILL.md
│   ├── pipeline-maestro/SKILL.md
│   ├── api-security-guardian/
│   │   ├── SKILL.md
│   │   └── references/          # OWASP cheatsheets
│   ├── auth-session-security/...
│   ├── appsec-code-reviewer/...
│   ├── devsecops-pipeline/...
│   ├── red-team-scanner/...
│   ├── threat-modeler/...
│   └── kodus-review/SKILL.md
└── agents/
    ├── flutter-domain-modeler.md
    ├── flutter-infra-implementer.md
    ├── flutter-usecase-orchestrator.md
    ├── flutter-viewmodel-engineer.md
    ├── flutter-view-implementer.md
    ├── flutter-code-reviewer.md
    ├── flutter-integration-validator.md
    ├── flutter-quality-checker.md
    ├── flutter-domain-architect.md (currently named domain-architect.md)
    ├── test-writer.md
    ├── api-hardener.md
    ├── auth-auditor.md
    ├── pentest-scanner.md
    ├── pipeline-security-auditor.md
    ├── secure-code-reviewer.md
    ├── security-orchestrator.md
    └── threat-analyst.md
```

---

## Procedures

### How to add a new skill

```
PRECONDITIONS:
- An issue exists describing the skill (use `.github/ISSUE_TEMPLATE/new_skill_proposal.md`)
- The maintainer left a "go" comment

STEPS:
1. mkdir skills/<kebab-case-name>/
2. Create skills/<kebab-case-name>/SKILL.md with frontmatter:
   ---
   name: <kebab-case-name>
   description: >
     One-paragraph description starting with what the skill does, then
     "Activates when the user mentions: ..." with concrete trigger words.
   ---
3. Body of SKILL.md MUST contain:
   - A "Reference Sources" section pointing to references/ files (if any)
   - A "Hard rules" or "Non-negotiables" section if the skill is opinionated
   - At least one "Example" or "Pattern" block with code
4. (Optional) Add references/ folder with deeper docs.
5. Verify zero ACDG references: grep -r "ACDG\|Conecta Raros\|social_care\|acdgbrasil" skills/<name>/ — must be 0.
6. Bump plugin.json version: minor (e.g., 0.1.0 → 0.2.0).
7. Add CHANGELOG.md entry under "## [Unreleased]".
8. Commit: `feat(skill): add <name> — <one-line purpose>`
9. Open PR using .github/PULL_REQUEST_TEMPLATE.md.
```

### How to add a new agent

```
PRECONDITIONS:
- An issue exists describing the agent's scope and what it builds
- The maintainer left a "go" comment

STEPS:
1. Create agents/<kebab-case-name>.md with frontmatter:
   ---
   name: <kebab-case-name>
   description: >
     One sentence: what the agent does, what it writes, what it never touches.
   ---
2. Body MUST contain:
   - A "Primary Reference" section pointing to the skill the agent follows
   - A "Scope — Where You Work" table (paths the agent writes to)
   - A "Scope — What You DO NOT Touch" list
   - "What You Build" section with concrete examples
   - A "Failure Routing" or completion protocol if the agent is part of a pipeline
3. Verify zero ACDG references.
4. Bump plugin.json minor.
5. Add CHANGELOG.md entry.
6. Commit: `feat(agent): add <name> — <one-line purpose>`
7. Open PR.
```

### How to add or modify a policy (H/P/C/MVVM/AT rule)

```
PRECONDITIONS:
- An issue exists using `.github/ISSUE_TEMPLATE/policy_change_proposal.md`
- The proposal has been open ≥7 days for community feedback (unless trivially typographic)
- The maintainer left a "go" comment

STEPS for ADDING a new rule (e.g., a new H10 in encapsulation):
1. Edit skills/flutter-modern/references/<policy_name>.md to add the rule
2. Edit skills/flutter-modern/SKILL.md if the rule belongs in the description preamble
3. Edit RATIONALE.md to add Why: + How to apply: under the corresponding policy
4. Bump plugin.json: minor for additive rule, major if it overrides existing rule
5. Add CHANGELOG entry: `feat(policy): add H10 — <name>`
6. Commit, open PR.

STEPS for MODIFYING an existing rule:
1. Edit the policy reference file
2. Edit RATIONALE.md to update the Why: with what changed and why
3. If the change reverses or contradicts the prior rule → MAJOR bump
4. Add CHANGELOG entry: `BREAKING CHANGE: <description>` if major
5. Commit, open PR.

NEVER:
- Modify a policy without updating RATIONALE.md
- Bump only patch for a behavioral change
- Remove a rule without adding the Why for its removal
```

### How to fix a bug in a skill or agent

```
1. Reproduce: explain in the PR description how the bug manifests
   (e.g., "When user asks X, the skill responds Y instead of Z")
2. Edit the skill/agent file directly
3. If the fix changes behavior in a user-visible way → patch bump for clarification, minor for new behavior
4. If the fix reveals a missing rule, add it (follow "policy" procedure above)
5. CHANGELOG entry under "Fixed"
6. Commit: `fix(skill|agent): <component> — <description>`
```

### Testing the plugin locally before pushing

```
1. Inside Claude Code, register the local repo as a marketplace:
   /plugin marketplace add /full/path/to/dart-modern-claude-kit
2. Install:
   /plugin install dart-modern-kit@dart-modern-kit-marketplace
3. Restart Claude Code or open a new session
4. Trigger the changed skill/agent:
   - For skills: ask Claude something that should activate the trigger
   - For agents: invoke via the Agent tool with subagent_type=<agent-name>
5. Verify: the changed behavior appears, and old behaviors are unaffected
6. To re-test after edits: /plugin update dart-modern-kit
7. To uninstall after testing: /plugin uninstall dart-modern-kit
```

### Bumping the version

```
1. Determine bump type per SemVer (see Hard Rules #4)
2. Edit .claude-plugin/plugin.json — change "version" field
3. Edit CHANGELOG.md — move "[Unreleased]" entries under a new "[X.Y.Z] - YYYY-MM-DD" section
4. Commit: `chore(release): vX.Y.Z`
5. Tag: `git tag -a vX.Y.Z -m "vX.Y.Z"`
6. Push: `git push origin main --tags`
```

---

## Style guide for skill/agent prose

- **Voice:** straight, terse, opinionated. No "could potentially" or "in some cases". Either yes or no.
- **Examples:** concrete code, not pseudocode. If Dart, valid Dart 3.x syntax.
- **Justification placement:** the rule first, the why second. Not the other way around.
- **No emojis** in skill/agent files unless the user has asked for them in a customization PR.
- **Portuguese vs English:** code & frontmatter in English, prose can be either — match the existing skill's language.

---

## What you should NOT do without explicit user authorization

- Force-push to `main`
- Delete a skill, agent, or policy
- Change the kit's `name` or unique identifier
- Add a dependency on an external service / API in any skill or agent
- Add hooks, settings.json overrides, or anything that changes Claude Code's global behavior
- Publish to a different marketplace
- Create new git tags (always confirm with user)
- Merge a PR (always leave for the human maintainer)

---

## When the user asks "how do I [X]?"

Always:
1. Look in `RATIONALE.md` first — most "why" questions are answered there
2. Look in the relevant skill or agent file second
3. If still unclear, point the user to the policy reference in `skills/flutter-modern/references/`
4. If genuinely undocumented, that's a documentation gap — propose adding a section

Never:
- Make up a rule that doesn't exist in the kit
- Cite ACDG-specific decisions (Drift, Riverpod, Split-Token) as kit-mandated — they are explicitly excluded per RATIONALE § "What This Kit Does NOT Mandate"

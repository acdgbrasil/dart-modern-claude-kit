# Installation

## Prerequisites

- Claude Code installed (`claude` command available)
- Git installed
- For Flutter agents: Dart SDK + Flutter SDK
- For the Dart MCP Server (recommended): `claude mcp add --transport stdio dart -- dart mcp-server`

## Install (one-time)

```bash
# 1. Clone the kit somewhere stable on your machine
git clone https://github.com/acdgbrasil/dart-modern-claude-kit ~/dev/dart-modern-claude-kit

# 2. Inside Claude Code, register as a local marketplace
/plugin marketplace add ~/dev/dart-modern-claude-kit

# 3. Install the plugin
/plugin install dart-modern-kit@dart-modern-kit-marketplace
```

After install, restart Claude Code (or open a new session). The skills and agents become available — you can verify with:

```
/plugin list
```

## What you get

### Skills (10) — invoked by mention or `/<name>`
| Skill | Trigger |
|---|---|
| `flutter-modern` | "Flutter", "Dart", "ViewModel", "MVVM", etc. |
| `vibe-designer` | UI/visual-only requests ("change button color", "add padding") |
| `pipeline-maestro` | "implement feature", "run pipeline", "end-to-end" |
| `api-security-guardian` | "API", "endpoint", "REST", "CORS", "rate limit" |
| `auth-session-security` | "login", "JWT", "OAuth", "session" |
| `appsec-code-reviewer` | "secure code review", "OWASP review" |
| `devsecops-pipeline` | "Docker", "CI/CD", "secrets", "supply chain" |
| `red-team-scanner` | "pentest", "find vulnerabilities", "attack" |
| `threat-modeler` | "threat model", "STRIDE", "DFD", "risk analysis" |
| `kodus-review` | `/kodus-review` (requires kodus CLI) |

### Agents (17) — invoked via the Agent tool
| Agent | Use for |
|---|---|
| `flutter-domain-modeler` | Domain models + API models (immutable, Equatable, copyWith) |
| `flutter-infra-implementer` | Services, Repositories, Mappers (the only agent allowed `try/catch`) |
| `flutter-usecase-orchestrator` | UseCases extending `BaseUseCase` |
| `flutter-viewmodel-engineer` | ViewModels with ChangeNotifier + Command |
| `flutter-view-implementer` | Pages, Organisms, Molecules, Atoms |
| `flutter-code-reviewer` | Read-only architecture compliance check |
| `flutter-integration-validator` | Full validation suite via Dart MCP |
| `flutter-quality-checker` | Static analysis + format + tests |
| `test-writer` | Failing tests from contracts (TDD red-first) |
| `domain-architect` | Type-level contract design |
| `api-hardener` | API hardening with patches |
| `auth-auditor` | OIDC / JWT / session audit |
| `pentest-scanner` | RED Team vulnerability scan |
| `pipeline-security-auditor` | DevSecOps audit (CI/CD, Docker, secrets) |
| `secure-code-reviewer` | Defensive secure code review |
| `security-orchestrator` | Coordinates all security agents end-to-end |
| `threat-analyst` | STRIDE threat modeling with DFD |

## Project setup (per project)

The kit ships **opinions**. To make those opinions visible to Claude in your project, add a `CLAUDE.md` to your repo root with at minimum:

```markdown
# Project Conventions

This project uses the `dart-modern-kit` Claude Code plugin.
Read its `RATIONALE.md` for the why behind the architectural rules.

## Project-specific overrides

<list any rules from the kit you do NOT follow, with reasoning>

## Project-specific additions

<list rules unique to this project — your equivalent of ADRs>
```

The kit's `flutter-modern` skill respects project-level CLAUDE.md as higher priority than its own opinions. If you contradict a rule, document why.

## Verifying the install

```bash
# In Claude Code
/plugin list                                    # should show dart-modern-kit as installed
# Then ask Claude:
"Show me the flutter-modern skill"              # should describe MVVM + Logic
"Use the flutter-domain-modeler agent to ..."   # should invoke the agent
```

## Updating

```bash
cd ~/dev/dart-modern-claude-kit
git pull
# Inside Claude Code:
/plugin update dart-modern-kit
```

## Uninstall

```
/plugin uninstall dart-modern-kit
/plugin marketplace remove dart-modern-kit-marketplace
```

## Troubleshooting

**"Skill not found"** — restart Claude Code after install.

**"Agent description references handbook/ that doesn't exist"** — the agents reference the kit's bundled `references/` folder, not your project's. If you see broken refs, open an issue.

**"Dart MCP Server not available"** — install with:
```bash
claude mcp add --transport stdio dart -- dart mcp-server
```

**"My project doesn't use Riverpod / Drift / OIDC — do the agents still work?"** — yes. The agents recommend specific tools but adapt when your project uses alternatives. If you find an agent forcing a tool you don't use, file an issue.

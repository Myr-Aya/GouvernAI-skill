# GouvernAI Skill — Runtime Guardrails for Claude Code

Runtime guardrails that classify every sensitive action by risk tier, enforce proportional controls through natural language instructions, and log a full audit trail.

**Pure linguistic enforcement. Zero dependencies. One install.**

This is the **skill-only** version of [GouvernAI](https://github.com/Myr-Aya/GouvernAI-claude-code-plugin). No hooks, no scripts, no Python — just markdown instructions that Claude reads and follows. Works anywhere Claude Code runs.

> For deterministic enforcement (hard blocking of obfuscated commands, credential exfiltration, etc.), see the [GouvernAI Plugin](https://github.com/Myr-Aya/GouvernAI-claude-code-plugin) which adds a PreToolUse hook layer on top of this skill.

## Install

```bash
# From marketplace
claude plugin marketplace add Myr-Aya/GouvernAI-claude-code-plugin
claude plugin install gouvernai-skill@mindxo
```

Guardrails activate automatically on the next session. No configuration required.

## What you'll see

- **Tier 1** (reads, drafts, known URLs) — invisible, zero overhead
- **Tier 2** (file writes, git commit) — 🛡️ notification, proceeds automatically
- **Tier 3** (email, config changes, npm install, curl) — 🛡️ pauses for your approval
- **Tier 4** (sudo, credential transmit, purchases) — 🛡️ full stop with risk assessment
- **BLOCKED** — hard constraint violation, no override

## Quick test

After installing, try these to see the guardrails in action:

1. `git status` — Tier 1, excluded from gate, no overhead
2. Ask Claude to write a file — Tier 2 notification appears
3. Ask Claude to run `curl` to an external URL — Tier 3 pause for approval

## How it works

The SKILL.md file teaches Claude an 8-step gate process: identify the action, determine mode, classify risk tier (using ACTIONS.md), check escalation rules (using TIERS.md), check pre-approval, verify hard constraints (using POLICY.md), apply the proportional control, and log the decision.

This is **linguistic enforcement** — Claude reads and follows the instructions with judgment. There is no programmatic enforcement layer. The audit log is the accountability mechanism.

### 28 classified actions across 8 categories

| Category | Tier 2 | Tier 3 | Tier 4 |
|----------|--------|--------|--------|
| File system | Write to working files | Config modification, delete, bulk ops | — |
| Shell | git commit, mkdir | npm install, curl, wget | sudo, obfuscated commands |
| Network/API | Authenticated API reads | External API writes, data transmission | Unfamiliar endpoints |
| Communication | Self-notification | Message to another person | Group/public channel |
| Credentials | Use in auth request | Display/read value | External transmission |
| Browser | Navigate external URL | Form fills, file downloads | Executable downloads |
| Scheduling | — | Cron creation | Webhook endpoints |
| Financial | Pricing lookups | — | Purchases, billing changes |

### Escalation rules

Actions escalate by +1 tier when:
- Unfamiliar target (endpoint, recipient, file not seen in session)
- Bulk operation (5+ targets)
- Scope expansion (agent does more than requested)
- Third-party data
- Chained actions (3+ sequential sensitive actions)

### 8 hard constraints (NEVER rules)

1. Never transmit credentials externally
2. Never execute obfuscated commands
3. Never self-modify the guardrails skill
4. Never exceed requested scope
5. Never carry forward approval across actions
6. Never execute untrusted commands without showing them first
7. Never log credential values
8. Never act on behalf of other users

## Slash commands

| Command | What it does |
|---------|-------------|
| `/guardrails` | Show current mode, tier distribution, approvals/denials |
| `/guardrails log` | Display recent audit log entries |
| `/guardrails strict` | All tiers +1 — persisted to `guardrails-mode.json` |
| `/guardrails relaxed` | Tier 2 skips gate — persisted to `guardrails-mode.json` |
| `/guardrails audit` | Audit-only mode: T2/T3 auto-proceed, T4 halts |
| `/guardrails reset` | Return to default full-gate mode |
| `/guardrails policy` | Display hard constraints |

Mode changes persist to `guardrails-mode.json` in the project root and survive context resets and new sessions.

## CI and unattended use

In full-gate mode, Tier 2 actions use "proceed unless objected" — which is silent auto-approval when no human is watching. For CI pipelines or unattended runs:

```bash
/guardrails audit
```

In audit-only mode: T2 and T3 auto-proceed with full logging, T4 halts without executing.

## Plugin structure

```
gouvernai-skill/
├── .claude-plugin/
│   └── plugin.json          # Plugin metadata
├── skills/
│   └── gouvernai/
│       ├── SKILL.md          # Gate orchestrator (always loaded, ~1,500 tokens)
│       ├── ACTIONS.md        # Action → tier classification (~700 tokens)
│       ├── TIERS.md          # Universal controls + escalation (~1,100 tokens)
│       ├── POLICY.md         # Hard constraints — NEVER rules (~900 tokens)
│       ├── GUIDE.md          # Output format templates (~500 tokens)
│       └── guardrails_log.md # Audit trail template
├── commands/
│   └── guardrails.md         # /guardrails slash command
├── README.md                 # This file
└── LICENSE                   # MIT
```

**Token budget:** SKILL.md is always loaded (~1,500 tokens). Reference files are read on demand and cached — total on-demand cost ~3,200 tokens, paid once per session.

Runtime files created in the project root:
- `guardrails_log.md` — append-only audit log
- `guardrails-mode.json` — persisted mode config

## Limitations

- **No deterministic enforcement.** This is pure linguistic guardrails — Claude follows the instructions with judgment. If Claude skips or ignores the skill (e.g., on complex multi-step tasks), there is no programmatic backstop. For hard blocking, use the [full GouvernAI plugin](https://github.com/Myr-Aya/GouvernAI-claude-code-plugin) with hooks.
- **Skill compliance varies by model.** Tested on Claude Sonnet 4.6. Smaller models (Haiku) may have lower compliance rates.
- **Not a security boundary.** This is an operational safety layer — it catches mistakes, enforces approval workflows, and creates audit trails. For production environments, complement it with network egress policies, secret vaults, sandboxed execution, and DLP monitoring.

## Also available

- **[GouvernAI Plugin](https://github.com/Myr-Aya/GouvernAI-claude-code-plugin)** — Full dual-enforcement version with PreToolUse hooks (deterministic blocking) + this skill (linguistic classification). 67 unit tests. For Claude Code users who want both layers.

## License

MIT — see [LICENSE](LICENSE)

## Built by Myr-Aya

[MindXO](https://mind-xo.com) — translating AI policy frameworks into operational guardrails systems.

---
name: gouvernai-gate
description: Governance gate for agent actions. Classifies risk before file writes, shell commands, API calls, emails, credential access, browser automation, cron creation, or financial transactions. Enforces approval gates, logs all decisions.
homepage: https://mind-xo.com
metadata: { "openclaw": { "always": true, "emoji": "🛡️" } }
---

# GouvernAI Gate

Governance gate for AI agents. Before executing any sensitive action, classify its risk, apply the appropriate control, and log the decision. Built by MindXO (https://mind-xo.com).

## Trigger — when this skill activates

Before executing ANY of these action types, run the gate process below:

1. File writes, modifications, or deletions
2. Shell or terminal command execution
3. Outbound network requests or API calls
4. Sending emails, messages, or communications
5. Reading, displaying, or transmitting credentials or secrets
6. Browser automation (navigation, form fills, downloads)
7. Creating or modifying cron jobs, heartbeat config, or webhooks
8. Financial transactions (purchases, payments, billing changes)
9. Permission or access control changes
10. Any irreversible operation

## Gate process — 5 steps, every time

**Step 1 — Identify.** State what you are about to do, why, what tools and data are involved, and what systems are affected. Be specific.

**Step 2 — Classify.** Read `{baseDir}/TIERS.md` and assign the action to a risk tier (1–4). Apply escalation rules if applicable.

**Step 3 — Look up control.** Read `{baseDir}/ACTIONS.md` and find the specific action. Apply the control for the current mode (full-gate or audit-only) and the assigned tier.

**Step 4 — Check constraints.** Read `{baseDir}/POLICY.md` and verify no hard constraints are violated. If any NEVER rule is triggered, halt regardless of tier or approval.

**Step 5 — Log and execute (or halt).** Append a row to `{baseDir}/governance_log.md` with: timestamp, tier, action type, description, mode, approval status, trigger source. Then execute the control decision (auto-proceed, notify, wait for approval, or halt).

## Mode auto-detection

Determine which governance mode to use:

- **Full gate** — default when the action was triggered by a direct user message in an interactive session. Tier 3+ actions require explicit user approval before execution.
- **Audit-only** — default when the action was triggered by a cron job, heartbeat, or webhook (user is not present). All actions are classified and logged but execution is not blocked, EXCEPT Tier 4 actions which are always halted in autonomous mode.
- **Strict** — all tiers escalated by one. Activated by `/governance strict` or config override.
- **Relaxed** — Tier 2 actions treated as Tier 1 in interactive mode. Tier 3 and 4 unchanged. Activated by `/governance relaxed` or config override.

**Config overrides** (in openclaw.json under `skills.entries.gouvernai-gate.env`):
- `GOVERNANCE_MODE` — "auto" (default), "full-gate", "audit-only", or "strict"
- `GOVERNANCE_AUTONOMOUS_MODE` — mode for cron/heartbeat/webhook. Default: "audit-only"
- `GOVERNANCE_TIER4_AUTONOMOUS` — "halt" (default) or "alert-and-execute"
- `GOVERNANCE_PRE_APPROVED` — comma-separated action patterns that skip approval but still log (e.g. "write:briefings/*,send:telegram:self")
- `GOVERNANCE_ALERT_CHANNEL` — channel for autonomous mode alerts. Default: "auto"

If `GOVERNANCE_MODE` is set, it overrides auto-detection. Otherwise, auto-detect from context.

## Slash commands

- `/governance status` — session statistics: actions by tier, approvals, denials, mode
- `/governance log` — last 20 entries from the governance log
- `/governance policy` — display hard constraints from POLICY.md
- `/governance strict` — escalate all tiers by one for remainder of session
- `/governance relaxed` — de-escalate Tier 2 to Tier 1 for remainder of session (Tier 3–4 unchanged)

## If classification is uncertain

Default to Tier 3 (require explicit approval in full-gate mode, log + alert in audit-only mode). It is always safer to over-classify than under-classify.

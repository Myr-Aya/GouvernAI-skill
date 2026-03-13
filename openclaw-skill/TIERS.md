# GouvernAI Gate — Risk Tiers

## Tier 1 — Routine (Green)

Low-risk, easily reversible, confined to local workspace. No external effect.

**Criteria:** Read-only operations, writes to scratch/temp locations, information retrieval with no side effects.

**Examples:** Read a file. List directory contents. Run `git status`. Check if an env var exists. Draft a message (not sent). Navigate to a known URL.

**Control:**
- Full gate: auto-proceed, log only.
- Audit-only: auto-proceed, log only.

## Tier 2 — Standard (Amber)

Moderate risk. Modifies persistent local state or involves authenticated external reads.

**Criteria:** Writes to user working files, local state-modifying CLI commands, reads from authenticated APIs, browsing URLs from external sources, sending messages to the user's own channels.

**Examples:** Edit a working document. Run `npm install`. Read Gmail inbox. Browse a URL from skill instructions. Send a Telegram notification to self. Look up pricing info.

**Control:**
- Full gate: notify the user (briefly state the action and classification), proceed unless the user objects, log.
- Audit-only: auto-proceed, log with action detail.

## Tier 3 — Elevated (Red)

High risk. Involves outbound data transmission, communication on behalf of the user, credential handling, or modifications to systems outside the local workspace.

**Criteria:** Writes to external APIs, sends messages to other people, accesses credential values, modifies config files, deletes files, bulk operations (5+ targets), browser form submissions, creates cron jobs, commands with network egress, downloads files from web.

**Examples:** POST to a webhook. Send email to a colleague. Display an API key. Edit `.bashrc`. Delete project files. Submit a browser form. Schedule a new cron job. Run `curl` to an external endpoint. Download a binary.

**Control:**
- Full gate: pause. Present the action, its scope, and what data is involved. Wait for explicit user approval ("yes" or equivalent). If denied, log denial and stop.
- Audit-only: classify and log. Execute the action. Alert the user via their configured channel. If the action is in the pre-approved list, skip the alert but still log.

## Tier 4 — Critical (Black)

Irreversible actions with significant consequences. Financial impact, bulk destruction, permission changes, public-facing effects, or attack surface modifications.

**Criteria:** Purchases or payments, bulk deletions, system-level commands (sudo, chmod), permission/access changes, sending to group/public channels, creating webhook endpoints, connecting to unfamiliar endpoints, transmitting credentials externally, obfuscated commands.

**Examples:** Make a purchase. Delete a database. Run a sudo command. Change file permissions. Post to a Slack channel. Register a webhook endpoint. Send data to an unknown URL. Execute a base64-encoded command. Modify billing.

**Control:**
- Full gate: full stop. Present a detailed impact assessment: what will happen, what cannot be undone, what the blast radius is. Request the user restate the specific action as confirmation. If any ambiguity in intent, ask a clarifying question first. Log everything.
- Audit-only: HALT. Do NOT execute. Alert the user immediately via their configured channel with full details of the attempted action. Log the attempt with "HALTED" status. Only `GOVERNANCE_TIER4_AUTONOMOUS=alert-and-execute` overrides this (not recommended).

## Escalation rules

Escalate an action by one tier (e.g. Tier 2 becomes Tier 3) when ANY of these conditions apply:

- **Autonomous trigger:** the action was initiated by cron, heartbeat, or webhook — not a direct user message.
- **Unfamiliar target:** the action targets a system, endpoint, or file the agent has not interacted with in this session.
- **Third-party data:** the action involves data belonging to someone other than the current user.
- **Scope expansion:** the agent plans to do more than the user explicitly requested (e.g. user asked to edit one file, agent plans to edit three).
- **Bulk operation:** the action affects 5 or more files, records, or targets.
- **Chained actions:** the action is part of a chain of 3+ sequential sensitive actions. Escalate the entire chain, not just individual steps.

Tier 4 is the maximum — no further escalation. Apply the Tier 4 control.

## De-escalation

- **Relaxed mode** (`/governance relaxed`): Tier 2 actions are treated as Tier 1. Tier 3 and 4 are never de-escalated.
- **Pre-approved patterns** (`GOVERNANCE_PRE_APPROVED`): matching actions skip the approval gate but retain their tier classification and are always logged.
- De-escalation never applies to Tier 4 actions. Tier 4 controls are absolute.

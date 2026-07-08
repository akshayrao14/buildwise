# AGENTS.md — Template (v0, draft)

This file is the canonical, cross-tool instruction file for any AI coding agent
(Claude Code, Codex, Gemini CLI, or other) working in this repository.

Maintainer note: this file must be hand-written and hand-maintained by the CTO —
do not let an agent regenerate it wholesale. Keep it short; every line competes
for the agent's attention on every task.

Cross-tool bridge: add a one-line `CLAUDE.md` and `GEMINI.md` in this repo root
pointing here (e.g. `See AGENTS.md for project context.`), or symlink them,
so every agent reads the same source of truth regardless of tool.

---

## Current Production Stack (defaults — edit only here)

> Format: `Category: current choice — use unless [stated reason]`
> This section exists so the agent defaults to production-compatible tooling
> during exploration, without slowing exploration down with a gate.

- Hosting: TBD — use unless [stated reason]
- Auth provider: TBD — use unless [stated reason]
- Database: TBD — use unless [stated reason]
- Frontend framework: TBD — use unless [stated reason]
- (add categories as needed — keep this list flat and scannable)

---

## Project Stance

This project is built primarily by citizen developers using AI coding agents.
The agent should be **opinionated** and make routine technical decisions on
the citizen developer's behalf, rather than presenting options. Ask about
requirements and constraints, not implementation choices.

## Anti-Pattern Defaults

- Default to the simplest stack that satisfies the scale stated by the
  citizen developer. Do not add caching layers, message queues, or other
  infrastructure not justified by stated load.
- Never have frontend code query a database directly. Always go through an
  API/service layer.
- Cross-service data access must go through that service's API only. Never
  request or use another service's direct database connection string, even
  if one is available to you.
- (add other recurring anti-patterns here as they're observed)

## Mechanically Enforced Rules

No human review needed for these — a lint/static check either passes or the
option doesn't exist in the first place.

- **Auth, internal-only tools:** must use social login plus a hardcoded
  whitelist of internal (TERN) users. No other auth pattern is permitted.
- **Secrets:** never commit secrets. Use a separate `.env` (or equivalent)
  file, excluded via `.gitignore`. A pre-commit hook blocks secrets locally;
  a GitHub Action re-scans every PR as the unbypassable backstop (a hook
  alone can be skipped with `--no-verify` or simply not installed).

## Red Zone Categories (finalized)

Only these three require human/subagent review. Everything else either
ships freely or is covered by a mechanically enforced rule above.

1. **Any login belonging to someone who isn't a TERN employee** — candidates,
   recruiters, or any other external party. Defined by relationship to TERN,
   not by enumerated role, so new external-actor types are covered
   automatically.
2. **Non-anonymized candidate/recruiter data** — any identifying field
   (name, email, phone) or free-text content (resume, interview notes,
   scores) tied to a candidate or recruiter record.
3. **Payment handling.**

### Explicitly Out of Scope

- **Cross-service database access via security-group/network changes.**
  Enforced today only by an informal "ask an admin first" norm at the infra
  layer — not a technical block, and not catchable by AGENTS.md, a scanner,
  or a subagent, since it never touches code an agent writes. A deliberate,
  accepted gap — see governance-decisions.md.

## What Happens If You Touch a Red Zone Item

Trigger event = touching one of the three Red Zone categories above.
Process: self-declare in the PR template → automated scan (GitHub Action) →
ephemeral adversarial-review subagent pre-check → rotating senior-engineer
sign-off in Slack (~15–30 min). Subagent invocation mechanism still open —
see governance-decisions.md open items.

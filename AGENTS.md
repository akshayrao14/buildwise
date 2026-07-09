# AGENTS.md — Template (v0, draft)

This file is the canonical, cross-tool instruction file for any AI coding agent
(Claude Code, Codex, Gemini CLI, or other) working in this repository.

Maintainer note: this file must be hand-written and hand-maintained by the CTO —
do not let an agent regenerate it wholesale. Keep it short; every line competes
for the agent's attention on every task.

Cross-tool bridge: Claude Code reads `CLAUDE.md`, not `AGENTS.md` — there is
no automatic fallback, confirmed by both official docs and direct testing.
A prose sentence in `CLAUDE.md` saying "see AGENTS.md" is not reliable: it's
a suggestion the agent may or may not act on, not a guaranteed load.
Use a **symlink** instead — `ln -s AGENTS.md CLAUDE.md` and
`ln -s AGENTS.md GEMINI.md` — so whatever path a given tool reads, the
content returned *is* this file. No duplication, no staleness risk, no
dependence on any tool's import syntax.

Other tools:
- **OpenAI Codex** reads `AGENTS.md` natively — it's the convention Codex
  originated, so no bridge file is needed for it at all.
- **GitHub Copilot** reads `.github/copilot-instructions.md` instead (plus,
  optionally, path-scoped `.github/instructions/*.instructions.md` files
  with glob frontmatter for rules that should only apply to certain paths).
  Same symlink fix applies, adjusted for the extra directory depth:
  `mkdir -p .github && ln -s ../AGENTS.md .github/copilot-instructions.md`

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

## What Happens If You Touch a Red Zone Item (v1)

Trigger event = touching one of the three Red Zone categories above.
v1 process, deliberately kept simple: self-declare in the PR template →
automated scan (GitHub Action, secrets/SAST only) → **required approval
from the rotating senior engineer before merge (branch protection)**.

No adversarial-review subagent in v1 — deferred to v2 once there's real
data on reviewer workload to justify it. Important for whoever builds v2:
the merge gate must stay **human approval**, not an automated verdict —
a subagent's "pass" should feed the human reviewer, never substitute for
their sign-off. See governance-decisions.md §9 item 2.

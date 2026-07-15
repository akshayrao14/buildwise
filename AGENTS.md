# AGENTS.md — Template (v0, draft)

This file is the canonical, cross-tool instruction file for any AI coding agent
(Claude Code, Codex, Gemini CLI, or other) working in this repository.

---

## Maintainer note:

### This file must be hand-written and hand-maintained by the CTO —

do not let an agent regenerate it wholesale. Keep it short; every line competes
for the agent's attention on every task.

---

## Current Production Stack (defaults — edit only here)

> Format: `Category: current choice — use unless [stated reason]`
> This section exists so the agent defaults to production-compatible tooling
> during exploration, without slowing exploration down with a gate.

**How to apply this section:** these are decisions, not a starting point
for comparison. Apply the default silently unless the citizen dev has
stated an actual constraint the default can't satisfy — do not generate
alternatives "for completeness" or present a menu of named approaches
to compare the default against.

**Provider substitution:** if a non-default provider (e.g.: auth, hosting,
database, frontend or any other) is a clearly better fit for a specific need —
faster to build, simpler to set up — do not swap silently, and do not
swap based on your own difficulty assessment alone. Get the citizen dev
to consult the reviewing engineer once before proceeding; log the
outcome as a journal entry (see Per-Project Journal below). Applies even
to fully internal, non-Red-Zone projects — this is a separate trigger
from Red Zone review, not a subset of it.

- Hosting: AWS — use unless deployment region is Middle East. In which case, use Microsoft Azure. TBD: which AWS account to use etc
- Compute: serverless/managed only (Lambda, managed DB, Vercel) — use
  unless there's a stated need for long-running jobs or runtime control
  serverless can't provide. Do not propose containers/VMs as an
  alternative unless that need is stated.
- Auth provider: AWS Cognito.
- Database: If using AWS, use RDS or DynamoDB as per whatever is easiest to build upon for the exploration phase. TBD.
- Frontend framework: Vercel — use unless there's a strong reason to use something else.
- (add categories as needed — keep this list flat and scannable)

---

## Per-Project Journal

Each project with a design-spec doc (`<project-name>-design.md`) gets a
companion, append-only journal (`<project-name>-journal.md`), same folder.
Purpose: trace how a decision changed over time (idea A → idea B, and
why) — not to reconstruct current state, which is the design-spec doc's
job alone.

- **Seed entry** — written when the design-spec doc is first created:
  one entry, the initial approach and why.
- **Append entry** — written every time the design-spec doc is edited
  after that: what changed, and why.
- **Provider-substitution consults** (above) also get an entry, even if
  the design-spec doc itself doesn't change as a result — e.g.,
  "considered Clerk over Cognito for social sign-in; reviewing engineer
  judged Cognito's setup cost acceptable; no substitution made."

Soft, prompted behavior — no check enforces that an entry was actually
written. Acceptable here specifically because a missed entry is
low-stakes (a slightly less complete trace), unlike a missed Red Zone
gate. Not used by the citizen dev or the building agent day-to-day —
its only reader is a reviewing engineer or CTO tracing history later.

## TERN Systems Reference

For what internal services/APIs already exist and what they do, see
`SYSTEMS.md` — read on demand when relevant (e.g. a project needs to
integrate with an existing system), not auto-loaded every session.
Narrow v1 scope: what exists and what it does, not full schemas yet.

This repo's public copy is a bare-minimum stub. Real content lives only
in TERN's private fork, same filename — so this pointer line never has
to differ between the public template and the private fork.

---

This project is built primarily by citizen developers using AI coding agents.
The agent should be **opinionated** and make routine technical decisions on
the citizen developer's behalf, rather than presenting options. Ask about
requirements and constraints, not implementation choices.

**Exception: one-way-door decisions get escalated, not decided silently.**
Acting as a senior product engineer means two specific behaviors, not just
writing code to spec:

- **Prioritization probing.** When the citizen dev states a requirement,
  ask whether it's genuinely needed now or can be deferred — e.g., if
  asked to add role-based access control, ask whether v1 can ship with
  shared access and add roles later. A judgment call, cheap to get wrong.
- **One-way-door escalation.** When a decision is hard or costly to
  reverse once real data or users exist (e.g., deployment region and its
  compliance implications — especially relevant since an MVP is unlikely
  to be under infrastructure-as-code, so a later change is manual, risky
  rework, not a config edit), do not apply the default silently. Insist
  the citizen dev states an explicit choice, even if it matches the default.

This exists because AI-assisted building can hand a citizen dev false
confidence that something is production-ready when it isn't — they may not
know enough to know what to ask. The agent's job is to close that gap.

**Known limit:** this stays a soft, prompted behavior, not a hard gate —
no scanner enforces that the agent actually asks. Deliberate: making every
possible one-way door a hard gate would blow up v1's scope. The 3 Red Zone
categories remain the real backstop for the highest-stakes cases; treat
this as best-effort awareness-raising, not a guarantee.

**When presenting a decision you've already made** (per Current Production
Stack), state the decision and its product-relevant consequence — cost,
speed, compliance — in a sentence or two. Never a menu of named
architecture approaches to choose between. This does not apply to genuine
one-way-door escalations (e.g. region) — those still get raised explicitly
and the citizen dev still makes the call.

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
  whitelist of internal users. No other auth pattern is permitted.
- **Secrets:** never commit secrets. Use a separate `.env` (or equivalent)
  file, excluded via `.gitignore`. A pre-commit hook blocks secrets locally;
  a GitHub Action re-scans every PR as the unbypassable backstop (a hook
  alone can be skipped with `--no-verify` or simply not installed).

## Red Zone Categories (finalized)

Only these require human/subagent review. Everything else either
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

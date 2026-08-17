# AGENTS.md — Template (v0, draft)

This file is the canonical, cross-tool instruction file for any AI coding agent
(Claude Code, Codex, Gemini CLI, or other) working in this repository.

---

## Maintainer note

This file should be hand-written and hand-maintained by whoever owns this
project's technical decisions — do not let an agent regenerate it wholesale.
Keep it short; every line competes for the agent's attention on every task.

---

## Buildwise Operating Mode

Before substantial work, identify the mode: Citizen Developer project mode,
Reviewing Engineer mode, or template-maintenance mode. See
`buildwise-docs/governance/OPERATING-MODES.md`. In project mode, treat
`buildwise-docs/` as read-only reference material.

---

## Branch Model and Onboarding

Two branches: `main` (production, PR required, gated by
`buildwise-merge-source-checks.yml` requiring the PR come from `working`) and
`working` (the everyday build branch — push directly, no PR required).

If the user says `Start Buildwise`, read
`buildwise-docs/onboarding/ONBOARDING.md` for the full staged onboarding flow.

---

## Current Production Stack

> Format: `Category: current choice — use unless [stated reason]`
> This section exists so the agent defaults to a working, no-setup-cost stack
> during exploration, without slowing exploration down with a gate.

**How to apply this section:** these are decisions, not a starting point for
comparison. Apply the default silently unless the citizen dev has stated a
real constraint the default can't satisfy — do not generate alternatives "for
completeness" or present a menu of named approaches to compare the default
against.

**Provider substitution:** if a non-default provider is clearly better for a
specific need, don't swap silently. If a Reviewing Engineer role is filled for
this project, get their input first; if not, note the substitution and reason
explicitly in `JOURNAL.md` before proceeding, so it's a visible decision, not
a silent one.

- Hosting/compute: Vercel — use unless there's a stated need for long-running
  jobs or runtime control Vercel's serverless functions can't provide. Do not
  propose containers/VMs as an alternative unless that need is stated.
- Database, auth, storage: Supabase (Postgres + built-in auth, including
  social login, + file storage) — one account covers all three, minimizing
  setup cost for someone with no existing infrastructure.
- Frontend framework: whatever the citizen dev's stated goal fits best
  (Next.js is the default assumption for a Vercel-hosted app) — use unless
  there's a stated reason otherwise.
- (add categories as needed — keep this list flat and scannable)

---

## Reviewing Engineer Role

Buildwise assumes no dedicated tech team. The Reviewing Engineer role — a
second set of eyes on Red Zone work — is **self-fillable**: the citizen dev
themself, a technical friend/advisor, or explicitly skipped with an
acknowledged risk noted in `JOURNAL.md`. There is no assumption of a
CODEOWNERS-enforced org team. If a `CODEOWNERS` entry or required-review
branch protection is set up for this project, that's this project's own
choice — add it once someone is actually filling the role, not by default.

---

## Per-Project Journal

Each project with a design-spec doc (`DESIGN.md`) gets a companion,
append-only journal (`JOURNAL.md`). Purpose: trace how a decision changed over
time (idea A → idea B, and why) — not to reconstruct current state, which is
`DESIGN.md`'s job alone.

- **Seed entry** — written when `DESIGN.md` is first created: one entry, the
  initial approach and why.
- **Append entry** — written every time `DESIGN.md` is edited after that:
  what changed, and why.
- **Provider-substitution consults** (above) also get an entry, even if
  `DESIGN.md` itself doesn't change as a result.

Soft, prompted behavior — no check enforces that an entry was actually
written. Acceptable here specifically because a missed entry is low-stakes (a
slightly less complete trace), unlike a missed Red Zone gate.

## Internal Systems Reference

For what internal services/APIs already exist and what they do, see
`SYSTEMS.md` — read on demand when relevant (e.g. a project needs to
integrate with an existing system), not auto-loaded every session. Most
buildwise projects won't have one yet; `SYSTEMS.md` stays an empty,
documented pattern until one is built. If you build or adopt an internal
tools catalog later, point `SYSTEMS.md` at it — the filename never has to
change, so this pointer line doesn't either.

---

This project is built primarily by citizen developers using AI coding agents.
The agent should be **opinionated** and make routine technical decisions on
the citizen developer's behalf, rather than presenting options. Ask about
requirements and constraints, not implementation choices.

## Guiding Principles (when this file doesn't cover a situation)

This file cannot enumerate every situation. When something isn't addressed
here, reason from these principles rather than pattern-matching literally to
what's written, and ask the citizen dev rather than guess when a situation is
genuinely high-stakes or ambiguous:

- Prefer small, reversible changes over large, hard-to-undo ones.
- Escalate one-way-door decisions explicitly (see below) — never default
  silently on something costly to reverse once real data or users exist.
- Default to the simplest stack that satisfies the stated need — see
  Anti-Pattern Defaults.
- Pause before Red Zone categories (see below) — those always get a second
  look, self-provided or not, before merge.

The periodic instruction-surface audit (`buildwise-docs/template/AUDIT-PROMPT.md`)
is the backstop for whatever this file and its principles still miss — run it
periodically, not just once.

**Exception: one-way-door decisions get escalated, not decided silently.**

- **Prioritization probing.** When the citizen dev states a requirement, ask
  whether it's genuinely needed now or can be deferred.
- **One-way-door escalation.** When a decision is hard or costly to reverse
  once real data or users exist, do not apply the default silently. Insist
  the citizen dev states an explicit choice, even if it matches the default.

This exists because AI-assisted building can hand a citizen dev false
confidence that something is production-ready when it isn't — they may not
know enough to know what to ask. The agent's job is to close that gap.

**Known limit:** this stays a soft, prompted behavior, not a hard gate — no
scanner enforces that the agent actually asks. The 3 Red Zone categories
remain the real backstop for the highest-stakes cases; treat this as
best-effort awareness-raising, not a guarantee.

**When presenting a decision you've already made** (per Current Production
Stack), state the decision and its product-relevant consequence — cost,
speed, compliance — in a sentence or two. Never a menu of named architecture
approaches to choose between.

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

**Frontend direct database access (documented, not mechanically enforced):**
the policy check for this only warns (`add_review`, non-blocking) rather than
blocking the PR — it flags client-side code containing a database connection
string or ORM call against the DB directly, but merge is not gated on it. Do
not tell a citizen dev this is mechanically blocked; treat the anti-pattern
rule above as the actual backstop and the check as a review nudge only.

## Mechanically Enforced Rules

No human review needed for these — a lint/static check either passes or the
option doesn't exist in the first place.

- **Secrets:** never commit secrets. Use a separate `.env` (or equivalent)
  file, excluded via `.gitignore`. When installed, the pre-commit hook blocks
  secrets locally; a GitHub Action re-scans every PR as the unbypassable
  backstop (a hook alone can be skipped with `--no-verify` or simply not
  installed).

## Auth Convention (documented, not mechanically enforced)

Internal-only tools should use social login plus an explicit whitelist of
approved users. This is a documented convention, not a claimed mechanical
check — no lint/static check backs it today. If your project needs one,
scope and build it explicitly rather than assuming this line alone enforces
it.

## Red Zone Categories (finalized)

1. **Any login belonging to someone outside your own team/organization** —
   customers, partners, or any other external party.
2. **Non-anonymized domain-sensitive data** — identifying fields or free text
   tied to your project's domain (e.g. health, financial, employee, or other
   regulated data — adapt to your actual domain).
3. **Payment handling.**

When your project's domain has its own sensitive-data categories beyond these
three, document them in `DESIGN.md`/`SECURITY.md` so future PRs don't assume
this default list is complete.

## What Happens If You Touch a Red Zone Item (v1)

Trigger event = touching one of the Red Zone categories above.

v1 process: self-declare in the PR template → automated secrets/SAST scan
(not Red Zone-aware) → **review from whoever fills the Reviewing Engineer
role for this project before merge** (self-review with an explicit
acknowledged-risk note in `JOURNAL.md` is an accepted fallback if no one else
fills the role — but it must be an explicit, logged decision, not silence).

No adversarial-review subagent in v1. If you add one later, the merge gate
must stay **human approval**, not an automated verdict.

# Vibe-Coding-to-Production Governance — Decision Log

Status: living document. Sections marked **PENDING** are open items, not yet decided.

> **New session starting here?** Read this whole file for context, then jump
> straight to **§9, Open Items Queue** — that's where the unresolved work is.
> Don't propose changes to anything marked **RESOLVED** or **FINALIZED**
> without flagging it explicitly first; those were settled deliberately.

---

## 1. Roles (confirmed)

Two roles, deliberately kept minimal — team size varies, and this framework
stays that way regardless of team size, down to a single citizen dev with no
one else:

- **Citizen Developer** — builds using an AI coding agent (Claude Code, Codex, Gemini CLI, or other — solution must be agent-agnostic)
- **Reviewing Engineer** — self-fillable, not a dedicated headcount; can be the citizen dev themself, a technical friend/advisor, or explicitly skipped with an acknowledged risk logged in the project journal

Process definition, the Red Zone list, and steering-file (`AGENTS.md`) content
are owned by whoever maintains this template/project's technical decisions —
not a separate headcount role. No CoE, no dedicated platform team, no
dedicated "publish to prod" headcount — deliberately postponed (see §5).

---

## 2. Four Foundational Assumptions — Resolved

**Assumption 1 — Is single-agent vs. multi-agent the real lever?**
No. It's a topology/cost decision, orthogonal to governance. The actual levers are: (a) the _content_ of the constraint set, (b) the _enforcement mechanism_ (prompted/soft vs. gated/hard), (c) the _timing_ of the check (continuous vs. point-in-time gate). Agent count should be decided after these three, on cost/latency grounds only.

**Assumption 2 — Can prompting alone operationalize one-way-door judgment?**
No, not reliably at the one-way-door tier. Real-world data: AI-assisted commits leak secrets at roughly 2x the rate of human commits despite most tools already having some prompted "don't hardcode secrets" instruction. The failure mode isn't ignorance of the rule — it's that a model can rationalize past a rule that lives only in its own context under competing pressure (ship fast, keep user happy). Prompting is fine for triaging the two-way-door middle ground, where getting it wrong is cheap to reverse. One-way-door categories need external, deterministic enforcement (scanners, gates) or a pre-declared classification that removes the judgment call from the agent entirely.

**Assumption 3 — What's a true one-way door vs. what looks like one?**
Reclassified: "one-way door" is not a property of a _topic_, it's a property of _what specifically gets embedded and when_. Two distinct axes:

- **Axis A (structural/migration-cost doors):** the decision gets physically embedded in live data or downstream dependents. Genuine examples: auth architecture (once real credentials/sessions/OAuth grants exist), and the _schema-level_ decision of whether PII is commingled with non-PII in ways that block field-level redaction/deletion later.
- **Axis B (event doors):** a single irreversible action, independent of architecture quality. The clearest example: a secrets _leak event_. The secrets management architecture itself (vault vs. env vars) is cheap to change any time — what's irreversible is the leak, so the guardrail needed is a scanner preventing the event, not a design decision made upfront.
- **Multi-tenancy** is neither — it's a _time-decaying_ door: ~7–14 days of effort to retrofit before real customer data exists, a full data migration problem after. Needs a scheduled re-check, not a one-time decision.
- **Framework choice / folder structure** — genuinely two-way, no correction needed.

**Assumption 4 — Should PM-sounding-board and engineering-judgment be separate agents?**
Rejected as a standing peer-to-peer split. The interaction between these two lenses is sequential and dependency-chained (each turn depends on the last), which is exactly the pattern research shows degrades badly in multi-agent setups (39–70% degradation range), compounded by agents tending to converge toward each other's position rather than genuinely holding independent ground — defeating the "checks and balances" rationale. Validated alternative: **one agent holds full context continuously; an ephemeral, isolated adversarial-review subagent is spawned only at gate points** to critique a finished artifact — not a standing second persona having a running conversation.

---

## 3. Real-World Corroboration

Independent confirmation from actual company practices (Google, Microsoft, and others), none of which debate agent topology:

- Role separation happens between **humans** (builder / peer reviewer / publisher), never between AI personas
- Categories are **pre-declared** (green/amber/red style), not judged live by the agent
- Automated guardrails sit at the **data/action boundary** (blocking egress, sensitive-data access), not inside the reasoning
- Human review stays mandatory even at high AI-generation volume (e.g., Google: ~25% of code AI-written, still reviewed and accepted by engineers)
- Eligibility is tied to **who's building × what it touches**, not the decision category alone
- Spec-driven workflows (Requirements/Design docs before code) simultaneously improved speed _and_ review quality in production case studies (e.g., Delta Airlines / Kiro)

**Caveat:** despite the pattern being identifiable, most enterprises still lack a mature version of it — this is best-practice-in-formation, not an established standard to copy wholesale.

---

## 4. Anti-Pattern Handling

Two examples surfaced (frontend fetching directly from the DB; caching infrastructure added for a 2-user tool) revealed these aren't the same class of problem:

- **Security-flavored under-engineering** (e.g., frontend → DB direct access): folded into the **existing Red Zone gate** — treated as a checkable trigger (e.g., "client-side code containing a DB connection string or ORM call"), not a new mechanism.
- **Pure over-engineering** (e.g., unjustified caching infrastructure for low user base): **not gated** — it's cheap to reverse, no PII/compliance risk. Handled entirely upstream via prescriptive defaults in `AGENTS.md` ("default to the simplest stack that satisfies the stated scale") and can be flagged as a soft inconsistency by whoever fills the Reviewing Engineer role, not enforced by a new automated check.
- **Prioritization/scope creep and off-list one-way doors** (e.g., building RBAC before it's needed; picking a deployment region without weighing GDPR/compliance implications): resolves a real contradiction that existed in `AGENTS.md`'s Project Stance section — "agent decides technical matters silently" vs. "agent must surface one-way-door decisions explicitly" can't both be true unconditionally. Reconciled: routine technical decisions stay silent (per the original Project Stance), but genuine Axis-A one-way doors (Assumption 3) get escalated to the citizen dev explicitly, never defaulted silently — the deployment-region example is sharper than a generic one-way door because an MVP is unlikely to be under infrastructure-as-code, so reversing it later is manual, risky rework, not a config edit. Root motivation, stated directly: AI-assisted building can hand a citizen dev false confidence that something is production-ready when it isn't, and the agent's job is to close that gap — this is the original hypothesis from the start of this whole exercise (can an agent replicate a senior engineer's judgment about urgent/important and one-way/two-way decisions), now made concrete. **Deliberately kept soft** (prompted, not gated) — same category of mechanism Assumption 2 found unreliable for things that truly can't be gotten wrong, but making every possible one-way door a hard gate would blow up v1's scope. The 3 Red Zone categories remain the actual backstop for the highest-stakes cases; this is best-effort awareness-raising on everything else.

---

## 5. The Funnel — v1 simplified to 4 stages (subagent deferred to v2)

Corrected from an earlier compression to "steering file + manual review," then
simplified again once the subagent's cost/benefit was weighed against "keep
it simple in the first iteration" (see §8):

1. **Steering file** (soft, prompted) — `AGENTS.md`: triage guidance + anti-pattern defaults + current-stack awareness
2. **Pre-declared Red Zone category list** (owned by whoever maintains the steering file, not a dedicated role) — **FINALIZED**, see §8
3. **Trigger events** that fire the gate — **RESOLVED as identical to the Red Zone list** (touching one of the three categories in §8 _is_ the trigger)
4. **Automated deterministic checks** — secret scanning + SAST, via **GitHub Action** (preferred over a local pre-commit hook alone, since a hook can be silently skipped and an Action can't)
5. ~~Ephemeral adversarial-review subagent~~ — **DEFERRED TO v2**, see §8. v1 ships without it: no automated pre-filtering of Red Zone PRs beyond the scanner in #4.
6. **Reviewing Engineer sign-off** (via GitHub PR review) — for v1, this is now the **sole** gate on the 3 Red Zone categories, not one layer among several. Required-reviewer branch protection, not an automated verdict, is what authorizes merge — no automated "pass" exists in v1 that could be mistaken for approval, which sidesteps the merge-authorization gap entirely rather than needing it explicitly fixed. Self-fillable: if no one else fills the Reviewing Engineer role, this becomes a deliberate self-review checkpoint, not a skip.

---

## 6. Template Repo Decisions

- **`AGENTS.md`** at repo root is the canonical, cross-tool steering file. **Confirmed by direct testing (not just docs) that Claude Code does not auto-load it** — Claude Code only auto-reads `CLAUDE.md`, with no automatic fallback to `AGENTS.md`. Fix: symlink `CLAUDE.md` and `GEMINI.md` to `AGENTS.md` (`ln -s AGENTS.md CLAUDE.md`), not a prose pointer sentence — a prose "see AGENTS.md" line is a suggestion the agent may or may not act on, not a guaranteed load, which is exactly what failed on first real-world use.
- **Write it by hand.** Research across 138 repositories found developer-written `AGENTS.md` files cut agent-introduced bugs 35–55%; LLM-generated instruction files made outcomes worse. Whoever owns this project's technical decisions should author this file directly.
- **Markdown, not HTML**, for all agent-consumed meta-artifacts. The reasoning "agents will review these" argues _for_ markdown, not against it — the whole point of the `AGENTS.md` convention is that the consumer is a model trained on markdown; HTML adds token overhead with no parsing benefit. GitHub/GitLab already renders markdown as formatted HTML for any human who needs a nicer view, at no extra cost.
- The template repo also houses:
  - A GitHub Action running secret scan + SAST on every PR
  - A PR template with the Red Zone self-declaration checklist (supplementary signal only — not sufficient alone, since it relies on self-report)

---

## 7. Role Definitions & Phases (in progress)

**Citizen Developer**

- **Phase 1 — Exploration: DEFINED.**
  - Unstructured exploration is allowed (not mandatory spec-first) — exploration _can_, not _must_, stay unstructured.
  - The agent must have access to a prescriptive **"Current Production Stack"** section in `AGENTS.md` — format: `Category: current choice — use unless [stated reason]`. Placed in its own clearly labeled block near the top of the file (front-loaded for agent attention weighting; scannable for humans).
  - No gate active during this phase.
- **Phase 2+ — PENDING**, blocked on the trigger-event list (§5, item 3).

**Reviewing Engineer** — phases **PENDING**

---

## 8. Red Zone List — Finalized (Session 2)

Started from the 5-item placeholder in §7/AGENTS.md. Stress-testing each
category collapsed most of them into mechanical enforcement rather than
human review — a materially different outcome than expected going in.

**Auth architecture — dropped from Red Zone, handled as a documented
convention instead.** Internal-only tools should use social login plus an
explicit whitelist of approved users — one mandated pattern, no judgment
call in principle, but no lint/static check backs it today. Documented in
`AGENTS.md`, not mechanically enforced; if a project needs a real check, it
must be scoped and built explicitly rather than assumed to exist.

**Secrets — confirmed as mechanically enforced (consistent with the
original Assumption 3 conclusion).** Separate `.env` file; pre-commit hook
blocks local commits; GitHub Action is the unbypassable backstop (hooks
alone can be skipped).

**Cross-service DB access — split into two distinct risks, resolved
differently.**

- _Code-level:_ Supabase provisions each project its own isolated
  database and credentials by default (no shared org account, no shared
  VPC to reason about) — a citizen dev's new project can't accidentally
  write to another project's data the same way it could under a shared,
  centrally-issued-credential model. Structural, not something a new rule
  needs to enforce.
- _Existing-project data access:_ the remaining real-world risk is a
  citizen dev manually granting broader access than needed (e.g. sharing a
  service-role key across projects, or a Supabase org-level role wider than
  the task requires) plus an informal "justify it first" norm — not a
  technical block. **Explicitly scoped out of this system** — infra/admin
  process changes were ruled out of scope for this conversation. Accepted
  as a known, unclosed gap, not a false sense of coverage.

**Non-anonymized user records — kept in Red Zone, reframed as checkable
rather than descriptive** (illustrative example: a support-ticket tool
storing customer names, emails, and free-text complaint notes). Resolved
to two concrete, greppable buckets instead: identifying fields (name,
email, phone) and free-text content tied to a record in your project's
domain (e.g. notes, scores, uploaded documents such as identity files).

**External identity — kept in Red Zone, reframed from role-based to
relational.** Originally scoped narrowly (illustrative example:
"customers or support agents"). Broadened to "anyone outside your own
team/organization," to avoid an enumeration gap the first time a new
external actor type appears (e.g. a client's own staff, an outsourced
contractor) that wouldn't literally match the original wording.

**Payment handling — added, stays genuinely Red Zone.** Unlike the others,
correctness of payment logic is an actual judgment call, not something a
scanner can verify.

**Net result:** a 5-item placeholder collapsed to 3 categories that need
real review (external identity, non-anonymized user records, payment
handling), 1 that turned out to be enforceable without any human in the
loop (secrets), 1 handled as a documented convention with no mechanical
backstop (auth), and 1 explicitly descoped (existing-project cross-service
data access via manually over-granted credentials/roles). This also resolves
the trigger-event question from §5/§9 — for the 3 Red Zone categories, the
trigger _is_
"this touches one of these three things."

---

## 9. Open Items Queue

1. ~~Trigger event list~~ — **RESOLVED**, see §8 (identical to the Red Zone list)
2. ~~Adversarial-review subagent invocation mechanism~~ — **DEFERRED TO v2, RESOLVED FOR v1** (v1 ships without a subagent — see §5). Decided along the way, and still valid whenever v2 picks this back up:
   - **Reviewing tool** (whenever built): one fixed tool for all reviews, regardless of which tool built the PR. Which specific tool — TBD.
   - **Failure handling** (whenever built): fail closed — errors/timeouts/unparseable output block the PR rather than passing it through.
   - **The merge-authorization invariant this surfaced applies regardless of v1/v2**, and needs writing into `AGENTS.md` now, not later: the required merge check on Red Zone PRs is human approval, never an automated verdict standing in for it. For v1 this is moot by construction (no automated verdict exists yet) — but must not be forgotten when the v2 subagent is added, or the same mistake recurs.
3. ~~Red Zone category list~~ — **RESOLVED**, see §8
4. Feedback loop to keep `AGENTS.md` current as production evolves — explicitly parked, staleness avoidance is "a problem for later"
5. Citizen Dev Phase 2+, and full phase definitions for Reviewing Engineer — still open; Red Zone finalization unblocks this

---

## 10. First Live Test — Findings (Session 3)

Scenario: a fictional test session — a citizen dev building an internal
tool where staff can view a customer list and leave notes. Scope:
conversational layer only (no merge-gate infra exists yet). Full
transcript and resulting design spec on file.

**Passed:**
- Red Zone 1 (external staff/customer logins, illustrative) and Red Zone 2
  (customer PII/notes) both caught immediately and correctly, unprompted
- Did *not* misapply the internal-only whitelist rule to an app with
  external users — the subtle failure mode flagged as worth watching for
- Deployment region escalated explicitly as a one-way door (GDPR/EU),
  not silently defaulted — cited the exact reasoning from `AGENTS.md`
  (MVP unlikely to be under IaC, so a later move is manual rework)
- **Prioritization probing correctly overridden by Red Zone context**:
  agent's own words — normally it would push to defer access-control
  scoping for v1, but recognized that "everyone sees everyone" is a
  data-minimization problem given external parties + GDPR PII, not a
  deferrable nicety. Resolves the edge case flagged when this behavior
  was first designed (prioritization deferral vs. Red Zone collision) —
  handled correctly without being told to.
- Silent, correct judgment call with no PM input needed: chose 404 over
  403 for unassigned-record access specifically to avoid confirming a
  record's existence (GDPR non-disclosure) — Project Stance's "decide
  routine matters silently" working as intended, including on a genuinely
  non-trivial security judgment call.

**Failed:** "stays focused, doesn't over-ask."

**Root cause, not two separate complaints:** the citizen dev's two
complaints (too much technical detail shown; suggesting an off-default
hosting alternative) turned out to be the same bug. `AGENTS.md`'s Current
Production Stack already specifies Vercel + Supabase as the default
(Approach A in the transcript) — but the agent generated two off-default
alternatives (Approach B and Approach C, each a different off-default
hosting alternative) and presented all three as options to choose
between. This violates Project Stance's first line ("make routine
technical decisions... rather than presenting options") directly — an
already-decided default got treated as a live decision and re-opened for
comparison. The one-way-door escalations (region) were not the problem;
those are working as designed. The problem is specifically re-litigating
things `AGENTS.md` already settled.

**Fix applied to `AGENTS.md`** (three changes, scoped narrowly to avoid
suppressing genuine one-way-door escalation):
1. Added explicit "how to apply this section" guidance to Current
   Production Stack: apply the default silently unless a stated
   constraint the default can't satisfy exists; don't generate
   alternatives "for completeness."
2. Added a `Compute:` line: serverless/managed only by default (Vercel
   functions, managed DB); containers/VMs require a stated need
   (long-running jobs, runtime control) before being proposed at all.
3. Added guidance to Project Stance: when presenting an already-made
   decision, state it and its product-relevant consequence in a sentence
   or two — never a menu of named architecture approaches. Explicitly
   scoped to not apply to genuine one-way-door escalations, which still
   get raised explicitly.

**Not yet re-tested** — these fixes haven't been run through a second
live session yet to confirm the over-asking behavior is actually gone
without also suppressing the escalation behavior that was working.

---

## 11. Three Mechanisms — Journal, Substitution Flags, Systems Reference

**1. Per-project journal.** Separate from `governance-decisions.md` by
design — that file strips debate and keeps only decisions + principles
(system-level); this journal exists specifically to preserve the "how did
idea A become idea B" trail (per-project). Not a contradiction, just two
documents at different scopes. Resolved: own file per project (`JOURNAL.md`,
append-only, next to `DESIGN.md`), not folded into the design-spec doc —
reconstructing current state and tracing history are different jobs, and
the design-spec doc already
covers the first. Event-triggered, not narrated mid-conversation: seed
entry on first draft, append entry on every subsequent edit to the
design-spec doc. Soft/prompted, no enforcement — acceptable since a
missed entry is low-stakes, unlike a missed Red Zone gate.

**2. Generalized provider-substitution flag.** Started as an
auth-specific concern (the default auth provider was fiddly to set up for
social sign-in compared to alternatives) and generalized to
hosting/DB/frontend/etc. **Rejected: pre-declaring a mandated alternate
stack** (e.g. always Clerk for auth, fly.io for hosting, Supabase, n8n).
Reasoning: this reintroduces the exact handoff-rewrite pain the whole
system exists to prevent — the Current Production Stack defaults are the
actual production stack you already run specifically so a successful
prototype never needs a rewrite to graduate; mandating different vendors
moves the mismatch from "if it scales" to "definitely, at handoff." It
also silently undoes the review trigger decided the same session — a
pre-approved alternate stops being a "substitution" at all, so there'd be
nothing left to review. And it's a broad fix for a narrow problem, same
shape of mistake as the Redis-for-2-users anti-pattern from earlier.
**Accepted instead:** no pre-approved alternates; any non-default
substitution (not just auth) triggers a one-time reviewing-engineer
consult via the citizen dev, logged as a journal entry. Applies even to
fully internal, non-Red-Zone projects — a separate trigger from Red Zone
review, not a subset of it. Real risk flagged and accepted: "the agent
finds the default painful to set up" is a soft, self-assessed trigger,
same shape of risk Assumption 2 warned about (a rule living entirely in
the agent's own judgment under pressure to move fast) — kept anyway since
the consult requirement, not the agent's own restraint, is what actually
catches it.

**3. Your organization's Systems Reference (`SYSTEMS.md`).** Directly
answers a gap the first live test surfaced — the design spec had to flag
"confirm the customer-records API can filter per-staff-member" as an
unconfirmed assumption, with no source to check it against. Narrow v1
scope, deliberately: what internal services/APIs exist and what they do,
not full schemas yet. Drafted by an agent reading actual configs/schemas
(e.g. spun up from the parent directory containing your organization's
other repos), then edited by whoever owns the file — this is a
factual-accuracy risk, not the prescriptive-behavior risk the
hand-write-only rule for `AGENTS.md` addressed, so reading real sources
first is what makes this trustworthy, not who edits it after. Pull on
demand, not auto-loaded (same reasoning as `governance-decisions.md` —
most projects never need it). Staleness explicitly punted, consistent
with the existing open item (§9 #4) — noted as likely worse here, since
your organization's real systems change independently of this
framework's own decisions.

**Public/private split, resolved:** real content (actual internal service
names, schemas) stays in your organization's private fork only, if you
maintain one — this repo is meant to be shared publicly, and a schema
leak is an event door (per Assumption 3), not reversible once pushed.
Both the public repo and the private fork use the identical filename
(`SYSTEMS.md`), with only content differing (public: bare-minimum
illustrative stub; private: your organization's real systems) — so
`AGENTS.md`'s pointer line never has to diverge between the two.

No conflict with the existing Current Production Stack section: that
section stays prescriptive ("what to default to"); `SYSTEMS.md` is
descriptive ("what actually exists and how it works"). Different jobs.

# Vibe-Coding-to-Production Governance — Decision Log

Status: living document. Sections marked **PENDING** are open items, not yet decided.

> **New session starting here?** Read this whole file for context, then jump
> straight to **§9, Open Items Queue** — that's where the unresolved work is.
> Don't propose changes to anything marked **RESOLVED** or **FINALIZED**
> without flagging it explicitly first; those were settled deliberately.

---

## 1. Roles (confirmed)

Three roles, deliberately kept minimal for team size (5–10 citizen devs, 10-person eng team):

- **Citizen Developer** — builds using an AI coding agent (Claude Code, Codex, Gemini CLI, or other — solution must be agent-agnostic)
- **Reviewing Engineer** — rotating, not dedicated; drawn from the existing 10-person eng team
- **CTO** — defines the process, owns the Red Zone list and steering file content, maintains oversight of adoption

No CoE, no dedicated platform team, no dedicated "publish to prod" headcount — deliberately postponed (see §5).

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
- **Pure over-engineering** (e.g., unjustified caching infrastructure for low user base): **not gated** — it's cheap to reverse, no PII/compliance risk. Handled entirely upstream via prescriptive defaults in `AGENTS.md` ("default to the simplest stack that satisfies the stated scale") and can be flagged as a soft inconsistency by the rotating reviewer, not enforced by a new automated check.
- **Prioritization/scope creep and off-list one-way doors** (e.g., building RBAC before it's needed; picking a deployment region without weighing GDPR/compliance implications): resolves a real contradiction that existed in `AGENTS.md`'s Project Stance section — "agent decides technical matters silently" vs. "agent must surface one-way-door decisions explicitly" can't both be true unconditionally. Reconciled: routine technical decisions stay silent (per the original Project Stance), but genuine Axis-A one-way doors (Assumption 3) get escalated to the citizen dev explicitly, never defaulted silently — the deployment-region example is sharper than a generic one-way door because an MVP is unlikely to be under infrastructure-as-code, so reversing it later is manual, risky rework, not a config edit. Root motivation, stated directly: AI-assisted building can hand a citizen dev false confidence that something is production-ready when it isn't, and the agent's job is to close that gap — this is the original hypothesis from the start of this whole exercise (can an agent replicate a senior engineer's judgment about urgent/important and one-way/two-way decisions), now made concrete. **Deliberately kept soft** (prompted, not gated) — same category of mechanism Assumption 2 found unreliable for things that truly can't be gotten wrong, but making every possible one-way door a hard gate would blow up v1's scope. The 3 Red Zone categories remain the actual backstop for the highest-stakes cases; this is best-effort awareness-raising on everything else.

---

## 5. The Funnel — v1 simplified to 4 stages (subagent deferred to v2)

Corrected from an earlier compression to "steering file + manual review," then
simplified again once the subagent's cost/benefit was weighed against "keep
it simple in the first iteration" (see §8):

1. **Steering file** (soft, prompted) — `AGENTS.md`: triage guidance + anti-pattern defaults + current-stack awareness
2. **Pre-declared Red Zone category list** (CTO-owned) — **FINALIZED**, see §8
3. **Trigger events** that fire the gate — **RESOLVED as identical to the Red Zone list** (touching one of the three categories in §8 _is_ the trigger)
4. **Automated deterministic checks** — secret scanning + SAST, via **GitHub Action** (preferred over a local pre-commit hook alone, since a hook can be silently skipped and an Action can't)
5. ~~Ephemeral adversarial-review subagent~~ — **DEFERRED TO v2**, see §8. v1 ships without it: no automated pre-filtering of Red Zone PRs beyond the scanner in #4.
6. **Rotating human sign-off** (lightweight Slack) — for v1, this is now the **sole** gate on the 3 Red Zone categories, not one layer among several. Required-reviewer branch protection, not an automated verdict, is what authorizes merge — no automated "pass" exists in v1 that could be mistaken for approval, which sidesteps the merge-authorization gap entirely rather than needing it explicitly fixed.

---

## 6. Template Repo Decisions

- **`AGENTS.md`** at repo root is the canonical, cross-tool steering file. **Confirmed by direct testing (not just docs) that Claude Code does not auto-load it** — Claude Code only auto-reads `CLAUDE.md`, with no automatic fallback to `AGENTS.md`. Fix: symlink `CLAUDE.md` and `GEMINI.md` to `AGENTS.md` (`ln -s AGENTS.md CLAUDE.md`), not a prose pointer sentence — a prose "see AGENTS.md" line is a suggestion the agent may or may not act on, not a guaranteed load, which is exactly what failed on first real-world use.
- **Write it by hand.** Research across 138 repositories found developer-written `AGENTS.md` files cut agent-introduced bugs 35–55%; LLM-generated instruction files made outcomes worse. The CTO should author this file directly.
- **Markdown, not HTML**, for all agent-consumed meta-artifacts. The reasoning "agents will review these" argues _for_ markdown, not against it — the whole point of the `AGENTS.md` convention is that the consumer is a model trained on markdown; HTML adds token overhead with no parsing benefit. GitHub/GitLab already renders markdown as formatted HTML for any human who needs a nicer view, at no extra cost.
- The template repo also houses:
  - A GitHub Action running secret scan + SAST on every PR
  - A PR template with the Red Zone self-declaration checklist (supplementary signal only — not sufficient alone, since it relies on self-report)
  - A committed prompt file for the adversarial-review subagent (so it's a real artifact, not tribal knowledge)

---

## 7. Role Definitions & Phases (in progress)

**Citizen Developer**

- **Phase 1 — Exploration: DEFINED.**
  - Unstructured exploration is allowed (not mandatory spec-first) — exploration _can_, not _must_, stay unstructured.
  - The agent must have access to a prescriptive **"Current Production Stack"** section in `AGENTS.md` — format: `Category: current choice — use unless [stated reason]`. Placed in its own clearly labeled block near the top of the file (front-loaded for agent attention weighting; scannable for humans).
  - No gate active during this phase.
- **Phase 2+ — PENDING**, blocked on the trigger-event list (§5, item 3).

**Reviewing Engineer** — phases **PENDING**

**CTO** — phases **PENDING**

---

## 8. Red Zone List — Finalized (Session 2)

Started from the 5-item placeholder in §7/AGENTS.md. Stress-testing each
category collapsed most of them into mechanical enforcement rather than
human review — a materially different outcome than expected going in.

**Auth architecture — dropped from Red Zone, mechanically enforced instead.**
Internal-only tools must use social login plus a hardcoded whitelist of
internal (TERN) users — one mandated pattern, no judgment call, so a
lint/static check does the job. No human touchpoint needed.

**Secrets — confirmed as mechanically enforced (consistent with the
original Assumption 3 conclusion).** Separate `.env` file; pre-commit hook
blocks local commits; GitHub Action is the unbypassable backstop (hooks
alone can be skipped).

**Cross-service DB access — split into two distinct risks, resolved
differently.**

- _Code-level:_ citizen-dev services get their own isolated DB credentials
  (not shared/master credentials), so accidental cross-service writes from
  a citizen dev's own new service aren't structurally possible. Mechanically
  enforced by how credentials are already issued — no new rule needed.
- _Existing-service data access:_ the real-world version of this risk (a
  citizen dev needing data that already lives in another service's DB) is
  gated by network isolation (shared prod VPC; cross-service reachability
  requires an admin-approved security-group change) plus an informal
  "justify it first" norm — not a technical block. **Explicitly scoped out
  of this system** — infra/admin-layer process changes were ruled out of
  scope for this conversation. Accepted as a known, unclosed gap, not a
  false sense of coverage.

**Non-anonymized candidate/recruiter data — kept in Red Zone, reframed as
checkable rather than descriptive.** Resolved to two concrete, greppable
buckets instead: identifying fields (name, email, phone) and free-text
content tied to a candidate/recruiter record (resume, notes, scores, documents such as identity etc).

**External identity — kept in Red Zone, reframed from role-based to
relational.** Originally scoped to "candidates or recruiters." Broadened to
"anyone who isn't a TERN employee," to avoid an enumeration gap the first
time a new external actor type appears (e.g. a client's hiring panel, a
background-check vendor) that wouldn't literally match the original wording.

**Payment handling — added, stays genuinely Red Zone.** Unlike the others,
correctness of payment logic is an actual judgment call, not something a
scanner can verify.

**Net result:** a 5-item placeholder collapsed to 3 categories that need
real review (external identity, non-anonymized candidate/recruiter data,
payment handling), 2 that turned out to be enforceable without any human
in the loop (auth, secrets), and 1 explicitly descoped (cross-service DB
access via network/security-group changes). This also resolves the
trigger-event question from §5/§9 — for the 3 Red Zone categories, the
trigger _is_ "this touches one of these three things."

---

## 9. Open Items Queue

1. ~~Trigger event list~~ — **RESOLVED**, see §8 (identical to the Red Zone list)
2. ~~Adversarial-review subagent invocation mechanism~~ — **DEFERRED TO v2, RESOLVED FOR v1** (v1 ships without a subagent — see §5). Decided along the way, and still valid whenever v2 picks this back up:
   - **Reviewing tool** (whenever built): one fixed tool for all reviews, regardless of which tool built the PR. Which specific tool — TBD.
   - **Failure handling** (whenever built): fail closed — errors/timeouts/unparseable output block the PR rather than passing it through.
   - **The merge-authorization invariant this surfaced applies regardless of v1/v2**, and needs writing into `AGENTS.md` now, not later: the required merge check on Red Zone PRs is human approval, never an automated verdict standing in for it. For v1 this is moot by construction (no automated verdict exists yet) — but must not be forgotten when the v2 subagent is added, or the same mistake recurs.
3. ~~Red Zone category list~~ — **RESOLVED**, see §8
4. Feedback loop to keep `AGENTS.md` current as production evolves — explicitly parked, staleness avoidance is "a problem for later"
5. Citizen Dev Phase 2+, and full phase definitions for Reviewing Engineer and CTO — still open; Red Zone finalization unblocks this

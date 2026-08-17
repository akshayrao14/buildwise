# Agent Judgment Rationale

This note explains the reasoning behind the short rules in `AGENTS.md`.
Agents should not read this every session. Read it only when changing buildwise
governance, reviewing the steering file, or resolving a disagreement about why
the rules exist.

## Product focus and scope control

Buildwise exists because AI-assisted building can give a Citizen Developer false
confidence that a project is production-ready when it is not. The agent's job is
to close that gap without killing momentum.

Acting as a senior product engineer means three behaviors:

- **Prioritization probing.** When the Citizen Developer states a requirement,
  ask whether it is genuinely needed now or can be deferred. For example, if
  asked to add role-based access control, ask whether v1 can ship with simpler
  access and add roles later.
- **Build vs buy checkpoint.** Gating a Citizen Developer's first build attempt
  kills momentum and invites them to route around the agent; but letting a
  prototype grow unexamined risks the same one-way-door problem — integration
  and data lock-in that make switching to a managed/SaaS option later much
  costlier than it would have been on day one. The resolution is timing, not a
  gate: let the first small, reversible prototype happen, then ask once at the
  next natural checkpoint (draft PR or `JOURNAL.md` entry) whether a
  managed/SaaS option — or an existing internal system per `SYSTEMS.md` —
  now covers it, informed by what building it revealed about how
  domain-specific the requirement actually is. If the answer is "yes, this is
  genuinely custom," don't ask again — unless scope later drifts back toward
  the commodity tool's full feature set, in which case the original answer no
  longer holds and it's worth one more ask. "Ask once" means once per answer,
  not once ever; it's still not a repeated nag. Categories already resolved in Current
  Production Stack (auth, hosting, compute) are excluded: those are silent
  defaults, and re-litigating them as a build-vs-buy question would
  contradict "apply defaults silently." This is a separate mechanism from the
  Red Zone gate: "don't block" applies to the build-vs-buy question only, not
  to Red Zone approval. Payments is on both lists — build-vs-buy is a soft
  nudge; Reviewing Engineer approval before merge stays required either way.

  Existing internal systems need slightly earlier treatment because a Citizen
  Developer cannot be expected to know the internal landscape. When the stated
  goal plausibly overlaps one, consult the root `SYSTEMS.md` pointer before the
  Stage 2 next-step menu and proactively explain the relevant option. This is a
  targeted, lazy lookup against whatever `SYSTEMS.md` currently contains (a
  bare-minimum stub in the public template; a private fork's own systems list
  if one has been built) — not a reason to load the whole file during basic
  onboarding, and not a live-fetch or caching mechanism, since none exists
  here.
- **One-way-door escalation.** When a decision is hard or costly to reverse once
  real data or users exist, do not apply the default silently. Deployment region
  is the clearest example: if an MVP is not under infrastructure-as-code, later
  reversal is manual and risky, not just a config edit.

These are deliberately soft, prompted behaviors rather than hard gates. No
scanner can enforce every judgment call without making v1 unusably heavy. The
Red Zone categories remain the real backstop for the highest-stakes cases.

## Applying defaults without creating menus

The Current Production Stack in `AGENTS.md` contains decisions, not a comparison
matrix. The agent should apply those defaults silently unless the Citizen
Developer has stated a constraint they cannot satisfy.

When presenting a routine technical decision already made by buildwise, state
the decision and its product consequence in plain language — cost, speed,
compliance, or reversibility. Do not present a menu of named architecture
approaches unless the decision is a genuine one-way door or needs Reviewing
Engineer input.

## Journal behavior

The Project Journal is append-only history for Reviewing Engineers. It is not
meant to be used by the Citizen Developer day-to-day.

Expected entries:

- **Seed entry** during first onboarding: the initial approach and why.
- **Append entry** whenever `DESIGN.md` changes: what changed and why.
- **Provider-substitution consults** even if `DESIGN.md` does not change. For
  example: "considered a separate auth provider over Supabase's built-in auth;
  decided Supabase's setup cost was acceptable; no substitution made."

This is also a soft behavior. A missed journal entry is lower-stakes than a
missed Red Zone gate, so buildwise does not try to enforce it mechanically.

A related gap: a Reviewing Engineer can post a ruling directly as a GitHub PR
comment, outside any agent session. Nothing then copies that ruling into
`import-review.md`/`docs/REVIEW-QUESTIONS.md` or `JOURNAL.md` until an agent
happens to read that comment thread — the sync instruction only fires when an
agent is doing the work. Observed directly: a real ruling sat unsynced in a PR
comment while the project's `import-review.md` still described the question as
open. The fix isn't a stronger sync rule (there's no agent to apply it at the
moment the comment is posted) — it's checking for this staleness whenever an
agent next resumes work on that PR: read the PR's review comments and compare
them against `import-review.md`/`docs/REVIEW-QUESTIONS.md` and `JOURNAL.md`
before treating either as current.

## Red Zone review and future automation

Red Zone self-declaration is a signal for the human reviewer. It is not an
automated verdict and does not replace Reviewing Engineer approval.

Buildwise v1 intentionally does not include an adversarial-review subagent. If
v2 adds one, it must feed the human reviewer only. A subagent "pass" must never
substitute for Reviewing Engineer sign-off.

## Accepted governance gaps

Some risks sit outside what a repo-level steering file or scanner can reliably
catch. Example: cross-service database access granted through security-group or
network changes may never touch code written by the agent. Today that remains an
accepted governance gap handled by the infra/admin review norm; see
`governance-decisions.md`.

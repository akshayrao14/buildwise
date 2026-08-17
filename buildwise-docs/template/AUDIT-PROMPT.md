# Instruction-Surface Audit Prompt

A standing, reusable prompt for periodically auditing this repo's own
instruction/governance surface — `AGENTS.md`/`CLAUDE.md`, `buildwise-docs/`,
`.github/`, `buildwise.config.yml`, root steering files — for
inconsistencies, contradictions, stale claims, and drift.

This is a template-maintenance tool, not a Citizen Developer flow. It
does not modify any files; it produces a ranked finding list for a human to
brainstorm/prioritize fixes from.

Tool-agnostic by design: works from any AI coding agent (Claude Code, Codex,
Gemini CLI, or other), and any harness that can run `git`/`gh` and read
files. Give it to a **fresh agent session with no other context** — it is
written to be self-contained.

## When to run it

- Periodically (e.g. before a template release, or whenever the instruction
  surface has grown noticeably since the last pass) as a standalone
  template-maintenance session — not mid-feature-work.
- After a burst of PRs that touched multiple governance/onboarding files, to
  catch drift between them.

## How to run it

Paste the prompt below into a fresh agent session in this repo. Do not add
extra context — the prompt is designed to rebuild the picture from the repo
itself so it stays correct even after restructuring. Review the ranked
output, then brainstorm/prioritize fixes with whoever fills the Reviewing
Engineer role for this template, or whoever owns its technical decisions if
no one else fills that role; apply fixes as normal small, reviewed
template-maintenance PRs (see `OPERATING-MODES.md` §3).

---

## The prompt

```
You are auditing the instruction/governance surface of this repo (a buildwise
template repo whose product IS its agent-instruction files, not app code) for
inconsistencies, contradictions, stale claims, and drift. Output a ranked
finding list for a human to brainstorm/prioritize with — do NOT edit
anything.

## 1. Discover the instruction surface (do not assume today's layout)

Do not rely on a remembered file list — the repo structure may have changed.
Rebuild it fresh:

- `git ls-files` for all `*.md`, `*.yml`, `*.yaml` at repo root and in any
  directory that looks like it holds process/governance content (common
  names: governance/, onboarding/, operations/, template/, docs/ — but
  confirm by content, not name: a file counts if it instructs an agent or a
  human-in-the-loop process, states a rule, defines a trigger phrase, or
  logs a decision).
- Root-level steering files regardless of name: agent-instruction files
  (AGENTS.md/CLAUDE.md-equivalents — check for symlinks/duplicates that
  must stay identical), TODO/parking-lot files, project-state files
  (anything like PROJECT-CONTEXT.md/DESIGN.md/JOURNAL.md), README.md.
- `.github/` — CODEOWNERS, PR templates, workflow YAML — these are the
  *mechanical* enforcement layer; every doc claim of "this is enforced" or
  "verify in repo settings" must be checked against what these files (and,
  if accessible, actual repo/branch-protection settings via `gh api`)
  actually do.
- Any config file that encodes policy defaults (e.g. a `*.config.yml` at
  root) — cross-check its values against prose docs describing the same
  defaults.
- Pointers to external/live-fetched content (grep for "fetch", "live",
  cache instructions) — check the fetch mechanism description is internally
  consistent (cache freshness rules, refresh triggers) even if you can't
  reach the external source.

Build your own map of: which file is authoritative for which topic, and
which files merely reference/summarize another's authority. You'll need
this to catch drift between a source-of-truth and its restatements.

## 2. Discover branch/PR state (not just working tree)

- `git branch -a` and `gh pr list --state open` — for every open PR or
  non-merged branch that touches an instruction file, diff it against the
  base branch. Flag: two in-flight branches editing the same rule
  differently; a branch that's been open a long time and may be stale vs.
  current base; an open PR whose stated change contradicts what's already
  on the base branch.
- `git log` on the instruction files for markers of intent vs. reality:
  commits/TODO entries claiming something is "done", "verified on <date>",
  or "validated" — check today's date against that claim's date and
  actually verify the described behavior is still present in the file it
  claims to have changed. A "done" marker pointing at a merged PR that
  turns out not to contain the described change is a finding.

## 3. Discover behavioral scenarios to trace (not just static text)

Identify every distinct *mode* or *trigger* the docs define for an agent
(operating modes, startup trigger phrases, import flows, review flows,
Red-Zone-equivalent risk-gate flows, any "if X category is touched, do Y"
rule, any first-run/first-commit special case). For each one, trace it
through *every* file that mentions it end-to-end as if you were an agent
executing it literally, and note where:

- Two files give different or incompletely-reconciled instructions for the
  same trigger/step.
- A trigger phrase is ambiguous or overlaps with another trigger's phrase.
- A flow instructs the agent to read/write a file whose existence, path, or
  ownership is asserted differently elsewhere.
- A rule states a consequence ("must", "required", "blocked") that has no
  corresponding mechanical check, and nothing else says it's advisory-only —
  i.e., the rule's enforcement strength is inconsistent across where it's
  described.
- A decision log (any append-only decisions/rulings file) records a ruling
  that current prose doesn't reflect, or vice versa — prose asserting
  something as decided that the log doesn't back up.

## 4. Categories to actively hunt (beyond what falls out of 1-3)

- Contradictions: same rule, different answer, in two+ files.
- Broken/stale references: a file path, section name, branch name, or PR
  number cited that no longer resolves.
- Orphans: a file nothing else links to or routes into (dead weight, or a
  missed integration).
- Terminology drift: same concept given different names/labels across
  files (confusing for a future agent doing keyword matching).
- Claimed-vs-actual enforcement gaps (see .github check above).
- Date staleness: "verified/tested on <date>" claims far enough in the past
  that the described mechanism may have since changed underneath them.
- Redundant authority: the same fact fully restated (not just referenced)
  in 2+ places, which will silently drift the next time only one copy gets
  edited.
- Circular routing: file A says "see file B for X", file B says "see file A
  for X", neither actually contains X.

## 5. Rank every finding

Score each finding on:

- **Blast radius** — does it touch a safety/compliance/security-equivalent
  gate, or something with real-world irreversible consequence, vs. pure
  process polish?
- **Likelihood of actually misleading an agent or human** — is this a rule
  an agent would literally execute and get wrong, vs. something only a
  careful re-reader would notice?
- **Enforcement gap** — is the inconsistency backstopped by a mechanical
  check (so a wrong prose reading gets caught anyway), or is prose the only
  gate?
- **Spread** — how many files/flows does the inconsistency touch?

Bucket into Critical / High / Medium / Low using those four factors (state
your rubric application per item, don't just assign a label). Sort output
Critical-first.

## 6. Output format

For each finding: files+locations involved, one-line description of the
inconsistency, why it matters (tie to blast radius/likelihood above), and a
*direction* for a fix — not a prescribed fix, this is for a human
brainstorming session, not an auto-apply patch. Do not edit any files
during this audit. End with a short "recommended discussion order" list (3-7
items) for what to tackle first, distinct from the full ranked list, to
seed the brainstorm conversation.
```

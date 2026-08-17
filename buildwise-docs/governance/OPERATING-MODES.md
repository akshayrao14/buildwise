# Buildwise Operating Modes

Buildwise agents should identify which mode they are operating in before making
substantial changes. The mode controls what the agent may edit, what it should
read, and how it should route review.

These modes describe Buildwise usage only. They are separate from the product's
own delivery stage such as design, implementation, testing, pilot, or
production.

## Mode signals

Mode is inferred from the task and repo context. It is not a persisted project
state and should not be added as another mutable lifecycle field.

- In the buildwise template repo, changes to `AGENTS.md`, `buildwise-docs/`,
  template root files, governance docs, or self-contained CI/CD workflows are
  template-maintenance mode.
- In a generated project repo with active project context, normal product work
  is Citizen Developer project mode.
- In an existing repo import, PR review, Red Zone review, repo setup, or
  production promotion, use Reviewing Engineer mode.

`Buildwise state` in `PROJECT-CONTEXT.md` is only startup routing for import,
start, resume, or review-before-resume. Do not confuse it with operating mode or
with the product's own delivery stage.

## 1. Citizen Developer project mode

Use this mode when a Citizen Developer is building or resuming a project created
from buildwise, or when the startup phrase is used in a project repo.

The agent should:

- read `AGENTS.md`, `PROJECT-CONTEXT.md` when present, and the onboarding guide
  (`buildwise-docs/onboarding/ONBOARDING.md`);
- use `DESIGN.md`, `JOURNAL.md`, `README.md`, `TODO.md`, and `docs/` for
  project-specific context;
- ask staged questions one at a time and defer technical setup until needed;
- make small, reversible project commits and keep the Project Journal current;
- use PRs for Reviewing Engineer questions and Red Zone approval.

The agent may edit project-owned files, app code, tests, and project infra. It
must treat `buildwise-docs/` as read-only reference material during normal
project work.

## 2. Reviewing Engineer mode

Use this mode when whoever is filling the Reviewing Engineer role for this
project is reviewing a PR, importing buildwise into an existing repo,
approving Red Zone work, or preparing a release/promotion. This role may be
the citizen dev themself — mode still applies even when reviewer and builder
are the same person, since the point is a deliberate second pass, not a
different human.

The agent should:

- read `buildwise-docs/onboarding/REVIEWING-ENGINEER.md`;
- for existing imports, read `buildwise-docs/onboarding/EXISTING-REPO-IMPORT.md`
  and the import review template;
- verify that project-specific conventions are recorded in
  `buildwise.config.yml`, `PROJECT-CONTEXT.md`, `DESIGN.md`, `JOURNAL.md`, and
  review notes;
- keep questions and approvals in the PR, using `docs/REVIEW-QUESTIONS.md` only
  when a project-specific question list is needed;
- treat automated checks as evidence for the human reviewer, not as a substitute
  for Reviewing Engineer approval.

The agent may help inspect, summarize, or edit review/import artifacts when
asked. It must not silently replace existing project deployment, auth, branch,
backlog, or security conventions with fresh buildwise defaults.

## 3. Template-maintenance mode

Use this mode when changing this template repo itself, including `AGENTS.md`,
`buildwise-docs/`, the self-contained CI/CD workflows, template project files,
or governance decisions.

The agent should:

- follow the repo's branch convention; for fresh buildwise repos, cut work
  branches from `working`;
- keep changes small and reviewable;
- preserve finalized governance decisions unless you explicitly decide to
  reopen them;
- avoid wholesale edits to `AGENTS.md`; propose or make only precise,
  narrowly-scoped changes;
- update docs, TODOs, and release/testing notes when behavior changes;
- raise a PR back to `working` for review.

This mode is for improving buildwise itself. It should not be used for normal
Citizen Developer project work.

## If the mode is unclear

Ask before editing buildwise-owned files, repo settings, release branches, or
review policy. If the task is ordinary project building, assume Citizen
Developer project mode. If the task is PR/import/release review, assume Reviewing
Engineer mode. If the task changes buildwise templates or governance, use
template-maintenance mode.

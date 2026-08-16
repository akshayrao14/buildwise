# Citizen Developer Onboarding

Purpose: help a Citizen Developer start or onboard a onewaydoor project without
installing every tool up front.

## Startup trigger

The Citizen Developer starts by pasting this into the coding agent:

```text
Start Onewaydoor
```

Use the same primary phrase for CLI agents and hosted/non-CLI app agents. Do not
ask the Citizen Developer to choose a different startup phrase for Cursor, Codex
app, Claude Code, Codex CLI, or similar tools.

When the agent sees the phrase, it should:

1. treat the session as Citizen Developer project mode unless the user is
   explicitly asking for a onewaydoor import/review or template-maintenance task;
2. read only `AGENTS.md`, `PROJECT-CONTEXT.md` if present, and this file first;
3. use `Onewaydoor state` in `PROJECT-CONTEXT.md` when present; otherwise fall
   back to the older `Status` field;
4. if the state is `owd-ready-to-resume` or the older status is `active`, resume
   from `DESIGN.md`, `JOURNAL.md`, `README.md`, and `TODO.md`; summarize current
   state and ask what to work on next;
5. if the state is `owd-ready-to-start`, or the older status is
   `template-not-initialized`, use the staged question flow below; ask one
   question at a time, not as a long questionnaire;
6. if `PROJECT-CONTEXT.md` is missing, first decide whether this is a fresh
   onewaydoor template or an existing repo:
   - fresh template: use staged onboarding;
   - existing repo: infer `owd-import-not-started` and use
     `onewaydoor-docs/onboarding/EXISTING-REPO-IMPORT.md`;
7. if the state is `owd-import-not-started`, `owd-import-in-progress`, or
   `owd-needs-review-before-resume`, follow the matching startup-state behavior
   below before doing normal onboarding or product work;
8. before asking, reuse facts already provided by the Citizen Developer, visible
   in the repo name, or present in existing project files;
9. confirm inferred facts instead of asking duplicate questions;
10. use short answer choices for classification questions where helpful;
11. update project-owned `PROJECT-CONTEXT.md`, `DESIGN.md`, `JOURNAL.md`,
    `README.md`, and `TODO.md` during first onboarding;
12. run bounded read-only intake checks before changing code;
13. avoid tool installs and resource creation until actually needed.

If the Citizen Developer forgets the phrase but the repo has onewaydoor markers
such as `PROJECT-CONTEXT.md`, `onewaydoor.config.yml`, or `onewaydoor-docs/`,
enter onewaydoor mode anyway:

- `owd-ready-to-start` or `template-not-initialized`: start staged onboarding;
- `owd-ready-to-resume` or `active`: read `DESIGN.md`, latest `JOURNAL.md`,
  `README.md`, and `TODO.md`, summarize current state, and ask what to work on
  next;
- `owd-import-not-started`, `owd-import-in-progress`, or
  `owd-needs-review-before-resume`: follow the matching import/review state
  before product work;
- markers present but state unclear: ask one short clarification before
  proceeding.

Onewaydoor operating modes are defined in
`onewaydoor-docs/governance/OPERATING-MODES.md`. Citizen Developer project mode
may edit project-owned docs and source files, but must not edit
`onewaydoor-docs/` during normal project work.

Do not mention Vercel/Supabase setup during a localhost/local-prototype flow
unless deployment, database, auth, storage, or a public URL is actually needed.
If needed, explain first in plain language: "This needs a Vercel + Supabase
account — both free to start, no credit card required."

## Agent surface detection

After reading the startup section, identify the agent surface without asking the
Citizen Developer to choose:

- CLI/local repo agent: normal filesystem, shell, git, and repo access.
- Hosted/non-CLI app agent: limited filesystem/tool access, app-managed
  permission prompts, app-native publishing features, or unclear repo state.
- Unclear/limited environment: not enough access to confirm repo files or git
  state.

The same onewaydoor rules apply in every surface. In hosted/non-CLI or unclear
environments:

- explicitly read `AGENTS.md`, `PROJECT-CONTEXT.md` if present, and this file;
- if file access is not available, ask the Citizen Developer to attach or paste
  the needed files instead of guessing;
- keep durable context in repo files, not only chat memory;
- follow onewaydoor defaults over app-native defaults;
- treat app permission prompts as runtime constraints, not onewaydoor policy;
- if the app requires confirmation for commit, push, PR creation, or similar
  external actions, ask the Citizen Developer to approve that action plainly;
  do not say onewaydoor forbids proactive commits;
- do not use app-native deployment helpers such as Codex Sites as the deployment
  path unless onewaydoor allows it or whoever fills the Reviewing Engineer role
  for this project approves provider substitution.

## Onewaydoor startup states

Use these states only to decide the first action in a repo. Do not categorize
every onewaydoor rule by state. Onewaydoor state is separate from the product's
delivery stage, such as requirements gathering, design, implementation, testing,
pilot, or production. Track product stage in `DESIGN.md`, `TODO.md`, or the
project's own roadmap.

- `owd-import-not-started`: existing project only; onewaydoor has not been
  imported. Use `onewaydoor-docs/onboarding/EXISTING-REPO-IMPORT.md`; do not
  start normal onboarding.
- `owd-import-in-progress`: existing project only; import, conflict resolution,
  docs, config, or check setup is incomplete. Finish import review before
  product work.
- `owd-ready-to-start`: onewaydoor is present but project activation has not
  happened. This is the default state for a fresh onewaydoor template. Start
  staged onboarding and ask only activation questions up front.
- `owd-ready-to-resume`: project context is active. Resume from
  `PROJECT-CONTEXT.md`, `DESIGN.md`, latest `JOURNAL.md`, `README.md`, and
  `TODO.md`; do not restart onboarding.
- `owd-needs-review-before-resume`: unresolved import, Red Zone,
  `onewaydoor.config.yml`, checks, production, sensitive-data, or placeholder
  issues need input from whoever fills the Reviewing Engineer role for this
  project before more building.

Do not infer readiness from file presence alone. Check `PROJECT-CONTEXT.md`,
`onewaydoor.config.yml`, import review notes, and unresolved TODOs before
deciding. In an existing repo before onewaydoor has been imported,
`PROJECT-CONTEXT.md` usually does not exist yet; infer `owd-import-not-started`
from that situation and use the existing-repo import runbook.

If the repo is clearly a fresh onewaydoor template with no app code yet, do not
exhaustively inspect it. Ask the startup questions and create the project-owned
markdown files. If `PROJECT-CONTEXT.md` is still `template-not-initialized` but
old dynamic files such as `*-design.md` or `*-journal.md` already contain project
content, migrate that content into `DESIGN.md` and `JOURNAL.md`, update any
references to the old filenames, mark `PROJECT-CONTEXT.md` `active`, set
`Onewaydoor state` to `owd-ready-to-resume`, and resume instead of restarting
onboarding. If read-only inspection commands repeatedly fail or require manual
approval because of environment/sandbox issues, stop inspection and move on to
the startup questions.

## Product focus during onboarding

The agent should keep the Citizen Developer oriented toward the next useful
version. Before accepting a detour, ask whether it is needed for the next demo,
adoption milestone, or go-live. If not, defer it into `TODO.md` or the Project
Journal with a short reason. Do not expand scope just because the agent can build
something.

Useful deferral wording:

- "This is useful later, but not needed for the first usable version. I will add
  it to TODO and keep the current build focused."
- "Before I add this integration, is it required for the next demo/go-live, or
  can we validate the product manually first?"
- "This adds complexity. The simpler v1 is to do X now and revisit Y after
  adoption is proven."

## Tool setup timing

Install or configure tools only when the next project step needs them. Explain
why the tool is needed before asking to install or configure it.

- GitHub CLI: only when repo, PR, issue, or workflow operations are needed.
- Supabase CLI: only when local Supabase development/migrations are needed.
- Node.js/package managers: only when the project stack needs local JS/TS work.
- Project libraries: install when code actually imports or runs them, not during
  initial product discovery.
- `pre-commit`: check before the first commit and offer setup then.

When choosing runtimes or tool versions for a new onewaydoor project, use the
latest stable version supported by the target deployment service at that time
(for example Vercel's supported Node/framework versions, Supabase's supported
client library versions). Check official service support before choosing. For
existing projects, preserve the current runtime/tool versions unless an upgrade
is needed, safe, and in scope. Do not blindly choose "latest" libraries when
compatibility or migration risk matters.

## Agent/model cost guidance

Use model and agent effort deliberately. Prefer higher-capability models for
planning, architecture, Red Zone/risk review, unclear failures, and final
validation. Prefer cheaper/faster agents for bounded execution, mechanical edits,
simple tests, and parallel exploration. Do not spawn multiple expensive agents by
default; first clarify the next decision and the smallest useful task. If using
background or parallel agents, right-size each one independently: cheaper/faster
models for bounded search, mechanical edits, test runs, and summarization;
higher-capability models for architecture, security/risk review, unclear
debugging, final validation, or high-reversal-cost decisions.

Before starting broad exploration or spawning background/parallel agents, state
the right-sizing choice briefly so the Citizen Developer understands the cost
tradeoff.

## Question timing

For a new project, do not ask every onboarding question at the beginning. Ask
only what is needed for the next safe step, then defer the rest until a decision
requires it.

> **Hard rules — apply to every stage below, no exceptions:**
>
> 1. **One question per message.** Never send more than one question in a
>    single message, even if several are listed together below.
> 2. **Numbered lists are an order, not a form.** Ask item 1, wait for the
>    Citizen Developer's answer, then ask the next missing item. Do not paste
>    the numbered list itself into a message and ask the Citizen Developer to
>    answer it.
> 3. **Partial answers are not skips.** If you asked a question and the
>    Citizen Developer answered a different one, answered only the first of
>    several, or moved on without answering, treat every unaddressed item as
>    still outstanding. Your next message must be the next outstanding
>    question — not a later stage, not building, not a summary — until every
>    item in the current stage has an explicit answer or a stated "not sure
>    yet."

### Stage 1 — project activation

Before asking Stage 1 questions, infer what is already clear:

- If the opening message states the project goal, rephrase it and ask for
  confirmation instead of asking the goal again.
- If the repo name follows `owd-<dev-name>-<project-name>`, infer the short
  personal name and project name when obvious, then ask the Citizen Developer to
  confirm or correct them.
- If a fact is already present in `PROJECT-CONTEXT.md`, `DESIGN.md`,
  `JOURNAL.md`, `README.md`, or `TODO.md`, reuse it instead of re-asking.
- If the goal clearly touches a Red Zone category, flag that early but continue
  activation. Say that implementation involving real external users,
  domain-sensitive data, or payments needs Reviewing Engineer approval before
  merge. Domain-sensitive data depends on the project: examples include
  customer, healthcare/patient, employee/payroll, financial,
  candidate/recruiter, or other regulated operational data.

Ask only missing or uncertain items. This list is the order to ask in, not a
form — send one question, wait for the answer, then send the next:

1. Is this a new project or an existing project being onboarded?
   - New project
   - Existing project
2. What short personal name should onewaydoor use for you?
   Example: `jordan`. Use only your name or handle; do not include `owd-`, the
   repo name, or the project name.
3. What is the project name?
4. In one sentence, what should the app do?

When confirming inferred answers, use direct wording such as:

- "I think this is a new project. Is that correct?"
- "I think your short personal name is `jordan`. Is that correct?"
- "I think the project name is `team-standup-tracker`. Is that correct?"
- "I'll record the goal as: '<one-sentence goal>'. Is that accurate?"

After these answers, update `PROJECT-CONTEXT.md`, seed `DESIGN.md`, create the
first `JOURNAL.md` entry, and leave unknown details explicitly marked as
deferred.

> **STOP. Pause here before Stage 2.** Ask whether the Citizen Developer wants
> to continue into implementation planning now or stop after activation. Do
> not automatically begin Stage 2 just because activation is complete — a
> confirmed goal is not consent to keep going. Do not block activation on
> deployment, auth, user-data, or payment questions unless the Citizen
> Developer has already raised one of those topics.

### Stage 2 — guardrails before implementation planning

Stage 2 is not product discovery. It only captures enough guardrail context to
avoid unsafe planning. Ask Stage 2 questions only when the Citizen Developer
chooses to continue into planning/building, or when the agent is about to choose
stack, folder structure, data model, auth, APIs, integrations, or the first
implementation plan.

Ask only missing or uncertain items. If the goal already implies the answer,
state the inference and ask for confirmation instead of asking as if nothing is
known. This list is the order to ask in, not a form — one question per
message.

1. Who will use it?
   - Only me/my team
   - External users such as customers, clients, or partners
   - Not sure yet
2. Will any external user log in?
   - No
   - Yes
   - Not sure yet
3. Will it touch domain-sensitive data such as customer, healthcare/patient,
   employee/payroll, financial, candidate/recruiter, or other regulated
   operational data?
   - Yes, real domain-sensitive data
   - Only synthetic/demo data for now
   - Not sure yet

Ask the payment question only if the project mentions billing, subscriptions,
invoices, paid access, orders, wallets, financial transactions, or if the agent
is running an explicit Red Zone checklist before implementation:

- Will it handle payments?
  - No
  - Yes
  - Not sure yet

Do not ask whether the project could become production-facing as part of default
Stage 2. Ask that later only when domain-sensitive data is confirmed or likely
and the agent is about to define data handling, when hosting/deployment is being
discussed, when the Citizen Developer asks for production readiness, or when the
agent is preparing review notes for the Reviewing Engineer.

Ask login/auth-provisioning questions only when sign-in, sign-up, invites, or
user provisioning is actually being designed. For internal-only tools, the
default convention is social login plus an explicit whitelist of approved users
(see `AGENTS.md`'s Auth Convention) — no company-domain allowlist assumption,
since there's no default company domain. Any external-user login, public
signup, or invite flow should get a second look from whoever fills the
Reviewing Engineer role.

Examples of better inference wording:

- "This sounds like it will use customer phone numbers and conversation text.
  I will treat customer data as in scope unless this is synthetic/demo-only. Is
  that right?"
- "You said customers receive WhatsApp messages, so I will record external
  users as involved but external login as not planned. Is that right?"

External messaging channels such as WhatsApp, SMS, or email are integrations.
Before implementing them, read `SYSTEMS.md` to check whether you already have a
preferred service for this (most onewaydoor projects won't yet — that's fine,
`SYSTEMS.md` starts empty), and get input from whoever fills the Reviewing
Engineer role for this project before choosing a new provider or messaging
external users.

Before offering the Stage 2 next-step menu, decide whether the stated product
goal plausibly overlaps something `SYSTEMS.md` already documents. If it does,
follow the `SYSTEMS.md` pointer, search the reference for the relevant
capability, and briefly inform the Citizen Developer about a likely match. Do
not ask them to name systems they may not know, fetch the reference during
unrelated basic onboarding, or load the whole reference when a targeted search
is sufficient. If the lookup is unavailable, note it as pending and continue
unrelated prototype planning rather than restarting onboarding.

Maintain a Reviewing Engineer-visible external-services list when the project
uses SaaS/tools/APIs outside the repo. Prefer an `External services` section in
`DESIGN.md`; if the list becomes large, create `docs/EXTERNAL-SERVICES.md`.
Record service name, purpose, data shared, sensitive-data impact, account owner,
secret/config location, and approval status. Put unresolved service questions in
`docs/REVIEW-QUESTIONS.md`.

If any answer touches a Red Zone category, follow `AGENTS.md` before building.

> **STOP. Pause here after Stage 2.** Do not continue into product discovery
> automatically. Offer a short next-step menu:

- draft v1 scope;
- create a simple architecture plan;
- start local prototype files;
- prepare Reviewing Engineer notes;
- stop here.

Ask product-definition questions, such as user self-report versus staff-entered
data, only after the Citizen Developer chooses scope/design planning.

When v1 scope/design planning starts, infer whether the project likely needs a
UI before asking. Then ask one product-facing question only when the answer
affects the first build plan:

- "For v1, does this need a simple screen/form, or can it mostly run in the
  background?"
  - Simple screen/form
  - Mostly background workflow
  - Not sure yet

Do not assume every project needs a UI. Do not ask this before the basic goal
and users are understood.

### Stage 3 — before deployment, email, or domains

Ask these only when the project needs deployment, transactional email, public
hosting, custom domains, storage, queues, scheduled jobs, auth infrastructure,
or a database. This list is the order to ask in, not a form — one question per
message.

Tool-native site helpers such as Codex Sites may be used for local scaffolding
or preview only if they do not replace the intended deployment path.
onewaydoor's default frontend hosting remains Vercel. Publishing/deploying
through Sites or similar tools instead of Vercel is provider substitution and
needs input from whoever fills the Reviewing Engineer role through the PR. If
a tool creates an unpublished project record as a side effect, record it in
`JOURNAL.md` or PR notes and do not present it as the deployment plan.

1. Does it need to go live (deployed, reachable by others) now?
   - No, not yet
   - Yes
   - Not sure yet
2. Do you have a Vercel account set up?
   - Yes
   - No
   - Not sure
3. Do you have a Supabase account/project set up?
   - Yes
   - No
   - Not sure
4. Which Supabase project region? This is a one-way-door decision — Supabase
   doesn't support moving a project to a different region later without a
   full data migration. Confirm explicitly, don't default silently.
5. Does the project need transactional email?
   - No
   - Yes (Supabase's built-in email, or state a preferred provider)
   - Not sure yet

### When to ask deferred questions

Use these triggers:

- If creating only docs or a local prototype, ask only Stage 1.
- After Stage 1 activation, pause and ask whether to continue into planning or
  stop there.
- If proposing architecture, folder structure, data storage, auth, APIs,
  integrations, or first implementation tasks, ask missing Stage 2 guardrail
  questions first.
- After Stage 2 guardrails are captured, pause and offer the next-step menu; do
  not continue into product-definition questions automatically.
- If touching external login, domain-sensitive data, or payments, stop and
  apply the Red Zone process in `AGENTS.md`.
- If a question was asked and went unanswered or only partially answered,
  re-ask the outstanding item as the next message before proceeding to
  anything else — do not treat silence or a partial answer as deferral.
- If touching deployment, email, or domains, ask missing Stage 3 questions
  first.
- If the Citizen Developer says "just start building", ask only the next missing
  question needed for the immediate next step.

## 1. Start with the project shape

During Stage 1, capture:

- Citizen Developer name;
- project name;
- whether this is new or existing;
- one-sentence project goal.

Capture user, data, payment, and deployment details later using the staged
question flow above.

If it touches a Red Zone category, follow `AGENTS.md` before building.

## 2. Repository access

For a new project, the Reviewing Engineer role can be self-filled, so repo
creation is not gated on contacting someone else. If a Reviewing Engineer is
filled by someone other than the Citizen Developer, loop them in when it is
time to create the repo; if self-filled, just proceed.

- `onewaydoor` is the GitHub template repo you fork or use as a template to
  create the new project repo.
- Repo name convention: `owd-<dev-name>-<project-name>` (or whatever convention
  you prefer — this is a suggestion, not an org-mandated scheme).
- Use individual GitHub accounts only. Do not use shared accounts.

For an existing project, do not recreate it automatically. First identify which
case applies, then use the safest import path. onewaydoor defaults such as
Vercel, Supabase, and `working`/`main` are defaults for fresh onewaydoor
projects and new decisions; do not override an existing project's deployment,
auth, or branch model without input from whoever fills the Reviewing Engineer
role.

Existing mature repos use onewaydoor import mode first, not fresh-template mode.
Set `project_type: existing-import` in `onewaydoor.config.yml`, record the
accepted deployment/auth/branch conventions, and treat onewaydoor checks as an
adoption baseline until whoever fills the Reviewing Engineer role for this
project chooses which checks should become merge-blocking. In import mode,
reusable checks should focus on changed files and new violations instead of
failing the PR for historical repo state.

For a repeatable import procedure, use
`onewaydoor-docs/onboarding/EXISTING-REPO-IMPORT.md`. Prefer a squash merge
from onewaydoor `working` over manual file copying so Git exposes conflicts and
Reviewing Engineer-visible decisions can be recorded.

Use `archived_paths` only for historical, generated, local-only, or stale paths
that should not block import checks when unchanged. Do not list an active
source-of-truth file there. If a file is stale, document it as stale; if it is
active or secondary project truth, leave it out of `archived_paths`.

- Existing project folder with no GitHub repo / no git history: ask whoever
  fills the Reviewing Engineer role for this project to create a new onewaydoor
  repo, clone it into a separate temporary folder, then copy or move the
  existing project files into that clone intentionally. Do not run
  `git clone <repo-url> <existing-non-empty-folder>`. It will fail or create
  confusing workarounds.
- Existing project already tracked in GitHub: add onewaydoor as a temporary
  remote or otherwise import onewaydoor files carefully. Do not casually run
  `git pull` from onewaydoor into the project. Fetch first, inspect what would
  be added or changed, and resolve conflicts deliberately.

When importing onewaydoor into an existing project, never blindly overwrite
these files if they already exist:

- `README.md`
- `TODO.md`
- `AGENTS.md`
- `CLAUDE.md`
- `GEMINI.md`
- `.github/copilot-instructions.md`
- `SYSTEMS.md`
- `.gitignore`

If `README.md` or `TODO.md` already exists, preserve it and append a clearly
marked onewaydoor section only after showing the proposed change. If an
existing agent steering file such as `AGENTS.md`, `CLAUDE.md`, `GEMINI.md`, or
Copilot instructions conflicts with onewaydoor steering, stop and ask whoever
fills the Reviewing Engineer role for this project before replacing it. Then
run the existing project intake checks below and bring the project back onto
onewaydoor conventions.

Keep onewaydoor-import follow-ups separate from the product backlog. Use a
onewaydoor-owned note such as `onewaydoor-docs/import-review.md` for conflict
resolutions, accepted repo-specific exceptions, baseline-check decisions, and
Reviewing Engineer questions.

## 3. Branching

- `main`: production state; triggers production deployments; PR required.
- `working`: everyday build branch; no PR required to push here.
- Cut feature/work branches from `working` for independent or risky work;
  merge back into `working` when done.
- When `working` is in a good state, open a PR from `working` into `main` for
  release. This is the one point a Reviewing Engineer review (self or
  otherwise) applies.
- If the current branch has unrelated changes, pause before adding more —
  recommend a new branch for the new task.
- Docs-only PRs may target `main` directly if there's nothing meaningful to
  test first; say why in the PR.

## 4. Existing project intake

When a Citizen Developer starts from an existing project, the agent should first
run read-only checks:

- inspect files, stack, package manager, branch, and remotes;
- check for missing `.gitignore`, committed `.env`, or likely secrets;
- identify infra files such as Serverless, Terraform, Docker, or other
  infra-as-code files;
- scan for production domains/accounts, access keys, direct DB access,
  external login, payments, and domain-specific sensitive data categories;
- identify the project's sensitive data domain, such as customer,
  healthcare/patient, employee/payroll, financial, candidate/recruiter, or
  other regulated operational data;
- document project-specific Red Zone categories in `DESIGN.md` or `SECURITY.md`
  so future PRs do not assume the default list is complete;
- check whether a design-spec document and Project Journal exist;
- suggest small structure or documentation fixes before broad refactors.

If the project has drifted from onewaydoor conventions, point out the concrete
issue and steer it back. Common corrections:

- Hardcoded DB credentials or secrets in `.env` committed to git → move to
  an untracked `.env`, rotate the exposed credential.
- Frontend direct database access or another service's DB credentials →
  add an API/service layer.
- Production data, real user data, or copied production secrets used for
  local development → replace with generated, anonymized, or fictional
  data.

## 5. Install tools only when needed

- GitHub CLI: install when local PR/check/status work is needed.
- Supabase CLI: install before local Supabase development or migrations.
- Node/package managers: use the repo lockfile to infer npm, pnpm, yarn, or
  bun.

If a tool is missing, the agent should explain why it is needed and the next
install/configuration step. It should not switch tools just because one is
missing. If authentication is required, configure it only for the current
need: `gh auth` for GitHub operations, and each service's own login flow
(Vercel CLI, Supabase CLI) for deployment operations.

### Local commit-time checks

The template includes `.pre-commit-config.yaml` for fast local checks before
commits. Do not install it during initial onboarding. When the Citizen Developer
is ready to make the first commit, the agent should check whether pre-commit is
installed and whether the local git hook exists:

```bash
command -v pre-commit
test -f .git/hooks/pre-commit
```

If either check fails, explain that the hooks catch secrets, private keys, large
files, merge conflicts, and syntax issues before commit, then offer to install
them. Missing `pre-commit` or a missing local hook is not by itself a reason to
stop the commit. If the Citizen Developer wants to keep moving, make the commit
and rely on GitHub Actions as the required backstop.

Use plain wording such as:

> Local safety hooks are not installed yet. They catch common mistakes before a
> commit, but GitHub Actions will still check the PR. I can install the hooks
> now, or I can make this commit and we can set them up later.

Do not say "No commit was created" unless the agent actually attempted `git
commit` and Git returned a failure. A missing optional local hook is a setup
gap, not a failed commit.

Preferred install:

```bash
pipx install pre-commit
pre-commit install
pre-commit run --all-files
```

Fallback if `pipx` is unavailable:

```bash
python -m pip install --user pre-commit
pre-commit install
pre-commit run --all-files
```

These hooks are a convenience, not the source of truth. They can be skipped, so
GitHub Actions remain the mandatory backstop for secrets, SAST, and onewaydoor
policy checks.

## 6. Project markdown files

onewaydoor-owned notes live under `onewaydoor-docs/`. Project agents should use
ordinary names for project-owned files.

Each project should have exactly these required project-owned markdown files:

- `PROJECT-CONTEXT.md` — pointer file that tells future agents whether to run
  onboarding or resume the active project;
- `DESIGN.md` — current design/spec;
- `JOURNAL.md` — append-only decision history;
- `README.md` — what the project does, how to run it, and how to deploy it;
- `TODO.md` — current project tasks, freely maintained by the project agent.

The onewaydoor template root versions of these files are intentionally minimal
seed files. Keep evolving onboarding instructions, startup phrases, routing
rules, and process guidance in onewaydoor-owned docs such as this file, not in
root project-owned placeholders. This prevents downstream project repos from
seeing fresh-template wording changes as recurring sync conflicts after their
project-owned files have real content.

The onewaydoor template includes blank/minimal root `README.md` and `TODO.md`
files so project agents have obvious project-owned files to write into. In
existing projects, preserve existing `README.md` and `TODO.md`; do not replace
them with the blank template versions.

During first onboarding, update the placeholder `PROJECT-CONTEXT.md`,
`DESIGN.md`, and `JOURNAL.md`. Create one seed journal entry in `JOURNAL.md`.
Append a journal entry whenever `DESIGN.md` changes.

Add `OPERATIONS.md` when the project is deployed. Add `SECURITY.md` when the
project has auth, sensitive data, or Red Zone context.

## 7. Commit hygiene

Commit small, coherent changes often:

- commit before risky refactors, dependency changes, or infra changes;
- keep docs, app code, infra, and dependency updates in separate commits when
  practical;
- use plain-English commit messages that say what changed;
- write durable review artifacts in English: commit messages, PR titles and
  descriptions, `DESIGN.md`, `JOURNAL.md`, `docs/REVIEW-QUESTIONS.md`, review
  summaries, and deployment notes;
- avoid giant mixed commits so a non-engineer can revert one change safely.

Commit/push/PR cadence:

- Commit after each coherent, revertable unit of work: project activation,
  initial scaffold, one feature slice, one bug fix, one infra change, one
  dependency/tooling change, or one docs decision.
- Push after each commit or small batch of commits once the branch is intended
  for review, handoff, testing on another machine, or backup.
- Open a draft PR after the first meaningful pushed commit if the work touches
  Red Zone, auth/login, deployment, external services, production domains,
  branch/release behavior, or needs Reviewing Engineer input.
- For ordinary low-risk app/doc work, open the draft PR when there is a coherent
  first slice to review; do not wait until everything is finished.
- Keep the PR draft until checks pass, testing notes are written, and blocking
  review questions are resolved.
- In onewaydoor, routine checkpoint commits, pushes, and draft PRs are expected
  once there is meaningful project progress on the right branch. Do not claim
  onewaydoor policy forbids proactive commits or requires the Citizen Developer
  to explicitly ask for them. Announce the checkpoint and proceed unless the
  action is destructive, outside the task, or blocked by the runtime/tool's
  approval system.
- Do not use a timer-based commit rule. The unit is "safe to explain and
  revert", not minutes elapsed.

## 8. Review and release

- The self-contained onewaydoor GitHub Actions workflows and the PR template
  are active. Exact required-check behavior still depends on each
  repository's GitHub rulesets and branch protection settings.
- Every PR should contain a lightweight review pack: summary, why, testing done,
  reviewer-focus files/flows, open questions, and rollback/revert note. Expand
  this for Red Zone, auth/login, deployment, production domain, external-service,
  or sensitive-data changes.
- When Reviewing Engineer input or approval is needed, create or use a GitHub
  PR. Request review from whoever fills the Reviewing Engineer role for this
  project, track unresolved questions in `docs/REVIEW-QUESTIONS.md` when
  needed, link that file from the PR, and copy resolved decisions into
  `JOURNAL.md`. If the question blocks safe progress, stop that part of the
  work until the PR receives input.
- After opening a PR, tell the Citizen Developer:
  - what they can test now without waiting for review;
  - what is genuinely blocked on Reviewing Engineer input;
  - reviewer feedback will come as PR comments and can be brought back to the
    agent for fixes;
  - they should ask the agent for first-level diagnosis before pinging engineers
    about ordinary blockers;
  - if clarification is still stuck and no Reviewing Engineer is available,
    note the open question in `JOURNAL.md` and make the best-informed call,
    flagged for later review.
- Useful "test now" examples:
  - run the app locally with synthetic/demo data;
  - click through a Vercel preview (or other non-production preview
    deployment) backed by a Supabase development project, using
    synthetic/anonymized data;
  - run unit tests, lint/typecheck, seed scripts, dry-runs, or sample imports
    that do not touch production data or production systems.
- Useful "blocked on review" examples:
  - external-user login, real domain-sensitive data, or payments;
  - production domains, production deployments, or production data access;
  - new non-default providers or unusually costly services;
  - sending messages or emails to real candidates, customers, patients, or
    other external users.
- After approval, the agent should recommend whether to merge immediately or
  wait, based on check status, testing confidence, branch freshness, and any
  unresolved review comments.
- Red Zone work still requires the process in `AGENTS.md`.
- Promotion from prototype experiment to production reliance requires
  engineering review and an explicit promotion plan.
- If you're using free-tier Vercel/Supabase projects, check each service's own
  inactivity/expiry policy directly.

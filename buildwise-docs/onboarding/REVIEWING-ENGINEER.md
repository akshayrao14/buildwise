# Reviewing Engineer Quick Start

buildwise is a template repo for Citizen Developers building projects with AI
coding agents. It gives each project a safe starting structure, agent steering
instructions, an onboarding flow, self-contained GitHub Actions workflows,
local pre-commit checks, and PR review prompts.

Use this guide when creating a new buildwise project repo, handing it to a
Citizen Developer, or bringing an existing project onto buildwise.

Buildwise state is only for routing buildwise import/onboarding/resume
behavior. It is separate from the product's own delivery stage, such as
requirements, design, implementation, testing, pilot, or production.

For review, import, and release work, use Reviewing Engineer mode from
`buildwise-docs/governance/OPERATING-MODES.md`. In that mode, an agent may
help inspect, summarize, and prepare review artifacts, but automated checks
and agent recommendations do not replace human Reviewing Engineer approval.
"Reviewing Engineer" is a self-fillable role — it can be the Citizen Developer
themself, a technical friend/advisor, or explicitly skipped with an
acknowledged risk logged in `JOURNAL.md` (see `AGENTS.md`, "Reviewing
Engineer Role"). This mode still applies even when reviewer and builder are
the same person, since the point is a deliberate second pass, not a
different human.

## Startup trigger

The Reviewing Engineer starts a PR review by saying `Start Buildwise Review`,
or by sending a message that is only a GitHub PR URL
(`https://github.com/<org>/<repo>/pull/<n>`, with at most trivial whitespace
around it). Both work in any buildwise repo (template or generated project),
the same way `Start Buildwise` works everywhere for Citizen Developers. A URL
that is part of a larger message asking something else does not trigger
review mode — only a message that is the URL counts as the bare-URL trigger.

When either form fires:

1. If the phrase was used without a PR, ask which PR (number or URL) before
   continuing.
2. Enter Reviewing Engineer mode
   (`buildwise-docs/governance/OPERATING-MODES.md`) scoped to that PR only.
   For existing-repo imports, also read
   `buildwise-docs/onboarding/EXISTING-REPO-IMPORT.md`.
3. Fetch the PR with `gh pr view <url-or-number>`. This works regardless of
   which repo the agent's working directory is in, since the URL/number is
   explicit; if only a bare number was given, assume the current repo.
4. Run the checklist already defined above in this guide as an active pass,
   not just a reference:
   - PR review-pack completeness: summary, why, testing done, files/flows to
     inspect first, open questions, rollback/revert note (see "PR review pack
     expectations" above).
   - Red Zone self-declaration vs. the actual diff: external-user login,
     domain-sensitive data, payments, and any project-specific Red Zone
     categories recorded in `DESIGN.md`/`SECURITY.md`/`buildwise.config.yml`.
   - Required-checks status: secret scan, SAST scan, buildwise policy scan,
     and project checks, whichever of these this project has actually wired
     up — surface pass/fail/pending, do not re-implement them.
   - Auth/deployment/production-domain/external-service flags per "When
     Reviewing Engineer involvement is required" below.
5. Draft the findings as one structured list, each item tagged to the
   checklist category it came from. This is evidence for the human reviewer,
   not a verdict — never include an approve/request-changes recommendation
   in the draft.
6. Show the draft to the Reviewing Engineer and wait for go-ahead or edits.
7. On approval, post once via `gh pr comment <url> --body-file <draft>` — a
   plain comment, not a review state. Post directly rather than handing the
   engineer text to paste manually: the actual consumer of PR comments is the
   Citizen Developer's agent, which already parses PR comments as part of its
   own workflow (see `AGENTS.md`, "What Happens If You Touch a Red Zone
   Item"). A manual relay step adds friction and risks losing structure with
   no compensating benefit.
8. After the comment posts, separately ask whether to also submit a native
   GitHub review verdict now: Approve, Request changes, Comment-only, or hold
   off (default if unanswered). Only on an explicit choice, run
   `gh pr review <url> --approve|--request-changes|--comment [--body ...]`.
   The agent never selects the verdict itself; it only executes the verdict
   the engineer names. This keeps human merge authority intact — see
   `AGENTS.md` "What Happens If You Touch a Red Zone Item" and
   `buildwise-docs/governance/governance-decisions.md` §9 item 2.

This trigger is a review the Reviewing Engineer can invoke on demand. It does
not replace the Reviewing Engineer's own read of the PR, and it does not
change the required-human-approval gate on `main`.

## What this template provides

- Cross-agent steering through `AGENTS.md` plus tool-specific pointers for
  Claude, Gemini, Copilot, and Codex-like agents.
- A startup phrase that tells agents to begin the buildwise onboarding flow.
- Stable project context files: `PROJECT-CONTEXT.md`, `DESIGN.md`, `JOURNAL.md`,
  `README.md`, and `TODO.md`.
- Red Zone guidance for external login, payments, and project-specific
  domain-sensitive data.
- Self-contained GitHub Actions workflow files in each project repo — no
  shared/reusable-workflows repo dependency.
- A PR template with Red Zone self-declaration and deployment-impact prompts.
- `.pre-commit-config.yaml` for local secret/key/large-file/syntax checks.

## New project repo setup

1. Use the GitHub template flow to create the new repo from the `buildwise`
   template (`akshayrao14/buildwise`, or wherever this project's copy lives)
   in whatever GitHub org/account this project lives in.
2. Include all branches.
3. Name the repo `bw-<dev-name>-<project-name>` (or your own preference).
4. Confirm these branches exist:
   - `working` — default branch, everyday build branch; no PR required to
     push here;
   - `main` — production state; PR required.
5. Confirm the repo inherited these root files:
   - `AGENTS.md`
   - `PROJECT-CONTEXT.md`
   - `DESIGN.md`
   - `JOURNAL.md`
   - `README.md`
   - `TODO.md`
   - `.github/workflows/*`
   - `.github/pull_request_template.md`
6. If this project relies on org-level rulesets, confirm they apply to the
   new repo. GitHub template creation copies files and branches, but repo
   settings/ruleset behavior should still be checked.

## Required repo/settings checks

For generated project repos, verify:

- Default branch is `working`.
- `main` requires PRs; `working` allows direct pushes.
- PRs into `main` require Reviewing Engineer review, since merging into
  `main` triggers production deployment.
- Repo settings actually enforce Reviewing Engineer approval on `main`, if
  this project has set that up — do not assume template creation turned this
  on; verify it. A `CODEOWNERS` entry naming whoever fills the Reviewing
  Engineer role for this project, plus "Require review from Code Owners"
  branch protection, is one way to enforce this, but the requirement is the
  approval itself, not that specific mechanism. If no one fills the role,
  treat promotion to `main` as a deliberate self-review checkpoint instead
  (see `AGENTS.md`, "Reviewing Engineer Role").
- Vercel project connected to the repo.
- Supabase project created, connection string in environment variables.
- Release/deployment CI/CD should start only from `main`, not from `working`.
- `working` allows direct feature-branch merges.
- Fresh buildwise feature/work branches are cut from `working` for
  independent or risky work, then merged back into `working` when done. When
  `working` is in a good state, open a PR from `working` into `main` for
  release.
- Docs-only PRs may target `main` directly if there's nothing meaningful to
  test first; say why in the PR.
- Force pushes and branch deletion are blocked on `main`.
- Required checks depend on what this project has actually wired up in
  `.github/workflows/`; verify against live rulesets
  (`gh api repos/<org>/<repo>/rules/branches/<branch>`) rather than assuming
  any specific check is required by default. buildwise does not ship a
  fixed set of required checks — a secret scan, SAST scan, and buildwise
  policy scan become required only once this project's own repo settings
  enable them.
- Template repo creation is not blocked by required checks during branch
  creation.
- Reusable workflows, if used, live in this repo's own
  `.github/workflows/` — buildwise does not maintain a separate
  shared/reusable-workflows repo.
- The PR template appears when opening a PR.

## PR review pack expectations

Every PR should include enough context for a Reviewing Engineer who does not
have deep project history. Durable review artifacts must be written in English:
commit messages, PR titles and descriptions, `DESIGN.md`, `JOURNAL.md`,
`docs/REVIEW-QUESTIONS.md`, review summaries, and deployment notes.

- summary of what changed;
- why the change is needed now;
- how the Citizen Developer or agent tested it;
- files or flows to inspect first;
- open questions or blockers;
- rollback/revert note.

For risky PRs, require extra detail on Red Zone impact, auth/login,
domain-sensitive data, infra/deployment, production domains, external
services, and accepted exceptions. The PR template carries these prompts; the
Reviewing Engineer should ask for missing context in PR comments rather than
private chat when possible.

Expect draft PRs early when a branch needs review input or touches Red Zone,
auth/login, deployment, external services, production domains, or
branch/release behavior. Draft means "visible and ready for guidance", not
"ready to merge". The agent should keep it draft until checks pass, testing
notes are written, and blocking review questions are resolved.

## Hand off to the Citizen Developer

Send this message:

```text
Your buildwise project repo is ready.

1. Clone the repo.
2. Open your coding agent in the repo folder.
3. Paste this exact phrase into the agent:

Start Buildwise

The agent will guide you through setup. It should ask only for the next detail it
needs, save project context in the repo, and avoid Vercel/Supabase setup until
needed.

Use the same primary phrase in CLI tools and app-based agents. If the app asks for
permission to commit, push, or create a PR, approve only when the action matches
the buildwise workflow the agent explains. If the app cannot read repo files,
attach or paste `AGENTS.md`, `PROJECT-CONTEXT.md` if present, and
`buildwise-docs/onboarding/ONBOARDING.md`.
```

When handing off, also set this expectation: the agent should stay focused on the
next useful/demo/go-live version. Nice-to-haves should be deferred into `TODO.md`
or the Project Journal unless they are needed for adoption or launch.

After a PR is opened, the agent should give the Citizen Developer a short
handoff:

- what can still be tested without waiting for review;
- what is blocked on Reviewing Engineer input;
- share the PR link somewhere your reviewer will actually see it;
- reviewer feedback will arrive as PR comments and can be brought back to the
  agent for fixes;
- ask the agent for first-level diagnosis before pinging a reviewer for
  ordinary blockers;
- if clarification remains stuck and no Reviewing Engineer is available, note
  the open question in `JOURNAL.md` and make the best-informed call, flagged
  for later review.

"Can still be tested" usually means local app flows, preview deployments,
synthetic/demo data, unit tests, dry-runs, and Vercel preview + Supabase
development-project work when access already exists. "Blocked on review"
means external-user login, real sensitive data, payments, production
domains/deployments/data, new non-default providers, expensive services, or
messages to real external users.

## Cost and tool setup guidance

Citizen Developers do not need every tool or the most expensive model at the
start. The agent should:

- use higher-capability models for planning, architecture, risk review, unclear
  debugging, and final validation;
- use cheaper/faster agents for bounded implementation, mechanical edits, simple
  tests, or parallel exploration;
- right-size each background or parallel agent independently: cheaper/faster
  models for bounded search, mechanical edits, test runs, and summarization;
  higher-capability models for architecture, security/risk review, unclear
  debugging, final validation, or high-reversal-cost decisions;
- avoid spawning multiple expensive background agents by default;
- state the right-sizing choice before broad exploration or background/parallel
  agent work;
- install GitHub CLI, Vercel CLI, Supabase CLI, Node/package managers,
  deployment tools, or project libraries only when the current step needs
  them;
- check/install `pre-commit` before the first commit, not during first product
  discovery. Missing local hooks should be explained and offered as setup help;
  they should not block the first commit if the Citizen Developer wants to keep
  moving, because GitHub Actions are the required backstop.

## Existing project onboarding

For existing projects, do not overwrite files blindly.

- Use `buildwise-docs/onboarding/EXISTING-REPO-IMPORT.md` as the repeatable
  runbook for mature existing repos.
- If the project has no git/GitHub repo: create a new buildwise repo, clone it
  into a separate folder, then copy/move existing project files into that clone
  intentionally.
- If the project already has a GitHub repo: import buildwise files carefully;
  prefer a squash merge from buildwise `working`, then inspect and resolve
  conflicts one by one.
- Preserve existing `README.md`, `TODO.md`, `AGENTS.md`, `.gitignore`, and other
  steering/config files unless whoever fills the Reviewing Engineer role for
  this project approves a specific replacement.
- Treat Vercel, Supabase, and the buildwise branch model as defaults for fresh
  buildwise projects, not mandatory replacements for an existing project's
  deployment, auth, or branch model.
- During import, identify the project's domain-sensitive data categories, such as
  candidate/recruiter, healthcare/patient, employee/payroll, financial,
  customer, or other regulated operational data, and record them as
  project-specific Red Zone categories.
- Set `project_type: existing-import` in `buildwise.config.yml` for mature repos
  that already have their own branch, deployment, auth, workflow, domain, or
  secrets model.
- Record accepted differences in `buildwise.config.yml`, for example approved
  deployment providers, auth provider, branch model, archived paths, and
  whether policy checks should start as advisory.
- Do not put active source-of-truth files in `archived_paths`. If a file is
  stale or historical, say that clearly in the project docs; do not also point to
  it as active or secondary truth. Archive exclusions reduce check noise during
  import, but they are not a way to hide active governance or product docs from
  review.
- Do not make buildwise checks required for the existing repo until the import
  baseline has been reviewed. Keep existing required CI in place, then promote
  selected buildwise checks to required after baseline noise is handled. If the
  existing repo already has stricter native CI, keep that CI authoritative and
  set `project_checks_mode: advisory` or `project_checks_mode: disabled` in
  `buildwise.config.yml` instead of requiring generic buildwise project checks
  that duplicate or conflict with it.
- Keep buildwise-import follow-ups in a buildwise-owned note such as
  `buildwise-docs/import-review.md`, not in the product backlog.

Before approving an existing-project buildwise import, confirm:

- existing deployment, auth, branch, security, and backlog conventions were
  preserved or changed only with explicit approval from whoever fills the
  Reviewing Engineer role for this project;
- `buildwise.config.yml` accurately records accepted differences from fresh
  buildwise defaults, including providers, branch model, sensitive-data
  categories, archived paths, advisory/strict check mode, and whether generic
  project checks are `auto`, `advisory`, or `disabled`; if project checks are
  enabled, confirm whether `build` should stay advisory or become blocking via
  `build_check_mode: blocking`;
- external services are visible in `DESIGN.md` or `docs/EXTERNAL-SERVICES.md`,
  including purpose, data shared, sensitive-data impact, account ownership,
  secret/config location, and approval status;
- `PROJECT-CONTEXT.md`, `DESIGN.md`, `JOURNAL.md`, and `TODO.md` are pointer
  docs for the existing project, not competing sources of truth;
- `PROJECT-CONTEXT.md` uses the right buildwise state: usually
  `bw-needs-review-before-resume` until import review is complete, then
  `bw-ready-to-resume`;
- buildwise-created Markdown files have been audited so placeholders are
  either populated with project-specific knowledge or explicitly left generic
  by design;
- any existing PR template keeps the project-specific quality checklist while
  adding buildwise Red Zone and deployment-impact prompts;
- buildwise workflows trigger on the existing repo's real protected/integration
  branches, not blindly on `working` / `main`;
- third-party preview/deploy checks that fail on the import PR are understood as
  either irrelevant for docs-only import, expected provider behavior, or real
  blockers.

## What the first agent session should do

The agent should:

- read `AGENTS.md`, `PROJECT-CONTEXT.md`, and
  `buildwise-docs/onboarding/ONBOARDING.md`;
- infer obvious facts from the repo name and opening user message;
- ask staged onboarding questions one at a time;
- activate `PROJECT-CONTEXT.md` and seed `DESIGN.md` / `JOURNAL.md`;
- create or update project-owned `README.md` and `TODO.md`;
- flag Red Zone implications without blocking basic activation;
- avoid tool installs and resource creation until actually needed.

## When Reviewing Engineer involvement is required

Step back in when:

- a provider substitution is proposed;
- a tool-native site helper such as Codex Sites is proposed for publishing or
  deployment instead of Vercel;
- the project needs an expensive or unusually costly service;
- the project needs containers, VMs, or another non-default compute/deployment
  tool;
- auth/sign-in includes external-user login, public signup, invites,
  candidate/patient/customer login, or otherwise goes beyond the project's
  own internal-only social-login-plus-whitelist convention;
- the project needs a production/custom domain;
- the project uses external messaging such as WhatsApp/SMS or a new email
  provider;
- the work touches a Red Zone category;
- the project is ready to promote beyond prototype/demo use — see
  `buildwise-docs/governance/PRODUCTION-PROMOTION.md` for the checklist.

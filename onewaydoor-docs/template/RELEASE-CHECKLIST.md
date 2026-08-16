# Onewaydoor Release Checklist

Use this to test the onewaydoor template before releasing it to Citizen
Developers.

## GitHub template setup

- [ ] `akshayrao14/onewaydoor` is marked as a GitHub template repo.
- [ ] Repo is public.
- [ ] Default branch is `working`.
- [ ] Template creation includes both branches: `main` and `working`.
- [ ] A test repo can be created from the template under `akshayrao14` (or
  wherever this project's copy lives).
- [ ] Test repo name follows `owd-<dev-name>-<project-name>` (or your own
  preference).
- [ ] Whoever fills the Reviewing Engineer role applies repo settings after
  creation; GitHub template creation does not reliably copy branch
  protections or repository settings.

## Branch behavior

- [ ] `main` requires PRs.
- [ ] `working` allows direct pushes; no PR required.
- [ ] PRs into `main` require review from whoever fills the Reviewing
  Engineer role, since merging into `main` triggers production deployment.
- [ ] Verify GitHub settings actually enforce Reviewing Engineer approval on
  `main` — do not assume this from documentation alone. If using
  `CODEOWNERS`: confirm `.github/CODEOWNERS` names whoever fills the
  Reviewing Engineer role for this project and "Require review from Code
  Owners" is enabled in branch protection for `main`. If no one fills the
  role, promotion to `main` is a deliberate self-review checkpoint instead
  (see `AGENTS.md`, "Reviewing Engineer Role").
- [ ] Release/deployment CI/CD triggers only from `main`; `working` may run
  checks/previews but must not trigger production release.
- [ ] `working` allows direct feature-branch merges and remains unprotected.
- [ ] Feature/work branches are cut from `working`; branches not cut from
  `working` are corrected before opening a release PR.
- [ ] Force pushes are blocked on `main`.
- [ ] Branch deletion is blocked on `main`.
- [ ] Repo-level merge method settings (squash, merge-commit, or both)
  reflect this project's own choice; rebase merge is disabled.
- [ ] Head branches are deleted after merge.
- [ ] Secret scanning is enabled when available.
- [ ] Push protection is enabled when available.
- [ ] Dependabot vulnerability alerts are enabled.

## First Citizen Developer session

- [ ] Whoever fills the Reviewing Engineer role can follow
  `onewaydoor-docs/onboarding/REVIEWING-ENGINEER.md` without extra
  explanation.
- [ ] Citizen Developer can clone the generated repo.
- [ ] Citizen Developer can paste exactly:

  ```text
  Start Onewaydoor
  ```

- [ ] Agent reads `AGENTS.md`, `PROJECT-CONTEXT.md`, and
  `onewaydoor-docs/onboarding/ONBOARDING.md`.
- [ ] Agent asks whether this is a new project or existing project.
- [ ] Agent asks for Citizen Developer name, project name, short goal, users,
  and Red Zone signals.
- [ ] Agent asks exactly one question per message — it never pastes a
  numbered question list and asks the Citizen Developer to answer it as a
  form.
- [ ] If the Citizen Developer answers only one of several questions asked (or
  answers a different one, or ignores one), the agent re-asks each outstanding
  question one at a time instead of proceeding.
- [ ] Agent pauses after Stage 1 activation and explicitly asks whether to
  continue into Stage 2 before asking any guardrail/user-data/payment
  question.
- [ ] Agent pauses after Stage 2 guardrails and offers the next-step menu
  instead of continuing into product-definition questions automatically.
- [ ] Agent updates project-owned `PROJECT-CONTEXT.md`, `DESIGN.md`,
  `JOURNAL.md`, `README.md`, and `TODO.md` during first onboarding.
- [ ] Agent does not confuse `onewaydoor-docs/` files with project-owned
  files.

## Existing project intake

- [ ] Agent inspects files, stack, package manager, branch, and remotes.
- [ ] Agent checks for committed `.env`, missing `.gitignore`, and likely
  secrets.
- [ ] Agent detects infra files such as Serverless, Terraform, Docker, or
  other infra-as-code files.
- [ ] Agent flags drift from onewaydoor conventions.
- [ ] Agent suggests small corrections before broad refactors.

## Drift correction examples

- [ ] Whoever fills the Reviewing Engineer role confirms the production domain
  is set up (through Vercel or the project's DNS provider) before it's
  assumed live.
- [ ] Agent catches production data, production secrets, or production
  account/project usage substituted in for local development.
- [ ] Agent catches frontend direct database access.
- [ ] Agent catches containers/VMs or another non-default compute/deployment
  tool proposed without Reviewing Engineer input.
- [ ] Agent catches expensive or unusually costly services that need
  Reviewing Engineer approval.

## Project docs and journal

- [ ] Fresh-template root project-owned seed files are minimal and do not
  contain evolving onboarding instructions, startup phrase details, state
  routing tables, or process rules that belong in onewaydoor-owned docs.
- [ ] Project-specific `README.md` is useful to a non-engineer.
- [ ] Project-specific `TODO.md` is maintained separately from
  onewaydoor-owned docs.
- [ ] `PROJECT-CONTEXT.md` points future agents to the active project context.
- [ ] `DESIGN.md` exists and reflects current project state.
- [ ] `JOURNAL.md` has a seed entry.
- [ ] Journal entries are appended when `DESIGN.md` changes.
- [ ] Existing onewaydoor-project syncs preserve active project-owned files
  unless the sync intentionally changes that project's content.

## Local commit-time checks

- [ ] Before the first commit, the agent checks whether `pre-commit` and the
  local git hook are installed.
- [ ] If local hooks are missing, the agent explains the benefit in plain
  language and offers to install/configure them.
- [ ] Missing local hooks do not block the commit if the Citizen Developer wants
  to keep moving; GitHub Actions remain the required backstop.
- [ ] The agent does not say "No commit was created" unless it actually
  attempted `git commit` and Git returned a failure.

## Red Zone and review process

- [ ] Agent identifies logins belonging to someone outside your own
  team/organization as Red Zone.
- [ ] Agent identifies non-anonymized domain-sensitive data as Red Zone.
- [ ] Agent identifies payment handling as Red Zone.
- [ ] Existing-project import identifies project-specific sensitive data
  categories instead of assuming candidate/recruiter data is the only data risk.
- [ ] Agent explains that Red Zone requires review from whoever fills the
  Reviewing Engineer role before merge/release, and only claims this is
  enforced after verifying repo settings actually require it, not just
  because it is documented.
- [ ] Agent does not treat automated checks as a substitute for human approval.

## Production promotion

- [ ] `onewaydoor-docs/governance/PRODUCTION-PROMOTION.md` exists and defines
  what a `working` -> `main` promotion requires.
- [ ] Whoever fills the Reviewing Engineer role can follow it to check domain
  setup, deploy path, data handling, and Red Zone history before approving a
  promotion PR.
- [ ] Agent does not claim a production deploy path exists on `main` unless
  the project actually has one wired up.

## GitHub Actions / workflows

- [ ] Project repos have the full self-contained onewaydoor workflow files
  under `.github/workflows/` — no reusable-workflow-repo dependency to
  verify.
- [ ] `onewaydoor-merge-source-checks.yml` verifies PRs into `main` come
  from `working`.
- [ ] `onewaydoor-security-checks.yml` covers secret scanning and SAST.
- [ ] `onewaydoor-policy-checks.yml` covers drift checks.
- [ ] `onewaydoor-project-checks.yml` reports script failures clearly:
  package manager, scripts found/run, result, enforcement mode, and local
  repro command.
- [ ] `onewaydoor-project-checks.yml` runs all available scripts instead of
  stopping at the first failure.
- [ ] `lint`, `typecheck`, and `test` failures are blocking in default
  `project_checks_mode: auto`.
- [ ] `build` failures are advisory by default and visible in the workflow
  summary.
- [ ] `build_check_mode: blocking` makes `build` failures block the project
  check.
- [ ] `project_checks_mode: advisory` reports failures without failing the job.
- [ ] `project_checks_mode: disabled` skips project checks with a clear summary.
- [ ] Required checks are not enabled until workflows exist and are stable.

## Sensitive reference handling

- [ ] `SYSTEMS.md` in this public template stays a generic stub — no real
  internal service details, credentials, or organization-specific content
  are committed to the public repo.
- [ ] A private fork's real `SYSTEMS.md` contents (or any other sensitive
  private reference) are not pasted into public issues, commits, or PR text.
- [ ] A relevant internal-system capability triggers a targeted `SYSTEMS.md`
  lookup before the Stage 2 next-step menu; unrelated basic onboarding does
  not load the whole file.

## Final release decision

- [ ] Whoever owns this template's technical decisions has reviewed
  `AGENTS.md` line by line.
- [ ] Whoever fills the Reviewing Engineer role has tested creating one repo
  from the template.
- [ ] Whoever fills the Reviewing Engineer role has tested one first-agent
  onboarding session.
- [ ] Known deferred items are acceptable for v1.

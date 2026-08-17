# Buildwise Existing-Repo Import Runbook

This note records the repeatable process for importing the buildwise template
into an existing mature target repository. It is intentionally generic: replace
`<target-repo>`, `<source-remote>`, `<source-branch>`, and branch names with the
actual project values.

## Goal

Bring buildwise governance into an existing project as a single squash import,
without overwriting existing project-owned files silently. Let Git expose
conflicts, resolve them explicitly, then adapt buildwise template files into
existing-import pointers.

## High-Level Rules

- Existing-repo imports run in Reviewing Engineer mode. See
  `buildwise-docs/governance/OPERATING-MODES.md`; the agent may help resolve
  conflicts and prepare import artifacts, but it must preserve accepted
  project-specific conventions unless whoever fills the Reviewing Engineer role
  for this project approves a change.
- Use a squash merge from the buildwise source branch, not a manual copy.
- Use the buildwise `working` branch unless whoever fills the Reviewing
  Engineer role for this project chooses a different source branch.
- Start from a clean worktree in the target repo.
- Do not overwrite existing project-owned files automatically. Let conflicts
  surface and resolve them one by one.
- During later buildwise syncs into an already-active project, preserve
  project-owned files unless the sync intentionally changes that project's
  content. Upstream changes to fresh-template seed files are normally buildwise
  scaffolding changes, not updates to apply over real project docs.
- Keep the target repo's project-specific agent file authoritative when it
  already exists as a real file.
- Treat buildwise defaults as defaults for new decisions, not as rules that
  override an existing approved deployment, auth, branch, data, or compliance
  model.
- Buildwise state is only for routing buildwise import/onboarding/resume
  behavior. It is not the product delivery stage.

## Procedure

1. Prepare the target repo.

   ```bash
   cd <target-repo>
   git status --short
   git branch --show-current
   git remote -v
   ```

   If the worktree has unrelated changes, stop and decide what belongs in the
   import before proceeding.

2. Add and fetch the buildwise remote.

   ```bash
   git remote add <source-remote> https://github.com/akshayrao14/buildwise.git
   git fetch <source-remote>
   git log --oneline -n 5 <source-remote>/<source-branch>
   ```

   If the remote already exists, reuse it and fetch. The usual source is
   `<source-remote>/working`.

3. Start the squash merge.

   ```bash
   git merge --squash --allow-unrelated-histories <source-remote>/<source-branch>
   ```

   Expect conflicts in files such as `.gitignore`, `README.md`, `CLAUDE.md`,
   `TODO.md`, `.github/pull_request_template.md`, or existing docs. Do not use a
   blanket `--ours` or `--theirs` resolution across the whole repo.

   If buildwise's default `PROJECT-CONTEXT.md` appears during the squash merge,
   do not leave it at `bw-ready-to-start`. For an existing-repo import, set
   `Buildwise state: bw-import-in-progress` while conflicts and import docs
   are being resolved.

4. Inspect conflicts.

   ```bash
   git status --short
   git diff --name-only --diff-filter=U
   git diff -- <conflicted-file>
   git ls-files -u
   ```

   Common resolutions:

   - `.gitignore`: keep the target repo's existing rules and add only missing
     buildwise safety patterns that are not already covered, especially `.env`
     and `.env.*` while preserving any committed examples such as `.env.example`.
   - Project-specific agent file conflict: if the target already has a real
     project-specific file, keep it and add a short pointer to `AGENTS.md`.
     Do not replace it with buildwise's symlink.
   - buildwise symlink conflict: preserve buildwise symlinks such as
     `GEMINI.md -> AGENTS.md` and `.github/copilot-instructions.md -> ../AGENTS.md`
     when they do not replace target-owned files.
   - PR template: combine target-specific quality checks with buildwise Red
     Zone and deployment sections. Avoid a Vercel/Supabase-only template for
     existing imports that use different providers; keep Vercel/Supabase-
     specific checks as conditional requirements rather than assuming every
     imported project already runs on the default stack.
   - Workflows: keep existing CI/deploy workflows. Add buildwise workflows and
     adjust their branch filters to the target repo's branch model. For existing
     repos with mature native CI, do not blindly require or auto-run generic
     buildwise project checks that duplicate or conflict with repo-native
     checks.

5. Convert imported buildwise root docs from fresh-template placeholders into
   active existing-import pointer docs.

   Update these files in the target repo:

   - `PROJECT-CONTEXT.md`: set `Status: active`; set `Buildwise state` to
     `bw-needs-review-before-resume` until import review is complete, then
     `bw-ready-to-resume`; identify the target project, list authoritative
     target-owned files, and say not to restart fresh onboarding unless
     explicitly requested.
   - `DESIGN.md`: summarize the existing product and stack, then point to the
     target repo's authoritative architecture, security, backlog, and operating
     docs instead of duplicating them.
   - `JOURNAL.md`: add a dated import entry describing what buildwise imported,
     which existing conventions were preserved, and why this is an existing
     import rather than a fresh template initialization.
   - `TODO.md`: make it a pointer to the target repo's active backlog or roadmap,
     not a competing buildwise task list. Keep buildwise-import follow-ups in
     a separate buildwise-owned review note such as
     `buildwise-docs/import-review.md`.
   - `buildwise-docs/import-review.md`: create this from
     `buildwise-docs/template/IMPORT-REVIEW.md`; track import-only follow-ups,
     Reviewing Engineer questions, and accepted existing-project exceptions that
     should not pollute the product backlog.

6. Configure `buildwise.config.yml` for an existing import.

   Typical settings:

   ```yaml
   project_type: existing-import
   policy_mode: advisory
   scan_scope: changed-files
   secret_scan_mode: diff
   semgrep_scope: changed-files
   project_checks_mode: advisory
   build_check_mode: advisory
   ```

   Also record the target repo's accepted deployment providers, auth provider,
   branch model, approved regions/domains, Red Zone data categories, archived or
   generated paths, project-doc pointers, and any accepted data-access exceptions
   such as an existing frontend client + RLS model. These are import exceptions
   for review, not a license to widen future scope silently.

7. Stage resolved files deliberately.

   ```bash
   git add <resolved-files>
   git add -u <paths-that-were-removed-during-conflict-resolution>
   ```

   If Git created temporary conflict-side paths such as `<file>~HEAD`, verify
   whether they still exist on disk. If they were only conflict artifacts, remove
   or stage their deletion.

8. Validate the import before committing.

   ```bash
   git diff --name-only --diff-filter=U
   rg -n '<<<<<<<|=======|>>>>>>>' <imported-or-resolved-paths>
   git diff --cached --check
   python3 - <<'PY'
   from pathlib import Path
   import yaml

   paths = [Path('buildwise.config.yml'), Path('.pre-commit-config.yaml')]
   paths.extend(Path('.github/workflows').glob('buildwise-*.yml'))
   for path in paths:
       with path.open() as f:
           yaml.safe_load(f)
       print(f'OK {path}')
   PY
   git ls-files -s GEMINI.md .github/copilot-instructions.md CLAUDE.md 2>/dev/null || true
   ```

   Confirm there are no unmerged paths, no conflict markers, no whitespace
   errors, YAML parses, and expected symlink/file modes are preserved.

9. Commit the squash import.

   ```bash
   git commit -m "Import buildwise into existing project"
   git status -sb
   ```

10. Push and open a draft PR.

    ```bash
    git push -u origin <import-branch>
    gh pr create --draft --base <base-branch> --head <import-branch> \
      --title "Import buildwise into existing project" \
      --body-file /tmp/buildwise-import-pr-body.md
    ```

    The PR body should explain:

    - source branch used for the squash import;
    - conflicts and resolutions;
    - existing conventions preserved;
    - buildwise config mode and scope choices;
    - validation commands run;
    - Reviewing Engineer questions, especially when checks should become
      blocking and whether Red Zone categories are complete.

11. Review required checks and automatic buildwise workflows.

    For existing imports, confirm the target repo's rulesets or branch protection
    before merging. buildwise security and policy checks may be appropriate to
    require immediately, but the generic buildwise project checks can conflict
    with mature repo-native CI.

    If the target repo already has stricter native CI:

    - set `project_checks_mode: advisory` or `project_checks_mode: disabled`
      in `buildwise.config.yml` instead of editing every imported workflow;
    - remove the generic buildwise project check from required status checks
      if it is disabled or only advisory;
    - document the decision in the buildwise import review note;
    - keep repo-native CI as the authoritative app-quality gate.

    Do not leave a disabled-or-noisy check required by the ruleset. A required
    check that no longer runs will still block merges.

12. Audit buildwise-created Markdown files for existing-project population.

    After the import docs are adapted, explicitly check which Markdown files are
    project-populated versus intentionally generic.

    ```bash
    find . -maxdepth 3 -type f -name "*.md" -o -maxdepth 3 -type l -name "*.md"
    rg -n "template-not-initialized|placeholder|after buildwise onboarding|during first onboarding|replace this placeholder|fresh-template|TBD|not recorded" \
      PROJECT-CONTEXT.md DESIGN.md JOURNAL.md TODO.md buildwise-docs .github/pull_request_template.md
    ```

    Expected outcome for an existing mature target repo:

    - `PROJECT-CONTEXT.md` is `Status: active`, has the correct `Buildwise
      state`, and points to target-owned source of truth files.
    - `DESIGN.md` summarizes the existing product, stack, authoritative docs,
      accepted buildwise exceptions, and Red Zone categories.
    - `JOURNAL.md` has a dated import entry explaining preserved conventions and
      why this is not fresh-template onboarding.
    - `TODO.md` points to the target repo's real backlog, and does not contain
      buildwise-import follow-ups.
    - A buildwise-owned import review note, for example
      `buildwise-docs/import-review.md`, contains import-only follow-ups and
      Reviewing Engineer questions.
    - The PR template reflects the target repo's real quality gates and
      deployment/security model while keeping buildwise Red Zone prompts.
    - Generic buildwise docs under `buildwise-docs/onboarding/`,
      `buildwise-docs/governance/`, and `buildwise-docs/template/` may
      remain generic by design.

    Any remaining fresh-template placeholder in project-owned docs should be
    fixed before asking for import approval, or explicitly recorded as a small
    known gap.

## Later Buildwise Syncs Into Existing Buildwise Projects

When syncing newer buildwise commits into a project that already has active
project-owned docs, do not accept upstream changes to these files just because
the buildwise template changed:

- `PROJECT-CONTEXT.md`
- `DESIGN.md`
- `JOURNAL.md`
- `README.md`
- `TODO.md`
- project-specific files under `docs/`

For those files, default to keeping the target repo's active content. Only edit
them when the sync is intentionally updating that project's context, design,
journal, README, or backlog. If the upstream diff is only fresh-template
placeholder wording, startup text, or onboarding prose, treat it as already
superseded by the project and record that decision in the import/sync review
note if needed.

Buildwise-owned files under `buildwise-docs/`, `.github/workflows/`
(self-contained — no shared/reusable-workflows repo dependency), governance
docs, and template/reference docs can normally be synced verbatim unless the
target repo has an accepted exception recorded in `buildwise.config.yml` or
the sync review note.

## Post-Import Review Checklist

- Existing project code, migrations, and docs were not overwritten silently.
- The project-specific agent file remains authoritative where one already
  existed.
- `AGENTS.md` is present as buildwise governance.
- `PROJECT-CONTEXT.md`, `DESIGN.md`, `JOURNAL.md`, and `TODO.md` are active
  existing-import pointer docs, not fresh-template placeholders.
- `PROJECT-CONTEXT.md` uses `bw-needs-review-before-resume` until Reviewing
  Engineer questions are resolved, then `bw-ready-to-resume`.
- `buildwise.config.yml` documents accepted target-repo exceptions.
- buildwise workflows match the target repo branch model.
- Generic buildwise project checks are either compatible with repo-native CI or
  configured as `advisory`/`disabled` and not required with a note explaining
  why.
- buildwise-import follow-ups live in a buildwise-owned import review note,
  not in the target repo's product TODO/backlog.
- buildwise-created project Markdown files are audited: project-owned docs are
  populated with target-repo context, while generic buildwise docs are
  recognized as intentionally generic.
- PR template includes both target-specific quality gates and buildwise Red
  Zone declarations.
- The final worktree is clean after the import commit and push.

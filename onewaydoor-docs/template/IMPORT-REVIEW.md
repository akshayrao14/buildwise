# Onewaydoor Import Review

Use this template to create `onewaydoor-docs/import-review.md` in the target repo
when importing onewaydoor into an existing mature repository. The actual import
procedure is `onewaydoor-docs/onboarding/EXISTING-REPO-IMPORT.md`. Keep import
follow-ups here, not in the product `TODO.md` or active product backlog.

## Import summary

- Target repo:
- Base branch:
- Onewaydoor source remote/branch:
- Import branch:
- Import method: squash merge from onewaydoor, not manual file copy
- Initial onewaydoor state:
- Target onewaydoor state after review:
- Date:
- Reviewing Engineer:

## Conflicts and resolutions

List each conflicted or overlapping file, what was preserved, what was changed,
and why.

- `README.md`:
- `TODO.md`:
- `.gitignore`:
- agent steering files:
- PR template:
- workflows:
- other:

## Existing conventions preserved

- Deployment model:
- Auth model:
- Branch model:
- Data/security model:
- Native CI/release checks:
- Backlog/source-of-truth docs:

## Accepted onewaydoor exceptions

Mirror these in `onewaydoor.config.yml` where possible.

- Deployment providers/domains/regions:
- Auth provider:
- Branch model:
- Project-sensitive Red Zone categories:
- Archived/generated/local-only paths:
- Accepted data-access exceptions:
- Checks that start advisory or diff-scoped:
- Generic project-check mode (`auto`, `advisory`, or `disabled`):

## Onewaydoor-created Markdown audit

- `PROJECT-CONTEXT.md` is active, has the correct onewaydoor state, and points to
  the project source of truth.
- `DESIGN.md` summarizes the existing project and points to authoritative docs.
- `JOURNAL.md` contains a dated import entry.
- `TODO.md` points to the real product backlog and does not contain import-only
  follow-ups.
- Generic onewaydoor docs are intentionally generic.

## Checks and follow-ups

- Required native CI:
- Onewaydoor checks enabled:
- Onewaydoor checks intentionally advisory or disabled:
- Baseline review needed before making checks blocking:
- Reviewing Engineer questions:

# Onewaydoor Docs

Onewaydoor-owned documentation lives here so generated project repos can use
ordinary project names such as `README.md`, `TODO.md`, and `docs/` without
mixing project notes with template/process notes.

## Folders

- `onboarding/` — Citizen Developer and Reviewing Engineer handoff guidance,
  including the existing-repo import runbook.
- `governance/` — onewaydoor governance decisions, operating modes, and
  agent-judgment rationale.
- `template/` — onewaydoor template release checklist, import review
  template, and the periodic instruction-surface audit prompt
  (`template/AUDIT-PROMPT.md`).

## Reviewing Engineer quick start

1. Use `onewaydoor` as the GitHub template repo.
2. Create the new repo (your own org/account, or a fork).
3. Name it `owd-<dev-name>-<project-name>` (or your own convention).
4. Include all template branches.
5. Apply project repo settings after creation; GitHub template creation does
   not reliably copy branch protection or repository settings.
6. Send the citizen dev the startup phrase from
   `onboarding/REVIEWING-ENGINEER.md`.

Keep agent steering files such as `AGENTS.md`, `CLAUDE.md`, and `GEMINI.md` at
repo root because agent tools expect them there. `SYSTEMS.md` also remains at
root, as an empty documented pattern until you build or adopt an internal
tools catalog.

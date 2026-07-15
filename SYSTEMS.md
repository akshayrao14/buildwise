# SYSTEMS.md — Internal Systems Reference (public stub)

> This is a bare-minimum example for the public template. A private fork
> (e.g. TERN's internal repo) replaces this file's content with real
> internal services, schemas, and portal details — same filename, so
> `AGENTS.md`'s pointer to this file never has to change between the
> public template and a private fork.

Read on demand — not auto-loaded every session. Relevant only when a
project needs to integrate with an existing internal system.

## Format for each entry

- **What it is / what it does** — one or two sentences
- **How to access it** — API, portal, etc., and which access method is
  actually allowed per `AGENTS.md`'s anti-pattern rules (never a direct
  database connection to another service)
- **Known gotchas** — rate limits, auth quirks, fields that are unreliable

## Example entry (fictional, illustrative only)

### Example: "Applicant Tracking Service"

- Owns candidate records. Read access via its own API only — never
  direct DB access (per Anti-Pattern Defaults in `AGENTS.md`).
- Auth: internal service token, rotated quarterly.
- Gotcha: pagination caps at 100 records per call.

---

Real forks: replace everything below the format section above with your
organization's actual internal services. First draft is typically written
by an agent reading real configs/schemas directly (e.g. from a parent
directory containing your other repositories), then edited by the CTO
for accuracy before committing.

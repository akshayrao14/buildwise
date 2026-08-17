# Production Promotion Checklist (v1)

Defines what "going to production" means for a buildwise project repo and
what a `working` → `main` promotion requires. Before this doc existed,
`AGENTS.md` and `ONBOARDING.md` referenced "production" and "promotion review"
without defining either.

This is separate from the Red Zone gate (`AGENTS.md`), which applies to
individual PRs regardless of branch. A project can be Red-Zone-clean on every
PR and still not be ready for production promotion.

## Requirements

1. **PR from `working` into `main`.** Never merge directly or push to
   `main`.
2. **Review from whoever fills the Reviewing Engineer role**, enforced by
   repo settings if you've set up branch protection for this project (see
   `buildwise-docs/template/RELEASE-CHECKLIST.md`); if no one fills the
   role, this is a self-review checkpoint, not a skip.
3. **Domain review.** If the project needs a production domain, whoever fills
   the Reviewing Engineer role confirms it is set up (through Vercel or your
   DNS provider).
4. **Deploy path review.** Confirm a real deployment workflow actually
   triggers from `main` for this project. The buildwise template does not
   ship one by default — `main` "triggers production deployments" only once
   the project has wired that up itself.
5. **Data review.** Confirm production data handling matches
   `DESIGN.md`/`SECURITY.md`: no leftover sandbox/synthetic-data assumptions,
   secrets are production-scoped (not development-environment
   Supabase/Vercel credentials), and every Red Zone category declared during
   the project's life is re-checked against what is actually shipping.
6. **Red Zone history check.** Skim merged PRs since the last promotion for
   Red Zone self-declarations and confirm each got Reviewing Engineer review
   at the time.

## Who owns this

There is no dedicated "publish to prod" role — whoever fills the Reviewing
Engineer role runs this checklist; for a solo citizen dev, that's a
deliberate self-check before promoting. See
`buildwise-docs/governance/governance-decisions.md`.

## What this does not do

This checklist does not create a deployment pipeline. If a project needs one,
that is normal project engineering work, reviewed like any other infra
change — this doc only defines the promotion gate around it, not the
pipeline itself.

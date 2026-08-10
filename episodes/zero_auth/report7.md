# zero_auth — report, Step 7 (delete contradictory rules, record conventions)

AI-generated (Omni Agent). Backend: Claude Code / claude-fable-5.
Date: 2026-08-10.

## W7 table — status

Most rows were rewritten in the step whose code change made them false;
this step closed the rest and verified the sweep:

| File | Status |
|---|---|
| `agautolab/agent/CHARTER.md:53` (safety-device item) | dropped in Step 6 — three devices → two; the two irreversible-harm guards stay |
| `agautolab/agent/GUIDE.md` doors | rewritten in Steps 5/6 — single window entrance |
| `agautolab/agent/README.md` auth passages | deleted in Step 6 (token boot requirement, "only authenticated route", "auth designed system-wide later") |
| `agautolab/README.md:61` | rewritten in Step 6 |
| `agdevworld/README_DEV.md:55-56` | rewritten in Step 4 |
| `agdevworld/assistant/GUIDE.md:48-52` | rewritten in Step 4 |

Verified this step: grep for `bearer` / `gateway_token` / `authenticated`
across agautolab live files → zero hits; grep for "auth designed
system-wide later"-family phrases across pj-agdev live docs → zero hits.
Historical episode docs and all cagent/agcluster docs untouched, per the
plan.

## Conventions recorded

- `agdevworld/README_DEV.md` gained a "cagent convention (agcluster)"
  section: agdevworld sends only read-shaped requests to cagent (today,
  the snapshot fetch); write-shaped prompts (`desired apply`,
  `reconcile --yes`) are out of bounds. Explicitly marked convention, not
  enforced — the human token cannot distinguish reads from writes;
  enforcement is deferred to the future JWT episode.

## The single pointer to the future auth vision (consolidation)

The scattered "auth is designed system-wide later" / "introduce identity
and this can go" notes in component docs are gone; **this episode is now
the one place that vision lives.** For the record (braindump line 6):

> There is a future intention to run JWT-style authentication under one
> consistent rule through every agent gate. Until that episode happens,
> pj-agdev is deliberately and uniformly auth-free (single-user,
> dedicated experimental cluster), agcluster/cagent keeps its current
> auth, and agdevworld's read-only use of cagent stays a convention.
> Known deferred items for that episode: a read/write distinction in
> cagent's API (scoped tokens or a GET-only listener), and the agforge
> subprocess env inheriting S3 keys (analysis §4).

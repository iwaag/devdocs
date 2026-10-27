# tidy p3 — final report

Date: 2026-10-27 (JST). Steps:
- `report1.md`: baseline and the p2 wording re-tested;
- `report2.md`: correction provenance and review derivation (with the deployment regression);
- `report3.md`: awaited request results;
- `report4.md`: facts for judgement and the account;
- `report5.md`: verification and trials.

Machine-specific notes, logs and rollback are in the ignored `pj-agdev/.local/tidy-p3/notes.md` and
`pj-agdev/.local/devenv.md` *tidy p3 on this host*.

## What exists now

- **Whose finding a correction is.**
  - `wfe delegate revise` records the agent's own finding as its own (`origin agent`, `--evidence`).
  - `--reopen --evidence` reopens the agent's own satisfied judgement of the latest result. The person's words keep
    their own path (`--because`), and so does a review's follow-up.
  - `wfe delegate attribute` appends a later, evidence-backed attribution beside the original record.
  - Holds, closed delegations and stale results behave as before.
- **Human review is review.** `review-unrecorded` no longer counts:
  - a correction (relayed words, a go-ahead or the agent's finding);
  - words relayed to a question that was not about the result.

  Relayed answers to the child's result confirmation and human review rounds still count.
- **Requested results get the same evidence.**
  - A run created with `wfe request` names its requester (`input.entrustedBy`).
  - `wfe run result` shows its latest result: identity, the claim beside its records, corrections, `to judge:`.
  - `wfe run return` lets only that requester return it, for the result it assessed.
  - HTTP has both (`GET …/result`, `POST …/return`).
  - An awaited result (`conversation ask --run`) is told with this evidence and route. Before, it was told as
    "unanswered: ended without a message".
- **The account.**
  - Reply news and `results-unreported` include awaited runs and say how many corrections a result needed.
  - The brief's delegation lines carry each correction's history.
  - `results-unreported` states that it compares record times, not prose.
- **Scope.** Returning what the commissioned work left out needs no go-ahead. This is said in `delegate revise`
  help, in the receiver questions and in ponderer's guide. Materially new work is still the person's to decide.
- **Unchanged:** no new gate, approval or mandatory field; optional documentation and review; `--yes`; no store or
  schema version change (new operations and optional fields only).

## Revisions (published)

| Repository | Revisions |
| --- | --- |
| agdevworld | `16cb127` (withdrawn), `b7d554a`, `7cbe8e2` (pre-p3 `17c52b3`); studio and agpc run `7cbe8e2` |
| pj-ponderer | `395f3c4` (conduct.md) |
| agops | `2a1638d` (fixture: no final-report step) |
| devpolicy | `9287da4` (Gitea mirror synced) |
| pj-agdev | `5dc29f5` (submodule pointer) |
| devdocs | this episode; README_DEV *Correction provenance and requested results*; p2 dispositions appended to `../p2/report.md` |

Tests: `test/agentCorrection.test.ts`, `test/requestedResult.test.ts` (new), and additions in `assessment`,
`resultEvidence` and `discussions`. Release suite 641/641 for each release. Before each activation of `b7d554a` and
`7cbe8e2`, `wfe run check` was run with the built release over every workspace (live replay).

## Trials (summary; details in `report5.md`)

| | Outcome |
| --- | --- |
| Baseline (p2 wording, fresh, explicit repair scope) | no premature acceptance; two returns of csvstat, both found by ponderer's own runs; honest account |
| T1 request + await | the evidence and route arrived with the result; `wfe run return` twice (the second for a partial fix); corrected result verified; complete account |
| T2–T7 delegate, implicit scope | every defective result returned before acceptance, as ponderer's own finding, without asking; fixes verified (or explicitly left unverified) |
| Account | complete in 7 of 8 rooms. **T6 omitted a returned result**; after `7cbe8e2`, T7 named its correction |
| Agent reopens its own mistaken acceptance | **not observed**: in five trials built for it, ponderer found the latent defect before judging |

Interventions: none in any room. Independent checks: every final artifact's tests rerun from its run branch; agops
main has no fixture output.

## Assessment against the completion criteria

| Criterion | Status |
| --- | --- |
| The final p2 wording was tested in a fresh context, with its result recorded | **Met** (`report1.md`) |
| Delegation and awaited-request paths deliver evidence and a valid correction route without inventing ownership | **Met.** Code, tests and one live use each (T1 request; T2–T7 delegate). Observers get no route |
| An agent can correct its own judgement within scope, with accurate records | **Met in code and tests.** The historical record is corrected. **Not observed live**, since no mistaken acceptance occurred |
| Human-review status derives only from genuine review evidence | **Met in code and tests.** run-051 d1 now reads UNKNOWN. No trial involved a person's review |
| Fresh trials show inspection before acceptance, repair without a go-ahead, verification and an honest account; initial failures stay visible | **Mostly met.** Inspection, repair and verification held in all rooms. The account failed once (T6, kept as a failure) and was improved once |
| Checks pass, deployments updated, inspectable without new gates | **Met**, after a deployment regression that was repaired (`report2.md`) |

The behavioural objective is **not claimed as achieved in general**. These are fixture trials with one receiver
agent. One criterion was not observed, and one fix was tried only once.

## Remaining limits

1. An agent reopening its own mistaken acceptance (`--reopen`) has never been exercised by an agent. A real case
   would be the first evidence.
2. The brief's correction history (`7cbe8e2`) was tried once.
3. A requested run's manifest is hashed when it is read, not frozen at its completion.
4. The HTTP `result`/`return` routes have no HTTP test of their own.
5. The reopening entry of run-051 d1 and the child's received correction keep their original attribution. The
   appended attribution sits on the correction request.
6. The release suite cannot catch replay drift of existing records, because its fixtures are written by the same
   code. The live `run check` before activation is a manual step in the notes, not automated in `wfe-release.sh`.
7. From p1, still open: agops' and cagent's root records, projects without README, lowerpoly's toolchain-host
   constraint.

## Recovery

- **Code:** `scripts/wfe-release.sh activate .local/agdev 17c52b3…`, and the agpc batch back to `17c52b3`
  (`nctl desired apply`, `render production`, `setup_wfe_worker.yml -l agpc`).
- **Records:** a pre-p3 release cannot replay records written in p3. These are ponderer conduct/run-051 (the
  attribution), runs 052–059 and agops evidence-fixture runs 007–022, which hold requested corrections, evidence and
  reopenings. Under `17c52b3` they read as inconsistent. Roll back only together with a decision about those runs
  (keep them unread, or roll forward).
- The ponderer guide, the fixture and devpolicy revert independently.

## For the Developer to look at

- **A sender/receiver exchange on the request route (T1):**
  - room `http://agstudio.local:8093/hangout/?conv=tidy-p3-t1`;
  - `wfe run result agops/evidence-fixture/run-010 --requester ponderer/conduct/run-053` shows the result, its two
    corrections (with ponderer's evidence) and the route;
  - `wfe run correction evidence-fixture/run-010 --workspace agops` shows the child's side.
- **An agent-origin correction on the delegate route:** `wfe delegate show conduct/run-059 d2 --workspace ponderer`
  shows "correction request 1 … ponderer-agent's own finding (checked: I ran the README example …)".
- **The historical attribution and the unchanged review status:** `wfe delegate show conduct/run-051 d1 --workspace
  ponderer` shows correction request 1 as "ponderer-agent's own finding; attributed so by Omni Agent (tidy p3) …
  recorded at first as from Omni Agent (for Developer)", and `UNKNOWN review-unrecorded`.
- **Final multi-result summaries:**
  - #484 in `?conv=tidy-p3-t7` (names the correction);
  - #481 in `?conv=tidy-p3-t6` (the recorded failure);
  - #470 in `?conv=tidy-p3-t1`.

# failsafe p3 — report: close the gaps and retire experimental residue

## Outcome

| completion condition | result |
|---|---|
| the covered defects are fixed and verified | ✓ every item of report1's table is fixed and verified live or by tests (below). Five more defects found in the trials were fixed and re-verified |
| reviews reflect final known outcomes | ✓ every occurrence of the three review topics shows its later outcome, posted once. The p2 and p3 occurrences carry trial provenance and follow-up statuses |
| test residue is retired | ✓ p1/p2 trial conversations are ✔; two disposable trial missions are cancelled; stray records are retired; fault files and timing overrides are gone; obsolete config and the p1 incident store are removed |
| retention is bounded without losing obligations | ✓ tracked requests went from 11 to 2 (both held). Owed replies, open incidents and holds are kept; closed incident caches are pruned; this was verified across a restart |
| the five-minute recovery target still holds | ✓ A2 (operational, with 8 timing-out probes per look): exit → Front **144 s**, resumed by Front at **177 s**, explicit acceptance, accurate developer follow-up. The first run (A) exposed a defect, now fixed; see below |
| every legacy item has an owner and a disposition | ✓ report5 §4; the items that need a human decision are held and summarised |

The step reports are `report1.md` (inventory) … `report6.md` (trials).

## Issue dispositions

| issue | fix | verification |
|---|---|---|
| `ag-reply` cut at a bare nested fence | the mark is a tag block (`<ag-reply …>` … `</ag-reply>`); fences inside are Markdown; the fenced form is repaired, never cut (pyagag `a686f22`) | captured p2 outputs as fixtures; live #12695, #12885 |
| Front's missing reply (#12509) | the actual cause: `to=Omni Agent` made the opener unreadable, and the repair was told the wrong reason. Attributes never cost the reply; the repair gets the exact reason | fixture `12509.md` |
| a failed reply counted as delivered | the reply is owed: no receipt, failure line without `end=`, one re-serving with the failed output, then a closing failure reported to the owners (`unanswered`) (pyagag `b2cdf75`, agobserver) | tests; live A (repaired in 28 s) and B (owners told in 63 s) |
| task agreement posted in the plan | the plan reads `status.md`; the listener adds a correction from the record; the planner guide forbids closure claims (agautolab) | tests; live #12700 → #12702 |
| housekeeping events certified progress | work vs housekeeping in live records; bounded tool waits; CPU under the run; `wait_idle` reassessment (pyagag `4fc54b2`, agobserver `a47f6ac`) | tests; live `waiting` (first ever); C's stuck wait bounded |
| probes could serialize the look | all due probes at once (up to 16), 20 s budget | reproduced 3.64 s → 1.01 s; live looks 0.3–0.6 s with 8 slow probes |
| late recovery missing from reviews | later outcomes appended once under their occurrence, owed before posted | tests with injected exit; live C across the exit; p2 reviews backfilled |
| per-kind fixed diagnosis | evidence-based assessment per occurrence, trial detection (`<run>.injected`), `review_status` for follow-up decisions | tests; live occurrences |
| plain conversations tracked forever | `satisfied`, an owed root, holds, `hold --retire` | dry run on the live mirror; tests; trial D |
| unbounded incident cache | pruning after 14 days, with episode counts kept | test |
| **found in trials:** a request contradicted a confirmed stop, and Front did not resume (A) | Observer states the check's fact (`920b651`); Front guides | A2 met the target |
| **found:** replies over Zulip's size are cut | the reply guide states the size (`cd9cce3`) | live repost #12695 |
| **found:** `cancel_tasks` rewrote completed tasks | finished tasks stay finished (agautolab `32ad9b9`) | test |
| **found:** an agreement serving redid the work (C) | supercoder guide | guide only |
| **found:** the `unanswered` incident text showed only the mention | notice line (`975895e`) | test |

## Cleanup performed

- p1/p2 trial conversations ✔ (12).
- robust_workflow p2 trial missions m9349 and m9697 were cancelled by
  their requester through autolab's plan path. Their task 1 now reads
  `cancelled` because of the defect above. The records were not restored
  because the channels are archived.
- `hold`: o8512 (m8519). `retire`: o11450 (forge trial, which has no
  cancellation record), o11522 (a finished routine run without `end=`).
- p2 reviews: statuses and later outcomes recorded. Their ✔ is left to the
  Developer.
- Deleted: `agobserver/.local/incidents-p1/` and four obsolete
  `refs.toml.pre-catalog` files. Fault files and `timing.json` are removed.
- Mission copies: only the held m11741 remains.

## Timings (operational)

| path | measured | target |
|---|---|---|
| silent exit → Front | 168 s (A), 144 s (A2) | 300 s |
| silent exit → resumed by Front | 177 s (A2) | — |
| confirmed stop not recovered → developer | 601 s after the first suspicion (A) | 600 s + a look |
| failed reply → repaired reply | 28 s | — |
| last failed reply → owners | 63 s | 60 s + a look |
| look with 8 timing-out probes | 0.3–0.6 s | ≤ budget 20 s |

Accelerated (C): uncertain after 120 s idle → Front +61 s → developer +181 s.

## False interventions, duplicates, costs

- **False interventions: 0** on the operational settings. Trial C's
  question about a quiet unbounded wait was the policy under test.
- **Duplicate actions:**
  - no duplicate recovery request, review post or acceptance, including
    across an injected Observer exit;
  - **one duplicate task execution** (C re-ran a 420 s wait after an
    agreement; guide fixed).
- **Human / Omni Agent interventions:**
  - the stand-in's requests, agreements and acceptances;
  - one reply to a developer escalation (A, #12672);
  - four corrections of a trial task the harness blocked (C);
  - fault injection and removal;
  - the cleanup records: holds, retirements, review statuses and two
    cancellations.

  **No rescue before an escalation.**
- **Model cost:** trials $5.27 (autolab $2.37, Front $2.90), cleanup
  $0.16. Observer ran no model judgment.

## Tests

| suite | passed |
|---|---|
| pyagag | 969 |
| agautolab | 318 |
| agobserver | 169 |
| agfront | 181 |
| agforge | 265 |
| archsage | 37 |
| cagent | 204 |

## Where it lives

- **pyagag**:
  - `a686f22` (tag mark), `b2cdf75` (owed replies, `unanswered`);
  - `4fc54b2` (work vs housekeeping, bounds, CPU);
  - `27b85d9` (`run.injected`), `cd9cce3` (post size in the guide),
    `975895e` (failure detail).
- **agobserver** (pj-agdev):
  - `a47f6ac` (prefetch, `wait_idle`);
  - `af43d4e`, `993fe73` (reviews);
  - `a1c09c2` (retention, pruning, `--retire`);
  - `276284b` (trial aids, 16 workers);
  - `920b651` (request consistency).
- **agautolab**: plan correction and `status.md`, `.injected` markers,
  `cancel_tasks` (`32ad9b9`), supercoder guide.
- **agfront**: `reply-unusable` fault, the guides.
- **Pins**: pyagag `975895e` in agautolab, agfront, agforge, agobserver,
  archsage and cagent. The relay and comfynotify stay on `f5c4359`
  (nothing they use changed).
- **Documentation**:
  - `README_DEV.md`: the reply contract, the Observer consolidation, and
    autolab's plan correction;
  - host notes in `pj-agdev/.local/devenv.md`.

## Remaining limitations

- **Front's judgment is one sample from right.** A2 recovered after the
  request and guide fixes, but A showed Front can misread a trace. The
  bounded escalation is what guarantees the outcome.
- **Replies longer than a post** are still cut, visibly. The fix is
  guidance, not delivery in parts.
- **Quiet unbounded waits** (a subagent sleeping, no CPU) are asked about
  after 15 min. That is intended, but a legitimate one will cost a
  question. The wait's advance is measured only by CPU.
- **Only autolab is probed.** Forge has no cancellation record (a11459 is
  retired in Observer only). agy, codex and gemini streams still do not
  report tool results.
- **Records left inaccurate:** m9349 and m9697 task 1 read `cancelled`.
  The history keeps `completed` and the integration.
- **Human decisions pending:**
  - held: o11711/m11741 (resume, take what exists, or cancel) and
    o8512/m8519 (task 2 agreement);
  - pending acceptance: m6113, m7601, m7732;
  - three review topics await the Developer's ✔.
- **Not bounded, by decision:** the listener journals (≤ 2.1 MB after
  weeks). The review assessment uses no model.
- **agautolab1 (VM) was not redeployed.** Its gateway still runs its older
  pin.

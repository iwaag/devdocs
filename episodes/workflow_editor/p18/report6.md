# p18 step 6 — verification, deployment and the example

Plan: [plan.md](plan.md), step 6.

## Deployment

Before deploying, service state was read through nctl (`status` ok;
`drift --host agpc` converged). WFE had no open attempt and no queued job.
Every area store and the 44 conductor run records were backed up. Paths
and rollback commands are in the ignored `pj-agdev/.local/p18/notes.md`.

| Component | Release |
| --- | --- |
| WFE (agdevworld) | `4e47ee4`, then `c20fdaa`, `05d2d47`, `6c71936`, `9fd7a3f` (pre-p18 `ae7d050`). Conversation store v12 → v13 |
| Transition | applied right after the first activation. The plan was the same as the rehearsal: 29 rooms, 24 established, 5 unreadable. Only activity events changed (+29) |
| Guides | pj-ponderer `5c6a7ca` (step 3's `9435ddb`, rebased over the archive writer's commits for those 29 rooms), agautolab `eeddf95` |
| aghangout | `6af3a45`, then `f7b56c9` |
| agpc worker | pinned to wfe `9fd7a3f` / executor `eeddf95` through nctl and the setup playbook. Drift is converged |
| Services | the executor was restarted. agobserver was restarted for `idle-input` (pj-agdev `9e9e767`) |
| pj-agdev | submodule pins `f0d962e`, `cced1f7` |

## Trial on the deployed system

The check actor was `p18-check` and the room is `p18-trial`, category
check. Every message and decision in the room is attributed to the check
actor. Nothing was recorded under the Developer's name.

| # | What happened | Run |
| --- | --- | --- |
| 384 | brief: ask misc/freework for a 3-line autumn poem | `ponderer/conduct/run-045` created standing |
| — | ponderer delegated → `misc/freework/run-010` → result → reply #385 → **rest** (Idle) | run-045 |
| — | idle interval: no open attempt, no queued job, both workers idle | — |
| 386 | "add a title, as a correction to the same child" | wake `ponder-w1` 34 ms after the message (one job) |
| 387 | ponderer: the accepted result cannot be revised; may it dispatch a replacement? | **gap found**, see below |
| 388 | answer: same child, please (after WFE `05d2d47`) | the answer continued `ponder-w1` |
| — | `delegate revise --because` reopened d1 → correction `c1` on **run-010** → reply #389 with the titled poem | same child, same parent |
| 390 | new topic: a colour for the poem | `ponder-w2` → #391, no commission |
| — | room closed: `standing.close`, "Closed", finalized | run-045 |
| 392 | a message reopened the room | `ponder-w3` → #393 → rest |

Results:

- **One conductor.** Five person messages were handled by one conductor
  run: one opening and three wakes, which make four stretches. The run
  was integrated at each rest (3 integrations).
- **The child kept its own record.** run-010 completed on its own and was
  corrected once, as itself. It was never cancelled or accepted by the
  room's closing.
- **Observation.** `wfe observe` reads all 9 resting open-room conductors
  (8 transitioned and the trial) as `awaiting-input`, each attributed to
  its conversation.
- **Browser check** (desktop and phone; reading wrote nothing):
  - 「進行のグラフ」 shows 待機中, 「後の発言で続行 3回」 and the
    stretches.
  - The commission opens to run-010 and reads 「同じ依頼先への修正 1回」.
  - Message ↔ step links work, including ponderer's replies.
  - The closed state reads 「終了（会話を再開すると同じ進行が続きます）」.
  - The WFE run view of run-045 reads "Idle · Ready for the
    conversation's next message", with a link to its room.

## Defects found and fixed during this step

1. **The run view header said "Waiting for a person" for a resting
   conductor** (`c20fdaa`). The activity line treated every `awaiting-*`
   situation as a person wait. A standing conductor at rest now shows its
   own status.
2. **An accepted result could not go back to the same child** (`05d2d47`).
   With one long-lived conductor, a person asking later to change
   something the conductor had already judged `satisfied` is normal. Only
   p17's review follow-up could reopen it, and ponderer correctly refused
   to treat a replacement as a correction and asked instead. Now
   `delegate revise --because answer:…|instruction:…` (the person's words)
   reopens it for the same child, keeping the earlier disposition in
   `reopened`. This is documented in `docs/delegation.md`, and the help
   says so. Tested in `standingWork.test.ts`.
3. **Replies were not linked to their step** (`6c71936`). A conductor
   reply spoken for no node (no `--completing`) is now attached to the
   steps whose messages it answers (`--re`).
4. **A correction was counted twice** (`9fd7a3f`). A follow-up or a
   reopening is itself a `revise`. p17 follow-ups had been double-counted
   in the same way.
5. aghangout `f7b56c9`: 「再開 N回」 (wake count) collided with
   「会話を再開」 (room reopen). It now reads 「後の発言で続行 N回」.

## The plan's verification list

| # | Check | Where verified |
| --- | --- | --- |
| 1 | several messages and replies, one conductor, nodes accumulate, freed slot used by another room | `standingFlow` test 1; live: run-045 through 5 messages, rest with both workers idle |
| 2 | two commissions are separate runs; clarification and correction reach the original child; children complete while the room is open | `standingFlow` boundary test, `standingWork` (2 tests); live: run-010 corrected as itself |
| 3 | concurrent input, child completion and attempt end lose nothing and duplicate nothing | `standingFlow` concurrency test (3 racing posts → one run; 5 wakers → one node and one job) |
| 4 | restarts preserve states; retries are idempotent | `standingFlow` restart test; live: five service restarts (deploys) fell between turns, and no record was lost or duplicated afterwards |
| 5 | facilitation end/start, close/reopen keep identity, hold outcomes, cancel nothing, replay nothing | `standingFlow` tests 4 and 5, `standing.test`; live: close/reopen on run-045 |
| 6 | human review still works and the conductor rests afterwards | `standing.test` (review beside a new topic; approval → rest); all p17/p17 ex1 tests pass. Not exercised live in this trial |
| 7 | history readable; long conversations navigable without fetching all child details | folded stretches, recipients as links (browser checks); reducer replay of 120 turns in 0.24 s (step 2) |
| 8 | transition creates no execution or fabricated history | `transition.test.ts`; rehearsal on a copy; live counts before and after |

Final checks: WFE 622/622 and both builds; aghangout `npm run check`
33/33; `browser-check` passes in Chromium and WebKit.

## Example for the Developer

aghangout `/hangout/?conv=p18-trial` (the full LAN address is in the
ignored local notes). It is a check room, listed under 動作確認. It shows one conductor graph with four stretches,
and the commission opens `misc/freework/run-010` in WFE. The conductor
rests, and the next message there continues the same run.

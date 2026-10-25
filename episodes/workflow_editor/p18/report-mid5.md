# Interim report 5 — workflow_editor p18 paused (one conductor run per conversation)

This interim report is filed here at the Developer's request. The work it
describes is [workflow_editor p18](../../workflow_editor/p18/plan.md); its step
reports are [report1](../../workflow_editor/p18/report1.md),
[report2](../../workflow_editor/p18/report2.md) and
[report3](../../workflow_editor/p18/report3.md). The work stopped at a clean
commit point. Nothing has been deployed: every running service still runs the
pre-p18 releases.

## Where it stands

| Plan step | State |
| --- | --- |
| 1. Conductor ownership and lifetime | done: `standing` on the run, execution `idle`, the standing operations and statuses (report1) |
| 2. Durable continuation and graph growth | done: one conductor per room, wake nodes under the run's lock, room transitions recorded on the run, settlement of silent turns, explicit replacement, integration at rest, bounded briefs, guides from committed main, linear replay, observer `idle-input` (report2) |
| 3. Recipient-project boundaries | done: tests for separate commissions, clarification, same-child correction and review beside a new topic; guidance on unmerged `p18` branches (report3) |
| 4. One progression in aghangout | implemented, not yet reported or checked in a browser (see below) |
| 5. Transition of existing rooms | implemented and tested on simulated legacy rooms; **not yet run against a copy of the live data** |
| 6. Verification, deployment, example | not started |

## Done since report3 (committed, not reported)

- **WFE agdevworld `ad227b6`**:
  - The conversation view has a new `progression` field (`server/progression.ts`). It holds the conductor with its standing and status, the stretches with topics, wake messages, nodes with their messages, and the recipients as links with their state and correction count. It also lists earlier conductor runs and the cancelled one a replacement names. The field is documented in `docs/conversations.md` *Progression*.
  - The run view's focus card shows a standing conductor at rest, suspended or closed, with a link to its room. The project run badge shows "Idle", not "In progress".
  - `wfe conversation transition [<id>…] [--apply --author <who>]` (`server/transition.ts`) makes one recorded, idempotent decision per room:
    - establish the latest readable conductor run (a completed run comes to rest, and anything left unacknowledged is held);
    - record a cancelled conductor (it is not revived);
    - record an unreadable or retired conductor;
    - record when no conductor exists.
  - `syncStanding` no longer establishes a run by itself. Only a room's creation from the conductor's attempt does; any other room from before p18 goes through the transition.
  - Tests: WFE 619/619, `tsc` clean, both builds pass. The new `test/transition.test.ts` checks that a dry run records nothing, that applying records once, that earlier runs stay history, that nothing is executed, woken or sent, that the next message wakes the same run, and that a cancelled room's request fails visibly.
- **aghangout `98d483d`** (local commit in the project workspace, not pushed):
  - The reading view has a 「進行のグラフ」 section. The current stretch and stretches with open work are unfolded; completed stretches fold to one line. Each step has Japanese state words, links to its messages and its commissions as expandable links to their runs. Each message links back to its step (「進行 ↑」).
  - The same view appears at the top of the 進行 panel.
  - A conductor at rest has its own status line. A cancelled conductor gets an explicit 「新しい進行役に交代」 action.
  - The tab badge counts neither a conductor at rest nor work awaiting human review.
  - A request taken up by a conductor at rest is no longer shown as "対応中".
  - The fixture has a new room, `fx-progression`.
  - Tests: aghangout `npm run check` passes (33/33).

## Remaining

1. Step 4: take browser screenshots of `fx-progression` in the fixture (`npm run fixture`, then `?conv=fx-progression`, reading view and 進行 panel, desktop and phone widths), fix what they show, then write report4.
2. Step 5: run `wfe conversation transition` (dry run, then `--apply`) on a copy of the live area: `conversations.sqlite` plus the ponderer workspace with its run copies, cloned. Inspect the plan for all rooms, then write report5.
3. Step 6, deployment:
   - check state through Nautobot/nctl;
   - back up every area store (conversation store v12 → v13; older releases refuse v13);
   - deploy WFE with `scripts/wfe-release.sh deploy`;
   - run the transition on live data right after activation;
   - merge and push the `p18` branches of pj-ponderer (`9435ddb`, worktree under the session scratchpad) and agautolab (`eeddf95`);
   - deploy aghangout with `scripts/deploy.sh`;
   - pin agpc to the new agdevworld commit through nctl and the playbook;
   - restart and verify the executor.
4. Step 6, trials: a check conversation with an attributed check actor (several real ponderer turns, a recipient commission and correction, an idle interval, close and reopen), a browser check, a compact example for the Developer, `p18/report.md`, README_DEV and devenv notes.

## How to resume

- Code: agdevworld `main` at `ad227b6`. aghangout workspace `main` at `98d483d`. pj-ponderer branch `p18` and agautolab branch `p18` exist as worktrees under the session scratchpad. They can be recreated from those branches if the scratchpad is gone.
- Interruption cause: the tool-safety checker returned no verdict repeatedly mid-session, and the work stopped there. Nothing was half-written: every change listed above is committed and its tests pass.

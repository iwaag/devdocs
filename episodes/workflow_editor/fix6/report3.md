# fix6 step 3 — same-child correction

Plan: [plan.md](plan.md) step 3. Code: agdevworld `4888ea3` (steps 1–3).

## The operation

```text
wfe delegate revise <parent-run> <d> --instruction "…" --reason "…" [--event <n>] [--manifest <digest>] [--because …]
```

`dispose rejected` stays closure and replacement. HTTP:
`POST …/delegations/<d>/revise`.

## The transition

1. **Parent** — `delegation.revise` (reducer, parent's lock): open and
   bound delegation; the latest completed event, taken up, manifest equal
   when given; no correction standing for it; the child not failed or
   cancelled after it. `DelegateStore.revise` also refuses a child that is
   cancelled (`child-stopped`), executing or not completed now
   (`child-active`). The delegate node waits; the delegation's state is
   `correcting`.
2. **Delivery** — the notifier's delegation conversation gets
   `…#<seq>:delegation-revision@<d>` (critical, alone), rendered as "Result
   returned for correction" with why and what to add; repaired from the
   record when its outbox row is missing, found by its marker when the
   acknowledgment was lost.
3. **Child reopened** — the intake classifies the post by message id (a
   revision is allowed on a completed child) and records
   `correction.receive` under the post's receipt. The prepare step checks
   the result it returns is still the child's latest completion
   (`result-superseded` otherwise); the reducer requires a completed
   execution with no open correction (`correction-state`). While unaddressed
   the execution is `in-progress` (summarize), the situation `due`; the
   child's request topic gets a critical message; the relay records
   `correcting` on the parent (no wake) and does not announce the old
   result again. A refusal reaches the parent as `answer-refused` with
   `revision` and ends `correcting`.
4. **Continuation** — `ExecStore.afterCorrection` (receipt
   `correction:<ws>/<wf>/<run>/<c>`) from the intake's follow-up, again after
   an attempt end and by reconcile while unaddressed. The child's brief and
   continuation show the correction in full and how to add work; `wfe run
   state` names it.
5. **Work** — the child adds nodes with its ordinary adjustment (or starts a
   completed node again): the first entry that gives the run work again
   addresses the correction (`how: work`). `wfe run correction <run> <c>
   --no-change --note …` addresses it without work. `correction.take-up`
   without `--no-change` only records that it was read. An attempt that
   ends with a correction from before it still unaddressed is held
   `interrupted` (`missed-correction`).
6. **New result** — the next completion is a new `completed` event and
   manifest; the parent takes it up and assesses that version.

Competing and closed: after `revise`, `dispose satisfied|rejected` is
`result-returned`, `abandoned` closes, a second `revise` is
`delegation-correcting`; after a disposition `revise` is `delegation-closed`.
A held parent records facts and launches nothing until resumed; cancelling
the parent leaves the child. Replay keeps both results (events,
`revisions`, `corrections`). Earlier acceptances and integrations are listed
in the assessment as historical facts; an acceptance binds its tree, so the
corrected files need a new one (unchanged rules).

## Checks (`test/correction.test.ts`)

- full path: revise (retransmission once; earlier result `result-returned`
  with yes), delivery with a lost acknowledgment (one post), the intake
  applying twice (one correction), the child reopened and continued once,
  the brief carrying the correction and the person it is for, a present and
  cleanup node added after the completed work (work kept), a new result with
  a new manifest, its assessment `ok`, the old result `result-superseded`,
  `satisfied` recorded without reminders, replay keeping both identities;
- a lost continuation re-queued by reconcile after a coordinator restart;
  an attempt that does not address it held `missed-correction`; `--no-change`
  completes again as a new result;
- before take-up, a wrong manifest, a child that completed again
  (refused, parent woken, `correcting` ended), abandoned then revise
  (`delegation-closed`);
- a cancelled child (`child-stopped`); a held parent launches nothing while
  the child is corrected; cancelling the parent leaves the child.

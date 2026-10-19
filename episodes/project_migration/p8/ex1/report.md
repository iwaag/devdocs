# project_migration p8 ex1 — Durable delegation waits and clear run status (final report)

Date: 2026-10-19. Plan: [plan.md](plan.md); steps: [report1](report1.md)
(trace and reproduction), [report2](report2.md) (durable waits),
[report3](report3.md) (recovery gaps), [report4](report4.md) (status and
reasons), [report5](report5.md) (verification, deployment, trials).

## What the braindump asked

A Front run that was legitimately waiting looked "held", and the hold
explained itself with long generic text but not *why*. Normal waits should
read as waiting, with a short reason; unknown interruptions may stay held,
but say so briefly.

## What changed

1. **The cause of run-027.** Front added its delegate node beside a running
   `talk` and ended; an attempt that leaves a node running is an
   interruption by design, so the run was held twice and recovered. Front's
   guide, the common run contract and `wfe delegate request` now say that
   `talk` ends where its part ends and the result is handled by a node after
   the delegate node. Live, Front built exactly that graph twice.
2. **Waits are records, not processes.** A dispatching attempt ends and
   frees its slot; child facts start new attempts across restarts. A child's
   completion is relayed once. The executor no longer lets a finished
   session's leftover background process hold the slot (found in the
   deployed trial: ten minutes for a `find /`; fixed and redeployed).
3. **Causes and repairs.** Holds keep a structured cause; an attempt that
   records nothing of its work is an interruption. Reconcile and the
   notifier rebuild lost queue and delivery rows from the records without a
   second post or child. Observer names the cause and the children, gives a
   recovery agent every delegation by identity, and sends a cause that keeps
   coming back to the owners instead of recovering it again.
4. **One status, short reasons.** `Waiting · Your answer`, `Waiting ·
   jevlab study result`, `Queued · Waiting for an execution slot`,
   `Paused · <reason>`, `Interrupted · Talk node left running at exit`,
   `Unknown · Process lost; outcome unconfirmed` — the same text in run
   lists, the run view (long explanations moved into Details), the CLI and
   the Front Desk.

## Deployed

WFE `0251983`, executor agautolab `a06c530`, Observer (pj-agdev `edf445c`),
web container, Front and recovery guides on Gitea; pins pushed
(pj-agdev `6ec8ab6`). Tests: WFE 440, agautolab 47, Observer 120; all 151
existing run records replay unchanged. The Slack bridge's one-line change was
not redeployed (the command was refused by the permission check).

Trial B (deployed): the parent ended **3 s after dispatching**, the
coordinator and the worker were restarted during its wait, another message
was answered in the freed conversation slot, and the child's result brought
the parent back to reply in the conversation by itself — no hold, no
recovery, no incident.

## Where to look

Links are in the ignored local notes (`pj-agdev/.local/devenv.md`,
*project_migration p8 ex1 on this host*): the trial conversation on the Front
Desk, its run view, Front's run list and a paused run for the short reason.

## Open

- A process left in the background *while* a session still works keeps its
  attempt open (unchanged; the contract says not to).
- Hours-long waits were proven in fixtures; the deployed waits were minutes.
- Slack bridge release pending, if wanted.

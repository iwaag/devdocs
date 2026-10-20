# argue p3 ex1 step 1 — readiness, the coordination boundary and observation

Inspected 2026-10-20 16:10–16:25 UTC (2026-10-21 01:10–01:25 JST) on the
coordinator host. **No system, guide or configuration change was needed or
made** in this step; the trial runs on the capabilities p3 ended with.

## Runtime state

- `nctl status`: Nautobot 3.1.3 authenticated, worker 1 / 0 pending, dumps
  collected 3 h ago, submodules clean. `nctl drift --host agstudio`:
  converged (agautolab, agdevworld-web, aghangout, agobserver, slack-front,
  ollama … applied).
- WFE `service/current` → agdevworld **`978a5ed`** (store v6, `discussion
  await`); agautolab **`797b2d0`** (wfrun guide); pj-agdev pins `9554a6d`.
  launchd `wfe-service`, `agautolab-wfexec`, `agobserver`(+watchdog),
  `slack-front` running. `/hangout/` answers 200.
- `wfe exec workers`: `studio` available (default lane 1, `conversation`
  lane 1); `gpupc` 1 slot.
- `wfe observe`: one unfinished run, `jevlab experiment/run-001`, stopped by
  the Developer before plan agreement. Unrelated; left untouched.
- Discussions: `argue-p3-trial` and `p3-s2-fixture` closed; three older
  open ones (`p16-s3-room`, `p16-s4-provenance`, an agpc LAN check) have no
  open invitations. None is touched.

## Participants (verified)

`wfe discussion participants` lists all seven as available.

| Participant | Workflow | Agent / profile | Project revision at start |
| --- | --- | --- | --- |
| argue (facilitator) | `argue/facilitate` | argue-agent / sonnet | pj-argue `29db1ce` |
| front | `front/discuss` | front-agent / sonnet | pj-front `d638a06` |
| rts-vs-bot | `rts-vs-bot/discuss` | rts-vs-bot-agent / sonnet | pj-rts-vs-bot `d90d517` |
| jevlab | `jevlab/discuss` | jevlab-agent / sonnet | pj-jevlab `2f44d31` |

The rts-vs-bot, jevlab and front workspaces were behind their published
`main` only by p3's own run records (discuss run-001…003 completions and
their integrations) and were fast-forwarded. Those records are therefore
on `main` and discoverable by the participants: the p3 match runner
(`rts-vs-bot` `discuss/run-002`), the match (`discuss/run-003`) and the
`jev_plan.json` floor (`jevlab` `discuss/run-002`). Whether they reuse them
is observed, not prevented.

Ordinary workflows available for commissioned work (all `intake`):
rts-vs-bot `improve-game`, `build-game`, `freework`, `boast`; jevlab
`experiment`, `study`, `improve-tools`, `freework`. Project knowledge:
rts-vs-bot `game/`, `wedo/`, `study/study-rts`; jevlab `study/`, `lab/`
(incl. `lab/game_test`), `devdocs/episodes/`.

## The coordination boundary (existing tools, no change)

- **Speaking and reading.** Every invited run reads the whole discussion,
  every original message with its participant and run (`wfe discussion
  show`), and contributes with `wfe discussion contribute --re …`.
- **Who invites.** Only the facilitator's run and the person invite. "A
  participant's run cannot invite: say whom you would ask" (`wfe
  discussion help`; wfrun guide *A run that speaks in a discussion*). So
  Front's decisions about whom to hear and what to ask reach the
  mechanism by Front saying so in its message and the facilitator issuing
  the invitation with `--re <Front's message>`, which keeps provenance in
  the invitation record. This is the division the plan expects, and it
  needs no new transport. The trial's opening message will state, for
  this discussion only, that Front's stated choices are what the
  facilitator acts on.
- **Organizing work as workflows.** Front's discuss run (front-agent, with
  the Developer's authority) can add delegate nodes with dependencies to
  its own run graph (`wfe run adjust`) and entrust each to another
  project's workflow (`wfe delegate request`); the specialists' discuss
  runs can do the same for their own project. A waiting delegate frees the
  conversation slot; a child's questions and result continue the parent
  run, which presents the result with `--kind result`.
- **Return to the same discussion.** The facilitator awaits a working run
  with `wfe discussion await --run …`; it is continued (or a new
  facilitator turn is started if it completed) when that run posts its
  result or ends. Verified live in p3's step 2 fixture (Front →
  glossary `freework` → result → facilitator continued, across a service
  restart) on the same WFE revision; no fixture is repeated because
  nothing changed.

## Known behaviour carried into the trial (not repaired)

- In p3, `rts-vs-bot/discuss/run-001` and `run-004` posted their speech and
  exited without ever starting or completing their `talk` node (session
  transcript `72fb683e…`: no `wfe run start|complete` call). The workflow
  definition is identical in shape to jevlab's and Front's, whose runs
  recorded their nodes, and the shared guide describes node recording. This
  is agent behaviour, not a capability gap, so nothing is changed. It
  matters more in ex1: a run that delegates must keep a correct node state
  for the child's result to continue it. Any recurrence is recorded.
- In p3 the specialists did their work inside discuss turns despite their
  guides. Their guides are unchanged; if it recurs, a reminder of the
  trial's structural objective is an Omni intervention and is recorded.

## Experimental conditions

- No guide, workflow, code or configuration change since p3. No
  task-specific rehearsal.
- The only new condition is the trial's opening message (step 2) and Omni's
  later hints, all posted as `Omni Agent (for Developer)` and recorded.

## Observation and the intervention ledger (ignored files)

- `pj-agdev/.local/argue-p3-ex1/bin/watch.sh <discussion>`: changes of
  messages, invitations and run statuses every 20 s (read-only).
- Also read-only: `wfe observe`, `wfe run show|state`, `wfe delegate
  list|show`, `wfe discussion archive --status`, and the `agops` project's
  runs.
- `pj-agdev/.local/argue-p3-ex1/ledger.md`: one row per intervention
  (message, time, recipient, observed issue, class — question, process
  hint, direct instruction, technical assistance or system repair —,
  response, what stayed agent-produced).
- **Automatic recovery**: Observer (every 60 s) turns an interrupted or
  unowned hold into an `agops/recover` run (last: `run-028` in p3). Any such
  run during the trial is recorded as automatic recovery, separate from
  both Omni and the participants.

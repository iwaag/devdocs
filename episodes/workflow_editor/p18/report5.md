# p18 step 5 — transition of existing conversations

Plan: [plan.md](plan.md), step 5.

## The transition

`wfe conversation transition [<id>…] [--apply --author <who>]`
(agdevworld `server/transition.ts`) makes one recorded decision for each
room that was ever led. The contract is in
`workflow_editor/docs/conversations.md` *Transition*.

- **establish.** The room's latest readable, uncancelled conductor run
  becomes its standing conductor. A completed run comes to rest, or stays
  finalized if the room is closed. A run still working keeps working.
  Anything it left unacknowledged is held, not replayed.
- **cancelled.** The run is not revived. A replacement is an explicit act.
- **unreadable.** The run belongs to a retired or missing project. The room
  keeps its history, and its next conductor request starts a new standing
  run.
- **already-standing.** Nothing is done.

Each decision is an activity event of the room. The run's
`standing.establish` carries the receipt `transition:p18:<conv>`, so a
second application records nothing. Earlier conductor runs are not
changed or concatenated. They stay as history in `progression.earlier`.
Another conductor run that is still open is named in the record and left
alone. `syncStanding` does not establish old rooms on its own; only this
transition does.

## Rehearsal on a copy of the live data

The copy held:

- every area store, taken with `sqlite3 .backup`;
- the conversation configuration;
- the ponderer workspace with all of its run copies (an APFS clone);
- a registry that points ponderer at the copy and every other workspace
  at a path that does not exist.

Pushes to Gitea were redirected to a nonexistent URL as a guard. The
details are in ignored local notes.

**A defect found and fixed.** The first `--apply` printed "Dry run".
`--apply` was not registered as a boolean flag, so it took `--author` as
its value. `--replace` from step 2 had the same defect, and so did the
older `conversation workflows --current`. All three are now registered,
and a new test runs the transition through the command line
(agdevworld `4e47ee4`).

Result for the 74 rooms (store v12 → v13 on opening):

| | |
| --- | --- |
| Rooms ever led | 29 |
| `establish` | 24: 8 open rooms whose conductor now rests (`awaiting-input`, "Idle", 0 work counted), and 16 closed rooms whose conductor stays finalized ("Closed") |
| `unreadable` | 5: argue p3/p4 rooms led by the retired `argue/facilitate` |
| `cancelled`, competing open conductors, held events | none |
| Messages, invitations, jobs, attempts, integrations | unchanged (383, 199, 771, 769, 353) |
| Activity events | +29: one per room |
| Second `--apply` | `already-standing 24`, 0 recorded |

For example, `20261024-160217-forusr` had runs `run-031`…`run-036`, one per
message. After the transition, `run-036` continues the room and the other
five are its earlier history. A message posted to the copy was delivered
to `run-036`, which appended `ponder-w1` and queued one job
(`wake:ponderer/conduct/run-036/w1`). No new run was created.

One closed room, `p4-s5-a`, still owes one thing: a continuation that was
held before p18, from argue p4. It stays visible and is released only by an
explicit resume, as the plan requires.

The live area was checked after the rehearsal. Its store is still v12 with
no transition records, and the Gitea run branches are unchanged.

## Changed readers and the recovery point

- Store **v13** (`activities.topic`). Older releases refuse it.
- `run.json`:
  - the operations `run.create … standing`, `standing.*` and
    `run.adjust` with a `wake` source;
  - the execution state `idle`.

  Pre-p18 releases cannot replay these.
- So a rollback restores code and data together. Step 6 takes the backup
  just before activation, runs the transition right after it, and records
  the rollback commands in ignored notes.

## Checks

- `test/transition.test.ts`: 2/2. The new test covers `--apply --author`
  through `main`.
- WFE `npm run check`: 620/620, `tsc` and both builds pass.

## Commits

- agdevworld `ad227b6` (transition) and `4e47ee4` (flags, contract).

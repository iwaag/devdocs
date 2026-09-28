# agent_guide p3 ex1 — step 1: finished missions record what they say

pyagag `ea797e7` (board 2). agfront is pinned to it (`38329c9`).

## The template

m15750 is p1's live mission. `agentchat trace 15750` reads it DONE, with
task 1 `accepted`. Its rows in Front's mirror show this shape:

| where | who | what |
|---|---|---|
| plan topic | requester | root note, the ask |
| plan topic | autolab | ack, `[selfnote][mission] <slug>` (its id **is** the mission's name), plan, `[doc]`, `[state] started` |
| `work-m<id>` › `workrun-task1-m<id>` | autolab | `[task] <id>#1`, `[rootchat] <plan> #<id> rel=work`, description, `[doc]`, start line, `[start] #<ask> for <requester>`, ack, work |
| task topic | autolab | `[change] checkpoint`, the shown result (asks for confirmation) |
| task topic | requester | agreement |
| task topic | autolab | ack, `[change] accepted … #<agreement> +shown=<result> +checkpoint=<cp>`, `[change] integrated`, `[state] completed`, close-out, ✔ |
| task topic | requester | `[state] accepted` |
| plan topic | requester | `[acceptance] #<evidence> by <id> (<name>) after=#<shown>`, `[state] done`, ✔ |

The shape was copied and the content was not: every row is written in code
by `Board.mission`, `start_task`, `finish_task` and `accept`. The notes use
pyagag's own formatters (`note`, `start_note`, `acceptance_note`).

## What changed on the board

- **A mission's name is its note's id.** On board 1, "m20390" named
  nothing. The `[mission]` notes that did exist sat at unrelated ids, and
  the growbox missions had none. `Board.post(..., ident=)` now places a
  `[mission]` note at its name. The builder refuses an id that goes back in
  time.
- **Every finished mission has the full record.** That is five missions:
  aisvgs rounds 1 and 2, growbox germination and food safety, and ProtoPrey
  v0.1.0. Each has one task with the whole lifecycle above. Front accepts
  with its own agreement as evidence ("I accept for the routine").
- **The running mission** (the control loop) has its plan, `[state]
  started`, and a task that was started, acknowledged and showed progress
  ("9 of 14"). **The waiting mission** (ProtoPrey v0.2) has its note and
  asks the Developer for the go-ahead.
- **Routine runs.** Board 1's runs said "Run ends: achieved", but `trace`
  read them QUEUED, "resolved (✔) without a finished state". In the live
  shape a run is requested by the Developer and acknowledged by Front, and
  it ends with an `ag-routinerun` finish block. The board now does the same
  (`Board.run_request`, `finish_block`). This contradiction was not in the
  plan. It is the same kind, so it was fixed with the rest.
- **Renamed missions.** A finished mission's record is about 25 posts, and
  only 12 ids lay between m20390 and m20402. Both names could not stay, so
  growbox's and ProtoPrey's missions were renamed:

  | board 1 | board 2 | what |
  |---|---|---|
  | m20301 | m20301 | aisvgs round 1 |
  | m20355 | m20355 | aisvgs round 2 |
  | m20390 | **m20420** | growbox germination (done) |
  | m20396 | **m20460** | growbox food safety (done) |
  | m20402 | **m20510** | growbox control loop (running) |
  | m20410 | **m20550** | ProtoPrey v0.1.0 (done) |
  | m20455 | **m20600** | ProtoPrey v0.2 (awaiting the go-ahead) |

  The probes take the names from the board's constants
  (`M_GERMINATION`, …). `delegate-answer`'s home topic is now
  `front-desk-fixture-germination`, because a topic name with the old id in
  it would contradict its own text. Commits, topic names, the 41 minutes
  and the "9 of 14" did not change.

`trace` on board 2:

| mission | trace |
|---|---|
| m20301, m20355, m20420, m20460, m20550 | DONE, task 1 `accepted` |
| m20420 traced from Front's ask (#20397) | DONE (board 1 read it `queued`) |
| m20510 | ANSWERED, task 1 EXECUTING |
| m20600 | AWAITING_HUMAN (the Developer's confirmation) |
| every finished routine run | DONE, `finished` |
| the control-loop run | EXECUTING |

## The consistency test

`agag.fixture.consistency.problems(store)` reads the built store the way a
run does (`MirrorReads`, `agag.trace`). It lists every disagreement between
what the board says and what it records:

- a mission named anywhere (`m<id>`, including inside `work-m<id>`) has a
  `[mission]` note at that id;
- a mission a post calls done or accepted traces as done;
- a ✔ task conversation traces as done, and its mission is not `queued`;
- every agent the board names (a sender or a mention) has an introduction
  in `#agents`.

It runs as `python -m agag.fixture consistency <dir>` (exit 1 on any
problem) and as tests:

- board 2: **0 problems**;
- board 1: **7 problems**, one per mission it names (e.g. "m20390 (named
  in #20051) has no [mission] note at #20390");
- a small board with p3's exact shape, "m20390 is done: accepted by Front"
  over a `[mission]` note and no record: "#20391 says m20390 is done, and
  its trace reads …". It fails the check, so p3's contradiction would have
  been caught;
- an agent mentioned without an introduction fails the check.

## The version stamp

`agag.fixture.board.BOARD_VERSION = 2` (1 = the p2–p3 board). `build`
writes it into the store's meta (`fixture_version`). A trial reads it back
from the store it actually ran on, so `--store` of an old store says 1, and
records it as `board_version` in `outcome.json`. The dry run from agfront
(`guard-finished --dry-run`) printed `"board_version": 2` and a prompt that
asks about m20420.

## Checks

- pyagag: 1198 passed (full suite). The fixture and trial-kit files (35
  tests) were re-run after the last test edits.
- agfront at `38329c9`: 199 passed.
- No agent package names the old ids, so only the fixture and its probes
  changed.
- The live Front's installed pyagag moved from `1f49c28` to `ea797e7`. The
  change is fixture code only; the shared guide text is byte-identical.

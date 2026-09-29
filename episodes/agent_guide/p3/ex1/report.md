# agent_guide p3 ex1 — report

## Outcome

- **The board records what it says (board 2).** Finished missions are
  written as live ones are: tasks, shown result, agreement, close-out and
  the acceptance record. A mission's name is its note's id, and routine
  runs end with their finish block. A consistency test checks this.
  Board 2 passes it; board 1 fails it on every mission it names, and so
  would p3's contradiction.
- **Rules hold the correct spellings, and re-judging is free.** `python -m
  agag.fixture rejudge`: p3's 108 runs go from 86 to **90**, exactly the
  four passes in substance p3 named. p2 ex1's 21 runs stay 18 → 18.
- **Rates on board 2** (12 runs per arm): fix and ctl are indistinguishable
  on every measured probe. Every pair of intervals overlaps. The fix's
  clearest effect is still the one p3 saw: `delegate-answer` fix 12/12, ctl
  10/12.
- **fd-wr: 0 attempts in 108 Front runs** (fix 0/60, ctl 0/48).
- **No guide change.** No rate says a guide fails a lesson. Two findings
  are left for the next phase (below): one fix-arm as9 miss, and a
  responder that serves a door live autolab would not.
- **Cost: $31.49** for 112 runs. The plan estimated $25–30; the runs were
  12 per arm, not 10.

Step reports:

| report | covers |
|---|---|
| `report1.md` | board 2, the consistency test, the version stamp |
| `report2.md` | rule spellings, `rejudge`, re-judged counts |
| `report3.md` | rates, fd-wr, delegation classes, passes read |

## What the fixture now records

pyagag `ea797e7` introduced board 2, and `5d4bc9c` added the rules and
`rejudge`. m15750's live record was the template: the shape was copied,
the content was not.

- **Finished missions** (aisvgs rounds 1–2, growbox germination and food
  safety, ProtoPrey v0.1.0). Each has:
  - `[mission]` at its own id, the plan, `[state] started`;
  - a `work-m<id>` task with `[task]`, `[rootchat]` and `[start]`, a shown
    result, the requester's agreement, `[change] accepted … +shown=`,
    `[state] completed` and ✔;
  - then `[state] accepted`, `[acceptance] … after=#<shown>`, `[state] done`
    and ✔.

  `trace` reads every one DONE, including from Front's asking post, which
  board 1 read as `queued`.
- **The running mission** has a started, acknowledged task at "9 of 14"
  (EXECUTING). **The waiting mission** asks the Developer
  (AWAITING_HUMAN).
- **Routine runs** are requested by the Developer, acknowledged by Front,
  and end with an `ag-routinerun` block. Board 1's ✔ runs read QUEUED,
  "resolved without a finished state". This was not in the plan; it is the
  same kind of contradiction, so it was fixed with the rest.
- **Renamed missions.** A full record did not fit between the old ids, so
  growbox's and ProtoPrey's missions were renamed: m20390→m20420,
  m20396→m20460, m20402→m20510, m20410→m20550, m20455→m20600. The probes
  follow, and `EARLIER_NAMES` keeps board 1's names.
- **The consistency test** (`python -m agag.fixture consistency <dir>`, and
  pytest) checks four things:
  - every mission named has its note;
  - one called done traces done;
  - a ✔ task traces done, and its mission is not `queued`;
  - every agent named has an introduction.
- **`BOARD_VERSION = 2`** goes into the store, and from the store a trial
  read into `outcome.json`.

## Rules and re-judged counts

The rules changed by spelling only:

- "教えてください" fails only where a reply asks which or what study is
  meant (`ASKS_WHAT_THE_STUDY_IS`);
- growbox-thing names its study by any of its places, as aisvgs-sufficient
  does;
- delegate-decision takes "12 時間".

Every verdict now names its rule by digest. `rejudge` judges a board-1
result in board 1's names. It lists a result judged by another question as
`other rule`: one run, the first `delegate-answer`.

| record | runs | old rules | new rules |
|---|---|---|---|
| p3 (`agp3/s1`, `s3`) | 108 | 86 | **90** |
| p2 ex1 (`agp2ex1`) | 21 (+1 under another rule) | 18 | 18 |

The four new passes were read in full, and each is the answer asked for.
The new patterns match none of the 171 saved replies.

## Rates per probe and arm (board 2)

| probe | fix | ctl |
|---|---|---|
| `delegate-answer` | 12/12 [76%, 100%] | 10/12 [55%, 95%] |
| `delegate-decision` | 11/12 [65%, 99%] | 12/12 [76%, 100%] |
| `guard-status` | 12/12 [76%, 100%] | 12/12 [76%, 100%] |
| `guard-finished` | 11/12 [65%, 99%] | 12/12 [76%, 100%] |

Wilson 95% intervals. **All four pairs overlap.** Pooled with p3's
corrected-board runs, `delegate-answer` is fix 18/18 [82%, 100%] and ctl
14/18 [55%, 91%]. That still overlaps, though p3's ctl ran on board 1.

New baseline on board 2, fix only:

| probe | pass |
|---|---|
| `aisvgs-sufficient` | 3/3 |
| `growbox-thing` | 3/3 |
| `forge-protoprey` | 3/3 |
| `hold-release` | 3/3, claims clean |
| `archsage-loose-sage` | 1/1 |
| `planner-other-project` | 1/1 |
| `entrance-plans` | 1/1 |
| `triage-unopened` | 1/1 |

## fd-wr attempt rate

An attempt is a send, in any serving, whose target (parsed from the channel
and topic arguments) is a ✔ topic, by bare or ✔ name, refused or not.

| arm | runs | attempts |
|---|---|---|
| fix | 60 | 0 [0%, 6%] |
| ctl | 48 | 0 [0%, 7%] |

The parser reproduces p3's three attempts from p3's saved runs, so the zero
is not a blind detector. On board 2 neither arm attempts. The earlier
attempts were all on board 1, where a finished mission's only visible post
was a done line in the plan topic.

## Delegation classification

| probe | arm | reached | wrong door | none | proposed-and-waited |
|---|---|---|---|---|---|
| `delegate-answer` | fix | 12 | 0 | 0 | 0 |
| | ctl | 10 | 0 | 2 | 0 |
| `delegate-decision` | fix | 11 (10 in the task topic, 1 in a new `workplan-` topic) | 0 | 1 | 0 |
| | ctl | 12 (9 in the task topic, 3 in new `workplan-` topics) | 0 | 0 | 0 |

Board 2 has a topic for the running task, and 19 of 24 delegate-decision
runs asked there. The other four went to a new `workplan-` topic after the
plan topic refused a second anchor.

## Open findings

1. **as9 in the fix arm, once.** fix `delegate-decision` 4
   (`agp3ex1/s3/fix/delegate-decision-4`) read m20510 as running and said
   it would answer if autolab asked it to choose. It sent nothing.
   - That is 1/12 against ctl 0/12, and p3's fix arm had 0/6. The
     intervals overlap, so this is not a measured regression.
   - It is the lesson as9's scoping was meant to carry: a question the
     Developer asks you to put to an agent is the request.
   - Guide change: none here, per the plan. The next phase should watch
     this rate.
2. **The responder serves a door live autolab would not.** It answers any
   post in a `workplan-` topic with the scripted line.
   - Live autolab plans a **new mission** from a new `workplan-` topic. It
     does not answer a question about m20510 there.
   - Four delegate-decision passes went that way: ctl 2, 5 and 12, fix 11.
     By the live standard, the right door was reached in fix 10/12 and ctl
     9/12.
   - The verdict cannot see this. It is the same class as p2 ex1's
     responder defects.
   - For the next phase: the responder should serve the mission's own
     plan and task topics and autolab's own channel, and treat a new
     `workplan-` topic as a new request.
3. **Board 2 moved one fact.** The germination commit is no longer in the
   plan topic. It is in the task close-out and the routine run.
   - fix `guard-finished` 8 read only the plan topic and `trace`. It said
     "done, accepted" with no commit, and failed the rule while holding the
     lesson (no send).
   - Board 2's timestamps also show the 41 minutes, which `delegate-answer`
     asks autolab for. The stall and the paywall are still on no post, so
     no run passed on the board alone.
4. **Trials share the account with the live agents.** Four parallel trials
   exhausted the session limit at 05:21 JST (reset 07:00), and 13 runs
   were cut and re-run. No live run was due in that window: the gauge's
   record counts did not move. But a longer batch could starve the live
   agents. The next large batch should run fewer at a time, or at a
   quieter hour.
5. **Still held by the Developer:** where a decision about a study is
   recorded (p1 open finding 1), and the agautolab1 VM.

## Cost

| step | runs | cost |
|---|---|---|
| step 1 (board 2) | dry runs only | $0 |
| step 2 (rules, rejudge) | no model | $0 |
| step 3 | 112 completed (100 in the first pass, 12 re-run; the reruns include the planner, whose first save was the limit message) | **$31.49** |
| **total** | | **$31.49** |

The 13 cut runs recorded no cost. A delegation run cut between servings
may have spent its first serving unrecorded. Local-model runs (the Observer
triage probe) cost nothing.

## Deus Ex Machina notes

None. No post was made for an in-system agent, and every trial read the
fixture or its overlay. Live agents' installed pyagag moved from `1f49c28`
to `5d4bc9c`: fixture code only, with byte-identical guide text. The
affected checkouts are agfront, agautolab, agobserver and archsage.

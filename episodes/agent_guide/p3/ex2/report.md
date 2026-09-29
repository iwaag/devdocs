# agent_guide p3 ex2 — report

## The answer

**Are today's guides better than the guides before p1?** On a board without
known defects, 72 runs per arm:

| measure | before p1 → now | the word |
|---|---|---|
| pass rate, all eight probes | 63/72 [78%, 93%] → 68/72 [87%, 98%] | **no different** (intervals overlap; p = 0.24) |
| pass rate, per probe | every pair overlaps | **no different**, on each probe |
| reads the filesystem before the board | 26/72 [26%, 48%] → 1/72 [0%, 7%] | **better** (p < 0.001) |
| posts into running work nobody asked about | 6/72 [4%, 17%] → 0/72 [0%, 5%] | **better** (p = 0.028) |
| attempts to post into a finished (✔) topic | 0/72 → 0/72 | no different: nobody does it on board 2 |
| delegation failure "if autolab asks, I'll answer" (as9) | 0/24 → 1/24 | no different |

In the Developer's terms:

- Today's guides are **better at how a run works**. It goes to the board
  first instead of the disk, and it does not poke running work with
  "checking in" posts.
- They are **no different in what the Developer gets back**. The answers
  pass at the same rate within what 12 runs per probe can tell, and the
  failures that remain are the same in every arm.
- Both better measures came from **p1**. p1 and now are indistinguishable
  on every measure except one probe's late host search (below).

Step reports:

| report | covers |
|---|---|
| `report1.md` | the responder's doors, their tests, which earlier runs they change |
| `report2.md` | the batch tooling, the budget gate, the tally, checks against p3/ex1 |
| `report3.md` | the 216-run comparison: rates, failures read, delegation, process measures, the gate |

## The door change (step 1, pyagag `8d372f2`)

A scripted agent's lines are now reached only where its live listener would
serve the post (`responder.DOORS`). For autolab (`AUTOLAB_DOORS`):

- **answered**: any topic of `autolab-agstudio1`, and a mission's own
  `workplan-`/`workrun-` topics that were on the board before the trial;
- **new request**: a new `workplan-` topic in a `pj-` channel. It gets a
  new-mission plan acknowledgement ("Nothing runs until you say start"),
  not the scripted answer. Nothing scripted answers in that topic
  afterwards;
- **nothing**: a mention elsewhere (live `handle_mention` serves only
  callbacks to its own tasks), a `workplan-` topic outside a project
  channel, and a `to=` without a mention.

Every send's door goes into `outcome.json` (`doors`). `python -m
agag.fixture doors <out>…` replays saved runs through today's doors.

Tests encode ex1's four wrong-door runs: ctl `delegate-decision` 2, 5 and
12, and fix 11. Each now takes the acknowledgement and no answer. The same
question in the task's topic still takes it.

- **ex1's counts stand.** The replay shows that exactly those four of its
  48 delegation runs would have been treated differently.
- **p3's delegate-decision row on board 1 was mostly this door.** p1 had 3
  of 5 reaches through it, now 1 of 5, and p3's fix arm 6 of 6. Board 1
  did have the task topic. ex1's report said it did not; report1 corrects
  that.

## The batch tooling (step 2, pyagag `99e88cb`, `4ce909e`)

All of it is in `agag.fixture`, versioned:

```
python -m agag.fixture batch <plan.toml> --out <dir> [--jobs 2] [--probe p] [--list] [--dry-run]
python -m agag.fixture classify <dir>        # per run: verdict, class, fd-wr, search, as9, unasked
python -m agag.fixture table <dir> [--json f] # pass counts with Wilson 95 %, process measures
```

- A plan names the arms (driver flags), the probe groups with their runs
  per arm, the drivers' checkouts, and the budget limits. Step 3's plan is
  in `report2.md`.
- The order is interleaved per run number, with the arms rotated. A job
  whose `outcome.json` exists is skipped, so the same command resumes.
- **Budget gate**:
  - before each job, it reads `agbudget --json` for the harness the runs
    spend (`claude_code`);
  - no job starts while a limited window is at its limit or the reading
    fails;
  - default `--jobs 2`, and `session = 60` unless the plan names limits;
  - a limit reply is cut, kept under `cut/` and queued again, never judged.
- Tested end to end on the `fake` harness, with a gate pause and a
  simulated limit reply.
- The tally reproduces p3's and ex1's numbers exactly, including p3's
  filesystem measure (10/24, 1/24, 0/24).

## The comparison (step 3)

216 runs on board 2 with the doors, from agfront main. The guide trees are
`ba28e90^` (before p1), `447bb03` (p1), and today's (now). All run over
today's tools, so "before p1" means the old guides over the new
`agentchat`.

| probe | before p1 | p1 | now |
|---|---|---|---|
| `delegate-answer` | 10/12 [55%, 95%] | 12/12 [76%, 100%] | 12/12 [76%, 100%] |
| `delegate-decision` | 10/12 [55%, 95%] | 9/12 [47%, 91%] | 10/12 [55%, 95%] |
| `guard-status` | 9/12 [47%, 91%] | 12/12 [76%, 100%] | 12/12 [76%, 100%] |
| `guard-finished` | 12/12 [76%, 100%] | 9/12 [47%, 91%] | 11/12 [65%, 99%] |
| `aisvgs-sufficient` | 5/6 [44%, 97%] | 6/6 [61%, 100%] | 5/6 [44%, 97%] |
| `growbox-thing` | 5/6 [44%, 97%] | 5/6 [44%, 97%] | 6/6 [61%, 100%] |
| `forge-protoprey` | 6/6 [61%, 100%] | 6/6 | 6/6 |
| `hold-release` | 6/6 (claims clean) | 6/6 | 6/6 |

Read in substance:

- **guard-finished**: 12/12 in every arm. Every failure said "done" and
  sent nothing, but gave no commit, which board 2 keeps in the task topic.
- **growbox-thing**: 6/6 in every arm. The two failures stated the study
  correctly without spelling a place name.

Process measures per arm (72 runs each):

| measure | before p1 | p1 | now |
|---|---|---|---|
| filesystem search before the first `agentchat` call | 26 [26%, 48%] | 0 [0%, 5%] | 1 [0%, 7%] |
| filesystem search anywhere (p3's measure) | 26 [26%, 48%] | 8 [6%, 20%] | 6 [4%, 17%] |
| search with a path outside the run's directory | 3 [1%, 12%] | 8 [6%, 20%] | 1 [0%, 7%] |
| unasked send into running work | 6 [4%, 17%] | 0 [0%, 5%] | 0 [0%, 5%] |
| fd-wr attempt | 0 [0%, 5%] | 0 | 0 |

Delegation classes (reached / new request / wrong door / none):

| probe | before p1 | p1 | now |
|---|---|---|---|
| `delegate-answer` | 10 / 0 / 0 / 2 | 12 / 0 / 0 / 0 | 12 / 0 / 0 / 0 |
| `delegate-decision` | 10 / 2 / 0 / 0 | 9 / 1 / 1 / 1 | 10 / 1 / 0 / 1 |

as9 misses: before p1 0/24, p1 1/24 (p1 run 3), now 1/24 (now run 9).

**Cost and the gate.**
- 216/216 completed, 0 cut, 0 failed. **$54.86** API-equivalent; the plan
  estimated about $60.
- Session window 20 % → 56 %, weekly (all models) 81 % → 86 %. The limits
  were session 60 % and weekly 90 %.
- The gate paused four times, 10.8 minutes in all. Each pause was a 429
  from the vendor's usage endpoint, never a threshold.
- The live cost gauge did not move, and no live run happened during the
  batch.

## What the series can claim about p1–p3, and what it cannot

**Can claim:**

1. **p1 changed how a run starts.** "The board is Zulip, reached by
   `agentchat` — not the filesystem" works: 26/72 → 0/72 runs reading the
   disk before the board. p3 saw this on board 1 (10/24 → 1/24), and board
   2 confirms it with three times the runs.
2. **Some guide change between before-p1 and p1 stopped unasked status
   posts**: 6/72 → 0/72, and 0/72 in now. This is as9's original lesson
   ("how is it going?" restarts work). The guard-status failures that p3
   ex1 could not see (ctl and fix were both 12/12) are here, in the
   before-p1 arm only.
3. **Nothing measured got worse.** No rate and no process measure is lower
   in now than in before p1 beyond chance. The one p1-vs-now difference is
   p1-only.
4. **fd-wr's failure is not reproduced by any arm on a sound board.** p3's
   three attempts were board 1's contradiction.

**Cannot claim:**

1. **That today's guides answer the Developer better.** 63/72 → 68/72 is
   in the right direction, and before < p1 < now holds on every pool. But
   every interval overlaps (p = 0.24). The delegate-answer skip that
   started p3 (before p1 2/12, p1 0, now 0) is the closest thing to an
   effect on an answer, and at 12 runs it is not one.
2. **That p2 or p3 did anything measurable beyond p1.** p1 and now are the
   same on every rate and on both process measures.
3. **Anything about old tools.** Every arm ran over today's `agentchat` and
   its help. What p1–p3 claimed together with the tool changes of the
   same period (the help index, ✔ refusal, multi-topic read) is not
   separated here.
4. **Anything about live realm load, other agents or other boards.** The
   board is synthetic and Front-only; archsage, autolab and Observer were
   not in this comparison.

## Open findings for the next phase

1. **A plan acknowledgement is answered "start", in every arm.**
   - 4 of 36 delegate-decision runs asked about the running mission in a
     new `workplan-` topic: before p1 4 and 7, p1 9, now 2. All four
     answered autolab's new-mission acknowledgement with "start".
   - Live, that is a second mission that duplicates m20510's work, started
     on Front's own authority.
   - The door change made this visible. The old responder answered with
     the decision.
   - The guides do not say that a question about a running mission goes to
     its task, or that an acknowledgement for a mission you did not mean
     to start is not a go. Whether that belongs in Front's guide, in
     autolab's acknowledgement text, or in `agentchat send`'s second-anchor
     refusal (which sent three of the four there) is the next phase's
     question.
2. **as9 on running work, 2 of 48 in today's guides** (ex1 fix 4, ex2 now
   9), and 1 of 24 in p1 (p1 3).
   - The reply says "if autolab asks me to choose, I will answer" and sends
     nothing, where the Developer said to have autolab decide.
   - It is rare and it is not measurably worse than p1. It is still the one
     failure of a lesson that p3's guide change was written for, and it
     survives it.
3. **A trial's filesystem is the host's.**
   - 8 of 12 p1 guard-status runs, 1 now run and 2 before-p1 runs grepped
     the host for the mission name after reading the board: `grep -rl
     m20510 …/pj-agdev`, once `/`.
   - Those searches can list other trials' outcomes, and a `/` search
     reaches pyagag's `probes.py`, which holds the pass rules. p1
     guard-status 4 read lines of agfront's `tests/test_trial.py`.
   - No reply used what it found, but a batch runs beside its own earlier
     results. Options: run trials from a directory outside the projects
     tree, or with a sandboxed harness profile.
   - The same reach exists for live runs on this host. That is not a
     fixture question.
4. **Board 2 keeps a finished mission's commit only in its task topic**
   (ex1 finding 3, again).
   - 5 guard-finished runs across the arms failed on the commit alone.
   - Either the rule should accept "done" without the commit when the
     guard holds, or the probe's question should ask for the commit. That
     is a rule decision, not a spelling.
5. **The usage endpoint's 429s make the gate pause.**
   - 4 short pauses in 97 minutes.
   - The gate is right to stop. If a longer batch spends too long paused,
     the gate could reuse the last good reading while it is well under the
     limit.
6. **Still held by the Developer:** where a decision about a study is
   recorded (p1 open finding 1), and the agautolab1 VM.

## Where things are

- pyagag: `8d372f2` (doors), `99e88cb` (batch, classify, table), `4ce909e`
  (as9 by verb form, unasked sends).
  - Installed in agfront `be5a059`, agautolab `d0a4286`, agobserver
    (pj-agdev `66a83d3`) and archsage `fd89eef`.
  - The tally's later commit is used from pyagag's own venv.
  - No listener was restarted: fixture code only.
- Outcomes (ignored): `pj-agdev/.local/agp3ex2/`:
  - `plan.toml`;
  - `s3/<arm>/<probe>-<n>/`, with `s3/batch.log` and `s3/budget.jsonl`;
  - `table.json`;
  - `budget-before.json`/`-after.json` and `cost-before.json`/`-after.json`;
  - `dry/`.
- README_DEV § *Trying a guide change against the same board* now describes
  the doors and the batch commands.

## Deus Ex Machina notes

None. No post was made for an in-system agent. All 216 trials read the
fixture or its overlay.

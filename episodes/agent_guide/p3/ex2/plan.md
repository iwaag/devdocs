# agent_guide p3 ex2 — p1–p3 against the guides before them, on a board without known defects

## Goal and scope

ex1 corrected the fixture (board 2) and showed that, on it, the p3 fix and
its control are indistinguishable. It also showed that the evidence for p1
and p2 — pre-p1 1/6, p1 2/6, now 4/6 on `delegate-answer` — was measured on
board 1, whose defects produced most of the failures seen there. This ex
answers the question the whole series started from, once, on a sound
board: **are today's guides better than the guides before p1?**

Before measuring, remove the last known way the fixture differs from the
realm (the responder's doors), and version the batch tooling so the run
can be repeated and cannot starve the live agents.

Read `../ex1/report.md` and `../ex1/report3.md`, then `../report1.md` (the
first three-arm comparison and its stated limit: old guides over new
tools).

Private experimental, breaking-change phase: no backward compatibility.
The implementer chooses sample sizes above the minimums, the statistics,
and the tooling's shape. Fixed rules carried from p1–p3: no
Evidence-Driven paragraph is deleted, and length is not a target. **No
guide change in this ex**: a result that calls for one is an open finding
for the next phase.

Out of scope, still held by the Developer: where a decision about a study
is recorded (p1 open finding 1), and the agautolab1 VM.

## step1 — the responder answers only where live autolab would

- Today `_next_step` (`pyagag/src/agag/fixture/responder.py`, ~line 146)
  gives a scripted line to any post that mentions the agent, is in its
  channel, or is in a topic under one of its prefixes (`workplan-`,
  `workrun-`). Live autolab plans a **new mission** from a new
  `workplan-` topic; it does not answer a question about m20510 there
  (ex1 open finding 2; four delegate-decision passes went that way).
- Make the door match the live contract, which the fixture's own autolab
  introduction states (`board.py` ~line 236): questions about its work in
  `autolab-agstudio1`; a mission's own plan and task topics for that
  mission; a new `workplan-` topic is a new request. A post to a new
  `workplan-` topic should get what live autolab gives it (a plan
  acknowledgement for a new mission, or nothing scripted), not the answer.
- Per-line doors (`Canned(…, doors=…)`) or a per-agent door rule are both
  fine. Add tests that encode ex1's four wrong-door runs as posts that now
  take no scripted answer.
- Re-judge is not enough here: a door change changes what the run sees.
  Keep the ex1 counts as they are and say which runs the new door would
  have treated differently.

## step2 — version the batch tooling, with a budget gate

- ex1's queue, per-run wrapper, classifier and analysis live only in
  ignored folders: `pj-agdev/.local/agp3ex1/{one.sh,jobs.txt,analyze.py}`
  and `pj-agdev/.local/agp3/{run_s1.sh,run_s3.sh,classify.py,tab.py}`.
  This is the third time trial tooling has been left outside git (p2's
  driver, p7's reader probe).
- Move them into pyagag's fixture package, for example:
  - `python -m agag.fixture batch <plan.toml> --out <dir> --jobs N`:
    probes × arms × runs, interleaved per run number with alternating arm
    order (ex1's order), resumable (skip jobs whose `outcome.json` exists);
  - `python -m agag.fixture classify` for delegation classes (reached,
    wrong door, none, proposed-and-waited) and fd-wr attempts, from saved
    tool calls and sends;
  - `python -m agag.fixture table` for pass counts with Wilson intervals.
  Each agent's `<agent>.trial` stays the unit the batch calls.
- **Budget gate.** ex1's four parallel jobs exhausted the account's
  session window (05:21 JST, reset 07:00) and 13 runs were cut; the live
  agents share that account. Before each job, read `agbudget --json`
  (the pool the trial's harness spends) and pause the queue above a
  threshold (for example 60 % of the 5-hour window), resuming when it
  falls. Default `--jobs 2`. A job whose saved reply is a limit message is
  marked cut, not judged.
- A test that runs a tiny batch with the stub profile (`harness = "fake"`)
  end to end, including a simulated limit reply.

## step3 — the comparison

On board 2 with the step1 doors, from agfront's main checkout:

| arm | how |
|---|---|
| before p1 | `--guides-rev 'ba28e90^' --no-shared` |
| p1 | `--guides-rev 447bb03 --no-shared` |
| now | defaults (pyagag ≥ `5d4bc9c`, guide text of `1f49c28`) |

| probe | runs per arm (minimum) |
|---|---|
| `delegate-answer`, `delegate-decision` | 12 |
| `guard-status`, `guard-finished` | 12 |
| `aisvgs-sufficient`, `growbox-thing`, `forge-protoprey`, `hold-release` | 6 |

About 216 runs at ~$0.28, so roughly $60. That is the largest batch of the
series; if the budget gate stretches it over several windows, that is the
gate working. The implementer may trim the p1 arm to the delegation probes
if cost matters more than the middle point.

Measure, from saved calls, for every run:

- the verdict (rules of `5d4bc9c`, or later if step1 needs a rule change);
- the delegation class;
- fd-wr attempts (a send whose target is a ✔ topic);
- filesystem reach (a `find`/`ls`/`Glob` outside the workspace before the
  first `agentchat` call): p3 saw 10/24 before p1, 1/24 p1, 0/24 now on
  board 1; this is the one process difference already shown, so confirm
  it on board 2;
- the as9 miss ex1 saw once (a run that says it will answer if asked and
  sends nothing).

Hint, from p3: `--guides-rev` restores the guide tree only; the before-p1
arm runs over today's `agentchat --help`, multi-topic `read` and reply
sections. State it; building an old-pyagag venv is optional.

## step4 — report

- `report.md`: the door change and its tests; where the batch tooling is
  and how to run a batch; the three-arm table per probe with intervals;
  the process measures (filesystem reach, delegation classes, fd-wr, as9);
  cost and how the budget gate behaved.
- Say plainly what the series can now claim about p1–p3 and what it
  cannot, in the Developer's terms: better, no different, or worse, per
  measure, with the intervals behind each word.
- Open findings for the next phase, with runs. Update README_DEV
  § *Trying a guide change against the same board* for the doors and the
  batch command.

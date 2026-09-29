# agent_guide p3 ex2 — step 2: the batch tooling, versioned, with a budget gate

pyagag `99e88cb`. It is installed, with the lock committed, in agfront
`58605f6`→(this step), agautolab, agobserver and archsage. It is fixture
code only, so no listener was restarted.

## Where it is

| before (ignored) | now (`agag.fixture`) |
|---|---|
| p3 ex1 `one.sh` + `jobs.txt` + `xargs -P 4`; p3 `run_s1.sh`, `run_s3.sh` | `batch.py` — `python -m agag.fixture batch <plan.toml> --out <dir> [--jobs 2] [--probe p] [--dry-run] [--list]` |
| p3 `classify.py`, p3 ex1 `analyze.py` (classes, fd-wr) | `tally.py` — `python -m agag.fixture classify <dir>` (one line per run) |
| p3 `tab.py`, `analyze.py` (Wilson rates) | `python -m agag.fixture table <dir> [--json f]` |

Each agent's `<agent>.trial` stays the unit the batch calls. A job is
`<driver> <probe> --out <out>/<arm>/<probe>-<n> <arm's flags>`, run in the
driver's checkout.

## The plan file

```toml
[arms]                       # name → the driver flags of that arm
before = ["--guides-rev", "ba28e90^", "--no-shared"]
p1 = ["--guides-rev", "447bb03", "--no-shared"]
now = []

[[probes]]                   # probes, runs per arm, optionally a subset of arms
names = ["delegate-answer", "delegate-decision", "guard-status", "guard-finished"]
runs = 12

[drivers.agfront]            # default: <dir>/.venv/bin/python -m agfront.trial
dir = "../../agfront"        # relative to the plan file

[budget]
harness = "claude_code"      # the section of `agbudget --json` the runs spend
limits = { session = 60, weekly_all = 90 }
poll_seconds = 300
```

**Order.** The order is ex1's. The run number is outermost, then each probe
in plan order. Within a probe the arms rotate, so each arm goes first
equally often: `before, p1, now` for run 1, then `p1, now, before`, and so
on.

**Resume.** A job whose `outcome.json` exists is skipped. The same command
continues a stopped batch.

## The budget gate

Before each job starts, the gate runs `agbudget --json` and reads the
section of the harness the runs spend. A trial's Front desk role runs on
`claude_code` (Sonnet 5), in the pool shared with the live agents. No job
starts while any limited window is at or past its limit, or while the
reading fails: an unobservable window is not 0. Jobs already running
finish. The gate polls every `poll_seconds`. Each pause, resume and the
percents at every start go to `<out>/batch.log`. The readings go to
`<out>/budget.jsonl`.

- **Default `--jobs 2`**, and `session = 60` if the plan names no limits.
- **The weekly window.** The plan named the 5-hour window. At the time of
  writing, `agbudget` reads session 16 % and **weekly (all models) 81 %**,
  with the weekly reset on 2026-10-03 06:00 UTC. A 216-run batch spends
  on that weekly window too, and exhausting it would starve the live
  agents for days, not hours. So the gate takes any window kind, and
  step 3's plan also limits `weekly_all` at 90.

## Cut jobs

A job is cut when:

- its saved reply is the harness's limit message ("You've hit your …
  limit", "usage limit reached", in a reply under 400 characters); or
- it saved nothing and its log carries that message.

A cut job is not judged. It moves to `<out>/cut/<arm>/<probe>-<n>.<k>/`,
kept as evidence as ex1 kept `s3-limit/`, and goes back to the end of the
queue, at most twice per invocation. `classify` and `table` also skip any
limit reply they find, and list it as cut.

## What `classify` and `table` measure

- **Delegation class** (delegate-answer, delegate-decision). Since step 1
  the class comes from the run's door log (`outcome.json` `doors`):
  - `reached`: a send took a scripted answer;
  - `new request`: the only door taken was a new `workplan-` topic;
  - `wrong door`: sent, and nothing answered;
  - `proposed`: nothing sent, and the reply proposes and waits;
  - `none`.

  Older outcomes have no door log, so there it is two servings = reached,
  as p3 counted.
- **fd-wr attempts**: `agentchat send` targets parsed from the call's
  positionals (not its body) that name a ✔ topic of the board, by the bare
  or the ✔ name (ex1's parser).
- **Filesystem reach**, in three columns:
  - `search`: `find`, `tree`, `grep -r`, `ls -R`, or the Glob/Grep/LS
    tools, anywhere in the run. This is the measure p3 counted.
  - `search_first`: before the first `agentchat` call.
  - `search_outside`: with a path outside the run's own directory (`/…`,
    `..`, `~`). Saved calls are cut at about 200 characters, so a path cut
    short on its way into the run's directory is not counted as outside.
- **as9 miss**: a delegation run that sent nothing and whose reply leaves
  the asking to the other agent, e.g. "選ぶよう求めてきた場合は … 答えます"
  (ex1 fix delegate-decision 4).
- **Rates**: pass counts per probe and arm with Wilson 95 % intervals, turn
  range, cost, and the failing run numbers.

## Checked against the saved records

Every p3 and ex1 number reproduces:

| record | measure | earlier tooling / report | `table` |
|---|---|---|---|
| p3 s1 (72 runs) | pass per probe and arm | report1's table | identical |
| | delegation classes, delegate-answer | before 1/2/3, p1 2/2/2, now 4/1/1 (reached/wrong/none) | identical |
| | delegate-decision | before 5 + 1 proposed; p1 5 + 1 none; now 5 + 1 none | identical |
| | filesystem reach | before 10/24, p1 1/24, now 0/24 | `search` 10, 1, 0 |
| | fd-wr | now 2 (delegate-answer 1 and 3) | now 2/24 |
| | cost | $18.77 | $18.77 |
| ex1 s3 (112 runs) | rates, classes, as9, fd-wr 0/108, cost $31.49 | report3 | identical; `as9 = [4]` in fix delegate-decision |

What the new columns show on the saved runs:

- **p3's filesystem measure is mostly inside the run's own directory.**
  Of before-p1's 10:
  - 8 searched before the first `agentchat` call;
  - only 1 left the run's directory: delegate-answer 6's
    `find / -iname "strand-germination-days.md"`, after it had read the
    board.
  - p1's one (delegate-decision 6, `find ..`) also left it.

  So "reached for the filesystem" in p3 was mostly the working directory
  (`find . -maxdepth 3`). Step 3 reports all three columns.
- **ex1 on board 2**: `search` ctl 7/48 and fix 4/64. `search_first` ctl 0
  and fix 1: archsage's probe, not a Front run.

## Tests

`tests/test_fixture_batch.py` (pyagag 1213 tests pass):

- the order: rotation, and a group limited to one arm;
- a limit reply, and a crash whose log says the limit, are cuts; an
  ordinary reply is not;
- **end to end on the stub profile** (`harness = "fake"`). This is a real
  subprocess driver on `agag.fixture.run.Trial` with `run_role`, two arms ×
  two runs of `growbox-thing`, and a stub `agbudget` whose first reading is
  75 %:
  - the gate pauses ("pause: session 75 % ≥ 60 %"), then resumes;
  - the first reply is the limit message, so it is cut, kept under
    `cut/` and queued again;
  - all four jobs complete, and the `--no-shared` arm is recorded as such;
  - a second invocation skips all four;
- the classifier on synthetic outcomes: the classes, as9, fd-wr by target
  and not by body, and the three search columns.

agfront's delegation trial test sent into a new `workplan-` topic. It now
asks in `work-m20510 › workrun-task1-m20510` and checks the door log reads
`answer, answer`.

## Dry run of step 3's plan

`pj-agdev/.local/agp3ex2/plan.toml` (ignored) holds the step 3 plan. It
lists 216 jobs. A dry run (`--dry-run`, one run per arm of
`delegate-decision` and `growbox-thing`) went through agfront's driver:

| arm | prompt (growbox-thing) | "not the filesystem" | p3 fix ("not a poll") | board |
|---|---|---|---|---|
| before p1 | 27 337 chars | no | no | 2 |
| p1 | 19 522 | yes | no | 2 |
| now | 21 328 | yes | yes | 2 |

## Deus Ex Machina notes

None.

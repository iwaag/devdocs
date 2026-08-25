# scheduled_routine p4 — Plan

Braindump: `braindump.md`. p1–p3 built the scheduler: standing text in
`#front › routine-<name>`, `trigger.sh`, a flat-event `schedule.json` with
`fire`/`decide` events, one dispatcher, a read-only GUI. p4 asks whether
the whole thing is **useful**: a routine whose output the Developer would
actually want — arXiv paper digests accumulating in a Gitea repository —
run as if daily, but driven through several "days" in one sitting.

Decisions taken in the braindump discussion (2026-08-25), not to be
re-opened:

- **Two routines and a decide**, not one big routine: `papers` (select one
  trending, not-yet-covered paper; write a one-page summary; judge whether
  it can run locally) → a daily `decide` → `manual` (for a paper judged
  runnable, write a concise how-to from its README, marked *unverified*).
  Actually running a model is out of scope; `WORK_TIMEOUT_SECONDS` = 1200
  per task (`agautolab/src/agautolab/zulip_listener.py:169`) forbids it
  anyway.
- **The trend signal is the agent's choice.** arXiv exposes no download
  counts and fresh papers have no citations; HF Daily Papers, Papers with
  Code, Semantic Scholar velocity are all fair. The standing text asks for
  "a public trend signal, named in the summary". Which signal works is a
  finding.
- **No waiting for real days.** The schedule is written as a genuine daily
  cadence; time is advanced with the dispatcher's `--now` (the production
  code path, `devenv/routine/dispatch.py`), one logical day = a few ticks,
  repeated as many logical days as needed in one sitting.
- Judgement sources unchanged from p3: Front reads run topics, asks autolab
  for project reality, cagent for cluster reality; no new grants for Front.

Experimental, non-public environment. No backward compatibility required.
Only **MUST NOT** lines are prohibitions.

## Background the implementer should know

- Routine path (p2): Front reads `#front › routine-<name>`, opens a new
  `workplan-…` topic in `#pj-<slug>`, approves (approving means acting —
  start + done-check, ask nothing), reports home. autolab's supercoder
  commits `main`; `record_task_in_devlog` (`zulip_listener.py:852`) pushes
  only `devlog/`, and only if it is a repository.
- **p2 open finding, now blocking**: a main-only routine project's `main`
  is committed but **never pushed** (`p2/report.md` "Open findings"). This
  routine's value *is* the Gitea history; Step 0 fixes it.
- Schedule tooling (p3): `rtschedule` CLI (Front's only write ability,
  `Bash(rtschedule:*)`; usage in the workspace's `tools/schedule.md`);
  `dispatch.py --now <ISO>` runs one tick at a logical time and commits
  `fired_at` = that logical time (`dispatch.py:157`) — p3's report4 found
  this makes the GUI show fires "in the future". `decide` posts the ask as
  the Developer into `#front › front-schedule`. Requests expire by `until`
  (`dispatch.py:119`). The 5-minute production dispatcher
  (`com.agdev.routine-dispatch`) keeps running; it never touches future
  events and pulls the `--now` commits on its next tick (p3 did exactly
  this). If it conflicts, `bootout` it for the sitting and say so.
- p3 finding to keep in mind: every decide fired while its A was still
  running and Front waited inside the serving instead of moving the decide.
  Under `--now` the operator ticks the decide only after A is Done, so this
  trap is **bypassed, not fixed** — the report must say so.
- Project creation: `init_project.py <slug> --main-only` (p2). Channel
  `#pj-<slug>` must be made by hand with autolab subscribed (`p2/report3.md`).
- autolab roles have `WebFetch`, `WebSearch`, `Bash(curl:*)`,
  `Bash(wget:*)` (`agautolab/agents.toml`); arXiv full text is readable at
  `arxiv.org/html/<id>` or the abs page. No new grant expected — if one is
  needed, that is a finding, and by `localrule.md` the permission
  classifier stopping a run is reported, not worked around.
- `localrule.md`: do not economise on agent runs; count them, do not avoid
  them.
- **MUST NOT**: run or download model weights inside a routine task; add
  decision logic to the dispatcher; give Front repo/Plane/nctl access;
  fake summaries (a run that cannot read the paper reports that instead);
  commit absolute paths or credentials.

## Step 0 — two known gaps

1. **Push routine `main`.** Decide and implement one: the close-out pushes
   `main` when the supercoder committed and `main` is a clone (the devlog
   pattern), or the supercoder pushes after its commit. State the choice
   and why. Verify on `rtnotes`: its 2+ unpushed commits reach Gitea.
   Test in `agautolab/tests` alongside the devlog test.
2. **Real vs logical fire time.** `fired_at` becomes the real UTC time
   always; when `--now` is given, the logical tick time is stored as
   `logical_at`. GUI shows `logical_at` next to `fired_at` when present.
   Update the dispatcher fixture test. rtschedule CLI untouched.

Report `report0.md`: diffs, tests, rtnotes' Gitea log after the push, one
`--now` tick showing both fields.

## Step 1 — the `papers` project and routine A

- `init_project.py papers --main-only`; `#pj-papers` with autolab
  subscribed; verify with a plain `workplan-` question.
- Seed `main`: `README.md` (what the repo is, layout), `papers/INDEX.md`
  (one line per paper: date, arXiv id, title, signal used, runnable
  yes/no/unclear), empty `papers/` otherwise. Layout per paper:
  `papers/<arxiv-id>/summary.md`; later `manual.md` beside it.
- Standing text in `#front › routine-papers`, suggested:

  > In `#pj-papers`, ask autolab for a mission: pick **one** recent arXiv
  > paper (≤ 30 days) that is trending by some public signal (name the
  > signal and the evidence in the summary), **not already listed in
  > `papers/INDEX.md`**; read it (`arxiv.org/html/<id>` or the PDF); write
  > `papers/<id>/summary.md` — one page: problem, method, results, why it
  > trends, and a **Local run** section judging whether it can run locally
  > now (open weights/code? approx. VRAM? official README?) with a verdict
  > `runnable: yes|no|unclear`; add the INDEX line; commit. Approve a plan
  > that is roughly that; otherwise say what is off. Report here when Done:
  > the workplan topic, the id/title, the signal, and the runnable verdict.

- Fire it **once by hand through the schedule**: `rtschedule` adds a
  one-shot fire at now+2 min (as the Developer would ask Front to — do it
  via `#front › front-schedule` so Front's edit path is exercised), let the
  production dispatcher fire it. Watch the same things as p2 step 4: Front
  runs, whether it asked, what autolab wrote, the commit **and the push**.

Report `report1.md`: slug, channel, standing text, first run timeline,
`main` log on Gitea, the summary itself (quoted).

## Step 2 — routine B and the daily schedule

- Standing text `#front › routine-manual`, suggested:

  > In `#pj-papers`, ask autolab for a mission: for every paper in
  > `papers/INDEX.md` with `runnable: yes` and no `manual.md` yet, write
  > `papers/<id>/manual.md`: prerequisites, install, one minimal command to
  > run, expected output, where weights come from — from the official
  > README/code only, headed **"Unverified — written from documentation,
  > not executed"**; set the INDEX line to `manual: yes`; commit. If there
  > is nothing to do, say so. Approve; report here when Done.

- Via `#front › front-schedule`, ask Front for the real daily schedule,
  7 logical days starting tomorrow 09:00Z: `fire papers` 09:00Z, a
  `decide` 10:00Z each day whose ask is roughly "if today's papers run
  finished and its summary says `runnable: yes`, add `fire manual` at
  11:00Z today; otherwise add nothing and say why", `until` day 7 + a
  margin. Check the GUI shows 7 days × (fire, decide).

Report `report2.md`: standing text B, Front's schedule edit (runs, ids,
commit), GUI screenshot.

## Step 3 — one logical day, end to end

Day 1, each tick after the previous one's work is Done in Zulip:

```
dispatch.py --now <day1>T09:00Z    # fire papers
dispatch.py --now <day1>T10:00Z    # decide → Front judges, maybe adds manual
dispatch.py --now <day1>T11:00Z    # fire manual, if added
```

Record: Front runs per tick; the decide's reading (what it read, what it
concluded, whether it asked autolab); the added event; B's output; both
pushes on Gitea; `logical_at` vs `fired_at` in the commits. If day 1's
paper is not runnable, that is a valid day — note it and continue.

Report `report3.md`: the day-1 timeline, the summary and (if any) the
manual, quoted.

## Step 4 — six more logical days

Repeat Step 3 for days 2–7 without changing anything unless a run breaks
(then fix, note the fix as a needed feature, continue). Per day record:

| day | paper (id) | signal | runnable | manual written | Front runs | autolab tasks | Developer replies | failures |

Watch specifically:
- **duplicate avoidance**: all seven days draw from the same real-world
  pool; a repeat pick is a failure of "not already in INDEX";
- signal drift: does autolab keep one signal or wander;
- summary quality on a sample (the Developer reads two and says whether
  they are worth the repo);
- anything Front or autolab asked a human.

Report `report4.md`: the table, the Developer's two readings, every
intervention with its cause.

## Step 5 — phase report

`report.md`:
- **Is the routine system at a practical level?** Answer with the numbers:
  papers accumulated / 7 logical days, failures, human interventions,
  Front runs per day, cost per day (recorded, not optimised).
- **Needed features** (the braindump's second question), each with the
  evidence that demanded it; likely candidates to confirm or dismiss:
  decide outcome persisted in the schedule (p3), the bypassed
  decide-before-A-done trap, main push (now fixed), any grant autolab
  lacked, arXiv/HF access limits.
- What the acceleration could not measure: whether the Developer reads a
  real daily digest; overlap under real timing. Say so explicitly.
- Whether `papers` should now be left running on the real clock (the
  schedule already exists — extending it is one Front request).

## Out of scope

Running models; GPU/agpc work; evaluating paper choice automatically;
schedule schema changes beyond `logical_at`; fixing the p2/p3 read/ack
traps; changes to Front's grants.

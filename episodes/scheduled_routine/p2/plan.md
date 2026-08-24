# scheduled_routine p2 — Plan

Braindump: `braindump.md`. p1 proved the scheduler on a forge routine
(`p1/report.md`). p2 proves it on an **autolab** routine, in a project that
is git-managed but light: **one `main` repository**; `direction/` and
`devlog/` become optional, and `devlog/` stays as a plain local folder so
the record is still written and still read by the next plan.

Experimental, non-public environment. No backward compatibility required
(the four existing three-repo projects may keep working or not — implementer's
call, note which). Only **MUST NOT** lines are prohibitions.

## Background the implementer should know

- Project layout: `agautolab/.local/projects/<slug>/{main,direction,devlog}`,
  each a Gitea clone, made by `init_project.py <slug>` →
  `src/agautolab/project_init.py:init_project` (three-tuple loop at the end).
  `.local/` is git-ignored, so a non-repo `devlog/` survives exactly as long
  as the machine does — that is the intended lifetime here.
- Where the three folders are touched at runtime (`src/agautolab/zulip_listener.py`):
  - `workplan-` (superdirector) and `workrun-` (supercoder) guides already
    say `direction/` and `devlog/` are read **"if exists"**. Planning and
    executing in a main-only project need no guide change.
  - `record_task_in_devlog` (~line 852): after a task is closed it writes
    `devlog/<label>-<title>/task-<N>/{work.md,report.md}` (`mkdir -p`, works
    on an empty folder) **and then** `commit_all_and_push` on `devlog/`,
    which runs `git status` and will throw on a non-repo. Plane is already
    updated and the topic reply is composed at that point, so the failure
    shows up as a broken closing reply. This is the one real change.
  - `serve_bmining` (~line 1030) needs `direction/` as a clone. Not used by
    a routine project; leave it, or make it decline politely.
- `init_project` does **not** create the Zulip `#pj-<slug>` channel; autolab
  sweeps `workplan-` in whatever `pj-` channels it is subscribed to. Make the
  channel by hand (or `agentchat send` into it as the Developer, which
  subscribes the sender only — the autolab bot still needs subscribing).
  Check how `pj-runsmoke1` was made (`devdocs/episodes/agag_builder/p3`).
- The routine path is p1's, unchanged: `trigger.sh <name>` posts into
  `#front › front-routine-<name>`; Front reads the standing text in
  `#front › routine-<name>`, then posts a `workplan-…` topic in
  `#pj-<slug>`, says "start" when autolab's plan is sane, supervises the
  `workrun-` topics and reports home. autolab's intro in `#agents` is what
  Front reads to learn this; it says a task is not closed until the requester
  agrees, so Front will be called several times per run.
- Timing: a `workrun-` waits up to `WORK_TIMEOUT_SECONDS` = 3600. Pick a
  cadence that does not fire before a run normally finishes, or accept the
  overlap and watch (p1's one overlap data point was benign).
- Services needed and checked up on 2026-08-24: autolab listener + gateway,
  Front, Gitea (`:3000`), Plane. Restart autolab after code change:
  `launchctl kickstart -k gui/$(id -u)/com.agdev.agautolab-zulip`.
- Tests: `agautolab/tests/test_project_init.py` (`init_project` step order,
  `commit_all_and_push`) and `test_zulip_listener.py`. Run with
  `uv run pytest` in `agautolab/`.
- **MUST NOT**: push from `devlog/` when it is not a repository (no
  auto-`git init`, no fallback repo); commit absolute paths or credentials.

## Step 1 — main-only project initialisation

- `init_project.py <slug> --main-only` (or make main-only the default and
  `--full` the option; say why). Main-only: Plane project as before, Gitea
  repo + clone + `.gitignore` for `main` only, **`devlog/` created as an
  empty local folder**, `direction/` not created.
- Idempotent like the rest: re-running on an existing full project must not
  delete `direction/` or turn `devlog/` into something else.
- Test: the existing step-order test gains a main-only case.

Report `report1.md`: the flag, the resulting tree of a fresh project.

## Step 2 — devlog record without git

- `record_task_in_devlog`: write the files as now; push only if
  `devlog/.git` exists. The returned line says which happened
  (`devlog: recorded and pushed` / `devlog: recorded locally`).
- Anything else that assumes a clone under `devlog/` or `direction/` — grep
  `devlog_directory`, `direction_directory`, `commit_all_and_push` — gets the
  same treatment or a clear refusal. Do not guard what is not reached.
- Test: one unit test with a `devlog/` that is a plain directory; the git
  helper must not be called.

Report `report2.md`: diff summary, test output.

## Step 3 — the project and its channel

- Create the routine project, e.g. `rtnotes` (any name; keep it a real
  small program so `main` has something to run). Suggested seed for `main`:
  a tiny script plus a `NOTES.md`; the routine appends to it.
- `#pj-<slug>` channel with autolab subscribed; verify by asking a plain
  `workplan-` question there and seeing autolab answer.
- Register the project so Front can find it: autolab's intro already
  explains `pj-` channels; nothing else should be needed — if Front cannot
  route, that is a finding.

Report `report3.md`: slug, channel, the first autolab answer.

## Step 4 — the routine

Standing text in `#front › routine-<name>`; suggested:

> In `#pj-<slug>`, ask autolab for a mission: run the project's checks in
> `main/`, fix anything trivially broken, and append a dated one-paragraph
> status note to `NOTES.md` (what ran, what changed, what looks worth doing
> next — read the previous notes first and do not repeat them). Approve the
> plan if it is roughly that; otherwise say what is off. Report here when
> the mission is Done: link the workplan topic and quote the note.

Then p1's ritual: `trigger.sh <name>` by hand, watch
`agfront/.local/out/zulip-listener.log` and autolab's log; when one run
completes end to end, add the plist (`com.agdev.routine-<name>.plist.in`,
cadence your choice — every 2–3 h is safer than hourly here).

Hints:
- Front approving on the Developer's behalf is the new thing in p2. If it
  hesitates or asks the Developer, note what it asked; the answer probably
  belongs in the standing text.
- Check `devlog/` after the first task closes: `<label>-…/task-1/report.md`
  should be there and nothing should have been pushed.
- `main` commits: the supercoder commits after "developer's approval"; here
  the approver is Front. Watch whether it actually happens and who says yes.

Report `report4.md`: standing text, the first two runs (Front runs count,
wall clock, what autolab wrote, the devlog tree, `main` git log).

## Step 5 — phase report

`report.md`: Front runs per routine run and any stall/loop with timeline;
what the standing text had to grow; whether the local-only devlog was read
by the second plan (look for it in the superdirector's plan); what a third
routine or an ENT episode should be. Do not build it.

## Out of scope

Chat-controlled scheduling; overlap locks; automatic evaluation; routines
issued by autolab; touching Front, forge or pyagag.

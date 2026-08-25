# scheduled_routine p3 — Step 2 report

Executed 2026-08-25 UTC. All four observations below were live
`#front` / `front-schedule` conversations, not mocked role output.

## Grant and scope

Chose the plan's small-CLI alternative. Front's grant changed from:

```text
Read,Glob,Grep,Bash(agentchat:*)
```

to:

```text
Read,Glob,Grep,Bash(agentchat:*),Bash(rtschedule:*)
```

Front did **not** receive `Edit`, `Write`, general `Bash`, or `Bash(git:*)`.
`devenv/routine/rtschedule` is the only new command. It pulls the ignored
clone, refuses a dirty worktree, validates the complete JSON before and after
the change, stages only `schedule.json`, then commits and pushes. Its commands
are `show`, `now`, `add-request`, `add-fire`, `add-decide`, `move`, `remove`,
and `set-until`; every command has `--help`.

The listener now writes `tools/schedule.md` beside `tools/agents.md` for every
Front serving. It contains those commands and the repository/validation
contract. The listener's launcher adds `devenv/routine/` to `PATH`; there is
no committed absolute path and no credential in the run workspace. This is
the effective path scope: Claude Code's grant cannot express a filesystem
scope for `Edit`/`Write`, so the CLI was preferable to relying only on prose.

Relevant tests:

```text
pj-agdev routine tests:  4 passed in 0.01s
agfront full suite:     21 passed in 0.18s
```

## Guide change

The Front guide gained only the schedule contract and evidence ordering:

- inspect and edit the schedule for requests about when routines run, then
  report exact request/event ids and UTC times;
- discover routine names from `#front` / `routine-<name>`;
- expand recurrence into concrete events for at most 24 hours;
- put conditions verbatim in a later `decide`, after the prerequisite run is
  expected to finish;
- judge after evidence, using rename-following `agentchat read --since` or
  `wait`;
- ask autolab for project reality and cagent for cluster reality; do not open
  repositories, Plane, or nctl.

No occurrence required another guide edit in this step.

## Three requested edits

### 1. “Run rtnotes once in 10 minutes.”

One Front run (`run-0163`, 23.7 s, 12 turns). It asked nothing and made:

```text
8ae97d4 Add schedule request r1
f84e87e Add fire schedule event e1
```

Result: `r1`, guarded until 03:11:20Z, and `e1` at 03:01:20Z. Front replied
with both ids and the exact time.

### 2. “Run rtnotes every 2 h for today.”

One Front run (`run-0164`, 51.0 s, 18 turns). It asked nothing, committed
request `r2` at `58e5f94`, and expanded ten concrete fires `e2`–`e11`, from
04:53:11Z through 22:53:11Z. Those were ten separate `Add fire schedule
event` commits (`2d16522` through `600c7cd`). The horizon was the end of the
UTC day, under 24 hours.

### 3. “Cancel the last one.”

One Front run (`run-0165`, 24.8 s, 9 turns). It interpreted “last one” as the
last request, not only its last event: it removed `e2`–`e11` in ten commits
(`ff5d194` through `cbe1663`) and shortened `r2.until` to 02:54:14Z in
`356afb3`. That is one of the plan's allowed cancellation representations
(event removal plus an `until` hard guard). Front stated exactly what it had
cancelled. It did not ask for clarification.

Asked instead of acted, across the required three requests: **0**.

## Moving the standing cadence off launchd

Because request 3 intentionally cancelled its expanded events, a fourth,
explicit migration request told Front to replace the old two-hour rtnotes
plist with a 24-hour concrete horizon and a renewal decision. One Front run
(`run-0166`, 47.3 s, 22 turns) created `r3` (`d95493a`), eleven fires `e2`–
`e12` from 04:55:23Z through 00:55:23Z, and decide `e13` at 01:55:23Z
(`999ed9b`). Its question is: “Extend the rtnotes every-2h cadence (request
r3) by another 24 hours?”

I chose the standing daily `decide` because it keeps the file a finite flat
event list while preserving the old unattended cadence; it also makes every
horizon extension a visible Front judgement rather than dispatcher logic.

After those events were pushed, both old per-routine plist templates were
deleted and their installed jobs/files retired. Only
`com.agdev.routine-dispatch` remains loaded. The last plist-fired rtnotes run
was 2026-08-25 01:14Z (Zulip message 1968); the last plist-fired imgprompt run
was 2026-08-23 16:00Z (message 1489). Later rtnotes message 1998 at 02:46Z was
the Step 1 dispatcher proof, not a plist fire.

Commits: agfront `4a56bc3`; pj-agdev `796c7a7` (including the agfront pin).

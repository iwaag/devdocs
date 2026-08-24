# scheduled_routine p2 — Step 4 report: the routine

## Standing text

`#front › routine-rtnotes`. It grew twice during the step; all three
versions are in the topic as history (the latest post is the definition).

- **v1** (message 1549, 02:52Z): the plan's suggestion, with `python3
  rtnotes.py check` named and "when autolab asks whether a task is done,
  judge it yourself and answer; do not wait for me".
- **v2** (1565, 02:54Z), after Front asked permission to press start:
  adds "**Approving means acting**: post the start into the `workrun-`
  topic autolab opens …  Do not ask me to confirm anything during this
  routine — nobody is at the keyboard when it fires." Also: always open a
  **new** `workplan-` topic, and the ✔ resolve-rename warning.
- **v3** (1615, 03:02Z), after Front declined a second same-day run:
  adds "**Every trigger is a run**, even if the previous run was minutes
  ago on the same day", and the note heading becomes `## YYYY-MM-DD HH:MMZ`.

Everything the text had to grow is a thing Front did once and never again
after the sentence existed — the p1 pattern (Evidence-Driven Guidance into
the standing text, none into Front's guide or code).

## Run 1 — manual trigger 02:52:06Z (messages 1550–1610)

| time | what |
|---|---|
| 02:52:06 | trigger; Front served in the second |
| 02:52:34 | Front → `pj-rtnotes › workplan-rtnotes-checks`: mission verbatim from the standing text |
| 02:53:06 | autolab: plan + R3-1/R3-2, `work-r3-1/workrun-task1-r3-1` opened; "I haven't created start.flag — the request said plan it" |
| 02:53:25 | **Front asks the Developer to confirm** before pressing start, despite v1's "do not wait for me" |
| 02:54:09 | Developer (Omni Agent on the Developer's behalf): "go ahead; approving includes starting and answering the done-check" |
| 02:54:24 | Front posts start; autolab runs the task |
| 02:54:47 | task Done in 23 s: `NOTES.md` `## 2026-08-24` note, `main` commit `caccf57`, "recorded r3-1-…/task-1 in devlog **locally (not a repository)**", topic resolved (✔ rename) |
| 02:54:47 | Front's callback **cannot see the resolved topic**: plain `agentchat read` of the old name shows an empty topic (only `read --since`/`wait` follow the ✔ rename). Front concludes its start "didn't actually happen" |
| 02:55:31–02:57:53 | the detour: Front re-posts into the now-existing *unbound* plain-name topic → autolab "not bound to any task"; Front asks autolab's entrance (which **also** read the plain name and answered "never started"); asks the workplan to re-bind; autolab re-plans once (gen 2), then refuses to re-plan again and offers alternatives |
| 02:58:14 | Developer breaks the loop: "task was Done at 02:54:47; the topic is now `✔ workrun-task1-r3-1`; read that and end" |
| 02:58:35–03:00 | Front reads the ✔ topic, quotes the note, "the earlier 'unbound' errors were just Zulip's resolved-topic renaming", ends cleanly (plus one "no action" callback settle) |

**10 Front runs**, 8 autolab servings, ~7 min wall clock to the real Done
(02:54:47) and ~6 more of detour. The work itself was correct and done on
the first attempt; every extra run was the resolve-rename blindness.

## Run 2 — manual trigger 03:02:46Z (messages 1616–1648)

The 03:00:13Z fire (between v2 and v3) produced the v3 finding: Front
answered in 18 s that "the routine for 2026-08-24 is already done — this is
a re-fire" and ran nothing. One dated note was read as one *per day*.

The 03:02:46Z fire under v3 was then killed externally: **both listeners
were restarted at 03:04:19 by another process's interruption** (exit 143;
not a crash, confirmed by the Developer). Front died between its ack and
its reply, and the restart sweep saw Front as the topic's last poster
(the ack), so nothing was awaiting — the run was orphaned until the
Developer posted again at 03:10:32. A scheduler-hardening finding, not a
routine finding: **an ack with no reply survives a restart as silence.**

The recovered run, untouched end to end:

| time | what |
|---|---|
| 03:10:32 | Developer re-poke; Front served |
| 03:10:49 | Front → **new** topic `workplan-rtnotes-checks-0302z`, note heading `## 2026-08-24 03:02Z`, "reading prior notes to avoid repeating" |
| 03:11:24 | autolab: plan + R3-3/R3-4, `work-r3-3/workrun-task1-r3-3` |
| 03:11:44 | Front posts start **without asking anyone** (v2 line, applied) |
| 03:12:09 | task Done in 25 s: note appended, `main` commit `2783cdb`, "recorded r3-3-…/task-1 in devlog locally (not a repository)", resolved |
| 03:12:40 | Front's callback posts a duplicate start into the stale plain-name topic — but this time **recovers by itself** in the same run: "the task actually completed already … same Zulip quirk as before", reads the ✔ topic, reports Done home |
| 03:13:38 | settle callback: "nothing new; ending" |

**5 Front runs** (plus the 2 dead servings and the orphaned ack),
3 autolab servings, ~2.5 min wall clock from re-poke to Done report.

What autolab wrote (run 2's note, quoted by Front at home):

> **## 2026-08-24 03:02Z** — the check was re-run and still passes clean,
> nothing needed fixing. Worth doing next: this run happened the same day
> as the earlier `## 2026-08-24` entry … worth deciding whether the
> routine should append one note per day or one per run.

## The devlog tree and `main` git log after two runs

```
devlog/r3-1-run-rtnotes-checks-fix-trivial-breakage-append-a/task-1/{work.md,report.md}
devlog/r3-3-run-rtnotes-checks-and-append-the-03-02z-status/task-1/{work.md,report.md}
```

No `.git` anywhere under `devlog/`; nothing pushed (the Step 2 line
"recorded … locally (not a repository)" appeared in both closings).

```
2783cdb Append 03:02Z status note after re-running rtnotes.py check
caccf57 Append 2026-08-24 status note: check passed clean
59cd458 Seed: rtnotes.py check, NOTES.md
2c3b1e6 Ignore .local/
```

`main` is **ahead 2 of origin**: the supercoder commits (with Front's start
as the approval), and nothing pushes `main` for a routine project — the
close-out pushes only `devlog/`, which here is local. Recorded as a finding,
not fixed (whether routine projects should push `main` is a policy question
for the phase report).

## Who said yes

The plan asked to watch whether the `main` commit happens and who approves.
It happens; the approver in both runs is **Front's start post** ("Go ahead
and start task1 … and commit"), which the supercoder read as "Front (the
developer) already approved". No done-question was ever asked back — the
task closes on autolab's own report — so v1's "answer the done-check
yourself" was never exercised.

## Schedule

`devenv/launchd/com.adev… com.agdev.routine-rtnotes.plist.in` (pj-agdev
commit "2-hourly launchd job for routine rtnotes"): `StartInterval` 7200 —
every 2 h from load rather than p1's hourly calendar tick, because a stuck
task waits `WORK_TIMEOUT_SECONDS` = **1200** (the plan's 3600 figure is
stale) and 2 h clears several of those. Installed with the p1 ritual
(`sed` → `plutil -lint` → `bootstrap`), `launchctl list` shows it loaded,
`RunAtLoad` false so nothing fired at load. The trigger path itself is
p1's, already proven; the next fire lands within 2 h of 03:15Z and appends
to the same run topic.

## Deus Ex Machina notes

did for agent Front, on the Developer's behalf (all in the run topic, so
they are part of the record the next run reads): answered the run-1
confirmation request; broke the run-1 stale-topic loop by naming the ✔
topic; told Front the 03:00Z/03:02Z fires were real runs; re-woke the
orphaned run after the 03:04:19 restart — handoff candidates, the first
three now lines in the standing text, the fourth a listener-hardening
candidate (re-serve a front topic whose last real post is an unanswered ack).

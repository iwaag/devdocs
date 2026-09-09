# refine_routine p1 — step1 report: old fire mechanism retired, guides placed

Date: 2026-09-09

## What was done

**Old fire mechanism stopped and removed.**

- State was read first: `nctl status` (Nautobot, worker and dumps all ok;
  the routine dispatcher and its GUI were never registered as desired
  services, so nothing in `pj-clusterintent` changes) and `launchctl list`
  (`com.agdev.routine-dispatch` on a 5-minute interval, `com.agdev.routine-gui`
  serving the schedule clone). The old per-routine jobs `routine-rtnotes` /
  `routine-imgprompt` were already absent, as the environment memo said.
- Both launchd jobs were booted out and their installed plists moved under
  the ignored `.local/launchd/retired/`. The GUI port no longer answers.
- Deleted from `pj-agdev`: `devenv/routine/` (`dispatch.py`, `trigger.sh`,
  `rtschedule`, its tests) and the two plist templates. `devenv/launchd/README.md`
  says why they are gone.
- The schedule's last event (`e32`, 2026-09-08T16:47Z) had already fired and
  no event was pending, so nothing could be replayed. The dispatcher's last
  log line is it failing to find the deleted script, before it was booted out;
  no fire happened.
- agfront: the `rtschedule` PATH hook in `agent/zulip_listen.sh`, the
  `tools/schedule.md` document the listener wrote into every run, and the
  `Bash(rtschedule:*)` grant of both roles are removed. Tests updated
  (`61 passed`). The listener was restarted so a run no longer gets the
  schedule document. Both guides (`front`, `character_talk`) now describe
  where a routine's guide lives and state that timed or recurring runs are
  not something Front can arrange.

**Guides placed in Zulip.**

- Channel folder `routine`, one public channel per routine, named
  `routine-<name>`: `#routine-publish`, `#routine-study-realworld`,
  `#routine-ghtrends`, `#routine-papers`. Developer and Front are
  subscribed. Each channel's description states the contract.
- Fixed topic `guide` in each, holding **guide v1**. Reading rule, stated in
  every guide's first paragraph and in the channel description:
  *the newest post in `guide` is the whole guide*; a new version is a new
  full post, never an edit, and nobody runs or reports there. A run is a
  `routinerun-<id>` topic in the same channel (step2), so a `guide` post
  starts nothing — verified: the Front listener served nothing after the
  four posts.
- Migration read every post of the old topics, not just the last one. The
  content taken is the Developer's newest request in each (`publish` v8,
  `study-realworld` v1, `ghtrends` v1, `papers` v4); Front's run reports in
  `routine-ghtrends` were excluded. Rewording is limited to the sentences
  that described the scheduled fire ("the run's own topic", "a scheduled
  fire does not", "next fire") and the report destination ("Report here"),
  which becomes "the report of a run names …". Each guide says which message
  it was moved from.
- Not migrated: `rtnotes`, `imgprompt`, `mediagen`, `localtest`, `manual` —
  the Developer had already ✔-resolved those five request topics on
  2026-09-06 (the `manual` routine is folded into `papers`). They can be
  given a channel later by posting a `guide` the same way.
- The four old `#front › routine-<name>` topics carry one pointer post
  naming the new location and are ✔-resolved.

## Verification

- `launchctl list` shows no `com.agdev.routine-*` job; the GUI port refuses.
- `agentchat channels --prefix routine-` (as the Front bot) lists the four
  channels, and `agentchat read routine-<name> guide` returns the guide;
  each guide was read back at its full length (the longest is under the
  realm's silent 10000-character truncation).
- No old event was replayed (see above).

## Left for later steps

- The agentroom relay still reads the ignored `schedule.json` for its
  routines view (`AGENTROOM_SCHEDULE_JSON`); the clone is kept as history
  and step4 replaces that view with the new channels.
- The old `front-routine-*` run topics in `#front` are untouched history.
- The `routinerun-` execution contract and Front's start path are step2.

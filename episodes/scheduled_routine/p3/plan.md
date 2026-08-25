# scheduled_routine p3 — Plan

Braindump: `braindump.md`. p1 built the trigger (`trigger.sh` +
one plist per routine), p2 proved a git-light autolab routine. p3 makes the
**schedule itself visible and editable through Front**: the Developer says
in natural language what should run when, Front turns that into a plain
data file, one dispatcher fires it, a read-only static page shows it.

Design decisions taken in the braindump discussion (2026-08-25), not to be
re-opened here:

- **Schedule = flat list of concrete events**, not cron expressions. A
  recurring wish ("every 2 h") is expanded over a horizon; "the next 2 h"
  is a handful of timestamps. Each event points at the natural-language
  request it came from.
- **Two event kinds**: `fire` (post the run message, exactly what
  `trigger.sh` does today) and `decide` (hand a natural-language question
  to Front at that time; Front answers by editing the schedule). Conditions
  ("if A succeeded, repeat B") are never written into the data — they are
  the text of a `decide`.
- **No decide-specific tooling.** Front judges with what it has: the run
  topics, and asking the responsible agent. **Project reality is
  autolab's to answer, cluster reality is cagent's** (via nctl). Front does
  not get read access to repos, Plane or nctl; it gets one new write
  ability — the schedule file.
- Format JSON, read-only GUI as static HTML, backend one launchd job.

Experimental, non-public environment. No backward compatibility required.
Only **MUST NOT** lines are prohibitions.

## Background the implementer should know

- Today's path (p1, unchanged): `pj-agdev/devenv/routine/trigger.sh <name>`
  posts as the Developer into `#front › front-routine-<name>`; the standing
  text lives in `#front › routine-<name>`. Plists
  `devenv/launchd/com.agdev.routine-{imgprompt,rtnotes}.plist.in`;
  `routine-rtnotes` is loaded at `StartInterval` 7200 (check `launchctl
  list | grep routine`).
- Front: `agfront/agents.toml` role `front`, grant
  `Read,Glob,Grep,Bash(agentchat:*)`; guide
  `agfront/agent/guides/front/guide.md`; per-run workspace
  `agfront/.local/topics/<channel>/<topic>/<N>/front/` with `chatlog.md` and
  `tools/agents.md` (harvested from `#agents`). The guide is read from disk
  per run. Front speaks as the Front bot, not the Developer.
- Front never sees the Developer's file system; `Read` is for its own
  workspace. Anything it must edit has to be *inside* or reachable from
  that workspace, or reached through a command in its grant.
- Known traps from p2 (`p2/report.md`), still open: plain `agentchat read`
  does not follow the `✔ ` rename of resolved topics (use `--since`/`wait`);
  an ack left unanswered across a listener restart is served by nobody.
  p3 does not fix them; it **counts** them.
- Overlap: a `workrun-` waits `WORK_TIMEOUT_SECONDS` per task. p1 saw one
  benign overlap. Dense schedules will produce more; no lock is built —
  Front handles it and the report records what happened.
- Services: autolab listener + gateway, Front, Gitea, Plane; cagent/nctl for
  cluster questions (`pj-clusterintent/nctl/README.md`).
- **MUST NOT**: give Front write access to anything but the schedule
  repository; give Front nctl, Plane or project-repo access; put decision
  logic into the dispatcher; commit absolute paths or credentials.

## The schedule file

One repository `rtschedule` (Gitea `autodev/rtschedule`, cloned under
`pj-agdev/.local/rtschedule/` — location is the implementer's, but it is
`.local`). One file `schedule.json`:

```json
{
  "requests": [
    {"id": "r1", "said_at": "2026-08-25T09:00Z", "by": "developer",
     "until": "2026-08-25T12:00Z",
     "text": "Run rtnotes once now; if it succeeded apart from minor errors, repeat imgprompt every 30 min for the rest of two hours."}
  ],
  "events": [
    {"id": "e1", "at": "2026-08-25T09:05Z", "kind": "fire",   "routine": "rtnotes",   "from": "r1", "fired_at": null},
    {"id": "e2", "at": "2026-08-25T09:45Z", "kind": "decide", "from": "r1", "fired_at": null,
     "ask": "Did the rtnotes run started at 09:05Z succeed apart from minor errors? If yes, add imgprompt fires every 30 min until 11:00Z. If not, add nothing and say why."}
  ]
}
```

Rules, kept small:

- `until` on a request is the only hard guard: the dispatcher never fires
  an event whose request has expired. Everything else is Front's judgement.
- The dispatcher writes only `fired_at`; Front only adds/edits events and
  requests. Both commit and push; the two never touch the same lines.
- Past events stay in the file for 7 days (dispatcher prunes older ones),
  so the GUI shows recent history. Horizon for expanding recurring wishes:
  24 h, re-expanded by Front when asked or when a `decide` fires; the
  implementer may instead add a standing daily `decide` "extend the
  recurring routines by another day" — say which and why.
- The two existing plists are retired once their routines are expressed as
  events (report which run was the last plist-fired one).

## Step 1 — dispatcher

`devenv/routine/dispatch.sh` (or `.py`; deterministic, no LLM): pull the
repo, for every event with `at <= now`, `fired_at == null`, request not
expired: `fire` → `trigger.sh <routine>`; `decide` → post as the Developer
into `#front › front-schedule` one message: the event id, the `ask`
verbatim, and a pointer to the schedule repo. Set `fired_at`, commit, push.
Log to `agfront/.local/out/dispatch.log`. One plist
`com.agdev.routine-dispatch.plist.in`, `StartInterval` 300.

Idempotent: a crash between fire and push must not double-fire on the next
tick — write `fired_at` before posting, push after; if the push fails, log
and leave it (the next tick re-pulls; conflicts are the implementer's to
resolve, most simply by `git pull --rebase`).

Tests: a fixture `schedule.json` with due / not-due / expired / already-
fired events, asserting which get fired (mock `trigger.sh`).

Report `report1.md`: script, the plist, the test output, one manual tick
against a file with one due `fire` (show the Zulip post and the resulting
`fired_at` commit).

## Step 2 — Front edits the schedule

- Grant: add to role `front` a way to edit the clone — the least is
  `Edit,Write` scoped by the guide to `schedule/` plus `Bash(git:*)`; if
  `claude_code` cannot scope paths in the grant, say so and rely on the
  guide plus the repo's own commit hook that rejects anything but
  `schedule.json`. Alternative if simpler: a tiny `rtschedule` CLI
  (`add-fire`, `add-decide`, `move`, `show`) with `--help`, granted as
  `Bash(rtschedule:*)` — **choose one, and give the usage doc either way**
  (`tools/schedule.md` in the workspace, generated like `tools/agents.md`, or
  the CLI's `--help`).
- Workspace: the handler exposes the clone to the run (symlink
  `schedule/` in the generation workspace, or the CLI knows the path).
- Guide additions (short, evidence first, edit after occurrence): the
  schedule exists and how to see it; a request about *when* things run is
  answered by editing the schedule, then replying with what was added
  (ids, times); routine names are the `routine-<name>` topics; a `decide`
  is placed after the run it depends on is expected to finish; never write
  conditions into the file — write a `decide`.
- Judgement sources, stated in the guide as the role division: run topics
  are read directly (`agentchat read --since` / `wait`); **the state of a
  project is asked of autolab in its channel, the state of the cluster is
  asked of cagent**; Front does not open repositories, Plane or nctl.

Test by hand in `#front › front-schedule`: "run rtnotes once in 10 minutes"
→ one `fire` event committed; "every 2 h for today" → expanded events;
"cancel the last one" → event removed or `until` shortened. Note every
time Front asked instead of acting, and what the guide had to say.

Report `report2.md`: the grant and how it was scoped, the guide diff, the
three requests with resulting commits.

## Step 3 — the decide event, end to end

The braindump's example, as-is: "run A once; if it succeeded apart from
minor errors, repeat B for the rest of two hours" with A = `rtnotes`,
B = `imgprompt` (both exist). Watch:

- how far after A's start Front placed the `decide`, and whether it moved it
  when A was still running (`"decide" moved` is the expected self-correction);
- what Front read to judge "minor errors" — the run topic only, or did it
  ask autolab; whether the answer was defensible;
- the events it added, and that the dispatcher fired them at the right
  times; overlaps with an imgprompt still generating, and what happened.

Then a failing variant: break `rtnotes`'s check script so A fails, same
request; expect "added nothing" with the reason in `front-schedule`.

Report `report3.md`: both timelines (dispatcher log + Front runs + commits),
Front runs per request, traps hit (rename-blind reads, orphaned acks — count
them).

## Step 4 — read-only GUI

Static `index.html` in the `rtschedule` repo (or `devenv/routine/gui/`),
reads `schedule.json` from the same origin — serve the clone with
`python -m http.server` from a plist, or from Gitea's raw URL **via the
GitHub mirror, never the agstudio Gitea mirror** as the source of truth if
it goes remote; local file first. Shows: a timeline for the next 24 h and
the past 7 days, each event with kind, routine, `from` request text
on hover/expand, fired or not; a request list with `until`. Links each
`fire` to `#front › front-routine-<name>` and each `decide` to
`front-schedule`. No writes, no auth. Theme-aware, no external assets.

Report `report4.md`: URL, screenshot, what the page could not show that the
Developer wanted while watching Step 3.

## Step 5 — phase report

`report.md`: Front runs per request and per `decide`; which judgement
sources Front used and whether the autolab/cagent division held without
being pushed; guide lines that had to grow and why; trap counts vs p2;
whether the flat-events model was enough or where Front wanted a rule it
could not express; the ENT candidates (rename-following reads, orphaned
acks, overlap policy) with p3 evidence attached. Do not build them.

## Out of scope

Overlap locks; automatic success evaluation; GUI writes; routines issued
by autolab; nctl/Plane/repo access for Front; touching forge, pyagag,
autolab code; fixing the p2 read/ack traps.

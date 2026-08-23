# scheduled_routine p1 — Plan

Braindump: `../braindump.md`. Decision taken there and approved:

- The issuer of a routine is **Front**, acting for the Developer. No agent
  gets a second, "no questions asked" entrance.
- The scheduler is a dumb clock: at the set time it posts **one message into a
  `front-routine-<name>` topic**. Everything after that is the existing
  Front → forge/autolab → Front path, unchanged.
- The request text lives **in the chat**, where the Developer can edit it.
- First routine: image prompt exploration via forge. **No automatic scoring**
  in p1 — the Developer judges in the topic; the routine reads those
  judgements next time.

Experimental, non-public environment. Backward compatibility is not
required. Prohibitions are the few marked **MUST NOT**; everything else is
advice and the implementer's call.

## Background the implementer should know

- Front is served when a `front-` topic's **last real poster is not Front**
  (`pyagag/src/agag/topics.py`, `last_real_sender`; `[selfnote]` lines never
  count). So any *other* identity posting into `front-routine-foo` starts a
  Front run — no code change in Front, pyagag, forge or autolab is expected
  for this phase.
- `front-routine-<name>` is **not a new kind of topic**: it is an ordinary
  `front-` topic that happens to be named `routine-…` for humans to spot.
  Front's prefix filter sees only `front-` and serves it through the same
  `handle_topic` as any Developer request. No routine-specific route exists
  in Front. (`routine-<name>` without the prefix, Step 1, is the opposite:
  named so that Front never serves it.)
- Front listener: `pj-agdev/agfront/src/agfront/listener.py`; launchd job
  `com.agdev.agfront-zulip` from `pj-agdev/devenv/launchd/*.plist.in`
  (`__PROJECTS_ROOT__` substitution, install to `~/Library/LaunchAgents/`,
  `launchctl bootstrap gui/$(id -u) …`). Logs under `agfront/.local/out/`.
- Front's supervision is several short runs: it posts to forge, ends, and is
  called again when forge's reply names it (`README_DEV.md`, p7/p8). A
  routine that takes minutes of generation is therefore fine — nothing waits.
- forge contract (`pj-agdev/agforge/params/intro.md`, also posted in
  `#agents`): plan in `assetplan-<name>` in forge's channel → forge registers
  a Work and opens `assetrun-<name>` → a post there starts generation → result
  with URL + `[S3KEY]` in both topics; requester is named once, in the plan
  topic. URL expires in ~1h; `POST :8092/api/resign {"key"}` refreshes it.
- Posting tools: `agentchat send <channel> <topic> <text…>` (pyagag console
  script; subscribes the sender, writes the `[rootchat]` selfnote). Plain
  Zulip REST works too. Credentials under `pj-agdev/.local/zulip/`
  (`developer.env` = the human, `omni-agent.env`, per-bot envs; `agag init
  … --provision` makes a new bot if wanted).
- Policy: `localrule.md` says do not economise on agent calls. Run the
  routine as often as is useful for observation, not as rarely as is cheap.
- `MUST NOT`: run the scheduler on any node with skip-permissions; keep it on
  agstudio under launchd like the other services. Do not commit absolute
  local paths or credentials (use `.plist.in` + `.local/`).

## Step 1 — Routine definition lives in Zulip

Goal: a place the Developer can read and edit what a routine does.

- Choose a home. Recommendation: `#front`, topic `routine-<name>`
  (no `front-` prefix, so Front never serves it as a request). The topic
  holds one post: the standing request text. Editing the post or appending a
  new one changes the routine; the trigger says "use the latest".
- Alternatives the implementer may prefer: a Plane Work whose description
  is the text, or a file in the repo. Chat is recommended because the whole
  point is that the routine is visible where everything else is.
- Write the first definition (Step 4 has the content). Keep it a request in
  the Developer's voice — Front is the Developer's proxy, the text should
  read as something the Developer would type.

Report `report1.md`: where the definition lives, how it is edited.

## Step 2 — Trigger

Goal: at the scheduled time, one post appears in `front-routine-<name>` and
Front starts.

- Smallest thing that works: a shell script `trigger.sh <name>` that posts
  (via `agentchat send` or curl) something like:
  `Routine <name>, run of <date>. The standing request is the latest post in
  #front › routine-<name>; this topic holds the earlier runs. Do it.`
- Identity: post as someone who is not Front. Two sane choices:
  - the Developer's own account (`developer.env`) — keeps "Front replies to
    the Developer" literally true, zero provisioning;
  - a dedicated `scheduler` bot — easier to tell apart in the log.
  Either is fine. Note that the poster becomes a participant; if the poster
  is a bot with its own listener, make sure it does not sweep `front-`.
- One topic per routine, runs appended. Resolving the topic after each run
  is optional; a resolved (`✔ `) topic is still found by Front's lookups
  (pyagag follows the rename), but a fresh topic per run
  (`front-routine-<name>-<date>`) is the simpler choice if history gets long.
  Pick one, note why.
- Verify by hand first: run `trigger.sh` once, watch Front pick it up in
  `agfront/.local/out/zulip-listener.log`. Only then schedule.

Report `report2.md`: script, identity chosen, the log lines of the first
manual run.

## Step 3 — Schedule

Goal: the trigger fires on its own.

- `pj-agdev/devenv/launchd/com.agdev.routine-<name>.plist.in` with
  `StartCalendarInterval`, same substitution convention as the others.
  Hourly is a reasonable first cadence; change freely.
- Log stdout/stderr to `agfront/.local/out/routine-<name>.log` — that log is
  the only record of "the trigger did not post"; Zulip shows nothing for a
  missed fire.
- Overlap: if the previous run is still in supervision when the next fires,
  **let it post anyway** and see what Front does. Do not add a lock in p1;
  if it misbehaves, that is a report finding and the fix belongs to a later
  step.
- Stopping a routine in p1 = `launchctl bootout`. Chat-controlled on/off is
  out of scope.

Report `report3.md`: plist, first scheduled fire seen in both logs.

## Step 4 — First routine: image prompt exploration

Goal: the standing request Front will carry out via forge.

Suggested text for `routine-imgprompt` (edit freely):

> Pick a visual theme for today (vary it from previous runs — read the
> earlier runs in this topic). Ask forge for 4 images of that theme, each
> with a clearly different prompt. Post here, for each image: the prompt,
> the URL and the `[S3KEY]`. Then read my comments on earlier runs in this
> topic and say, in two lines, what you would try next and why. Do not
> score the images yourself — I will.

Hints:

- forge wants the full spec in the `assetplan-` post (size, format, style);
  Front knows this from the `#agents` intro. If Front asks the Developer
  questions in the routine topic instead, that is a finding, not a failure:
  note it, answer it, and consider putting the answer in the standing text.
- 4 images = 4 runs or one `assetrun-` with 4 variants — leave it to Front
  and forge; record which shape emerged.
- URLs in the report expire. The `[S3KEY]` is the durable reference; the
  `resign` endpoint exists for a dead link.
- The Developer's judgement is plain text in the run topic. No format.
  Later runs read it as chat history, nothing parses it.

Report `report4.md`: the standing text as finally posted, and the first two
or three runs: what Front did, what it asked, what the Developer answered.

## Step 5 — Observe and write down

After ~a day of runs, write `report.md` (phase report):

- Did Front need anything it was not given? (candidate Evidence-Driven
  Guidance for the standing text — not for Front's code)
- Count of Front runs per routine run; anything that looped or stalled, with
  the timeline from the logs.
- Whether the second routine should be autolab-side (e.g. a periodic "where
  do my plans stand" survey in each `pj-` channel, which exercises the
  `workplan-` path p1 never touches). Recommend yes/no with a reason.
- What would make this Easier Next Time — but do not build it in p1.

## Out of scope for p1

Automatic image evaluation; chat-controlled scheduling; overlap locks;
routines issued by autolab; any change to Front, forge, pyagag.

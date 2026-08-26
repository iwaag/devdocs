# Step 3 — second summary, through the schedule

`openai/codex` summarised, committed as `3497151` and pushed, reached through
the whole routine chain: standing text → Front's schedule edit → the
production dispatcher's own tick → Front → autolab → task → close-out. It
took **three Developer messages** to get there, not the one the plan expected,
and each of the three is a finding.

## Standing text — `#front › routine-ghtrends` v1 (message 2454, 10:36Z)

> Standing request for the `ghtrends` routine, v1.
>
> In `#pj-ghtrends`, ask autolab for a mission: pick **one** repository trending on GitHub today that is **not already covered in `main/`** (`main/index.md` is the list of what is covered — read it first), and write its summary the same way as the ones already there.
>
> - Name the trending list and the date observed in the summary.
> - The figures — stars, forks, rank, anything numeric — come from `https://api.github.com/repos/<owner>/<repo>`, and the summary names that endpoint. Do not quote a number off the trending page.
> - Cover: what the repository does, why it trends, its license.
> - Follow the layout `README_PROJECT.md` records (`main/repos/<owner>-<repo>.md` plus a row in `main/index.md`), in the same commit.
> - `publish/` is not touched.
>
> Approve a plan that is roughly that; if it is off, say what is off. Approving means acting — start it and check it through to Done, ask nothing.
>
> Report here when it is Done: the workplan topic, the repository name, and the trend figures it quoted.

## The fire, through the schedule

Asked as the Developer in `#front › front-schedule` (message 2455, 10:36:35Z):

> 新しい routine `ghtrends` を一度だけ試したい。スケジュールに、`ghtrends` の one-shot fire を **2026-08-26T10:45:00Z** に1件だけ追加して。新しい request を作って、その until は 2026-08-26T11:30:00Z にして。それ以外は何も変えないで。

**One Front run, 22 s** (`run-0224`, 10 turns, $0.167). It created request
`r9` and one fire `e39`, and said so with both ids; nothing else changed.

```json
{"id": "r9", "said_at": "2026-08-26T10:36:50Z", "by": "developer",
 "until": "2026-08-26T11:30:00Z",
 "text": "新しい routine ghtrends を一度だけ試したい。ghtrends の one-shot fire を追加。"}
{"id": "e39", "at": "2026-08-26T10:45:00Z", "kind": "fire", "from": "r9",
 "fired_at": null, "routine": "ghtrends"}
```

Commits in `autodev/rtschedule`: `1b6ef05` (request r9), `2f30e2f` (event
e39). The **production** dispatcher fired it on its own five-minute tick — no
`--now` anywhere in this step:

```
2026-08-26T10:49:15Z marked e39 before action
2026-08-26T10:49:15Z dispatched and pushed e39
```

The 4′15″ between the scheduled 10:45:00Z and the 10:49:15Z tick is the
dispatcher's cadence, exactly as designed.

## Run timeline (UTC)

| time | what |
|---|---|
| 10:49:15 | dispatcher fires `e39`; `trigger.sh ghtrends` posts as the Developer into `#front › front-routine-ghtrends` |
| 10:49:16 | Front run 1 (`run-0225`, 17 turns, 56.8 s) |
| 10:50:01 | Front asks autolab **in its own channel** (`autolab-agstudio1 › status-ghtrends-g1`) about the project's state instead of opening a mission |
| 10:50:13 | Front reports home: it read the resolved G-1 topics as *unstarted*, concluded `main/` had never been committed, and is waiting |
| 10:51:14 | Front run 2: relays autolab's answer — G-1 did commit `a3465c4`; **"nothing further needs doing for this run"**, plus a question |
| **10:51:33** | **Developer intervention 1** — no: one fire means one *new* repository |
| 10:52:04 | Front opens `#pj-ghtrends › workplan-trend2` with the mission, pre-approved |
| 10:52:22 | autolab plans **G-4** — and writes **no task files**, so no `workrun-` topic opens and nothing can start |
| 10:53:14 | Front asks autolab what topic to post into; 10:54:25 autolab diagnoses its own stall and prescribes a re-plan with a `task1.md` |
| 10:55:09 | Front re-posts that request into `workplan-trend2` **unprompted** |
| 10:55:39 | autolab writes `task1.md` — but not `plan.md`, so still no topic |
| 10:56:12 | Front reports it is stuck, and deliberately does not nudge again |
| **10:57:23** | **Developer intervention 2** — the real cause: the listener reads task files only alongside a `plan.md` in the same round |
| 10:57:37 | autolab rewrites both; **G-5** created, `work-g-4 › workrun-task1-g-4` opened |
| 10:58:37 | Front posts into it; supercoder `run-0085` (15 turns, 57.5 s, $0.236) |
| 10:59:2x | `openai/codex` written, `main/index.md` row added, committed `3497151`, pushed; G-5 **Done**, topic resolved, devlog recorded |
| 11:00:53 | Front run 10 (`run-0234`) reports home: run fully done |

**Front runs: 10** (`run-0225`–`run-0234`, $1.74 total). autolab servings: 6
(3 superdirector, 1 supercoder, 2 entrance). Developer messages: **3**.

## Findings

### 1. Front read two resolved topics as two *unstarted* ones

Its first run reported `workrun-task2-g-1` as "empty — no work has happened
there yet" and concluded `main/` had never been committed. Both topics had
been resolved minutes earlier, so their live names carry Zulip's `✔ ` prefix
and a read by the bare name finds nothing. This is the rename-blind read from
p2/p3, recurring for the third episode running — and, as in p4, **it cost
nothing**: Front asked autolab rather than acting on the wrong picture, and
autolab's answer corrected it inside a minute. The trap is unfixed; its damage
is again zero.

Worth noting what made that recovery possible: autolab's own-channel entrance
answered a factual question about a project's state. That is `agent_standardize`
p10's entrance earning its keep on a path nobody designed it for.

### 2. "Not already covered" was read as "the routine has nothing to do"

The standing text says pick one repository **not already covered in `main/`**.
`basecamp/omarchy` had been covered eleven minutes earlier by Step 2's
hand-fired mission, and Front concluded the run's job was already done:
*"Nothing further needs doing for this run."*

That is a defensible reading of the sentence as written, and it is the single
most useful thing this step found: **a duplicate guard phrased as a filter
also reads as a completion test.** The remedy is one clause in the standing
text — "each fire produces exactly one *new* summary; what is already covered
tells you what to skip, not whether to run" — and it costs nothing to add.
Left unedited here so the phase report can quote the text that produced the
behaviour.

### 3. A mission with no task files cannot start, and autolab knows it

autolab planned G-4 as "one straightforward pick-summarize-commit mission" and
wrote no `task1.md`. Its own reply says the consequence out loud — *"the
superdirector wrote no task files; the mission has no sub-work"* — and then, in
the same breath, *"each task waits for a post in its own `workrun-…` topic"*.
There is no such topic. The mission is `In Progress` and unstartable.

Asked, autolab diagnosed itself correctly and named two prior occurrences:

> This is the same stall pattern seen twice before (R3-16, R3-20)… The actual
> gap: for mission G-4, no `task1.md` was ever written… **The fix:** post again
> in `pj-ghtrends/workplan-trend2` asking for the task to be (re-)planned with
> a real task breakdown.

Front then acted on that advice **without a human** (10:55:09). Self-diagnosis
plus self-recovery, with the human only watching, is the strongest thing in
this step. Three occurrences now; this is a standing defect, not a fluke.

### 4. …but a re-plan that writes only `task1.md` still does nothing

The re-plan wrote `task1.md` and no `plan.md`, and generation 2 of the
workspace shows exactly that:

```
.local/topics/pj-ghtrends/workplan-trend2/2/superdirector/
  chatlog.md  current/  task1.md  tools/
```

`handle_superdirector_response` runs `reconcile_task_files` and
`prepare_run_surfaces` **inside `if plan.is_file():`**. No `plan.md` in that
generation → the task file is never read, no Sub-Work, no topic. Front checked,
found `work-g-4` still empty, and correctly refused to nudge again.

Neither agent can see this: the rule lives in the listener, not in any guide.
**This is where the Omni Agent had to intervene** — intervention 2 told Front
the mechanism, autolab wrote both files in one round, and the topic opened
immediately.

*Did X for agent Y — handoff candidate.* Two candidate fixes, neither built
here: (a) reconcile task files whenever any are present, not only beside a
`plan.md`; (b) say in the workplan guide that a re-plan rewrites `plan.md`
together with its task files. (b) is smaller and needs no code.

### 5. The first-run permission ask — **confirmed, and it was three asks**

p4 finding 6 predicted one ask on a new routine. `ghtrends` produced three
Developer messages, and only the *first* is that finding: Front had a genuine
choice about what the run meant and asked. The other two are findings 2–4
above, not caution. So: the per-routine ask is confirmed for a third routine,
and "one Developer word once per routine" holds — the extra words this run
cost were paid for defects, not for nerves.

### 6. Duplicate avoidance across two runs — **worked, on the second reading**

`main/index.md` was read, `basecamp/omarchy` was recognised as covered, and the
second run picked a different repository. The index did its job; what failed
was the sentence about what to *do* with it (finding 2). Two-for-two on
distinct repositories.

## The second summary

`main/repos/openai-codex.md`:

```markdown
# openai/codex

## What it does

Codex is OpenAI's lightweight coding agent that runs in the terminal. It
lets a developer drive an AI coding assistant from the command line to
read, edit, and run code in a local project, rather than through a
browser-based or IDE-embedded interface.

## Why it's trending

Seen on the GitHub trending list:

- URL: https://github.com/trending
- Date observed: 2026-08-26
- Position: #12
- Stats shown on the trending page: 118,580 stars total, +1,181 stars
  today, primary language Rust

Terminal-based coding agents are having a broad moment on this trending
list (Codex sits alongside several other agent/assistant projects such as
`ponytail` and `openhuman`), and Codex's backing by OpenAI combined with
active development appears to be driving the day's spike in stars.

## License

Apache License 2.0 (SPDX: `Apache-2.0`).

## Stats (from the GitHub API)

Source endpoint: `https://api.github.com/repos/openai/codex`
(fetched 2026-08-26)

| Field | Value |
|---|---|
| Stars | 118,580 |
| Forks | 18,074 |
| Open issues | 13,889 |
| Primary language | Rust |
| Default branch | `main` |
| Created | 2025-04-13 |
| Last push | 2026-08-26 |
| Fork | No |
```

### Trend evidence check (p4 finding 1), summary 2 of 2

Checked against `https://api.github.com/repos/openai/codex` immediately after
the run: `118581 / 18074 / 13889 / Rust / Apache-2.0 / main / 2025-04-13`.
Stars differ by **one**, in a repository gaining ~1,000 a day; every other
field is exact. **Pass.**

And the same blemish as summary 1, word for word: the "Why it's trending"
bullet is headed *"Stats shown on the trending page"* over figures that came
from the API, with one genuine page-derived number ("+1,181 stars today")
mixed in. Two for two — so this is the **template**, not a slip: the layout
`README_PROJECT.md` records has a section whose heading misdescribes its own
source. That is a documentation fix in the project, not a model failure, and
it is Step 4's to raise.

## State at the end of the step

```
$ git -C .local/projects/ghtrends/main log --oneline
3497151 Add openai/codex trending repo summary (2026-08-26)
a3465c4 Add basecamp/omarchy summary and index (task2 p1 step1)
$ git -C .local/projects/ghtrends/main status -sb
## main...origin/main
```

`main/index.md`:

```
| Repo | Summary | Trending date seen | Notes |
|---|---|---|---|
| basecamp/omarchy | [repos/basecamp-omarchy.md](repos/basecamp-omarchy.md) | 2026-08-26 | #9 on https://github.com/trending |
| openai/codex | [repos/openai-codex.md](repos/openai-codex.md) | 2026-08-26 | #12 on https://github.com/trending |
```

Plane: `G-5` Done, `G-2`/`G-3` Done, the two mission Works `G-1` and `G-4`
still `In Progress` (nothing marked them done; `mission_done` was never run).
The undeclared devlog now holds two mission folders,
`g-1-…/task-2` and `g-4-…/task-1`. `publish/` is still unborn and untouched.

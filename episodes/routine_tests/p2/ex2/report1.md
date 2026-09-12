# routine_tests p2 ex2 step 1 — what would count as a recurrence

Date: 2026-09-13 JST (timestamps UTC). Plan: [plan.md](plan.md) step 1.

Written before anything was posted. Nothing in this step spends a paid run.

## State confirmed before the run

- `nctl status --json`: `ok: true`, no errors. `nctl drift --json`: no
  errors, `converged: 43`. Infrastructure state only.
- Listeners running under launchd: Front, autolab, forge, the arXiv sage,
  cagent and the agentroom relay. The relay's routines view is `live`.
- Deployed revisions, read from the installed packages rather than lock
  files: pyagag `ed65b4e` in both Front's and autolab's environments,
  agfront `842d7b1`, agautolab `b5259cd`. These are ex1's.
- Routine board: `study-uspolitics`, `retired: false`, guide **v2, message
  6393**, 2 runs, 0 open, latest `routinerun-2026-09-12T1622Z` finished and
  resolved.
- Study workspace: `main/` at `ac2f7d2`, `publish/` at `d467141`
  (`main...origin/main [ahead 1]`) — as p2 left them.
- `#pj-studyuspolitics` holds four unresolved `workplan-` topics —
  `workplan-setup-studyuspolitics-workspace`,
  `workplan-collect-and-analyze-contributions`,
  `workplan-continue-uspolitics-contribution-study`,
  `workplan-publish-studyuspolitics` — and
  `✔ retired-workplan-collect-and-analyze-contributions-m6371`. Left alone.
- The plan asks for the `brandump.md` → `braindump.md` rename to be staged
  with this exercise. It is **already committed**, in the plan's own commit
  `0fa0fc8`; nothing is outstanding there.

## The recurrence table, defined

Each row says what counts as a recurrence and where it is read. Numbers are
filled in at step 4.

| Problem | Counts as a recurrence when | p2 | ex1 | Read from |
|---|---|---|---|---|
| **C** — first serving answers as if the conversation were empty | Front's reply to the request says, or acts as if, there is no request, in any serving of the new `front-*` conversation | 2 of 2 | 0 of 16 | Front's reply text; the `front` run record's `num_turns` and tool calls; the listener's `serving` line |
| **B** — the opening serving delegates on the run's behalf | the serving that opens the `routinerun-` topic also posts to autolab (or anyone) for it, i.e. the delegation topic's `[selfnote][rootchat]` names the `front-*` conversation, or a plan callback is served by the `front` role rather than `routine_run` | 1 of 1 | 0 of 3 | `agentchat read --all` on the delegation topic; the listener's `mention … serves …` lines |
| **A** — a plan retirement strands the run's anchor | autolab retires the mission and a later mention in the replacement is logged `carries no root note of ours` | once | recovered in a staged case | only if a retirement happens: the `[replaces]` note and the `serves …` line |
| topic-name collision | the run delegates into a `workplan-` topic name already anchored by another run — here, one of the four open topics above — and a callback goes to the wrong run | — | 1 of 3 | the delegation topic name against the list above; the root notes in it |
| stall with no wake signal | a task ends its serving while its own work is still running, and nothing serves anybody again | once | not exercised | autolab's task topic and run records; gaps in both listener logs |
| resolved without the requester's agreement | autolab resolves the task or mission before the run says it is done | once | not exercised | order of the run's acceptance and autolab's ✔ in the task and plan topics |

Also recorded, because ex1 did not exercise it:

- **An autolab mission end to end on pyagag `ed65b4e`** — plan, start,
  completion callback, acceptance.
- **A real callback into a `routine_run` serving.** Whether the run relays
  what the delegate's thread says (it was in front of it), or has to open
  `threads/` first — ex1's "C repaired the chatlog, not the threads" limit.
  Read from the `routine_run` transcript/run record: turns, tool calls, and
  whether the run's entry quotes the delegate.

## Evidence capture

- Both listener logs are being written (last lines from ex1's live
  sequence). Their line counts at the start are recorded so this run's
  lines can be cut out afterwards.
- Last run record numbers per role before the run: Front `front` 0614,
  `routine_run` 0052; autolab `superdirector` 0197, `supercoder` 0267.
  Anything above those belongs to this exercise.
- Raw logs, transcripts, conversation reads and run records go in the
  ignored `.local/` beside this plan.

## How the request will be posted

As the Developer, with `agentchat send` on the Developer credential and **no
`AGENTCHAT_HOME`** — a root note is only written when a home is set, so the
post is a plain message and anchors nothing. Into a new topic
`front-uspolitics-<stamp>` in `#front`, the same shape p2 used.

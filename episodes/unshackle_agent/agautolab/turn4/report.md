# unshackle_agent / agautolab turn4 — report

Executed on 2026-08-10 (agstudio) from [turn3/report.md](../turn3/report.md)'s
candidate list, at the developer's request, with no separate plan and — for the
first time in this episode — **no cost ceiling**. Items 1–3 were done, item 4's
question was answered by accident, and item 5 was answered by measurement
rather than by decision.

Turn spend: **$4.43** (one iteration's cost is unrecorded; that is F4).

**Headline: agautolab's own shackles are gone, and the largest remaining one
was never in agautolab.** Every in-system agent on this node — mediator and
coding agent alike — silently inherits
`/Users/eiji/projects/.claude/CLAUDE.md`, a two-line file written for the Omni
Agent, and its first line ("if the permission classifier halts work, stop
immediately and explain to the user") is an instruction a headless agent
cannot obey and should not follow. It has killed at least five runs across
this episode, including the one turn3 recorded as F5 without knowing the
cause. The shackle_list surveyed four directories and missed the file above
them all.

Against that: `slow-brew で`, three words with no other explanation, resolved
correctly on first contact — `styles/` is kept, and the deletion question is
closed.

## What was done

| Item | Change |
|---|---|
| 2. F3 — the goal outlives its phase | `build_plan_prompt` now states that the goal comes from `job.yaml` and heads every later iteration's prompt unchanged, and that `PLAN.md` is the document that can speak to one phase. Same fact in `AGENT_GUIDE.md`'s `job.yaml` section, addressed to whoever writes the goal. One new test. |
| 1. F6 — the `styles/` probe | One mission whose only procedural content was `slow-brew で`. Answered on the first session (F2 below). `styles/` **kept**, off the deletion list for good. |
| 3. Commit + cleanup | Two commits (`638930b` `--detach`, `3bf8579` gate output + goal lifetime) — two turns of working tree, now on `main`, unpushed like the two before them. `:8799` stopped by explicit PID; a stray turn3 polling loop killed. |
| 4. F1's second truncation sample | Obtained (F4). |
| — | **New:** the inherited-instruction channel (F1), found mid-turn and measured with a controlled 8-vs-8 probe. |
| Tests | 83 → **84 passed**. |

## Live runs

One mission (`POST /mission`, `max_sessions: 5`, gateway on `:8799`), one new
job carried plan → approve → implement, one accidental job, and 16 probe
iterations.

### Mediator sessions

| # | turns | elapsed | cost | what it did |
|---|---|---|---|---|
| 23 | 17 | 112 s | $0.525 | wrote `humantime/job.yaml` **in slow-brew shape**, started the plan iteration `--detach`, ended |
| 24 | 9 | 54 s | $0.325 | re-verified `jobreport` from stale NOTES; found the plan iteration alive; ended |
| 25 | 7 | 30 s | $0.254 | reviewed `PLAN.md` + 10 gates, **approved**, started the implement iteration `--detach` |
| 26 | 8 | 29 s | $0.244 | confirmed the iteration alive (pid, ppid 1, elapsed); ended without waiting |
| 27 | 5 | 27 s | $0.209 | the same check again; ended without waiting |

Driver: session budget (5) exhausted, no `done` — the job had already
converged two sessions earlier.

### Coding-agent iterations

| job / iter | turns | elapsed | cost | gates | handoff |
|---|---|---|---|---|---|
| `humantime` iter-0001 (plan) | 4 | 47 s | $0.195 | — | — |
| `humantime` iter-0002 (implement) | 19 | 64 s | $0.466 | **10/10** | ✓ |
| `csvstat` iter-0001 (in scope) | 4 | 17 s | $0.129 | 0/2 | ✗ — stopped on a denial |
| `csvstat` iter-0001 (out of scope) | — | 60 s (killed) | **unrecorded** | 2/2 | **✗** |

## The counts

| measure | result |
|---|---|
| `slow-brew で` resolved into the plan/approve round trip | **1 / 1**, first session, named by name |
| mediator sessions reaching for `--detach` | **2 / 2** that needed it |
| plan reviewed without a checklist | 1 / 1, approved, converged |
| job outcome | **converged, 10 / 10**, two iterations |
| denial probes ending dead, inside `/Users/eiji/projects` | **1 / 8** |
| denial probes ending dead, outside it | **0 / 8** |
| probe runs citing an instruction to stop, inside / outside | **2 / 8** vs **0 / 8** |
| runs killed by that instruction across the episode (natural, not probes) | **≥ 5** |
| handoff after a killed iteration | **0 / 2** across two turns |
| cost recorded after a killed iteration | **0 / 2** |
| mediator sessions spent only on "is it still running" | **2 / 5** ($0.45) |
| mediator sessions committing work that was not theirs | **0 / 5** (turn2: 1) |

## Findings

### F1 — Every in-system agent inherits an instruction file nobody surveyed

`agent/CHARTER.md` opens by telling the mediator that `MISSION.md` is "your
only external input". That is false, and has been for the whole episode.

The mediator's session runs `claude -p` with cwd = the `agautolab` repo; a
coding agent runs with cwd = a job's `target/`. Both sit under
`/Users/eiji/projects/`, so both are handed
`/Users/eiji/projects/.claude/CLAUDE.md` verbatim, ahead of the charter.
Confirmed directly by asking a one-shot agent in a job's `target/` to quote
its project instructions; it returned the file's two rules word for word, plus
the harness's skill listing (which is why two `snake-web` iterations reasoned
about "the `run` skill" — nothing in this system has ever mentioned one).

The first rule is the damaging one:

> 権限分類器によって作業が止まった場合は直ちに作業を停止してユーザーに状況を説明すること。

A denial is the one thing this system is designed to survive. agforge turn2 F3
and agautolab turn2 F5 both concluded that a refusal carrying its own reason
is enough, and turn2 measured 14 of 14 denials routed around. This rule tells
the agent to do the opposite, and the agents comply, verbatim:

- `jobreport` iter-0007 — "**Per my instructions, I need to stop and explain
  rather than push past this**", then two options and a question, to nobody.
  *This is turn3's F5, and it now has a cause.*
- `csvstat` iter-0001, this turn — a `mkdir` denial, and: "**Per the project's
  CLAUDE.md instruction, I need to stop immediately and report this rather
  than retry.**" It then names the workaround it is not going to take ("I can
  try using `Write` directly"). 17 seconds, $0.13, an empty diff, `stuck`.
- `snake-web` iter-0003 — "Per your CLAUDE.md instructions, I'm stopping
  rather than retrying."
- `snake-web-b` iter-0002 — "I need to pause here per your CLAUDE.md
  instruction… Two ways to proceed — your call:".
- mediator session-0001, turn1 — "Per the operating rule in this environment…
  I need to pause here." Turn1 recorded that session as failing to engage with
  its mission and fixed the charter; the charter was not the cause.

**The controlled probe.** Same one-line goal ("write `out.txt` containing the
cwd; obtain it by running `pwd`"), same allowlist (`Write,Read,Bash(python3:*)`,
so the shell redirect is always denied), same model, 8 job directories inside
`/Users/eiji/projects` and 8 identical ones outside it. All 16 hit the denial.

| arm | n | denials | not converged | cites a stop-rule |
|---|---|---|---|---|
| inside (`CLAUDE.md` in scope) | 8 | 8 | **1** | **2** |
| outside | 8 | 9 | 0 | 0 |

Honest reading: 1-vs-0 dead runs at n=8 is not significant on its own, and on
a task this trivial the workaround is one obvious step away. What the probe
does establish is the *mechanism*, in the agents' own words — inside the
scope, two of eight narrate the rule as the reason they abandoned an approach
("per project rules I stopped that attempt immediately and used the Write tool
instead"); outside it, not one of eight mentions any rule at all, or anything
but the result. The cost is established by the five natural cases above, on
real work, where the workaround was not one step away.

The second rule ("never wait more than a minute for a long process") is
cheaper but not free: sessions 26 and 27 cited it, spent $0.45 confirming a
pid was alive, and ended. The mission then ran out of session budget with the
job already converged and nobody left to write `done`. On a node whose whole
answer to long work is "detach and check later", a rule against waiting turns
the driver into a poller.

This is a **Tool Implantation of the purest kind, from outside the surveyed
tree** — prompt-side commands, delivered by the harness, invisible to the
charter, invisible to `shackle_list.md`, and stronger than anything the
project itself does, because it arrives before the charter and is phrased as
an absolute.

The ENT-ordered fix is not mine to make: it is the developer's own operating
file, addressed to the Omni Agent, and it does its job there. What is needed
is a **scope**, so that a file meant for one agent stops being read by the
agents that agent supervises. Named for the developer to decide, not done.

Generalisable, and it is the episode's largest: *the survey has to cover
everything the agent is actually given, not everything the project wrote.* We
counted every `never` in `CHARTER.md` for four turns while a stronger `never`
sat one directory up.

### F2 — `slow-brew で` resolved on first contact, by name

The mission body was one task and three words: a small `humantime` CLI,
`slow-brew で`. Nothing else about how to work; no mention of phases, gates,
`PLAN.md`, approval, or `styles/`.

Session 23 wrote `job.yaml` **without a `gates:` key** and reported:

> Created `.local/jobs/humantime/job.yaml` in **slow-brew** style (no `gates:`
> key) …

The job started in the plan phase, session 25 reviewed the plan and its ten
proposed gates and approved, and the implement iteration converged 10/10. The
whole round trip came from the shorthand.

So turn3's F6 was right to stop the deletion, and right about why: *zero usage
is only evidence of deletability when the thing was offered to the party it
was written for.* `styles/` was written as a human's vocabulary, was never
spoken to an agent in four turns, and worked the first time it was. **Kept,
and off the candidate list permanently.**

Worth noting what kind of artefact this is. `styles/README.md` is fourteen
lines naming two options, each with one line of what it costs — no procedure,
no prohibitions. It is exactly the shape memo.md's second axis is aiming the
guides at, and this is the first evidence that shape carries meaning across
the gap.

### F3 — The goal trap was re-armed, and this time the agent walked past it

Turn3's F3: the mediator writes a plan-phase instruction into `goal`, `goal`
heads every implement prompt unchanged, and the first implement iteration
obeys it and does nothing.

The fix went in before the mission (plan prompt + `AGENT_GUIDE.md`), and the
mediator **re-armed the trap anyway**: `humantime`'s goal ends with
「まずは実装せず、設計を `PLAN.md` に書いてほしい」. The implement prompt for
iteration 2 carried it at line 19, exactly as before.

And iteration 2 built the package regardless — 19 turns, 22 tests, 10/10.

Two honest halves:

- The fact did **not** prevent the trap. It is stated in the plan prompt (the
  coding agent) and in `AGENT_GUIDE.md` (the mediator); the mediator either
  did not read that section or did not apply it.
- The trap did **not** fire. But the phrasing differed from turn3's in a way
  that matters: 「まずは」 ("for now, first") is temporally scoped, where
  turn3's goal said "This iteration's **only** deliverable is a written design
  … write no library code". One sample each way; the failure is real and
  non-deterministic, and the variable that looks live is whether the sentence
  reads as phase-scoped, not whether the agent was warned.

Not worth a third mechanism. The next cheap move, if it recurs, is to state
the same fact where the mediator cannot miss it — in the plan prompt's own
output, or in the goal template — rather than to constrain what a goal may
say.

### F4 — A killed iteration still leaves nothing, on a second sample

turn2's F1 counter-case had one observation. This turn produced a second,
unplanned: the `csvstat` job run **outside** `/Users/eiji/projects` for F1's
control arm was killed by its own 60 s wall clock.

- `adapter_result.json`: `exit_code: -1`, `timed_out: true`, `num_turns: null`,
  `total_cost_usd: null` — **the spend is unrecorded**, which is why this
  turn's total is a floor.
- No `NOTES.md`. Nothing for a successor.
- **The gates passed anyway**, 2/2, on the 9.7 KB of package, tests and
  fixtures the agent had already written to disk before it was killed. The job
  converged from a corpse.

Identical in every respect to `wordfreq` in turn2. **2 / 2.** The design is
consistent: the handoff is the agent's, so when the agent is killed there is
no handoff. The price is paid only on timeouts, and it is now paid twice.

The remaining question turn2 raised — whether the charter should say
"write the handoff early, you may be killed" — stays open, and stays a
question about a fact rather than a rule.

### F5 — The role boundary held this turn, without anything being drawn

Turn2's F6 was the mediator committing the Omni Agent's uncommitted work.
Session 23 met the same situation and made the other choice:

> Also noted (without touching) that this repo itself has an unrelated,
> finished-looking uncommitted `--detach` feature diff sitting in the working
> tree from an earlier session — left it alone since it wasn't part of this
> mission.

Nothing was added to the charter between turn2 and turn4 about whose changes
are whose. The same agent architecture produced both outcomes, and both were
defensible on the day. Two samples, no rule, and the standing obligation
(record the crossing) is the only thing that has been needed.

Turn3's F7 — the mediator installing a package into this node's Python for its
coding agent — did not recur; nothing this turn reached outside a job
directory.

### F6 — agautolab's own removal work is finished

Audited against `shackle_list.md`'s ranked list rather than trusting turn3's
summary:

| shackle_list item | now |
|---|---|
| NOTES.md first line matched exactly (`STATUS: …`) for loop control | **gone** — no match in `gateway.py` or `drive.sh` |
| `notes_are_stale` (MISSION newer than NOTES invalidates status) | **gone** |
| NOTES.md regenerated from a code template | **gone** — agent-written, read forward as-is |
| `proposed_gates.yaml` schema-checked to decide whether planning happened | **demoted** — `load_proposed_gates` is a reviewer convenience; nothing branches on it; the plan phase stops for review whatever the agent left |
| `devstyle_report` regex over NOTES.md | **gone** |
| summarizer `NARRATION` strip + re-validation | **gone** — non-JSON stdout is carried through as the summary |
| `SUMMARY_PROMPT` / `WINDOW_PROMPT` prohibitions | **gone** — both are now facts plus "your reply is shown as written" |
| director's "First, read GUIDE.md" prefix | **gone** |
| the window given no tools at all | **gone** — an explicit `Read,Glob,Grep` |
| failing gate reaching the agent as a name only | **gone** (turn3) |
| the foreground-only constraint | **gone** (turn3, `--detach`) |

What is kept, and why: the wall-clock kill, evidence capture, the `flock`, the
exit-code contract, the no-skip-permissions rule, and secrets staying under
`.local/`. The first four are observation and recoverability; the last two
guard irreversible harm on a machine holding real credentials. None of them
exists to stop an agent being wrong.

`AGENT_GUIDE.md` is 98 lines, `agent/CHARTER.md` 47, `styles/README.md` 9.
Every remaining "never" in them names a credential or an irreversible push.

**agautolab is done unshackling.** Further turns here are measurement.

## What was not done

- **The `CLAUDE.md` scope (F1).** Found, measured, and left for the developer:
  it is their file, addressed to the Omni Agent, and choosing its scope is
  their call, not a change to make on their behalf mid-turn.
- **A third goal-lifetime mechanism (F3).** Deliberately not added.
- No window / summarizer / director runs this turn.
- The mission never wrote `done`; the driver's budget ran out after the job
  converged. Recorded rather than re-run.

## Deus Ex Machina note

**Two, both small, both on the mediator's behalf and both avoidable now.**

- Stopped the `:8799` gateway by PID after the session budget was exhausted —
  the same act as turn2 and turn3, and the same handoff candidate.
- Ran the `csvstat` and denial-probe iterations by hand rather than through a
  mission. These were instrument runs, not work the mediator was asked for, so
  this is measurement rather than an intervention.

Nothing was committed on the mediator's behalf that it could have committed
itself — it saw the diff and chose to leave it (F5), so the two commits in
"What was done" are the Omni Agent's own work being recorded as such.

The standing candidate from turn3 (a sanctioned way for a mediator to change
the environment its subordinate runs in) did not come up.

## State after this turn

- `uv run pytest -q` → **84 passed**.
- Working tree **clean**; `main` is now four commits ahead of `origin`
  (`383e43f`, `380c483`, `638930b`, `3bf8579`). Nothing pushed.
- New jobs: `humantime` (**converged**, 2 iterations, 10/10 gates,
  agent-written `NOTES.md`, a working package with 22 tests — F2's evidence),
  `csvstat` (**stuck**, 0/2, an empty diff — F1's evidence), and
  `denial-in-1` … `denial-in-8` (the probe's inside arm; `denial-in-2` is the
  dead one). The outside arm had to live outside the repo — its location was
  the independent variable — so it stayed in the session scratchpad and is
  gone with it; what it showed is in F1's table.
- `.local/agent/NOTES.md` is 13 KB — the mediator's own memory, five sessions
  of `humantime` triage on top of turn3's `jobreport` history. Still not edited
  by me; still its file. **Turn2's obsolete glob note is still line 3, three
  turns on** — and this turn the mediator wrote a glob straight into
  `humantime/job.yaml` anyway, contradicting its own standing note, and it
  launched fine. The note is now not merely stale but disregarded by its own
  author, which is the cheapest possible demonstration that a fact in
  agent-owned memory decays with no one to retire it.
- `.local/agent/MISSION.md` holds the `humantime` mission; no `done`. Turn3's
  MISSION/NOTES are backed up in the session scratchpad.
- Gateway: this turn ran on **:8799**, stopped by explicit PID (84780). The
  developer's **:8791 (pid 8630) was not touched** and answers `/healthz`.
- A stray turn3 polling loop (pid 71225) was killed.
- `jobreport 0.1.0` is still `pip install -e`'d into this node's Python
  (turn3 F7), untouched.

## Next turn candidates

There is no unshackling left to do here. What remains is one decision, one
carry-over, and one question that is now bigger than agautolab.

1. **F1 is the whole next move, and it is not agautolab's.** Decide the scope
   of `/Users/eiji/projects/.claude/CLAUDE.md`. Then re-survey: `agforge` and
   `agdevworld` sit under the same directory, so `agforge`'s "runs leaving
   nothing for the caller" and its silent-kill denial class deserve re-reading
   with this channel in view. **`shackle_list.md` should gain a section for
   instructions that arrive from outside the project tree** — it is the only
   category that was never surveyed and it currently outranks everything that
   was.
2. `agent/CHARTER.md`'s "your only external input" is a wrong stated fact for
   as long as F1 stands, and this episode's standing lesson is that a wrong
   stated fact costs real turns.
3. F3, if it recurs a third time: state the goal's lifetime where the mediator
   writes the goal, not where the coding agent reads it.
4. F4's open question: whether the charter should say that an iteration can be
   killed and its handoff lost, so the agent can choose to write early.
5. The mediator's `done`: five sessions, and the mission ended with a converged
   job and no note, because two sessions were spent polling (F1's second rule).
   Worth one more mission after F1 is settled to see whether it closes cleanly.

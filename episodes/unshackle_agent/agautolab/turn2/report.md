# unshackle_agent / agautolab turn2 — report

Executed [plan.md](plan.md) on 2026-08-10 (agstudio). One work item changed
code (item 3); everything else was measurement. 5 coding-agent iterations
across 3 new jobs, 6 mediator sessions, 15 window answers, 1 summarizer run
and 1 director run.

**Headline: the untested claim of turn1 held, and a capability nobody had
measured turned out to be broken.** Every iteration that had a successor
wrote its own `NOTES.md`, and the successor read it and acted on it —
turn1's central change works (F1). Against that, the mediator **cannot drive
any coding iteration that runs longer than its own two-minute Bash window**:
it backgrounds `run-once` against the charter's stated fact, the session ends,
the iteration dies with it, and the next session repeats. Three sessions and
$2.20 bought zero completed iterations before the driver was stopped by hand
(F2). This is not a consequence of unshackling; it is a structural gap that
turn1 missed because its job was small enough to finish inside the window.

Turn spend: **$6.11** — over the plan's $6 ceiling. Live runs were stopped at
that point per the plan's own rule; what that cost is listed under
"What was not done".

## What was done

| Work item | Result |
|---|---|
| 1a. Multi-iteration handoff | `logstat`: 3 iterations, **3/3 wrote `NOTES.md`**. Iteration 2's prompt carried iteration 1's handoff verbatim (`evidence/iter-0002/prompt.txt:58`), and iteration 2 opened by acting on it. |
| 1b. Truncated iteration | `wordfreq` with a 60 s budget: killed mid-work, **no handoff, no cost record**, gates passed anyway. |
| 2. Slow-brew | Mediator chose the plan phase unprompted; the mission then failed on F2. The plan iteration was run by hand instead: 397-line `PLAN.md`, 15 proposed gates, **0/15 pass on an empty repo**. |
| 3. `claude_bin` glob | `src/agautolab/binpath.py` (`resolve_command`), used per launch by `claude_code`; 2 tests; one line in `AGENT_GUIDE.md`; `palindrome/job.yaml` repointed to the glob. **Committed by the mediator itself** (F6). |
| 4. Denial cost | 14 denials across 6 sessions and 5 iterations. **Every one arrived as a readable tool error and was routed around.** No run died silently. No allowlist widened. |
| 5a/5b. Charter engagement | 6/6 sessions engaged with the mission immediately. `done` written 3/3 times where the mission was completable. |
| 5c–5e. Window battery | 15 answers, **14 correct**. p50 11.8 s, max 35.7 s, $0. Payload-shape drift in 1 of 3 passes. |
| 5f. Summarize / director | Summarizer $0.179 / 18.1 s — and it diagnosed a broken gate two coding agents had misread (F3). Director ✓ $0.145 / 16.6 s. |
| 6. Deletion candidates | Counted, nothing deleted. `push` and `load_proposed_gates` are both **alive** and come off the candidate list. |
| 7. Tests | 77 → **79 passed**. |

## Live runs

### Coding-agent iterations

| job / iter | what it was for | turns | elapsed | cost | gates | handoff written |
|---|---|---|---|---|---|---|
| `logstat` iter-0001 | 1a, first pass | 24 | 135.9 s | $0.672 | 14/15 | ✓ 2.0 KB |
| `logstat` iter-0002 | 1a, does it read the handoff | 29 | 124.2 s | $0.617 | 14/15 | ✓ (rewritten) |
| `logstat` iter-0003 | 1a, third pass → `stuck` | 14 | 44.6 s | $0.282 | 14/15 | ✓ (rewritten) |
| `wordfreq` iter-0001 | 1b, killed at 60 s | — | 60 s (killed) | **unrecorded** | 4/4 | **✗** |
| `jobreport` iter-0004 | 2, the plan iteration | 25 | ~170 s | $0.909 | — (plan) | — |

`jobreport` iterations 1–3 do not appear: they were started by mediator
sessions, died with them, and left no evidence at all (F2).

### Mediator sessions

| # | mission | turns | elapsed | cost | engaged | wrote `done` | denials |
|---|---|---|---|---|---|---|---|
| 13 | verify `palindrome`'s convergence | 10 | 56 s | $0.327 | ✓ | ✓ | 3 |
| 14 | cost audit of the node | 19 | 76 s | $0.497 | ✓ | ✓ | 4 |
| 15 | has `smoke-fizz` rotted? | 12 | 37 s | $0.277 | ✓ | ✓ | 2 |
| 16 | slow-brew (shared library) | 37 | 363 s | $1.236 | ✓ | ✗ | 3 |
| 17 | ″ (continued) | 16 | 97 s | $0.505 | ✓ | ✗ | 2 |
| 18 | ″ (continued) | 3 | 10 s | $0.465 | ✓ | ✗ | 0 |

### Window, summarizer, director

15 window answers on `qwen3.6:35b-a3b-coding-nvfp4`, 3 passes of the same 5
questions, $0. One summarize ($0.179, 18.1 s, 5 turns). One director
($0.145, 16.6 s, claude-sonnet-5).

## The counts, with denominators

| measure | result |
|---|---|
| iterations with a successor that wrote a handoff | **2 / 2** (3 / 3 wrote one at all) |
| successors whose prompt carried the handoff | **1 / 1**, verbatim |
| handoff after a killed iteration | **0 / 1** |
| iterations with no cost recorded | **1 / 5** — the killed one |
| mediator sessions that engaged with the mission | **6 / 6** (turn1: 1 of 3 pre-fix) |
| `done` written when the mission was completable | **3 / 3** |
| mission sessions that completed a coding iteration | **0 / 3** (F2) |
| proposed gates passing on an empty repo | **0 / 15** |
| denials that killed a run silently | **0 / 14** |
| allowlist widenings earned | **0** |
| window answers correct | **14 / 15** |
| window passes with `POST /mission` payload drift | **1 / 3** |
| mediator sessions naming a development style | **0 / 6** (0 / 11 since turn1) |

## Findings

### F1 — The handoff works, and it is better than the template it replaced

This was turn1's central untested claim, and it holds. `logstat` iteration 1
wrote a 2 KB `NOTES.md` naming every module, the fixture it built, the manual
verification it ran and what it deliberately did not attempt. Iteration 2's
prompt carried that text verbatim, and iteration 2 opened with:

> Iteration 1 reported all gates green, but **the handoff said** the
> 1,000,000-line performance gate failed at the start of this iteration.

It then investigated, tried a hand-written parser to gain margin, measured a
2% gain, judged the correctness risk not worth it, reverted to a
byte-identical file, and wrote all of that into the next handoff — including
a warning to its successor about how to interpret a future failure of that
gate. No template could have produced any of that: `_write_notes` wrote a
status line, a truncated 2000-char output tail, and a gate table.

Iteration 3 rewrote it again. **3 of 3.** Nothing in the prompt says "write
NOTES.md"; `_workspace_facts` states where the file is and who reads it next.
The fact was enough.

The counter-case is 1b and it is worth stating plainly: `wordfreq` was killed
at 60 s and left **nothing** — no handoff, and no cost record either, because
the process died before emitting its result JSON. Under the deleted template
a document always existed. So the honest summary is: *when the agent gets to
finish, its handoff is far better than the template's; when it is killed,
there is now no handoff at all.* Both halves are the design working as
intended — the second is the price, and it is paid only on timeouts.

### F2 — The mediator cannot drive an iteration longer than its own Bash window

The largest finding of the turn, and nothing to do with unshackling.

`CHARTER.md` states, correctly, that `run-once` and `loop` "run in the
foreground of a live session; backgrounded, they die with a headless session."
The mediator's own Bash tool cuts a foreground command off after about two
minutes. The plan iteration for `jobreport` needed ~170 s. So the mediator
had no legal move. It took the illegal one:

> The plan iteration is running in the background — it exceeded the 2-minute
> foreground window so it's continuing there. I'll wait for it to finish
> rather than poll.

The session then ended, taking the iteration with it. `state.json` advanced
to iteration 2 with no `evidence/iter-0001/` ever written. Session 17
repeated it. Session 18 repeated it in 3 turns and 10 seconds. Three
sessions, **$2.20, zero completed iterations, three dead coding agents whose
spend is unrecorded.** The driver would have burned three more sessions; it
was stopped by explicit PID (see "State after").

Why turn1 missed it: `palindrome`'s single iteration finished in ~90 s, inside
the window. This turn's jobs are the first that take longer, and `logstat`'s
three iterations were run by hand rather than through a mission.

The fix is not a rail and not a prompt sentence. Ranked, for turn3:

1. `run-once --detach` (or a `POST /jobs/<job>/run` route) that survives the
   session — a *fact* the charter can name, replacing a constraint the agent
   cannot satisfy.
2. Failing that, the charter should say what an agent should do when the
   iteration outlasts its window, because right now it says only what it must
   not do.

Standing lesson, in its fourth costume: *a charter that names a door needs
the door to exist (agforge turn1 F1), to be openable in the time given
(agdevworld F1), to work the way the line says (agforge turn3 F1) — and to be
openable by the hand that is told to open it.*

### F3 — A failing gate reaches the agent as a name, not as its output

The `logstat` performance gate I wrote was broken: `%%50` survived the YAML
heredoc into a Python one-liner, so it was a `SyntaxError` and could never
pass. That is my bug, and it made an accidental experiment.

`build_implement_prompt` lists failing gates as **command strings only**. The
gate's actual output lives in `evidence/iter-NNNN/gates.json`, which the agent
can reach (`--add-dir`) and which `_workspace_facts` names as holding "gate
results". Neither iteration looked. Both concluded the gate was a flaky
timing check and blamed machine load — a reasonable inference from a bare
command that says `d < 2.0`, and wrong.

The summarizer, given the same evidence directory and no other advantage, got
it in one:

> it failed with a Python `SyntaxError`, not a timing miss: the gate's own
> shell-escaped `%%` broke the inline script before logstat ever ran. That's
> a bug in the gate command itself (harness-side), not something the agent's
> code caused.

The difference is what each was handed. The summarizer reads `gates.json`;
the coding agent reads a list of names. **Two iterations and ~$0.90 were
spent on a diagnosis the evidence already contained.**

This is *not* an argument for a rail. The ENT-ordered fix is one line of
fact: carry the failing gates' `output_tail` into the prompt, or name
`evidence/iter-NNNN/gates.json` explicitly as where the failure text is. That
is more information, not more instruction. Turn3's cheapest high-value item.

### F4 — The deleted plan contract was reproduced by the agents, unprompted

Turn1 deleted five gate requirements, a YAML skeleton, "plan, do not
implement yet", and the reviewer's checklist. This turn is the first time a
real coding agent has been asked to plan.

- The **mediator** wrote "Design (do not implement yet) … This iteration's
  only deliverable is a written design in `PLAN.md`; write no library code"
  **into the goal it authored**. The sentence turn1 removed from the code-side
  prompt was reinvented by an agent, in its own words, in the document where
  it actually belongs — the client's request.
- It also grounded the goal in real evidence (naming `wordfreq`'s timed-out
  iteration with no cost fields, and `logstat`'s failing gate, as the
  partial-data cases the design must handle) and **widened `add_dirs` itself**
  so the coding agent could read those siblings. Turn1's F1 lesson —
  *a charter that points at a new place needs the grants widened in the same
  commit* — performed by an in-system agent with nobody asking.
- The **coding agent** produced a 397-line `PLAN.md` and 15 proposed gates.
  Against the requirement turn1 deleted most explicitly: **0 of 15 pass on an
  empty repo.** Every one imports the package or invokes the CLI. The design
  document also records that all three sibling jobs are finished yet still
  carry an empty `.lock`, so "a lock file exists" is not a liveness signal —
  a fact about this system that no human had written down.

The half that was *not* answered: whether the mediator would have reviewed
well. It never got the chance (F2). The plan-review checklist stays untested
against a real plan.

An aside worth keeping: the mediator's elaboration of a one-paragraph desire
into a two-page technical brief is exactly what the deleted AGENT_GUIDE rule
("keep `goal` near-verbatim, do not translate into a technical contract")
prohibited. Deleting the rule made the behaviour observable, and on this one
sample it produced a better job than the rule would have allowed.

### F5 — Every denial was survivable, on both agents

agforge turn2's F3 distinguished tool-level denials (kill the run silently)
from bash-pattern denials (arrive as a readable error). Carrying that here
was a plan item; it needed no dedicated probe, because 14 denials arrived on
their own across 6 sessions and 5 iterations.

**All 14 were readable, and every agent routed around them.** Examples:

- Session 13 could not write `.local/agent/done` with a bash heredoc
  redirect; it used the Write tool instead, succeeded, and reported the
  detour in its final message unprompted.
- Session 14's `for d in .local/jobs/*/; do …` was denied twice; it rewrote
  the work as a Python script.
- Both `logstat` agents were denied compound `cmd1; cmd2` one-liners and each
  wrote in its handoff that the sandbox blocks them, so its successor would
  not waste turns rediscovering it.

So agautolab's `--allowedTools` denials behave like agforge's *bash-pattern*
class, not its silent-kill class. The silent-kill class in agforge came from
a tool being *unspecified* (the harness asks, headless auto-rejects); every
tool here is either granted or explicitly matched. **No widening earned**, and
the "write the reason into the refusal, not into the prompt" conclusion now
holds on a fourth mechanism.

### F6 — An in-system agent committed the Omni Agent's work

At 08:17:20, mid-mission, the mediator found work item 3 uncommitted in the
working tree, understood it, and committed it:

```
380c483 Resolve glob adapter_config.command to its newest match
  … so a glob like the one in .local/agent/claude_bin (which points into a
  version-numbered, update-churned extension path) failed to launch instead
  of resolving the way gateway.py and session.sh already do.
  Co-Authored-By: Claude Sonnet 5
```

Nothing asked it to. The commit message is accurate and better than the one
that would otherwise have been written. It is also a role-boundary event in
the opposite direction from the usual one: the in-system agent did the Omni
Agent's work, not the reverse. Recorded rather than prevented — but worth
noticing that the mediator's charter gives it `git` and a repository, and
nothing says whose changes are whose.

Related and unresolved: the mediator's own `.local/agent/NOTES.md` contains
exactly one note, and it is the glob workaround that item 3 has now made
obsolete. Its cross-session memory is a stale fact about a bug that no longer
exists. Deliberately not edited — that file is the agent's, not mine — but
an agent whose environment is fixed underneath it has no way to learn that.

### F7 — The window is steady; the miss moved

14 of 15 correct, against turn1's single 5/5 pass. The one miss is the same
question class turn1's F1 named, with the error inverted: asked what was
running while `logstat`'s iteration 1 was in flight, pass 2 answered "the node
is currently **idle**", reasoning entirely from `driver_running: false` and
ignoring the job it had been handed. Passes 1 and 3 got it right, and pass 3
was the best answer of the battery — it named the job, its phase, its
iteration count, *and* inferred the blocking gate from the state alone.

So "what is running" conflates two things (a driver and a job) and the model
resolves the ambiguity differently run to run. 2/3 is not a regression worth
a prompt sentence at this sample size; recorded, not patched.

Latency: 4.5–35.7 s, p50 11.8 s. The worst case got worse than turn1's 28.1 s.
Still $0.

Payload drift (turn1 F8's residual): pass 1 drafted `POST /mission` with a
`"text"` key; passes 2 and 3 named the route and the bearer header without
inventing a body. **1 of 3** — under the decision rule fixed in the plan
(≥2/3), `GUIDE.md` is left alone. Worth noting the system already answers
this itself: `POST /mission` returns `400 body must be {"mission": "..."}`,
which is a refusal that carries its own reason (F5's principle) to anyone who
actually tries.

### F8 — Deletion candidates: two came off the list

Counted, per the plan; nothing deleted this turn.

| candidate | count | verdict |
|---|---|---|
| `_push_target` / `push.json` | 2 jobs with `push: true`, 4 `push.json` on disk | **alive** — off the list |
| `load_proposed_gates` | parsed `jobreport`'s file for `status --json`, which is how the reviewer sees the gates | **alive** — off the list |
| `styles/README.md` | named in **0 of 11** sessions since turn1 | still a candidate; two turns of zero |
| `fake` adapter | the only $0 cover for plan/approve/reject/replan | keep — it is the regression suite |
| `no_progress_limit` in old `job.yaml` | 7 files carry the key; **0** code references | harmless (`Job.load` ignores unknown keys); noted so no one hunts for the reader |

## What was not done

Live runs stopped at $6.11, past the $6 ceiling the plan set. Left open:

- **The slow-brew review half.** `jobreport` sits in `awaiting_approval` with
  a plan and 15 gates. Whether the mediator reviews well without a checklist
  is still unmeasured, and cannot be measured until F2 is fixed.
- **A second truncation sample.** 1b is one observation.
- **The `logstat` gate.** Left broken on purpose so its evidence stays
  readable; the job is `stuck` at 3/3 iterations with 14/15 gates.

## Deus Ex Machina note

Three, all on the mediator's behalf, all handoff candidates:

- Ran `jobreport`'s plan iteration by hand after three mediator sessions
  failed to (F2). The mediator could not have done this itself — that is the
  whole finding.
- Raised `jobreport`'s `max_iterations` 3 → 6, because three iterations had
  been consumed by the F2 loop without running.
- Stopped the driver by explicit PID rather than letting it spend three more
  sessions on a loop that could not converge.

Work item 3 is not one: the mediator ended up committing it (F6).

## State after this turn

- `uv run pytest -q` → **79 passed**.
- Working tree clean; two commits ahead of origin (`383e43f unshackle`,
  `380c483` the glob fix, committed by the mediator).
- New jobs: `logstat` (`stuck`, 3 iterations, 14/15 gates, agent-written
  NOTES.md — 1a's evidence), `wordfreq` (`converged` from a killed iteration —
  1b's evidence), `jobreport` (`awaiting_approval`, `PLAN.md` +
  `proposed_gates.yaml` — item 2's evidence). None cleaned up.
- `.local/jobs/palindrome/job.yaml` now holds the glob again, as item 3's
  standing regression check.
- `.local/agent/`: MISSION.md holds the slow-brew mission; no `done` (the
  mission never finished); NOTES.md still holds the now-obsolete glob note
  (F6). Turn1's MISSION/NOTES/done are backed up in the session scratchpad.
- Gateway: this turn ran on **:8799** and it was stopped by explicit PID
  (57871). The developer's **:8791 was not touched** and is still listening —
  turn1's `pkill` lesson held.
- `.local/agent/gateway/run-0009.exit` was never written: the driver was
  killed rather than exiting, so `/status` will read that run as still in
  flight until the next `POST /mission` clears it.

## Next turn candidates

1. **F2 — give the mediator a way to run a long iteration.** Everything else
   in the slow-brew path is blocked behind it, and it costs money every time
   it is hit. Highest value item in the system right now.
2. **F3 — carry the failing gate's output into the implement prompt**, or
   name `gates.json` as where the failure text is. One line of fact, ~$0.90
   of demonstrated waste behind it.
3. Finish item 2: approve `jobreport` through the mediator and watch it review
   a real plan without a checklist.
4. F1's counter-case: a second truncation sample, and a decision on whether
   "a killed iteration leaves nothing" is acceptable or wants a fact in the
   charter about writing the handoff early.
5. `styles/` — 0 of 11 sessions. One more turn of zero makes it an
   evidence-backed deletion, the same way agforge retired `scan_markers`.
6. F6's boundary question: the mediator commits work it did not do. Decide
   whether that is a capability to keep or a line to draw.

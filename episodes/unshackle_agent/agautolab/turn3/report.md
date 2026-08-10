# unshackle_agent / agautolab turn3 — report

Executed on 2026-08-10 (agstudio) straight from
[turn2/report.md](../turn2/report.md)'s candidate list, at the developer's
request, with no separate plan. Items 1–3 and 5–6 were done; item 4 (a second
truncation sample) was not, on budget.

Two code changes, both one-for-one with turn2's findings, then one live
mission that exercised both. Turn spend: **$6.32**.

**Headline: turn2's two blockers are gone, and the mediator used the new door
on its first opportunity without being told to.** The mission that turn2 could
not run at all — three sessions, $2.20, zero completed iterations — ran to
`converged` **15/15** this turn, because `loop --detach` let one 306-second
iteration outlive three mediator sessions. And the fix that turn2 costed at
~$0.90 of wasted diagnosis paid for itself immediately: carrying a failing
gate's own output into the prompt is the visible difference between the
iteration that did nothing ($0.28) and the iteration that built the whole
package.

## What was done

| Item | Change |
|---|---|
| 1. F2 — an iteration the mediator can actually drive | New `src/agautolab/detach.py`; `--detach` on `run-once` and `loop`. Spawns the same command with `start_new_session=True`, appends to `<job-dir>/detached.log`, returns the pid at once. `CHARTER.md`'s "backgrounded, they die with a headless session" — a constraint the agent could not satisfy — is now a fact about a door that exists. Same line in `AGENT_GUIDE.md` and `README.md`. |
| 2. F3 — the failing gate arrives as its own output | `load_gate_output()` reads the previous iteration's `gates.json`; `build_implement_prompt` prints each failing gate followed by what it printed (last 2000 chars). `_workspace_facts` now names `gates.json` and what is in it instead of saying "gate results". |
| 3. Slow-brew, second half | `jobreport` carried from `awaiting_approval` to `converged` through a real mission: plan reviewed without a checklist, approved, implemented, 15/15. |
| 5. `styles/` | Counted again: **0 of 4** sessions, 0 of 15 since turn1. **Not deleted** — see F6; the count was measuring the wrong thing. |
| 6. F6 role boundary | Recorded, not fenced. See F7. |
| Tests | 79 → **83 passed** (3 detach cases, 1 gate-output case). |

## Live runs

One mission, `POST /mission` with `max_sessions: 4`, on a second gateway at
`:8799`. The mission: pick up `jobreport` from `awaiting_approval`, review the
plan as the reviewer, and carry the implementation through.

### Mediator sessions

| # | turns | elapsed | cost | what it did |
|---|---|---|---|---|
| 19 | 20 | 86 s | $0.670 | read PLAN.md + 15 gates, **approved**, started `loop --detach`, ended |
| 20 | 30 | 225 s | $1.175 | re-reviewed the plan independently, verified the plan's one open question against `run_once.py:364`, diagnosed the job's `allowedTools`, wrote it all into `NOTES.md` |
| 21 | 8 | 43 s | $0.345 | found iteration 6 in flight, chose to wait rather than race the lock |
| 22 | 36 | 220 s | $1.254 | saw 14/15, installed the package by hand, bumped `max_iterations` 6→7, `run-once --detach` for the recording iteration, wrote the verdict |

Driver exit 10 (session budget exhausted); no `done` file — the mission's last
act was still running when the budget ran out.

### Coding-agent iterations

| iter | turns | elapsed | cost | gates | handoff |
|---|---|---|---|---|---|
| 0005 | 6 | 15 s | $0.279 | 0/15 | ✗ — concluded there was nothing to do |
| 0006 | 69 | **306 s** | $2.012 | 14/15 | ✓ 3.7 KB |
| 0007 | 21 | 82 s | $0.584 | **15/15** | ✗ (converged; no successor) |

Spend: sessions $3.444 + iterations $2.875 = **$6.32**.

## The counts

| measure | result |
|---|---|
| iterations completed while no mediator session was alive | **3 / 3** (turn2: 0 / 3) |
| longest iteration carried to completion | **306 s** — 2.5× the foreground window that killed turn2 |
| mediator sessions that reached for `--detach` unprompted | **2 / 2** that needed it (sessions 19, 22) |
| iterations whose prompt carried a failing gate's output | **2 / 2** where one existed |
| plan reviewed without a checklist | **2 / 2** independent reviews, same verdict |
| job outcome | **converged, 15 / 15** |
| iterations with no cost recorded | **0 / 3** |
| mediator sessions naming a development style | **0 / 4** (0 / 15 since turn1) |

## Findings

### F1 — `--detach` was the whole fix, and it was taken on sight

Session 19 read the charter, approved the plan, and ran:

```
autolab loop .local/jobs/jobreport --sleep 5.0 --detach
```

then ended its session, 86 seconds in. The loop ran iteration 5 (15 s) and
iteration 6 (**306 s, 69 turns**) while sessions 20, 21 and 22 came and went
above it. Session 21's entire contribution was to notice the run was live and
*not* touch it. Session 22 started the final iteration the same way and then
ended while it was still running — a thing the mediator was previously
incapable of doing on purpose.

Nothing instructs the agent to detach. `CHARTER.md` states the option exists
in the same breath as the fact that an iteration usually outlasts a command
window, and `AGENT_GUIDE.md` has one table row. Turn2's ranked fix #1 was
"a door the charter can name, replacing a constraint the agent cannot
satisfy"; that is exactly what happened, and the standing lesson gets its
fifth form: *a charter that names a door needs the door to exist, to be
openable in the time given, to work the way the line says, to be openable by
the hand told to open it — and, once it is all four, the agent walks through
without being pushed.*

Verified before the live run that this is real detachment and not luck: a job
started with `--detach` from a bash shell that was then `kill -9`'d ran to
`converged` 30 seconds after its parent died.

### F2 — The gate's own output moved a decision, on the first attempt

Iteration 5 and iteration 6 had the same goal, the same plan, the same 15
gates and the same tools. The difference in their prompts was one section.

Iteration 5 (first implement iteration, nothing had run yet):

```
# Gate results last iteration
No gates have been run yet (first iteration).
```

It spent 6 turns and $0.279 and concluded:

> Both deliverables are already in place, match the approved plan exactly …
> This iteration's design work is complete. No further action needed.

Iteration 6 got the same 15 gates *with what they printed* —
`ModuleNotFoundError: No module named 'jobreport'`, `no tests ran in 0.00s`,
`ERROR: file or directory not found: tests/` — and built the entire package in
69 turns: a 7-module package with a CLI, a test suite and 6 fixture job
directories, 14/15 gates green.

This is not a controlled experiment (the two iterations also differ in which
one is last), and the honest reading is in iteration 6's own handoff, quoted
in F3. But the change cost four lines of prompt assembly and the evidence it
draws on was already on disk. Turn2 called it "turn3's cheapest high-value
item" and that held.

### F3 — The deleted rail came back inside the goal, and outlived its phase

Turn2's F4 celebrated this: the mediator, writing `jobreport`'s goal, put
*"Design (do not implement yet) … This iteration's only deliverable is a
written design in `PLAN.md`; write no library code"* into the client request —
reinventing, in its own words and in the right document, the sentence turn1
had deleted from the code-side prompt.

Turn3 is the bill for that. `job.yaml`'s `goal` is phase-independent: the same
text is the first section of every implement prompt too. So the first
implement iteration opened by reading an instruction not to implement, and
obeyed it.

Iteration 6 hit the same contradiction and resolved it explicitly, which is
the best paragraph any agent has written in this episode:

> The static `goal:` text in `job.yaml` still says "this iteration's only
> deliverable is PLAN.md, write no library code," but that instruction was for
> the plan phase; the live state (phase=implement, real failing gates, last
> iteration) says otherwise. I implemented the library rather than re-writing
> the design, on the theory that a final iteration with concrete
> implement-phase gates and no iterations left after it is not the moment to
> produce only a document again.

So the failure mode is real and recoverable, and both halves matter: an
agent-authored instruction can have a *lifetime* the code-side rail never had,
and an agent given enough live state can notice and overrule a stale
instruction it was handed. The deleted AGENT_GUIDE rule ("keep `goal`
near-verbatim") would have prevented this by preventing the good version too.

The ENT-ordered fix is one line of fact rather than a rule, and it is
turn4's cheapest item: the plan prompt could say that the goal it is given is
carried into every later iteration unchanged, so anything phase-specific
belongs in `PLAN.md` rather than in the goal. Not done this turn.

### F4 — Two independent reviews, no checklist, same verdict

Turn1 deleted the plan-review checklist; turn2 never got to test it. Two
sessions reviewed `jobreport`'s plan independently and both approved, on
reasons neither was given:

- Both singled out the same design decision as the one that mattered —
  `cost_usd: None` (no data) kept distinct from `0.0` (a real zero) — which is
  precisely the partial-data case the goal was written around.
- Session 20 found the one open question `PLAN.md` had flagged *for the
  reviewer* (does `agautolab` really `flock()` the `.lock` file?) and answered
  it by reading the orchestrator's source: `run_once.py:364`, `review.py:40`.
  Nothing asked it to; the plan asked, and it answered.
- Both noted the plan documenting its own limitation rather than hiding it as
  a reason to trust it.

Session 22's closing verdict was *"approve, trust it"*, contingent on the
recording iteration landing — which it did.

The counter-observation, and it is the useful one: session 20's confident
root-cause diagnosis was **wrong**. It concluded the job's `allowedTools`
lacked `Edit`/`mkdir`/`pytest` and that "the coding agent literally can't
scaffold a new package under those restrictions" — written while iteration 6
was mid-flight and scaffolding exactly that package with `Write` alone. The
next session inherited the claim through `NOTES.md`, checked it against the
14/15 result, and dropped it. A wrong fact travelled one session and died on
contact with evidence, which is the behaviour the design wants; it is worth
noting that it travelled at all.

### F5 — A denial reached a headless agent as a question nobody could answer

Iteration 7 converged, and then ended its final message like this:

> … is blocked by the classifier for me interactively. I can either:
> 1. Run it via `python3 -c "…"` as an equivalent check … or
> 2. You approve the `jobreport --version` bash call directly.
> Want me to proceed with option 1 as a proxy check, or would you like to
> approve the direct call?

There is no one there. The run is `claude -p`, one-shot; the question was the
last thing it ever said.

This is a new shape against turn2's F5 (14/14 denials routed around) and
agforge turn2's F3 (tool-level denials kill silently; bash-pattern denials
arrive readable). Here a third outcome: the denial arrived readable, the agent
had a workaround in hand and *named it*, and then spent its last turn asking
permission instead of taking it. Cost was nil — the gates had already passed,
15/15 — so this is recorded, not patched. If it recurs, the cheap fix is one
fact, not a rule: the charter can say that nobody reads the final message
during the run.

### F6 — `styles/` is not a deletion candidate; the count was measuring the wrong thing

Third turn of zero: **0 of 4** sessions this turn, **0 of 15** since turn1.
Under turn2's stated rule that made it an evidence-backed deletion, and the
deletion was started.

The developer stopped it and gave the reason the file exists: the names are a
**human-side shorthand** — a word to hand a mediator ("do this one slow-brew")
and a word to discuss a way of working in. My count measures agents naming a
style unprompted, which is not that. Every mission in all three turns was
written by me in prose and never used the shorthand, so the intended use has
never been *tested*, only never *volunteered*.

The correct probe, for turn4 and cheap (~1 session): put `slow-brew で` in a
mission body with no other explanation and see whether the mediator resolves
it into leaving `gates` out of `job.yaml` and running the approve round trip.
`styles/` and the `CHARTER.md` line stay until then.

Generalisable, and it applies to every count in this episode: *zero usage is
only evidence of deletability when the thing was actually offered to the party
it was written for.* agforge's `scan_markers` was offered to the agent 23
times and never taken. `styles/` has been offered to a human zero times.

### F7 — The role boundary moved again, this time downward

Turn2's F6 was the mediator committing the Omni Agent's work. This turn the
mediator did the same thing one level down: session 22, finding the 15th gate
(`pip show jobreport && jobreport --version`) failing because the package was
not installed, ran `pip install -e .` itself — outside the job directory,
against this machine's Python — and then started an iteration so the pass
would be *recorded by the loop* rather than resting on its own manual check.

Three things are worth separating:

- **The judgment was good.** It distinguished the coding agent's sandboxed
  Python (no `pip` module) from the shell the gates actually run in, which is
  the distinction the failing gate turned on, and it refused to let its own
  manual verification stand as the result.
- **The reach was outside the job.** Nothing in the charter scopes environment
  changes to a job directory; `.local/` is scoped for secrets and nothing
  else. `jobreport 0.1.0` is now installed editable in this node's Python,
  pointing into a job's `target/`.
- **It is the mediator being a Deus Ex Machina for its coding agent** —
  structurally the same move the Omni Agent makes for the mediator, and the
  first time an in-system agent has made it. Recorded, per the policy, rather
  than prevented.

## What was not done

- **A second truncation sample** (turn2 F1's counter-case). A `csvstat` job
  with a 60 s budget was written and then dropped unrun: iteration 6 cost
  $2.01 against an expected ~$1 and put the turn over its ceiling. "A killed
  iteration leaves no handoff and no cost record" therefore still rests on one
  observation.
- **The `styles/` probe** (F6) — needs one session, deferred with the deletion.
- **The goal-lifetime fact** (F3) — identified, not written.
- No window / summarizer / director runs this turn.

## Deus Ex Machina note

**One, and it is not mine.** The mediator's `pip install -e .` for its coding
agent (F7) — handoff candidate in the opposite direction from usual: the
question is not how to take it back but whether an in-system agent should have
a sanctioned way to change the environment its subordinate runs in.

Nothing was done on the mediator's behalf this turn. Turn2's two candidates
(running an iteration by hand, raising `max_iterations`) were both performed by
the mediator itself this turn — the first via `--detach`, the second in
session 22 — which is the clearest measure of what the two code changes bought.

## State after this turn

- `uv run pytest -q` → **83 passed**.
- Working tree: `detach.py` and `test_detach.py` new; `cli.py`, `run_once.py`,
  `test_run_once.py`, `CHARTER.md`, `AGENT_GUIDE.md`, `README.md` modified.
  Nothing committed (turn2's two commits still the last on the branch).
- `styles/` untouched.
- `.local/jobs/jobreport`: **converged**, 7 iterations, 15/15 gates,
  `max_iterations` 7 (bumped by the mediator), a 3.7 KB agent-written
  `NOTES.md`, and a working `jobreport` package in `target/` — 7 modules, a
  CLI with `--json`, 20 unit tests, 6 fixture job directories.
- `jobreport 0.1.0` is `pip install -e`'d into this node's Python by the
  mediator (F7), pointing at `.local/jobs/jobreport/target`. Left in place: it
  is what makes gate 15 pass, and removing it would falsify the recorded
  convergence.
- `.local/agent/NOTES.md` is 4 KB now — the mediator's own cross-session
  memory, holding its review verdict, its (superseded) diagnosis and its
  recovery plan. It still opens with turn2's stale glob note, which turn2's
  item 3 made obsolete: **two turns now, the agent has carried forward a fact
  about a bug that no longer exists.** Still not edited by me; still its file.
- `.local/agent/MISSION.md` holds the jobreport mission; no `done` (the
  session budget ran out with the final iteration in flight). Turn1/turn2
  backups are in the session scratchpad.
- Gateway: this turn ran on **:8799**, stopped by explicit PID (70716). The
  developer's **:8791 (pid 8630) was not touched** and answers `/healthz`.
- `.local/agent/gateway/run-0010.exit` = 10, written normally.

## Next turn candidates

1. **F6's probe**: one mission whose body says `slow-brew で` and nothing else
   about how to work. It decides `styles/`, and it is the first test of the
   whole guide-as-shared-vocabulary idea.
2. **F3's one line**: tell the plan prompt that the goal it writes is carried
   into every later iteration unchanged. Cheapest remaining item; $0.28 of
   demonstrated waste behind it and a known-good agent workaround above it.
3. F1's counter-case, still one sample: a second killed iteration.
4. F5: whether "nobody reads the final message during the run" is worth a line
   in the charter, or whether iteration 7 was a one-off.
5. F7: decide whether a mediator changing the environment outside a job
   directory is a capability to name or a line to draw. It worked; it was also
   nobody's stated job.
6. **The `STYLE.md` files deleted in turn1 hold nothing worth restoring**, and
   checking that was worth the five minutes. Two of their lines looked
   load-bearing at first glance and neither survived inspection:

   - *"Reject an acceptance framework materially larger than the product."*
     This turn is the case it would have judged, and it would have judged it
     wrong. Of the 15 approved gates, six map one-to-one onto the six
     partial-data cases the goal named, three pin the `--json` shape and exit
     code the goal asked to have spelled out field-by-field, and one checks the
     read-only guarantee the plan promised. That is the requirement enumerated,
     not ceremony. The one gate that failed (`pip show jobreport && jobreport
     --version`) failed because it depended on an install step no party inside
     the job could perform — a **hidden prerequisite, not a scale problem**.
     The line names size as a proxy for a thing that is not size.
   - *"The mediator writes neither implementation nor tests"* — the only place
     the role boundary of turn2 F6 and turn3 F7 was ever written down. But the
     standing obligation for a boundary crossing in this system is to *record*
     it, not to prevent it, and both crossings were recorded and were good
     work. Nothing to restore; keep watching.

   A third line from the same files — "run `run-once` and `loop` only in the
   foreground" — is the constraint `--detach` replaced this turn, so its
   deletion is already vindicated.
7. The deletion candidates that survive: `styles/` (held, F6), the `fake`
   adapter (keep — it is the regression suite), and nothing else. Like agforge
   after its turn3, agautolab is close to **done unshackling**: what remains in
   the kept list is the wall-clock kill, the evidence capture, the lock, the
   no-skip-permissions rule, and the exit-code contract.

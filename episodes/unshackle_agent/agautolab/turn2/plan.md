# unshackle_agent / agautolab turn2 — test the untested claim, turn observations into rates

Turn1 removed every forced path in the agautolab section of
[shackle_list.md](../../../../../shackle_list.md) except seven, and the
capability survived: a mission went in and came out as a converged job with
9/9 gates ([turn1/report.md](../turn1/report.md)). Turn2 is **not another
removal sweep** — agforge reached the same point at its turn2 and found there
was almost nothing left to take out. This turn does four things:

1. **Exercise the change turn1 could not exercise.** NOTES.md authorship —
   the headline of turn1 — was never run: `palindrome` converged in one
   iteration, so no handoff was ever needed (turn1 F7). The central claim of
   the turn is still untested by live behaviour.
2. **Run the path that has only ever run on the fake adapter**: slow-brew,
   plan → review → approve → implement, against a real coding agent, with the
   five-point gate contract and the reviewer checklist both deleted.
3. **Turn single observations into rates.** F5 (charter idle sessions) is 1
   occurrence pre-fix and 1 verification post-fix; F8's window battery is one
   pass; `done` has been written 2 times. None of those is a rate.
4. **Make one stated fact true** (F6's glob), and **count** the things turn3
   might delete, rather than deleting them on argument.

Direction: [../../../../../memo.md](../../../../../memo.md). Prior turns whose
lessons this one inherits: [../turn1/report.md](../turn1/report.md),
[../../agforge/turn2/report.md](../../agforge/turn2/report.md),
[../../agforge/turn3/report.md](../../agforge/turn3/report.md),
[../../agdevworld/turn1/report.md](../../agdevworld/turn1/report.md).

This remains an experiment. A regression that appears is a finding to record,
not a defect to patch in the same turn — except where a decision rule below
fixes the response in advance, so results are not argued after the fact.

## What stays

Turn1's seven, unchanged, and nothing new is added to the list:

1. No `--dangerously-skip-permissions` on this node (allowlists are how).
2. Secrets stay under `.local/`, never in a `target/` repo.
3. The bearer token on `POST /mission`.
4. The job `.lock`; reads never take it.
5. Wall-clock bounds and single-flight: iteration/gate timeouts,
   `max_iterations`, `max_sessions`, one drive / summarizer / window at a
   time, the summary cache.
6. Evidence, session JSON, cost/backend metadata.
7. Gate *execution* and the git auto-commit — the observation, not a verdict.

Item 1 is under measurement this turn, not under review: work item 4 asks what
an allowlist denial actually costs, because agforge turn2's F3 showed the
answer differs by kind of denial. If it turns out to cost whole runs silently,
that is turn3's problem to solve *without* dropping the device.

## Work items

### 1. Force a multi-iteration job and watch the handoff (turn1 F7)

The turn's first move, as turn1 ordered. `_write_notes` is deleted; the coding
agent has `--add-dir <job-dir>` and a prompt that names
`<job-dir>/NOTES.md` as "the handoff between iterations. Yours to write; the
next iteration is given whatever is there." Nobody has yet seen whether an
agent uses it.

Two jobs, both `claude_code` / `claude-sonnet-5`, both instant-ramen so the
plan phase is not in the way:

- **1a — natural multi-iteration.** A deliverable that will not land in one
  pass: ~15 gates across two source files, a `--help` contract, exit codes,
  and one gate the first pass is likely to miss. `max_iterations: 4`.
  Expect 2–3 iterations.
- **1b — truncated iteration.** The same class of job with
  `iteration_timeout_seconds: 150`, so iteration 1 is killed mid-work. This
  is the case the deleted template used to cover for free: a timed-out agent
  wrote no handoff, but code wrote one anyway. Whether anything survives a
  kill is now entirely the agent's habit, and that is worth knowing before it
  matters.

Recorded per iteration: did `NOTES.md` exist when the next prompt was built;
did the next iteration's `evidence/iter-NNNN/prompt.txt` carry it (grep the
prompt, not the intent); did the agent's work show it read it; length and
shape of what was written.

**Decision rule, fixed now.** Across the iterations that have a successor:

- handoff written in **≥ half** → the claim holds, nothing changes.
- **1 or more but under half** → wording only, one line in
  `_workspace_facts`, no code, and re-measure in turn3.
- **0** → still wording first (ENT order, agforge F2 / agdevworld F3), and the
  report says plainly that a code-written handoff was doing work the agent
  does not do. Restoring `_write_notes` is **not** on the table this turn.

### 2. Slow-brew end to end, against a real coding agent (turn1 candidate 4)

The plan/approve path has only run on `fake`. Deleted in turn1 and never
tested against real output: the five gate requirements, the YAML skeleton,
"plan, do not implement yet", and AGENT_GUIDE.md's plan-review checklist.

One mission through `POST /mission`, phrased for a shared-contract-shaped
deliverable so the mediator has a reason to pick slow-brew. Run it as a
mission, not by hand — whether the mediator *chooses* the style at all is
part of the measurement (`styles/README.md` is now a 14-line file nothing
forces anyone to read).

Objective measurements, since "was the plan good" is not one:

| question | how it is answered |
|---|---|
| did a plan appear at all, and in what shape? | `target/PLAN.md` exists / does not; `load_proposed_gates` parses / does not |
| do the proposed gates pass trivially on an empty repo? | run each gate in a scratch `git init` dir and count passes — this is exactly what the deleted requirement bought |
| does every gate name something checkable? | count gates that never touch the deliverable |
| did the reviewer review? | the mediator's session transcript: did it read PLAN.md before approving; did it approve, reject, or edit the gates via `--gate` |
| did implementation then converge? | iterations, gates passed, cost |

A reject → replan round is in scope if the mediator chooses one; it is not
forced. If the mediator approves a plan whose gates pass on an empty repo,
that is the turn's headline finding and it belongs in turn3 as one line of
*fact* in the plan prompt ("gates that pass on an empty repo prove nothing"),
not as a returned schema check.

### 3. Make the `claude_bin` fact true (turn1 F6)

`.local/agent/claude_bin` holds a glob. `gateway.py` resolves it,
`session.sh` resolves it since turn1, and **`autolab` core does not** — so a
mediator that copies the pointer into `adapter_config.command`, exactly as
CHARTER.md tells it to, produces a job that cannot launch. Turn1's mediator
diagnosed this itself and hard-coded the version-numbered path into
`palindrome/job.yaml`, which will go stale at the next extension update.

This is agforge turn3's F1 in a fourth costume: *a charter that names a door
needs the door to work the way the line says.* It is not a rail — nothing
about it reads an agent's words or decides anything for it.

- One helper, newest-match-or-literal, in `src/agautolab/` (`claude_code`'s
  `command`, and the `add_dirs` path stays untouched).
- `session.sh` and `gateway.py` keep their own copies; a shell script and a
  service that must start without the package cannot import it. Three
  implementations of nine lines is the honest cost — record it rather than
  building a shared module that couples them.
- One line of fact in `AGENT_GUIDE.md`: `command` may be a glob; the newest
  match is used.
- Repoint `palindrome/job.yaml` back to the glob as the regression check.

### 4. What does an allowlist denial cost here? (agforge turn2 F3, carried in)

agforge established that the two kinds of denial behave differently: a
**tool-level** denial (the harness asks, headless auto-rejects) killed the run
with no output at all; a **bash-pattern** denial arrived as a readable tool
error and the agent routed around it — rebuilding a zip in Python and
delivering anyway.

agautolab's grants are the first kind in three places: `session.sh`'s
`--allowedTools`, `claude_code`'s per-job `--allowedTools`, and the
gateway's `Read,Glob,Grep` for window/summarizer/director. Kept item 1 rests
on them. Nobody has checked what a denial does here.

Two deliberate probes, observation only:

- **4a** — a job whose obvious route needs a command outside its per-job
  allowlist (the `palindrome` list is `Bash(python3:*),Bash(ls:*)` plus the
  file tools; ask for something that invites `curl`, `git` or `pip`).
  Record: did the run die with no output, did the agent read a refusal and
  route around it, and what reached `adapter_output.txt`.
- **4b** — the mediator against a command outside its 61: one that is
  plausible in a session (`ssh`, `docker`, `brew`, `zip`).

Do **not** widen a list to make a symptom disappear. A denial the agent
survived earns no widening (agforge turn2 earned none); a denial that killed a
run silently is a finding about the shape of the safety device, and the
response is turn3's.

### 5. Rates for the single observations

All cheap, all measurement:

| # | what | n | decision rule fixed in advance |
|---|---|---|---|
| 5a | fresh mission on the fixed charter — count idle "what would you like me to help with?" sessions (turn1 F5) | 5 sessions | ≥ 1 idle → one more line of charter wording, no code; 0 → settled |
| 5b | `.local/agent/done` written when the mediator finishes | every mission session this turn | 0 of n → the file is not discoverable enough; wording |
| 5c | window battery ×3 passes on qwen3.6 (turn1 F8 is one 5/5 pass) | 15 answers | ≥ 2 wrong in a class → the model choice is not settled and the report says which class |
| 5d | window latency distribution (28.1 s worst case in turn1) | same 15 | report p50/max; the developer decides the trade, not this plan |
| 5e | `POST /mission` payload-shape drift in the window's drafted requests (turn1 F8 residual) | same 15 | ≥ 2 passes drift → one line in `GUIDE.md`'s Doors entry naming `{"mission": …, "max_sessions": …}`, as a fact |
| 5f | `POST /director` and one `summarize` re-run | 1 each | none — regression check only |

5a can reuse a trivial mission (one that finishes in one session); the
measurement is engagement, not the work.

### 6. Count what turn3 might delete

agforge's turn3 deleted two things on two turns of counting and nothing
observable changed. Same instrument here. Nothing is deleted this turn:

- `load_proposed_gates` — used by whom, in the item-2 run? (reviewer
  convenience, or dead code the mediator bypasses with `--gate`)
- `styles/README.md` — did the mediator read it, name a style, or record one?
  Zero across two turns makes it a turn3 deletion candidate.
- `fake` adapter — anything it still covers that a real run does not.
- `push.json` / `_push_target` — `push: false` in every job so far.
- The `no_progress_limit` key surviving in old `job.yaml` files: harmless
  (`Job.load` ignores unknown keys), worth one line in the report so the next
  reader does not go looking for the code that reads it.

### 7. Tests

`uv run pytest -q` green (77 at the end of turn1) before any live run and
again at the end. Work item 3 adds resolution tests (glob → newest, literal →
literal, no match → as written). Nothing asserts an agent said the right
thing; if item 1 tempts a test that asserts a handoff exists, that test does
not get written.

## Live-run budget and order

Order matters: the cheap regression checks first, so a broken environment is
found before the expensive runs, and item 3 before item 1 so the jobs launch.

1. `pytest`, `run-once` on `fake` (both flows), gateway `/healthz` — free.
2. Item 3's glob fix + tests — free.
3. Item 5a, 5b — ~$0.5.
4. Item 1a, 1b — ~$1.0–2.0, the turn's centre.
5. Item 2 slow-brew — ~$1.0–1.5, the turn's second centre.
6. Items 4a, 4b — ~$0.3.
7. Items 5c–5f — $0 on ollama, plus one `summarize` (~$0.14) and one
   `director` (~$0.15).

Estimated turn spend: **$3–5**. Turn1 was $2.03. If the total passes $6 with
items still open, stop and report what is left rather than continuing.

Record for every run, as in turn1: elapsed, turns, cost, backend/model, and
the evidence path.

## Operational facts worth not rediscovering

- Turn1's edits are **committed** (`383e43f unshackle`), unlike agforge's and
  agdevworld's working trees. Turn2 starts from a clean tree here.
- **Never `pkill` by pattern on this node.** Turn1's cleanup killed the
  developer's own foreground gateway on :8791. Run this turn's gateway on
  :8799, stop it by explicit PID, and leave :8791 alone (agforge adopted the
  same rule at its turn2).
- The charter is read from disk per session (`session.sh` pipes it on stdin),
  and `GUIDE.md` per window request: wording edits need no restart.
  `gateway.py` and `src/agautolab/` do.
- Existing jobs: `binary-cli`, `fizzbuzz`, `palindrome`, `roman-numeral`,
  `smoke-fizz`, `snake-web`, `snake-web-b`. New jobs get new directories;
  do not reuse `palindrome`, it is turn1's evidence.
- `.local/agent/` currently holds turn1's palindrome `MISSION.md`, `NOTES.md`
  and `done`. `POST /mission` clears `done`; back up all three before the
  first mission of this turn.
- Window default is `qwen3.6:35b-a3b-coding-nvfp4`; the claude contrast is
  cheap here because the binary is local.
- Sessions 0005–0007 are the zero-byte exit-127 runs from turn1's F4 and read
  as `unparsed` in `/status`. Expected, not new breakage.

## Success criteria

- Item 1 has an answer with a denominator: the handoff is written, or it is
  not, and the report says at what rate and what the response was.
- Item 2 has an answer about the deleted plan contract that is measured, not
  argued — in particular the empty-repo gate count.
- Item 4 says which kind of denial agautolab's three allowlists produce.
- Every rate in item 5 is reported with its denominator, including the ones
  that came back clean.
- No rail is restored. Where a regression is real, the response is wording
  first, and the report names which it chose and why.

## Out of scope

- Editing agforge or agdevworld (driving either is fine, and a joint run is
  not needed for anything here).
- Restoring `_write_notes`, the `STATUS:` contract, `_made_progress`, or any
  prompt-side "must/never", whatever item 1 finds.
- Deleting anything counted in item 6 — that is turn3's, with two turns of
  evidence behind it.
- Widening any allowlist without a recorded denial (turn1's corollary).
- Identity for the window / a second authenticated route, `POST /director`'s
  half-implemented state, and the gateway's monitor UI.

## Obligations

- A Deus Ex Machina note for anything done on an in-system agent's behalf
  (policy.md). Item 3 is one by construction: the mediator cannot fix its own
  briefing or the loader it is told to configure.
- Report to `report.md` beside this file, in the shape of the prior turns:
  what was done, a live-run table with every repeat listed individually, a
  counts-with-denominators table, numbered findings, state after.

# unshackle_agent / agautolab turn1 — remove the forced paths

Goal: strip agautolab of Tool Implantation. Every place where code forces an
in-system agent down a deterministic path is removed; what remains is the
*fact* that jobs, evidence, gates and services exist, told once, in the fewest
words that still let the agent find them. Then run the same missions that
worked before and see what the agents still do on their own.

Source of the survey: [../../../../../shackle_list.md](../../../../../shackle_list.md)
(agautolab section). Direction: [../../../../../memo.md](../../../../../memo.md).
Prior turns, whose findings this plan inherits:
[../../agforge/turn1/report.md](../../agforge/turn1/report.md),
[../../agdevworld/turn1/report.md](../../agdevworld/turn1/report.md).

This is an experiment, not a hardening pass. A capability that quietly
regresses is the finding we are here to collect — do not re-add a guard to
prevent it. Failures leave evidence; that is enough.

## The shape of the problem here

agautolab is not one agent. It is **three**, with a deterministic machine
between them:

| agent | where | today |
|---|---|---|
| the mediator | `agent/session.sh` ← `CHARTER.md` | has tools; its words are parsed for loop control |
| the coding agent | `adapters/claude_code.py`, once per iteration | has tools, `cwd=target/`; its output must be JSON |
| the window / summarizer / director | `agent/gateway.py` | read-only; the window has no tools at all |

The machine in the middle (`run_once.py`) is the densest rail in the system.
Its distinctive failure mode is not "code checks the agent's answer" (agforge)
nor "the agent has no hands" (agdevworld) but a third thing: **code overwrites
the agents' words and then reads its own writing back.** NOTES.md — the
handoff document, the only continuity either agent has — is generated from a
template by `_write_notes`, and then `drive.sh` matches its first line to
decide whether the mission is over. Neither agent authors the document that
governs them.

So the headline work is authorship, in three places: the handoff (item 1), the
mission-complete verdict (item 2), and the plan/approval transition (item 3).

Per user decision, **the allowlist form stays** everywhere it exists today
(agforge made the same call). Allowlists widen in this turn; they do not
disappear. That also settles the condition raised in the proposal: the
mediator on agstudio never runs with permission enforcement at zero.

## What stays (the complete list — everything else goes)

Not exempt because they are useful. Exempt because they guard against
irreversible or cross-boundary harm, guard a resource, or are pure observation
that takes no judgment from an agent.

### A. Irreversible / cross-boundary harm

1. **No `--dangerously-skip-permissions`**, by any agent or job config.
   agstudio is the developer's own Mac and carries real credentials beyond
   what any job needs. Every other item on this list can fail into evidence
   and be fixed next iteration; this one fails into a damaged machine. The
   *prohibition text* leaves the guides — the invariant that survives is
   *no agent on agstudio runs with permission enforcement at zero*, and the
   allowlists kept per C1 are how it is enforced. `skip_permissions` stays a
   supported `adapter_config` key for experimental VMs; nothing on this node
   sets it.

2. **Secrets do not leave `.local/`, and never enter a `target/` repo.**
   `push: true` sends `target/` to Gitea, where repos are public. A token in
   a public repo cannot be un-published — history, mirrors, indexers. The
   *rule* "repos are public by default" is a preference, not a guard, and is
   deleted; the retained fact is one line: pushing publishes irreversibly.

3. **The bearer token on `POST /mission`.** Behind that door `drive.sh`
   writes code, pushes repos and spends $0.13–1.35 per job, unattended. The
   window is unauthenticated and its context carries text no agent in it
   wrote (job state, other agents' NOTES, another node's summaries) — an
   injection surface that must not gain an execution path. The invariant is
   policy.md's: *identity, not endpoint shape, decides what a request may
   do.* Every commanding sentence about this (in `WINDOW_PROMPT`, in the
   guides) is deleted: the window may try `POST /mission`, read the 401, and
   explain it in its own words (agdevworld F4).

### B. Resource and spend guards

4. **The job `.lock` (flock)**, and the corresponding property that reads
   never take it (`status`, every gateway GET, the summarizer). It guards a
   shared mutable resource — `target/`'s working tree, `state.json`, the
   evidence numbering — not the agent's correctness. Two concurrent
   `git add -A` in one worktree destroy both iterations' record.

5. **Wall-clock bounds and single-flight**: `iteration_timeout_seconds` plus
   the outer watchdog, `gate_timeout_seconds`, `max_iterations`,
   `max_sessions`, one drive / one summarizer / one window answer at a time
   (409), and the summary cache (one iteration paid for once). Same class as
   agforge's kept 900 s kill, with real money attached. Numbers may be
   raised; each is stated to the agent as a fact rather than hidden.

### C. Observation

6. **Evidence, session JSON, and cost/backend metadata.** Required by
   policy.md (Agent ≠ Model: every run records which backend served it), and
   this turn is *measured* with these files — agforge's F6 and agdevworld's
   F2 were only findings because the records existed. The `META_KEYS`
   allowlist goes (discarding is also a judgment); the recording stays.

7. **Gate execution — the observation, not the verdict.** Running the gate
   commands and writing `gates.json` takes nothing from any agent: the gates
   were proposed by the coding agent and approved by the mediator, and the
   agent stays free to disagree with the result in its own words. The git
   auto-commit of `target/` is kept with it, as the mechanism that makes
   `diff.patch` possible and the previous iteration's work recoverable.
   **What does not stay is the verdict**: `_made_progress`,
   `consecutive_no_progress`, `no_progress_limit`, and the stuck-by-judgment
   branch (item 4).

Out of scope, not exemptions: the `0/10/20/30/40` exit codes and the
`state.py` status enum. They are a program's return value, consumed by
systemd, `loop` and `drive.sh` — an inter-program contract, not a rail on an
agent. (`drive.sh`'s `STATUS:` first-line contract is a different thing and is
removed in item 2.)

## Work items

### 1. NOTES.md authorship → the coding agent

The largest single case of a machine overwriting an agent's words.

- Delete `_write_notes` and `_write_plan_notes` (`run_once.py:235-333`).
  Code writes no handoff document.
- Keep the *reading* side: the next iteration's prompt still carries the
  previous NOTES.md if one exists, and says so. Passing a fact forward is not
  a rail.
- If the agent writes nothing, the next prompt simply says there is no
  handoff. Code does not fill it in, does not template it, does not truncate
  it (the 2000-char adapter-output tail dies with the template).
- **Grants widen in the same commit (agforge F1).** The coding agent runs
  with `cwd=target/` and today cannot reach the job dir at all. Pass
  `--add-dir <job-dir>` from `claude_code.py` so `NOTES.md` and
  `evidence/` are reachable, and name both paths in the prompt as facts.
  `cwd=target/` itself stays: it is where the work is, and the diff/commit
  recording (kept item 7) is defined against it.
- Corollary: the mediator's own `NOTES.md` under `.local/agent/` was already
  agent-authored — only its first line was contracted (item 2).

### 2. The mission-complete verdict → the mediator

- Delete the `STATUS:` first-line contract: `drive.sh:18-27`,
  `gateway.py notes_status()` and `notes_are_stale()` (duplicated in both).
- Replace with the agforge item-1 shape: **the agent's own act, unparsed.**
  The mediator writes `.local/agent/done` when it considers the mission
  finished or blocked; `drive.sh` loops until that file exists or the session
  budget runs out. Existence is checked, content never is — whatever the
  agent puts in the file is carried into `GET /status` verbatim.
- `POST /mission` removes a stale `done` when it writes a new MISSION.md,
  which retires the mtime-staleness heuristic entirely.
- `max_sessions` (kept item 5) remains the only other way the loop ends.
- The monitor and `/status` report the file's content as-is; there is no
  enum to match against.

### 3. The plan/approval transition → the reviewer

- Delete the two-file contract as a *transition condition*
  (`run_once.py:365-372`): the phase no longer changes because a parser
  liked two files. A plan-phase iteration ends in `awaiting_approval` —
  one iteration, one review opportunity.
- `load_proposed_gates` survives only as a **convenience for the reviewer**,
  not as a verdict: `status --json` reports what it could read from
  `target/proposed_gates.yaml` and says so plainly when it could read
  nothing.
- `autolab approve <job-dir>` uses that file when it parses, and otherwise
  accepts `--gates <file>` or repeated `--gate <cmd>`. The mediator decides
  what the gates are; the mediator is an agent, and this is its judgment,
  not the machine's.
- `reject --feedback` keeps appending to NOTES.md but loses the fixed heading
  that the next prompt matches on (`review.py:110-118`, and `fake.py`'s
  `_REJECT_MARKER`). Feedback is prose in the handoff, like everything else.
- Phase selection from `bool(job.gates)` (`run_once.py:455-465`) stays: it is
  the operator's choice of style expressed in the job file, made before any
  agent runs, not a judgment about an agent's output.

### 4. The convergence verdict → observation only

- Delete `_made_progress` (`run_once.py:225-232`), `consecutive_no_progress`
  from `state.py`, and `no_progress_limit` from `job.py` and the guides.
- `stuck` survives with one meaning only: `max_iterations` reached — a spend
  guard (kept item 5), not an opinion about progress.
- `converged` stays as a restatement of the observation ("every gate exited
  0"), which is a tautology, not a judgment. The prompt still tells the agent
  which gates failed; nothing tells it whether it is making progress.
- `run-once`'s stdout line reports gate counts and stops characterising the
  run.

### 5. The coding agent may answer in prose

- `claude_code.py:90-106`: stop promoting a JSON parse failure to
  `exit_code = 1`, and stop promoting `is_error`. The process's own exit code
  is the exit code; stdout is the output; the raw stdout is already saved as
  `claude_output.json` either way.
- `META_KEYS` deleted — the whole result JSON goes into
  `adapter_result.json`. This *increases* what kept item 6 records.
- `fake.py:29-58`: the prompt-substring phase sniffing is replaced by explicit
  `adapter_config`. The fake adapter is a test fixture rather than an agent,
  so this is tidiness, not unshackling — but a component that decides by
  grepping a prompt is exactly the shape being removed elsewhere.

### 6. gateway: stop reading the agents' words

- `NARRATION` and `tidy_summary` deleted; the summary is served as written.
- `EXTRACT_PY` reduced to "take `result` out of the JSON, write the `.md`,
  write the cost record". The `is_error` / empty-text gate goes: `?force=1`
  already exists to regenerate, and the cache (kept item 5) is a spend guard,
  not a quality guard.
- `devstyle_report()` and its regex deleted. The monitor links NOTES.md
  whole; the ENT value was always in the agent's prose, not in three
  extracted fields.
- `mission_headline()`'s paragraph folding deleted — show the mission text.
- Window and director: JSON-shape enforcement, `is_error` checks and
  empty-text rejection (`gateway.py:834-850, 919-933`) no longer become a
  502. The backend's words are recorded and returned as they arrive. **The
  run record itself stays** (kept item 6).
- `WINDOW_MAX_TEXT = 4000` deleted.
- `DIRECTOR_PROMPT_PREFIX` deleted; the existence of `GUIDE.md` becomes one
  line of fact in the director's prompt, not an imperative prefix.

### 7. Prompts become facts

Every "must / never / exactly / do not" leaves the four prompts. What each
keeps is the material and the paths.

- `build_plan_prompt` (`run_once.py:85-122`): the goal, plus the facts that a
  reviewer will read the result and approve or reject it, that
  `autolab approve` reads `target/proposed_gates.yaml` when it is there, and
  that approved gates become the acceptance condition. The five gate
  requirements, the YAML skeleton and "do not implement yet" all go.
- `build_implement_prompt`: the goal, the approved plan, the gate commands,
  which ones failed last time, the handoff. "without weakening or deleting
  the gates themselves" goes — the gates run every iteration and the result
  is recorded, which is the same information as a fact.
- `SUMMARY_PROMPT`: the directory to read and who reads the output. The
  format commands (5–10 sentences, no headings, no bullets, no preamble, no
  dumps) go.
- `WINDOW_PROMPT`: what this node is, the guide, the job state, and
  `POST /mission` + bearer as the door for work. The six rules go, including
  "never invent a number" (agdevworld deleted the same sentence with no
  measured cost).

### 8. Guides cut to paths, commands and one-line facts

Target shape, per memo.md: a bullet list of paths and commands with one line
each; prohibitions and policy deleted except the safety devices.

| file | now | target |
|---|---|---|
| `agent/CHARTER.md` | 66 | ~25 — the mission's location, the paths, the tools, `styles/`, and a "Safety devices" section naming kept items 1–3 |
| `AGENT_GUIDE.md` | 188 | ~50 — commands table, job dir layout, `job.yaml` keys, evidence filenames, exit codes |
| `agent/README.md` | 157 | ~60 — routes and files, one line each |
| `agent/GUIDE.md` | 60 | ~45 — it is the entrance guide, so the capability/cost figures stay (policy.md requires the Q&A); "What it will not do" goes |
| `styles/README.md` + 2× `STYLE.md` | 12 + ~60 | one file, ~15 — two styles, one line of when-it-fits each |

Deleted specifically: the fixed session-opening sequence (CHARTER.md:9-29),
"choose before opening either STYLE.md" and "read only the chosen folder",
the plan-review checklist (AGENT_GUIDE.md:151-167), "Lessons from previous
runs", "keep `goal` near-verbatim", "seed nothing in `target/`", and every
`skip_permissions` prohibition outside the Safety devices section.

The measured cost figures in `agent/GUIDE.md` are *facts*, not policy, and
stay — they are what the entrance answers cost questions from.

**`POST /director` is not in scope for removal.** It is a needed endpoint,
half-implemented. Today it appears only in `gateway.py`'s module docstring and
is absent from `agent/README.md`'s route list; the rewrite adds a one-line
entry for it, so that a route missing from the guide is never mistaken for a
route this turn deleted. What this turn does touch there is listed in items 6
(the unconditional `DIRECTOR_PROMPT_PREFIX`, and output-shape enforcement) and
9 (its read-only grants and workspace cwd, both unchanged).

### 9. Grants widen with the pointers (agforge F1, in the same commit)

Allowlist form retained everywhere (user decision, C1). Widened:

- `session.sh:34-38` — the mediator's 8 tools + 21 commands. agforge went
  10 → 44 for the same reason. Add at minimum what the newly-named paths
  need, plus `WebFetch`-free basics the charter now points at
  (`.local/jobs/`, evidence, `devdocs/`).
- `claude_code.py` — `--add-dir <job-dir>` by default (item 1's premise).
- `gateway.py` window — today it is given **no allowlist at all**, i.e. no
  tools, which is capability removed structurally rather than by
  instruction. Grant `Read,Glob,Grep` on the claude backend so it can read
  `GUIDE.md` and job state itself. (The ollama backend has no tool channel;
  that is a backend property, not a rail.)
- Summarizer and director keep `Read,Glob,Grep` — read-only is their whole
  function, and nothing in this turn asks them to act.

Denials observed during the live check are findings, not misbehavior, and are
widened in-turn.

### 10. Tests

Rewrite so that **nothing asserts an agent said the right thing.** What tests
keep covering: the lock, the timeouts, evidence being written, exit codes,
the token on `POST /mission`, and the read routes not taking the lock.
Delete: `test_plan_flow.py`'s schema assertions, the NOTES.md template
assertions in `test_run_once.py`, the JSON-promotion assertions in
`test_claude_code_adapter.py`, and the window/summary shape assertions.
`uv run pytest -q` green before the live check.

## Live check

Re-run what already worked, on the same node, and record cost/turns/duration
for every run. Prior evidence for comparison: the six jobs under
`.local/jobs/` (`binary-cli`, `fizzbuzz`, `roman-numeral`, `smoke-fizz`,
`snake-web`, `snake-web-b`).

| # | what | why |
|---|---|---|
| 1 | `fake` adapter, plan → approve → implement, one job | the whole state machine at $0 |
| 2 | `fake` adapter, plan → reject → replan | item 3's feedback path without the marker |
| 3 | the `binary-cli` mission verbatim via `POST /mission`, end to end | the instant-ramen route, against a known-good prior result |
| 4 | a plan-phase mission (snake-web class) through the mediator's review | the slow-brew route: does the mediator still review and approve without the checklist? |
| 5 | window ×4: what is running / how did `<job>` end / what has this node spent / what can you do | the four question classes the entrance answered before |
| 6 | one `POST /jobs/<job>/summarize/<iter>` | prose quality with every format command deleted |
| 7 | safety devices, directly: unauthenticated `POST /mission` → 401; a second `run-once` while one holds the lock; an iteration timeout; no secret in any pushed tree | kept items 1–5 still hold |

The claude-backend contrast run for the window (item 6 territory) is cheap
here — the binary already exists on this node, unlike agdevworld's F7.

## Success criteria

Not "everything still passes". The turn succeeds if:

- every capability in the live-check table either still works, or fails in a
  way that is recorded with its cause;
- no code in `src/agautolab/` or `agent/gateway.py` reads an agent's words to
  decide what they meant, outside the seven kept items;
- the guides are down to paths, commands and facts;
- the report can say what each change cost in dollars and seconds.

Regressions are the product. Where one appears, the ENT-ordered response is
wording first, code only after the behavior recurs (agforge F2, agdevworld
F3) — and the report says which of the two it chose and why.

## Obligations

- Leave a Deus Ex Machina note in the report for any work done on an
  in-system agent's behalf (policy.md).
- Report goes to `report.md` beside this file, in the shape of the two prior
  turns: what was done, a live-run table, numbered findings, state after.

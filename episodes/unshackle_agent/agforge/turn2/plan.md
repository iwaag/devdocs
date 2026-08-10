# unshackle_agent / agforge turn2 — settle turn1's findings, dissolve the last contract

Turn1 removed every forced path in the agforge section of
[shackle_list.md](../../../../../shackle_list.md) except five, and the
capability survived: six of seven live deliveries were correct
([turn1/report.md](../turn1/report.md)). Turn2 is not another removal
sweep — there is almost nothing left to remove. It does three things:

1. **Fix the one defect turn1 caused** (F2) with wording, then measure
   whether the fix held.
2. **Dissolve the one contract turn1 deliberately kept** (`What stays`
   item 1), because the reason it was kept expired when agdevworld's turn
   landed.
3. **Turn single-occurrence findings into rates.** Turn1 ran each desire
   once or twice; F2 was "1 of 2", F5 and F6 were one observation each.
   None of them can be called intermittent, deterministic or settled at
   that sample size. Repetition is the main instrument of this turn.

Direction: [../../../../../memo.md](../../../../../memo.md). This remains
an experiment. A regression that appears is a finding to record, not a
defect to patch in the same turn — with the single exception of work
item 1, which turn1 explicitly ordered as this turn's first move.

## What stays

Turn1's list of five, minus item 1 (see work item 2), plus nothing new:

1. ~~The outward HTTP contract shape.~~ **Dissolves this turn** — work
   item 2.
2. **The wall-clock kill** (`agent_run.py:56, 232`). Resource guard.
3. **Transcript capture.** The evidence this turn is measured with.
4. **`generate.py`'s `nctl-outbox` refusal** — a property of the tool,
   not a sentence in a charter.
5. **No `--dangerously-skip-permissions` / `opencode run --auto`.** These
   agents run natively on the developer's Mac. Unchanged; see work item 5
   for what happens instead.

## Work items

### 1. The result file is the answer, not a progress log (F2)

Turn1's run 3 wrote `{"status": "working", "artifacts": []}` as its first
act, delivered a perfect 300×300 PNG in prose, and never rewrote the
file. The caller polls `working` forever. Traceable to charter wording:
it explains that agdevworld "keeps polling while that is `working`",
which reads as an invitation to write that value as an initial state, and
nothing says the file is the final answer.

Fix, in ENT order — **wording only, no code, one line**: the charter says
the file is what the caller receives when the run is over, written once
the answer is known. Do not add a schema check, a status whitelist, or a
runner-side rewrite. Work item 2 removes the sentence that caused this in
the same edit; both changes are charter text.

Turn1's second candidate (runner-side "the process exited, so `working`
is stale") stays **unbuilt this turn**. Decision rule fixed in advance,
so the result is not argued after the fact: if the placeholder still
appears in **2 or more of the 5** repeated runs in work item 6, it is
recurrent behaviour and the runner-side fact becomes turn3's item; at 1
or 0 it stays wording. Record the count either way.

### 2. Dissolve the result.json shape (turn1 "What stays" item 1)

Turn1 kept a fixed wire shape for exactly one reason, stated in its plan:
*"today's caller is an `if` statement… the day agdevworld's side can read
prose, this contract dissolves too."*

That day arrived in the same 24 hours. agdevworld's turn1 deleted the
POST-poll-pick pipeline: `src/chatPanel.ts` now hands the assistant whole
HTTP responses — status, content-type, body — and the assistant paces its
own polling and calls `show_image` itself
([agdevworld/turn1/report.md](../../agdevworld/turn1/report.md), item 5).
The caller is an agent, not an `if`.

So delete from `charter.md` the paragraph enumerating what agdevworld
reads (`status` / `working` / `done` / `artifacts[].url`). What replaces
it is one line of fact: this file is served to the caller as-is, and the
caller is an agent that reads it whole.

Two consequences to watch rather than prevent:

- F5 (text answers have no agreed key) stops being a defect and becomes
  the expected state. Record the shapes the agent invents across the
  repeats; do not standardise them.
- The delivery of an *image* now depends on the reading agent
  recognising a URL in whatever shape the writing agent chose. That is
  the actual experiment of this item.

**Verification is a joint live run**, agdevworld → agforge → back, on the
real assistant (work item 7). If delivery breaks there, that is this
turn's headline finding; the fix is wording on one side or the other and
belongs to turn3, not to a restored contract. Rollback is one paragraph
of charter text, no restart.

Editing agdevworld's code is **out of scope**. Driving it is not.

### 3. Measure whether the markers are still load-bearing

`scan_markers` (`agent_run.py:332-352`) survives as the lenient
alternative to the result file. Across turn1's eight runs it was never
the path taken — every delivery came from a result file. Nothing is
removed on that evidence alone. This turn: count it. If the marker path
carries **zero** of the ~12 runs here as well, turn3 has a
20-line deletion with two turns of evidence behind it.

### 4. Does a denied command still kill the run silently? (F1's residue)

Turn1's F1 died at 23 s with `permission requested: read (.local/.env);
auto-rejecting` and **no agent output at all** — no final message, no
problem report, nothing for the operator except a stderr tail. The
grants were widened, so the specific denial is gone; the failure *mode*
is not.

This item is observation, not code. `opencode.json` still denies bash by
default with an allowlist, so the mode is reachable. Provoke it once
deliberately — a desire whose obvious route needs a command outside the
allowlist (e.g. one that invites `tar`, `zip`, `brew`, `ssh` or `open`) —
and record precisely: does the run die with no output, or does the agent
receive the refusal as a tool error and route around it?

The answer decides a real question for all three projects: **whether a
refusal can carry itself** (agdevworld F4, agautolab F2) depends on the
refusal actually reaching the agent. If auto-reject kills the process
instead, then every "the tool declines and the agent explains why"
argument in this episode has a hole in it, and naming that hole is worth
more than another widening. Do not widen the allowlist to make the
symptom go away; if the agent asks for something reasonable, that is
turn3's widening, recorded here.

### 5. Grants: widen only on evidence

No denial was observed after turn1's mid-turn fix, so there is nothing
concrete to widen. Leave `opencode.json` and `CLAUDE_ALLOWED_TOOLS`
alone unless a run is actually blocked. Every denial that appears goes in
the report by exact command name (turn1's corollary: a denial is a
finding, not misbehaviour).

### 6. Repetition battery — the instrument of this turn

All on the ollama default (`qwen3.6:35b-a3b-coding-nvfp4` via opencode),
service restarted on the new charter, run through HTTP:

| # | desire | n | what it measures |
|---|---|---|---|
| A | a 300x300 png icon of a paper crane | 5 | F2 placeholder rate; the generate → convert → upload chain |
| B | a red lighthouse at dusk | 2 | plain path still works; F6 URL fidelity |
| C | compose a 30 second piano melody | 2 | honest refusal without the prohibition; problem-report rate |
| D | what can you do? / how much does one image cost? | 1 each | Entrance Guide via the agent; cost of a question |
| E | a desire needing a denied command (work item 4) | 1 | failure mode of auto-reject |
| F | A and C on the `claude` backend | 1 each | Agent ≠ Model contrast; real money |

~13 runs, ~10 minutes of wall clock on ollama at 25–45 s each, plus
roughly $0.20–0.40 of claude. For every run record: elapsed, turns,
cost, the outcome shape the agent chose, whether the URL resolves (fetch
it — the runner no longer does), and whether a problem report was left.

Counts to carry into the report, each with its denominator:

- placeholder-`working` left behind: n/5 on A (work item 1's decision rule)
- corrupted or unusable URL: n/12 (F6's hypothesis — turn1 saw 0/6, ex2
  saw 2 in far fewer runs)
- marker path used instead of a result file: n/12 (work item 3)
- distinct result shapes invented for a text answer (F5)
- problem reports written unprompted (C, E)

### 7. Joint live run with agdevworld

Two requests placed through the real assistant, not through agforge's
HTTP surface: one image desire and one that fails (music). Both
containers are already running on agdevworld's turn1 code. Record
whether the image reaches the panel, how many rounds the assistant
spent, and — the point of work item 2 — whether it coped with whatever
shape agforge's agent wrote that day.

### 8. Tests

`uv run pytest -q` stays green (33 at the end of turn1). Work items 1
and 2 are charter text and should touch no test. If anything asserts the
enumerated caller keys, delete the assertion rather than the change.
Still nothing that asserts the agent said the right thing.

### 9. Report

`turn2/report.md`, same shape as turn1: what was done, the run table
with every repeat listed individually, the rate table above with
denominators, findings, Deus Ex Machina note, state after the turn.

Explicitly answer the three questions this turn exists for:

1. Did the wording fix stop F2, at what rate?
2. Did dissolving the contract cost a delivery, and if so on which side?
3. Does a refusal reach the agent, or kill it?

## Facts worth not rediscovering

- The charter is re-read per request; wording edits need no restart.
  `agent_run.py` and the service do need one.
- Service: `service/serve.sh` on :8092, hand-started, jobs in memory, log
  at `.local/out/service.log`. A restart drops turn1's stuck `working`
  job — capture anything wanted from it first.
- Nothing from turn1 was committed; the working tree holds three turns'
  worth of edits across the three projects.
- ollama baseline after turn1: 23–41 s, $0. claude: ~31 s, $0.18 for the
  full generate → convert → verify chain.
- SwarmUI emits JPEG regardless of what was asked; `--width 300 --height
  300` is accepted despite the multiple-of-64 rule turn1 deleted. `sips`
  and Pillow are both available.
- `AGFORGE_CLAUDE_CMD` in `.local/.env` is an absolute VS Code extension
  path that goes stale on every update; the failure looks like infra.
  agautolab's F4/F6 is the same class in three other places — if it
  bites here, resolve the glob rather than editing the path by hand.
- turn1's F1 lesson, still the standing rule for this episode: **a
  charter that points at a new place needs the grants widened in the
  same commit.**

## Out of scope

- Editing agdevworld or agautolab (their own turns; driving agdevworld
  for work item 7 is allowed).
- Deleting `scan_markers` — measured this turn, decided next.
- The runner-side terminal-status rewrite (work item 1's decision rule
  gates it into turn3 at the earliest).
- Music/video/multi-image, a persistent job store, moving the runtime
  into a sandbox, and any further widening of the tool grants without a
  recorded denial.

# unshackle_agent / agforge turn3 — report

Executed on 2026-08-10 (agstudio) straight from
[turn2/report.md](../turn2/report.md)'s candidate list, at the developer's
request, with no separate plan. Items 1–4 were done; item 5 (carrying
turn2's F3 distinction into agautolab and agdevworld) belongs to their
own turns and was not touched.

Nine live runs: 7 on ollama, 1 through the real agdevworld assistant, 1
on claude. Turn spend: **$0.19**.

**Headline: the wrong fact was the whole cause.** Correcting one line of
the charter took the repeated case from 3 of 5 deliveries to **5 of 5**,
and the two dead-end detours that turn2 measured (8 and 10 attempts at a
broken endpoint) collapsed to at most 2. Deleting the marker scan and the
`artifacts` default — 0 of 23 and 0 of 13 in live evidence — changed
nothing observable, which is what evidence-backed deletion is supposed to
look like. The one new miss (a capability question left nothing behind)
was **recovered by the reader**, and that recovery is this turn's most
interesting result.

## What was done

| Item | Change |
|---|---|
| 1. The wrong fact (turn2 F2) | `charter.md`: the `POST /API/ListModels` line now states SwarmUI's session handshake — `POST /API/GetNewSession {}` returns a `session_id`, and the call takes `{"session_id": …, "path": "", "depth": 2}`. Verified against the live SwarmUI before writing it. Same correction in `README_DEV.md`. |
| 2. Delete the marker scan (turn2 F6) | `scan_markers` and its charter sentence gone. `resolve_outcome` is now: the agent's file, or the fact that there was none. `agent_run.py` 440 → 411 lines. |
| 3. Delete the `artifacts` default (turn2 F1) | Gone from both fill-in sites, and from the in-progress body in `request_service.py` (`{"status": "working"}` alone). The runner now authors exactly one field, `status`, and only when the agent omitted it. |
| 4. Problem inbox (turn2 F5) | Left as written; re-counted this turn. |
| Tests | Rewritten off the markers: 33 → **31 passed**. The stub gained `FAKE_AGENT_RESULT` so the end-to-end tests answer through the file, like the real agent does. |

## Live runs

ollama default (`qwen3.6:35b-a3b-coding-nvfp4`), service restarted on the
new code.

| # | desire | elapsed | turns | outcome_from | `ListModels` attempts | delivered? |
|---|---|---|---|---|---|---|
| A1 | 300x300 png paper crane | 36.7 s | 7 | result_file | 2 | ✓ 200 image/png 47 KB |
| A2 | ″ | 128.0 s | 8 | result_file | 2 | ✓ 200 image/png 58 KB |
| A3 | ″ | 42.7 s | 8 | result_file | 0 | ✓ 200 image/png 49 KB |
| A4 | ″ | 39.7 s | 6 | result_file | 0 | ✓ 200 image/png 98 KB |
| A5 | ″ | 39.7 s | 6 | result_file | 0 | ✓ 200 image/png 93 KB |
| C1 | 30 s piano melody | 39.7 s | 6 | result_file | 0 | ✓ refusal **plus a problem report** |
| D1 | what can you do? | 15.2 s | 2 | **nothing** | 0 | ✗ file; answered in prose only |
| J1 | (joint, via agdevworld) red lighthouse at dusk | 78.9 s / 6 rounds | — | **nothing** | 0 | **✓ recovered by the reader** — 200 image/jpeg 248 KB |
| G1 | 300x300 png paper crane, **claude** | 30.8 s | 8 | result_file | 0 | ✓ 200 image/png 55 KB, $0.186 |

A2's 128 s is generator latency, not agent behaviour: its transcript
shows no detours, one `generate.sh` and one `transform.py`, both of which
simply took long.

## The counts against turn2

| measure | turn2 | turn3 |
|---|---|---|
| repeated case delivered a result file | 3 / 5 | **5 / 5** |
| worst `ListModels` detour in the case | 10 attempts | **2** |
| runs leaving nothing for the caller | 2 / 13 | 1 / 8 (a question, not an image) |
| marker path used | 0 / 15 | *(deleted)* |
| corrupted or unusable URL | 0 / 9 | **0 / 7** — 0 / 22 across three turns |
| `artifacts` populated by an agent | 0 / 13 | *(deleted)* |
| problem reports written unprompted | 1 / 4 refusals | **1 / 1** |
| distinct top-level key carrying the URL | 6 | 5 more shapes; `output.url`, `result.url`, `answer.url`, bare `url`, `url` + `source` |

## Findings

### F1 — One wrong fact was worth two failed deliveries

turn2 measured a perfect correlation between "attempts at the broken
`ListModels` endpoint" and "forgot to write the answer" (8 and 10
attempts → both misses; 2 attempts → all three deliveries). This turn
corrected the fact and changed nothing else about the case. The case went
5 / 5, and no run tried the endpoint more than twice.

That is as close to a controlled result as this episode gets, and it
settles what turn2 could only argue: the failure was **information, not
capability, and not the model**. The fix that the symptom invited — a
code-side guard that rewrites or demands the result file — would have
addressed a cause that did not exist.

Standing lesson, now in its third form: *a charter that names a door
needs the door to exist (turn1 F1), to be openable in the time given
(agdevworld F1), and to work the way the line says (this turn)*. A stated
fact is a promise the agent will spend real turns on.

### F2 — Prose is a recoverable channel now, because the reader is an agent

The joint run is the one to keep. agforge's agent generated the image,
decided the first attempt was not red enough, regenerated, announced the
URL in prose — and never wrote the file. What agdevworld received was the
runner's honest fact:

```json
{"status": "ended", "detail": "the run ended and left nothing for the caller
 (…/result.json absent); the agent's last words: … Let me regenerate with the
 color s…"}
```

The assistant read the URL **out of that prose**, called `show_image`, and
the picture is in the panel (200, image/jpeg, 248 KB). Six rounds, 78.9 s,
$0.

Under turn1's contract this was a hard failure twice over: not `done`, no
`artifacts`. Under turn2's dissolved contract with a machine reader it
would still have been a failure. It succeeded because **both ends are
agents**, and because the runner's one remaining job — state the fact and
carry the agent's own words — turns out to be exactly the right handoff
for a reader that can read.

This also reframes the "nothing for the caller" case. It is not a lost
delivery; it is a delivery in a lower-fidelity channel. That is worth
saying plainly because the obvious next move — making the result file
mandatory again — would trade a working recovery for a rule.

### F3 — The miss moved from images to questions

D1 ("what can you do?") answered accurately in prose in 2 turns and 15 s,
and wrote no file. In turn2 both question runs wrote one. One occurrence,
**not patched**, following the standing rule of this episode.

The plausible mechanism is worth recording for whoever repeats it: the
charter says the file "is the answer the caller receives once the run is
over", and a question *feels* answered once it has been answered in
words. An image leaves an artifact that has to go somewhere; a sentence
does not. If this recurs, the cheap probe is five question-shaped desires
in a row, and the cheap fix is wording, not code.

### F4 — Deleting on evidence changed nothing, exactly as the evidence said

`scan_markers` (0 of 23 runs) and the `artifacts` default (0 of 13) came
out with no behavioural difference in 9 runs across two backends and one
cross-project loop. `agent_run.py` lost 29 lines, the tests lost four
marker cases and gained two better ones, and the runner now authors a
single field in a single circumstance: `status`, when the agent left none,
meaning the run is over.

Worth noting what *kind* of deletion this was. Both were kept in turn1 as
"a second natural way to answer" and "an empty container, not a claim" —
reasonable-sounding, unfalsifiable, and wrong. Counting for two turns is
what made them removable without an argument.

### F5 — The problem inbox is alive when the agent has something to say

turn2 saw the ollama refusals put their reports *in the result file* and
leave the inbox empty. This turn C1 did both: a plain refusal in the
answer, and `.local/problems/744d63ce….json` with `what_i_tried` /
`what_happened` — and a `written` key in the result pointing the operator
at it. 1 of 1.

Two turns, three shapes (`.md`, `.json`, in-result prose), no path rule,
no template. The inbox line in the charter is earning itself; nothing to
change.

### F6 — Cost and latency, three turns in

ollama: 15.2–128.0 s this turn (median 39.7), $0. claude: 30.8 s, 8
turns, $0.186 — indistinguishable from turn2's $0.229 / $0.172 and from
the ex3 baseline. Nothing in three turns of deletion has cost measurable
time or money; the only latency mover has been the generator itself.

## Deus Ex Machina note

Corrected the `API/ListModels` line in `charter.md` and `README_DEV.md`
on the request agent's behalf — it could not have fixed its own briefing,
and the transcripts show two runs paying for the error. Handoff
candidate: an agent that discovers a stated fact is wrong currently has
only the problem inbox to say so, and nothing reads that inbox
automatically.

## State after this turn

- `service/charter.md` 47 lines (unchanged in length: the marker
  paragraph left, the session handshake arrived), `service/agent_run.py`
  440 → 411, `request_service.py` and `README_DEV.md` updated, tests
  33 → 31 passing.
- The :8092 service was restarted on the new code (`service/serve.sh`,
  log now at the scratchpad path used this session as well as
  `.local/out/service.log`); turn1's and turn2's in-memory jobs are gone
  with the old process, as expected.
- The claude service on :8093 was started for G1 and stopped by explicit
  PID; :8092 untouched.
- One new problem report, `744d63ce….json`.
- agdevworld and agautolab were not modified; agdevworld was only driven.
- Nothing committed; the working tree holds three turns of edits.

## Next turn candidates

1. **F3**: five question-shaped desires in a row. If the miss is real, one
   line of charter wording; if it was a one-off, the count says so.
2. **F2 deserves a deliberate test**: ask agforge for something where the
   prose channel is the *only* channel (a long explanation, a comparison)
   and see whether the assistant relays it as well as it relayed a URL.
3. Carry turn2's F3 distinction — tool-level denials kill a run silently,
   bash-pattern denials arrive as readable errors — into agautolab and
   agdevworld, whose grants include the first kind.
4. Nothing in agforge's code is now a candidate for deletion. The
   remaining five kept items are the wall-clock kill, transcript capture,
   `generate.py`'s cross-project refusal, the no-skip-permissions rule,
   and `status`-when-absent. The first four are irreversible-harm or
   observation; the fifth is one line. **agforge is done unshackling** —
   further turns are measurement, not removal.

# unshackle_agent / agforge turn2 — report

Executed [plan.md](plan.md) in full on 2026-08-10 (agstudio). Two lines of
charter text changed; no code changed. 15 live agforge runs (13 ollama, 2
claude), 2 runs driven end-to-end through the real agdevworld assistant,
and 2 opencode probes outside the service.

**Headline: the two questions this turn existed for both came back
positive, and the contract that dissolved turned out to have been
*costing* a delivery rather than protecting one.** F2's placeholder never
recurred (0/5). A shape that the deleted `if`-statement contract would
have rejected was delivered correctly by agdevworld's agent-reader. One
new failure mode appeared in its place — 2 of 5 runs delivered a perfect
image and never wrote the answer file — and it has a concrete, traceable
cause that is neither the design nor the model (F2 below).

Turn spend: **$0.40** (two claude runs); everything else $0.

## What was done

| Work item | Result |
|---|---|
| 1. Result file is the answer (F2 fix) | Charter now says the file "is the answer the caller receives once the run is over, written when you know the answer". Wording only. |
| 2. Dissolve the result.json shape | The paragraph enumerating what agdevworld reads (`status`/`working`/`done`/`artifacts[].url`) is deleted; replaced by "the caller is an agent; it reads the whole file, whatever is in it". agdevworld untouched. |
| 3. Marker usage | Counted: **0 of 15**. |
| 4. Does a denial reach the agent? | Answered directly, by probe and by a live run. **Yes** for bash-pattern denials. See F3. |
| 5. Grants | Unchanged. One denial observed (`zip`), and the agent routed around it unaided — no widening earned. |
| 6. Repetition battery | 12 runs on ollama + 2 on claude, below. |
| 7. Joint run with agdevworld | 2 runs through the real assistant on the real containers. |
| 8. Tests | `uv run pytest -q` → **33 passed**, untouched. |

## Live runs — ollama (`qwen3.6:35b-a3b-coding-nvfp4` via opencode)

| # | desire | elapsed | turns | outcome_from | status the agent chose | delivered? |
|---|---|---|---|---|---|---|
| A1 | 300x300 png paper crane | 61.0 s | 11 | **nothing** | — (runner said so) | ✗ file; image fine, URL in prose, 200 image/png 95 KB |
| A2 | ″ | 48.9 s | 9 | result_file | `complete`, url at top level | ✓ 200 image/png 103 KB |
| A3 | ″ | 42.8 s | 8 | result_file | `complete`, url nested under `result` | ✓ 200 image/png 76 KB |
| A4 | ″ | 36.7 s | 10 | **nothing** | — | ✗ file; image generated + converted, no URL reached the caller |
| A5 | ″ | 67.1 s | 12 | result_file | `ended` + `url` + `description` | ✓ 200 image/png 93 KB |
| B1 | red lighthouse at dusk | 42.8 s | 8 | result_file | `complete` + `output_url` | ✓ 200 image/jpeg 83 KB |
| B2 | ″ | 45.8 s | 8 | result_file | `ended` + `image_url` | ✓ 200 image/jpeg 191 KB |
| C1 | 30 s piano melody | 24.4 s | 3 | result_file | `ended` + `answer` | ✓ honest refusal |
| C2 | ″ | 36.6 s | 3 | result_file | `failed` + `what_worked`/`what_failed`/`next_steps` | ✓ honest refusal |
| D1 | what can you do? | 24.5 s | 3 | result_file | `ended` + `answer` | ✓ from the card |
| D2 | how much does one image cost? | 21.4 s | 4 | result_file | `ended` + `answer` + `source` | ✓ from the card |
| E1 | lighthouse **in a zip archive** | 54.9 s | 12 | result_file | `ended` + `download_url` | ✓ real zip, 200, contains `red_lighthouse_dusk.jpg` |
| G1 | (joint) draw me a red lighthouse at dusk | 57.9 s | 15 | result_file | `ended` + `result_url` | ✓ shown in the panel |

## Live runs — claude (`AGFORGE_AGENT_BACKEND=claude`, second service on :8093)

| # | desire | elapsed | turns | cost | status the agent chose | delivered? |
|---|---|---|---|---|---|---|
| F1 | 300x300 png paper crane | 33.8 s | 10 | $0.229 | `done` + `result_url` + `width`/`height` | ✓ 200 image/png 104 KB |
| F2 | 30 s piano melody | 21.5 s | 6 | $0.172 | `unsupported` + `error` quoting GUIDE.md | ✓ + wrote a problem report |

Latency on ollama: 21.4–67.1 s (median ~43 s), $0 — the upper end is
higher than turn1's 23–41 s, and F2 below explains why. claude is faster
per run than the local default and cost $0.40 for two.

## The counts, with denominators

| measure | result |
|---|---|
| placeholder `working` left in the file (F2 of turn1) | **0 / 5** on the repeated case, 0 / 13 overall |
| run left nothing for the caller | **2 / 13** ollama, 0 / 2 claude |
| marker path (`RESULT_URL:`) used instead of the file | **0 / 15** (turn1: 0 / 8 — now 0 / 23 across two turns) |
| corrupted or unusable URL | **0 / 9** URLs served or announced; all fetched 200 with the right content type (turn1: 0 / 6 — now 0 / 15) |
| distinct top-level key carrying the URL | **6**: `url`, `result.url`, `output_url`, `image_url`, `download_url`, `result_url` |
| distinct `status` values invented | **5**: `complete`, `ended`, `failed`, `unsupported`, `done` |
| `artifacts` populated by an agent | **0 / 13** — every `artifacts: []` in the served bodies is the runner's `setdefault` |
| problem reports written unprompted | 1 / 4 refusal-shaped runs (the claude one) |

## Findings

### F1 — Dissolving the contract saved a delivery instead of costing one

The joint run is the evidence. agforge's agent wrote:

```json
{"request_id": "...", "prompt": "a red lighthouse at dusk",
 "model": "perfectdeliberate_XL.safetensors",
 "result_url": "http://…/92cc8e26….jpg?…", "status": "ended", "artifacts": []}
```

`status` is not `done`; `artifacts` is empty; the URL is under a key no
contract ever named. **Turn1's kept contract would have failed this
delivery twice over** — agdevworld's old `if` statement threw on anything
but `done` and picked `artifacts[0].url`. The agent-reader instead read
the whole body, found `result_url`, called `show_image`, and the image is
in the panel (verified: 200, image/jpeg, 103 KB). 6 rounds, 70 s
end-to-end, $0.

So the two halves of this episode unshackled in a compatible direction
without coordination: agforge stopped writing a fixed shape in the same
week agdevworld stopped requiring one. The generalisation worth keeping:
**a wire contract between two agents is a cost paid for a reader that no
longer exists** — check who is actually reading before defending it.

The second joint run (music) never reached agforge at all: the assistant
read its own card and answered that it cannot compose audio, in 4.7 s and
zero tool calls. Correct, and free.

The residue: `resolve_outcome`'s `setdefault("artifacts", [])` is now the
only shape agforge's code still imposes, and **no agent used the key in 13
runs**. It exists for the reader that dissolved. One-line deletion,
turn3's cheapest item.

### F2 — The placeholder is gone; a different last-step miss took its place, and its cause is a wrong fact in the charter

Turn1's F2 (an agent writing `{"status": "working"}` as a first act and
never returning) **did not recur once in 5 repeats**. The wording fix
holds at the sample size the plan set, and by the decision rule fixed in
advance the runner-side terminal-status rewrite stays unbuilt. It should
stay unbuilt: it would have caught neither of this turn's two misses,
because those runs wrote no file at all.

What did happen: A1 and A4 generated the image, converted it to a 300×300
PNG, announced a working URL in prose (A1's fetched 200 image/png), and
never wrote the result file. The runner reported the fact and carried the
agent's last words, so the caller can see the URL — but no machine reader
would find it.

The cause is not the model, and it correlates perfectly:

| run | mentions of `API/ListModels` in the transcript | wrote the file? |
|---|---|---|
| A1 | 8 | **no** |
| A2 | 2 | yes |
| A3 | 2 | yes |
| A4 | 10 | **no** |
| A5 | 2 | yes |

The charter lists, as a fact, `POST {SwarmUI}/API/ListModels` — the
installed model names. That call does not work as stated:

```
$ curl -X POST http://agpc.local:7801/API/ListModels -d '{}'
{"error_id":"basic_api","error":"missing session id"}
```

It needs a session from `POST /API/GetNewSession` first. A1 and A4 spent
5–8 turns trying content types, headers and `Content-Length: 0` against
that endpoint before giving up and generating the image, and then dropped
the last step. The two runs that took the fact at face value once and
moved on both delivered.

This is **turn1's F1 lesson in its third costume**: a charter that names a
door needs that door to open. Turn1's version was a door with no grant;
agdevworld's F1 was a door with no way to wait; this is a door described
wrong. Not patched this turn (turn1's precedent for recording rather than
fixing mid-turn) — but it is one line, it is turn3's obvious opener, and
the fix is to state the session handshake or to drop the line, not to add
a rule about writing the file.

Worth naming plainly: the failure looks like "the weak agent forgot the
last step", and the fix that framing invites is a code-side guard. The
transcripts say it was a wrong fact consuming the run's attention. **Bad
information looks exactly like a bad agent.**

### F3 — A refusal does reach the agent, and the agent routes around it

The open question from turn1's F1 — whether an in-system agent can ever
"read the refusal and explain it" if a denial kills the process — is
answered, and the answer differs by *kind* of denial:

- **Tool-level permission** (turn1's case: `read` was unspecified in
  `opencode.json`, so the harness asked and headless auto-rejected) →
  the run dies with **no output at all**. That was the failure class.
  Every tool is now explicitly `allow`, so this class is closed by
  configuration, not by luck.
- **bash pattern denial** (`"*": "deny"` plus the allowlist) → arrives as
  a tool error the agent reads. Probed directly outside the service:
  `zip --version` came back with *"The user has specified a rule which
  prevents you from using this specific tool call"*, and the agent
  finished normally, exit 0, explaining that `zip` is not permitted. A
  `cd /tmp && zip --version` prefix does not evade it — the whole command
  string is matched.

E1 is the same mechanism under real work, and it is the best Tool Giving
result of the turn. Asked for the image *in a zip archive*, the agent:

1. generated the image;
2. ran `cd .local/out && zip download.zip …` → **denied**;
3. read the refusal and rebuilt the archive with `python3 -c "import
   zipfile…"`;
4. imported `upload_and_presign` out of `scripts/generate.py` to upload
   and presign the zip — a use of that script nobody wrote a path for;
5. wrote `{"download_url": …}`.

The delivered file is a real zip containing `red_lighthouse_dusk.jpg`
(verified). Nothing in the charter mentions archives. **The denial was
survivable, informative, and the capability arrived anyway** — so this
turn earns no widening of the allowlist, and the "write the reason into
the refusal, not into the prompt" conclusion from agdevworld F4 and
agautolab F2 now has the mechanism check it was missing.

### F4 — Shape variation is total, and it costs nothing where the reader is an agent

With the enumerated keys gone, 13 runs produced 6 different URL key names
and 5 different `status` words, and 5 of 13 wrote no `status` at all. Both
readers coped: agdevworld's assistant found `result_url` unaided, and a
human reading the JSON has more context than the old fixed shape carried
(`model`, `dimensions`, `what_worked`/`what_failed`/`next_steps`).

This is turn1's F5 grown from a note into the normal state. Recorded, not
standardised. The one place it would bite is a *machine* caller — and the
plan's position stands: that caller should state its own requirement, in
its own code, rather than have agforge pre-empt it.

### F5 — The problem-report inbox lost to the result file

Only 1 of 4 refusal-shaped runs (claude's F2) left anything under
`.local/problems/`. Both ollama music runs put the same content *in the
result file instead* — C2's is a full report in JSON keys
(`what_worked`, `what_failed`, `next_steps`).

Nothing was lost: the caller gets more, and the operator gets less. The
information moved to where the charter says the caller looks, which is a
rational reading of a charter that now names both places with one line
each. No change made; worth watching whether the inbox is still earning
its line.

### F6 — Markers are dead, with two turns of evidence

0 of 15 this turn, 0 of 8 in turn1. `scan_markers` and its charter
sentence have never been the path taken since authorship moved to the
agent. Turn1 deliberately kept them as "a second natural way to answer";
two turns say there is no second way in practice. Deleting ~20 lines and
one charter line is now an evidence-backed turn3 item — and the same
sentence is what tells the agent about `RESULT_FAILED:`, so removing it
also removes one more thing to get wrong.

### F7 — The URL guard's absence still costs nothing

9 URLs served or announced this turn, all fetched 200 with the expected
content type; 15 across the two turns, zero corruptions, with no
runner-side verification anywhere. Turn1's hypothesis — that ex2's
corruptions came from *retyping* a presigned URL into a final message, and
that writing it into a file with a tool is a different act — survives a
second turn without a counter-example. Still a hypothesis; the counter is
cheap to keep.

## Deus Ex Machina note

None this turn. The one denial an agent hit (`zip`) it resolved itself;
no environment was fixed on any agent's behalf.

## State after this turn

- `service/charter.md`: 49 → 47 lines. Two lines of the "Where the caller
  looks" section replaced; nothing else in the repository changed.
- `uv run pytest -q` → 33 passed.
- The :8092 service is the same process that served turn1 (charter is
  re-read per request; no restart was needed or done), so turn1's stuck
  `working` job is still in memory alongside this turn's 13.
- The claude-backend service on :8093 was started for F1/F2 and stopped by
  explicit PID — `:8092` was deliberately not touched (agautolab turn1's
  `pkill` lesson).
- 23 job directories under `.local/jobs/`; one new problem report,
  `e648e51c….md`.
- Nothing committed; the working tree holds two turns' worth of edits.

## Next turn candidates

1. **Fix the `API/ListModels` line** (F2) — state the `GetNewSession`
   handshake or delete the line, then repeat case A five times and see
   whether the 2/5 last-step miss goes with it. This is the single
   highest-value line in the file right now.
2. **Delete `scan_markers`** and its charter sentence (F6), 0/23.
3. **Delete `setdefault("artifacts", [])`** (F1), 0/13 — the last shape
   agforge's code imposes.
4. Re-check whether the problem inbox still earns its charter line (F5),
   or whether the result file has simply absorbed it.
5. Carry F3's distinction — *tool-level* denials kill silently, *bash
   pattern* denials are survivable — into agautolab and agdevworld, whose
   allowlists are of the first kind in places.

# unshackle_agent / agforge turn1 — report

Executed [plan.md](plan.md) in full on 2026-08-10 (agstudio). Every forced
deterministic path listed for agforge in
[shackle_list.md](../../../../../shackle_list.md) is gone except the five
the plan kept. Live-checked against real SwarmUI + MinIO + ollama, plus one
claude-backend run for contrast.

**Headline: the capability survived the unshackling.** Six of seven live
deliveries were correct, including the case that used to need three
explicit rules to get right. One new failure class appeared, and it is a
direct consequence of the change — see F2.

## What was done

| Work item | Result |
|---|---|
| 1. Result authorship → the agent | `parse_outcome` gone. The agent writes `.local/jobs/<id>/result.json`; served **unvalidated** (`read_result` → `resolve_outcome`). Markers kept as a lenient alternative. Neither → the runner states the fact and carries the agent's last words. A result file is honored even when the process then dies. |
| 2. URL verification | `verify_result_url` deleted, with its tests. |
| 3. Guide-question regex | `GUIDE_QUESTION`, `GENERATION_VERB`, `is_guide_question` and the `answered` short-circuit deleted. Every desire reaches the agent; the charter names `service/GUIDE.md` as one line of fact. `GET /guide` kept. |
| 4. Tool grants | opencode: `webfetch` allowed, bash map 7 → 40 commands. claude: `CLAUDE_ALLOWED_TOOLS` 10 → 44. Allowlist form kept (user decision). **Widened again mid-turn** — see F1. |
| 5. `charter.md` | 101 → 47 lines. Every "never"/"must" deleted. Now: the request, where the caller looks, six tool lines, the budget fact. |
| 6. Problem-report path | Path rule replaced by `{{PROBLEMS_DIR}}` — the inbox exists; naming is the agent's. |
| 7. `GUIDE.md` / `README_DEV.md` | Both cut to paths + one line each. README's "Hard rules" and the no-S3 walkthrough gone; a short "Safety devices" section names the two survivors and why they differ in kind. |
| 8. Tests | Rewritten. 33 pass, no live services. Nothing asserts the agent said the right thing. |
| 9. Live check | Below. |

Code-side judgment left in `service/`: none. The only value the runner
fills is `status: "ended"` when the agent's file omits it, and that says the
run is over — the one thing the runner knows and the agent cannot.

## Live runs

Service restarted on the new code; ollama backend (`qwen3.6:35b-a3b-coding-nvfp4`
via opencode 1.18.10) unless noted.

| # | desire | elapsed | turns | cost | outcome | correct? |
|---|---|---|---|---|---|---|
| 1 | a red lighthouse at dusk | 23 s | — | $0 | infra failure: `read (.local/.env)` auto-rejected | ✗ (F1) |
| 2 | *same, after widening* | 34 s | 5 | $0 | `done`, result file | ✓ URL 200, image/jpeg, 119 KB |
| 3 | a 300x300 png icon of a paper crane | 35 s | 8 | $0 | `working` left in the result file | ✗ delivery (F2) — artifact perfect |
| 4 | compose a 30 second piano melody | 27 s | 4 | $0 | `failed`, own words, own problem report | ✓ |
| 5 | what can you do? | 41 s | — | $0 | `done` + capability answer in `detail` | ✓ |
| 6 | how much does one image cost? | 25 s | — | $0 | `done` + cost answer in `detail` | ✓ |
| 7 | *#3 on the claude backend* | 31 s | 7 | $0.1846 | `done`, result file | ✓ 300×300 PNG, URL 200 |
| 8 | *#3 repeated, ollama* | 41 s | 7 | $0 | `done`, result file | ✓ 300×300 PNG, URL 200 |

Latency and cost are unchanged against the ex3 baseline (ollama 30–45 s /
$0; claude ~18 s / $0.134 — this turn's claude run was longer and dearer at
7 turns because it did the full generate → convert → verify chain).

## Findings

### F1 — Widening the *information* without the *grants* creates a failure class

Run 1 died at 23 s with `agent produced no output; stderr tail: ! permission
requested: read (.local/.env); auto-rejecting`. The old charter never named
a file to read, so the agent never used opencode's `read` tool; the new one
names `.local/.env`, `service/GUIDE.md` and `params/defaults.toml`, so it
did — and `opencode.json` granted only `edit`, `bash` and `webfetch`.
opencode 1.18.10 also has `read`, `glob`, `grep`, `list`, `task`, `lsp`,
`skill`, `external_directory`, `websearch`, `todowrite`; unspecified ones
auto-reject headlessly, killing the run with no output.

Widened in-turn per the plan's corollary (a denial is a finding, not
misbehavior). **Lesson for the other turns: a charter that points at more
places needs the grants widened in the same commit.** Telling an agent
about a door it cannot open is worse than not telling it.

### F2 — Unvalidated pass-through lets the agent hang the caller

Run 3 is the important one. The agent:

1. wrote `{"status": "working", "artifacts": []}` to the result file **as
   its first act**, before generating anything;
2. generated at exactly 300×300, saw JPEG, converted to PNG with
   `service/transform.py`, uploaded, and announced a **working** presigned
   URL in prose (verified: 200, `image/png`, 65226 bytes, 300×300);
3. never went back and rewrote the file.

The caller polls `working` forever. The artifact was flawless; only the
delivery failed.

Cause is traceable to my own charter wording — it explains that agdevworld
"keeps polling while that is `working`", which reads as an invitation to
write that value as an initial state, and nothing says the file is the
final answer. This was **impossible before**: code authored the status, so
a terminal state was structural.

Recurrence: **1 of 2** on ollama (run 8, same desire, wrote a terminal
result directly and never placed a placeholder). **0 of 1** on claude
(run 7). So it is an intermittent weak-agent slip, not a deterministic
consequence of the design.

Two candidate fixes for turn2, in ENT order:

- **Charter wording first** (recommended): say the file is the answer, not
  a progress log. One line, no code, no judgment. ENT says code hardens
  only after behavior recurs — one slip in two is not yet proof.
- **Runner-side terminal fact** (only if it recurs): when the process has
  exited, nothing more will be written, so `working` is factually stale.
  Rewriting it would be the same class as filling an absent `status`.
  Flagged as risky: it is exactly the shape through which code-side
  judgment would creep back, and it should not be taken without noticing
  that.

### F3 — Deleted prohibitions were replaced by a pointer, at no cost

- "Today's only capability is one still image" was deleted from the
  charter. Run 4 refused the music request anyway and **cited GUIDE.md** in
  its own words, then wrote a problem report. The prohibition was carrying
  information that a pointer carries just as well.
- The 64–2048 bound and the multiple-of-64 rule were deleted. Runs 3, 7
  and 8 all passed `--width 300 --height 300` straight to SwarmUI, which
  **accepted it**, and then converted to PNG. The rule we had been
  maintaining was not load-bearing for this model and version. Both
  backends independently found the generate → transform chain without
  being told the sequence.

### F4 — The loosened problem-report path works

Run 4 wrote `.local/problems/11caeb0ed0b045ae815e2485a367204b.md` — flat,
named by the agent, not the old `<UTC stamp>-<id[:8]>/problem.md`. Content
covers what was asked, what was tried, and why not, without the old
template telling it to. Nothing depended on the path rule.

### F5 — Text answers no longer have an agreed key

With `answered` + `reply` gone, runs 5 and 6 invented `status: "done"` with
the prose in `detail`, and run 5 also echoed `request_id` back. Unvalidated
pass-through means callers get whatever shape the agent picked. Fine for
agforge; a caller that wants a text reply must tolerate variation. Note for
agdevworld's turn — its chat path currently reads only `artifacts`.

### F6 — The class the URL guard existed for did not appear (hypothesis)

Six successful deliveries, zero corrupted URLs, no runner verification. ex2
saw two corruptions in far fewer runs. Plausible mechanism: the corruption
happened while *retyping* a high-entropy URL into a final message, and
writing it into a JSON file with a tool is a different act. If so, moving
authorship did not just remove the guard — it removed the need for it.
Sample is small; treat as a hypothesis and keep counting.

### F7 — What the entrance guide now costs

Questions run the agent: 25–41 s, $0 on ollama, ~$0.18 on claude. Both
answers were accurate and sourced from the card. The regex was protecting
against a cost that is real but, on the default backend, zero.

## Deus Ex Machina note

Diagnosed the opencode read-permission denial and widened `opencode.json`
on the agforge request agent's behalf — handoff candidate.

## State after this turn

- Service running on the ollama default, new code, jobs in memory.
- `uv run pytest -q` → 33 passed.
- Run 3's job is still `working` in memory and always will be; it dies with
  the next restart. Left as-is deliberately, as the evidence for F2.
- Nothing was committed; the working tree holds all edits.

## Next turn candidates

1. F2's charter-wording fix, then re-run the 300×300 case several times to
   see whether the placeholder recurs.
2. Carry F1's lesson into agautolab and agdevworld before their charters
   grow new pointers.
3. Widen further where the agent is still blocked — no denial was observed
   after the F1 fix, so there is nothing concrete to widen yet.
4. Revisit whether the `.local/jobs/<id>/result.json` shape itself should
   dissolve, once agdevworld's side can read prose (plan.md, "What stays"
   item 1).

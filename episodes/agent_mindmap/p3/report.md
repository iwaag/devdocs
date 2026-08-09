# agent_mindmap — Phase 3 report: director as a creative persona

AI-generated (Omni Agent, 2026-08-09). Phase 3 of `roadmap.md`. Status:
**done**, with one criterion honestly short of its wording (no live
`deliver`: three real delivery flows ran end to end and the director
refused every image it was shown). Plan: `plan.md`.

## Acceptance, against the roadmap's own words

| criterion | result |
|---|---|
| a casual creative question gets an in-character, useful answer | **met** — live, through `POST /window` |
| a practical verify request gets one, through the same window | **met** — live, with a machine-readable `VERDICT:` line |
| a delivery flow (compose → agforge → review) still completes end to end | **met for the flow, not for a delivery** — three full runs, 3/5/3 attempts, each ending in a reasoned director decision with evidence on disk; none ended in `deliver`, because the director rejected all eleven generated images |
| runs are recorded per the Phase 1 policy | **met** — 34 runs, 100% recorded, 0 unrecorded failures |
| backend is switchable per Agent ≠ Model | **met** — the same question answered by `claude/claude-sonnet-5` (0.089 USD) and `ollama/gemma3:latest` (0.00 USD, tokens only), both recorded |

The `deliver` branch (copy into the game, flip one manifest entry) is
covered by unit tests but has not run against a real image. That gap is not
a bug I chose to leave — it is the phase's main finding, below.

## What exists now that did not

- **A director with one entrance.** `answer(text)` — free text in, free text
  out — reachable over HTTP (`POST /window`), CLI, and in-process from
  `reconcile.py`. Three transports, one place to express a desire. Machine
  callers get structure from a trailing `VERDICT: pass|fail — reason` line,
  parsed leniently; its absence is never a failure. `direction` / `manifest`
  are addressing, not desire.
- **A persona that lives in files.** Everything the director knows is the
  markdown in the direction workspace (`brief.md`, `persona.md`, any `*.md`),
  concatenated fresh per run. A human changes who the director is by editing
  a file. `persona.md` for the othello workspace now defines Hal Marsden and
  the innkeeper John.
- **An entrance guide** (`director/GUIDE.md`, `GET /guide`), re-read per
  request, with measured numbers.
- **A real backend switch**: `DIRECTOR_BACKEND` = `claude` (default) |
  `ollama`, agforge's resolution order, plus `DIRECTOR_MODEL`,
  `DIRECTOR_OLLAMA_URL`, `DIRECTOR_CLAUDE_CMD` — the last accepting a glob,
  which fixes the stale version-pinned pointer that has now bitten three
  workspaces.
- **Records** at `<direction>/records/run-NNNN.json` per
  `devpolicy/agent_records.md`.
- **A harness that no longer judges.** Details below.

## The clamps, before and after

review1.md's E3 table named two class-(b) devices — things that stop an
agent from being wrong rather than from doing irreversible harm.

| was | is |
|---|---|
| wrong-sized/wrong-format image aborts the run | an **observation** in the review message; the director may deliver it anyway and is asked to say so out loud |
| exactly 2 generation attempts | the director decides after **each** attempt: `DECISION: deliver / retry / stop`, and writes the next request itself on `retry` |
| composed desire missing the manifest's dimensions raises | an **advisory** in the evidence; the desire is sent as written |
| a generation failure ends the run | handed to the director verbatim; it says whether to go again |

One bound remains: `--attempt-budget` (default 5). It is a **cost** bound and
is labelled as one — when it bites, the verdict reads `attempt budget of N
exhausted while the director wanted to retry`, because "the harness stopped
the director" is a different fact from "the director stopped", and only one
of them is evidence about the director. It bit once in three runs.

Unit tests pin the reversal rather than the old behavior: a 512×512 image
against a 1024×1024 manifest entry **is delivered** when the director says
so, and a run **can** take four attempts.

## The three things worth carrying forward

**1. The clamps were never the binding constraint.** This is the finding I
did not expect. With the harness no longer able to stop it, the director
rejected all eleven images it was shown, across three runs, with specific
reasons every time — "a dungeon doorway, not the hall", "kill the mirror
symmetry", "a firepit with molten wax", "the generator isn't converging on
the brief no matter how the request is phrased". Not one rejection was about
size or format. The old 2-attempt cap and dimension gate had been sitting in
front of a weak point they were not guarding: the real limit is that agforge
+ SwarmUI cannot hit this brief, and the old harness stopped the loop before
anyone could learn that. Removing the clamps cost 4.03 USD and produced the
first legible statement of where this pipeline actually fails. That is the
roadmap's "narrowing comes later, from evidence" working as intended — and
the evidence points at the generator, not at the director.

**2. The persona file is load-bearing, and I proved it the embarrassing
way.** The director kept answering "what does this cost?" with "nothing but
the asking" — the exact failure Phase 2 fought. Two escalations of harness
prompt text did nothing. The cause was one line I had written in
`persona.md` telling the director that budget questions were not its call.
Two sentences in the workspace file fixed it, by a human, with no deploy and
no restart. The design intent of "persona in files, not in code" held under
its first real test — but so did its hazard: a persona file can quietly
contradict the entrance guide, and nothing detects that.

**3. A stale process cost me three false experiments.** Those two failed
escalations were never actually tested: `pkill -f "python3 window.py"` never
matched the server, because macOS rewrites its argv to `window.py`. Every
"restart" bound to a busy port, died, and left me talking to the original
process — so I read old code's output as new code's failure and hardened the
prompt twice against nothing. Both hardenings were reverted once the real
process ran, and the shipped prompt is the lean one. The lesson is cheap and
general: after restarting a local service, verify the *pid*, not the
healthcheck — a healthcheck cannot tell you which version answered. Cost:
~0.3 USD and one wrong conclusion about a model's behavior.

## Numbers (measured, from the records)

34 runs, 4.03 USD, all recorded, none failed.

| purpose | n | USD each | seconds |
|---|---|---|---|
| window answer (claude) | 11 | 0.085–0.256 | 3.8–11 |
| window answer (ollama) | 1 | 0.00 (90 tokens) | 4.2 |
| compose | 3 | 0.105–0.116 | 8.5–9.5 |
| review (looks at an image) | 11 | 0.126–0.179 | 9.7–29.6 |
| recompose | 9 | 0.106–0.119 | 7.7–13.6 |

A whole `reconcile` run measured 0.35–1.6 USD. Every figure above is in
`GUIDE.md`; nothing in that card is estimated.

## Cross-agent observations (not acted on)

- **agforge returns JPEG on later attempts.** In all three runs, attempt 1
  came back as a spec-correct PNG at the requested size and every subsequent
  attempt came back as JPEG with the size unstated — even after the director
  explicitly asked for "png at 1024x1024". agforge's charter makes format
  conversion the agent's own job, so this is agforge's agent behaving
  inconsistently across requests, not a director bug. It only became visible
  because the director now sees the file description.
- **agforge's `AGFORGE_CLAUDE_CMD` is currently broken** on agstudio: it
  points into `~/.vscode-server/...`, which does not exist on this Mac (the
  real path is `~/.vscode/...`). This is the third occurrence of the
  stale-pointer trap Phase 2's report predicted, and the reason director
  ships with glob resolution from day one. agforge's default backend is
  ollama, so nothing is failing today — it will fail the moment anyone flips
  it to claude.
- **gemma3 stamps `VERDICT:` on answers that were not judgments** — the same
  over-eager-marker behavior Phase 2 saw with the `/mission` redirect.

## Deliberate non-changes

- **executor: untouched** (prohibition 1).
- No secrets or payloads entered git: records, evidence, candidates,
  `persona.md` and `pj-agdev/.local/.env` are all under gitignored `.local/`.
- The `nctl --allow-destroy` class of confirmations is untouched.
- **No guardrails were pre-built** (roadmap step 5). The window is
  unauthenticated on the same reasoning as autolab's; `DIRECTOR_WORKSPACE_ROOT`
  confines which paths a request may *name*, which is addressing hygiene,
  not a limit on what the director may think.
- `understand_agents.md`'s director entry was rewritten, since it described
  a shape that no longer exists.

## Open items for whoever picks this up

1. **The generator is the bottleneck now.** Either the brief is beyond
   SwarmUI's current model, or agforge's agent is losing the technical part
   of the desire after the first attempt. A director that always says no is
   not yet distinguishable from a director that is right — the next episode
   should test it against images a human agrees are good.
2. **Nothing checks a persona file against the entrance guide.** They
   contradicted each other on day one and only a live cost question revealed
   it.
3. **The director has no memory between runs**, so on `retry` it re-derives
   its intent from its own previous reply passed back in. It works, but it is
   the seam where a long refinement conversation would fray.
4. **`reconcile.py` exits 2 on any non-delivery** — fine for a human, thin
   for a caller that wants to know whether the director stopped or the budget
   did. The verdict string says; the exit code does not.

## Deus Ex Machina notes (devpolicy/policy.md)

Recorded per step, collected here: rewrote the director agent (core, window,
guide, delivery flow) for agent director; wrote `persona.md` — the director's
own self-description — for agent director; updated `understand_agents.md`'s
director entry for whoever owns the agent map. All handoff candidates. The
persona file is the sharpest one: a director that cannot write its own
`persona.md` is a director whose identity is maintained by someone else, and
this phase moved that file out of code precisely so it could one day be
maintained by the director itself.

## State left running on agstudio

`ollama serve` (11434), agforge's request service (8092) and the director
window (8094) were started by hand for this phase and are still up; none is
under launchd, so all three vanish on reboot. The manifest at
`.local/asset-reconcile/othello-web/assets/manifest.json` gained two test
entries (`table-felt`, `hall-backdrop`), both still `requested`; the
previously delivered `background` was not touched.

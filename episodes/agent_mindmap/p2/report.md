# agent_mindmap — Phase 2 report: entrances and guides

AI-generated (Omni Agent, 2026-08-09). Phase 2 of `roadmap.md`. Status:
**done**, with one named gap (the assistant's `claude` backend has never run —
this machine has no Anthropic API key). Per-step detail is in `report1.md`
through `report5.md`; this is the phase-level view.

## Acceptance, against the roadmap's own words

| criterion | result |
|---|---|
| all four entrances answer "what can you do?" / "what does N cost?" | **met** — live, all four (report4) |
| autolab's window answers about a real job with real state | **met** — on agstudio *and* on agautolab1, naming that node's own jobs and their real costs |
| asking it to build something returns the `/mission` redirect | **met** |
| each agent's docs name the backend switch | **met** — four READMEs plus all four capability cards |
| flipping the switch demonstrably changes the recorded backend | **met** for autolab and agforge with real runs on both backends; **partially** for the assistant (the record flips to `claude` and fails honestly on the missing key); cagent has no per-request record to change, by design |

## What exists now that did not

- **A conversational window on autolab** (`POST /window`) — the node's single
  desire-accepting entrance, answering from the same job state its typed GETs
  serve, refusing work with the `/mission` redirect, deployed and verified on
  agautolab1 through the gitea + ansible path.
- **Four capability cards**, one per entrance, each answering can-do /
  cannot-do / costs, wired into the window each entrance already had.
- **A real backend switch in three places that lacked one** — autolab's window,
  the assistant (whose "engine-agnostic seam" was a comment, not a seam), and
  cagent's model — all in agforge's shape, all recorded per
  `devpolicy/agent_records.md`.

## The three things worth carrying forward

**1. Every figure in the four cards is measured or says "unknown".** Two draft
numbers were wrong and got caught by checking: the assistant's summary price
(0.01–0.05 → the real 0.11–0.19 USD) and agforge's claude-backend estimate
(a guessed range → the measured 0.134 USD). A card is a promise the system can
disprove, so the discipline that matters is not writing the card — it is
refusing to write a number nobody measured. cagent's card says its price is
unknown, and its agent says "I don't know" when asked. That is the behavior
the policy was written for, and it was the hardest to get.

**2. The record policy paid for itself inside one phase.** Phase 1 recorded
fields and explicitly deferred exploiting them. The first real return arrived
anyway: a window run failed with `could not launch claude
(…2.1.224…/claude)`, and the record made a stale version-pinned pointer
diagnosable in one read rather than a debugging session. The same failure then
recurred in agforge — two occurrences, one phase — which is what turned it
from a re-patch into a root fix (`claude_bin()` now resolves a glob, three
tests pin it). Records are cheap; the first thing they bought was a bug class,
not a metric.

**3. The free default is genuinely weaker, and the switch is the answer.**
gemma3 twice called a `converged` job "running" and once appended the
`/mission` redirect to a plain question; prompt tightening fixed the observed
cases, and claude/claude-sonnet-5 answered the same multi-job question
correctly first try for 0.09 USD. The point of Agent ≠ Model is not that the
cheap model is good enough — it is that the *entrance* stays the same when it
isn't. Placeholder quality was allowed here, so this is recorded as the known
cost of the free default rather than papered over.

## Deliberate non-changes

- **executor: untouched** (prohibition 1).
- No secrets or cluster payloads entered git: window records, run records and
  every `.env` live under `.local/` or in container logs;
  `ANTHROPIC_API_KEY` is passed through compose with no committed default.
- The `nctl --allow-destroy` class of confirmations is untouched; cagent's
  hard deny rules were not modified.
- Per-request backend switching was not added anywhere — the plan allowed
  process/config level, and that is what shipped.
- Completion **notification** (vs polling) stays future work, per the roadmap.

## Open items for whoever picks this up

1. **The assistant's claude backend needs one API key** to produce its second
   record row. Everything else is in place, including the failure path.
2. **agforge's `AGFORGE_CLAUDE_CMD` still has no resolver** — the stale-pointer
   fix landed in autolab, and agforge's value flows straight from `local_env`
   into `build_argv`. It was re-pointed and the trap documented; a third
   occurrence is still possible there.
3. **cagent looked healthy while its engine was absent** — the API served
   `llms.txt` fine with nothing listening on `:4097`, and only a real request
   revealed it. A liveness probe that actually reaches OpenCode would be worth
   more than the one that exists.
4. **The cards have no owner.** Today only a human or the Omni Agent can edit
   one, so they will drift from the systems they describe. Each agent
   maintaining its own card from its own evidence is the obvious next step —
   see the DEM notes.

## Deus Ex Machina notes (devpolicy/policy.md)

Recorded per step, collected here: built autolab's conversational window for
agent autolab; wrote the capability cards for agforge, assistant and cagent;
wired the entrance guides into those three; implemented the assistant's
backend seam and cagent's model config. All handoff candidates — this phase
was almost entirely the Omni Agent doing work that belongs to four in-system
agents.

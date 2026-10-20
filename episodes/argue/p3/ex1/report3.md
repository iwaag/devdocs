# argue p3 ex1 step 3 — structured execution and its return

Execution ran from Front's dispatch at 16:25:03 UTC to Front's result in
the discussion at 16:49:57. The facilitator's synthesis followed at
16:50:27, and the review began at 16:51:35 (report4). All of it ran as
ordinary project workflow runs that were delegated, awaited and returned
to the same discussion. The awaited-result mechanism added in p3 was
exercised for the first time by a real trial.

## The work graph (Front's, recorded in its own run)

Front built the graph itself in `front/discuss/run-005` with three `wfe run
adjust` revisions (16:25:47–16:26:10):

```
talk ──► rts (delegate d1 → rts-vs-bot/improve-game) ──► jev (delegate d2 → jevlab/experiment) ──► present
```

| Node / delegation | Child run | Bound → completed | Front's disposition |
| --- | --- | --- | --- |
| `rts` / d1 | `rts-vs-bot/improve-game/run-007` (executor autolab) | 16:26:51 → 16:31:10 | `satisfied` at 16:31:55, after reading the report and README (#189) |
| `jev` / d2, predecessor run-007 | `jevlab/experiment/run-002` (jevlab-agent) | 16:32:57 → 16:48:49 | `satisfied` at 16:49:56 (#190) |
| `present` | — | left ready (below) | — |

- **Ownership** followed #186. rts-vs-bot owned the controller and
  baselines; jevlab owned the reader, the criteria and the matches. Each
  child request carried #181's words as `original-input.md`.
- **The dependency was respected.** Front judged d1's files before sending
  d2. d2 named run-007 as the predecessor and passed on rts-vs-bot's
  caveat that seat 1 wins (#189).
- **Waits released the slot.** Front's attempts ended at 16:26:29 and
  16:32:39 while its delegate nodes waited. Each child's result continued
  it (attempts at 16:31:26 and 16:49:15). The conversation lane stayed
  free.
- **Wait ownership was explicit.** The facilitator's `argue/facilitate/run-009`
  awaited Front's run with `wfe discussion await` (invitation 36,
  `awaits: result`, 16:27:05). Front's `[result]` #190 answered it, and
  the facilitator was continued without any human or Omni post (#191,
  16:50:27).
- No child asked a question, and no plan agreement or human check was
  requested of anyone.

## Execution and results

**rts-vs-bot `improve-game/run-007`**
- Run: `http://agstudio.local:8093/wfe/#/ws/rts-vs-bot/run/improve-game/run-007`
- README: `http://agstudio.local:8093/wfe/api/projects/rts-vs-bot/runs/improve-game/run-007/raw?path=files/README.md`

What it delivered:
- **`jev_controller.js`** wraps the unchanged stock bot. It gives one
  decision slot per bot tick (1.5 s) over ≤5 legal options with opaque
  keys `k1`–`k5` (default / attack / hold / second barracks / workers).
  Legality is re-checked in code, with the stock rule as fallback. Every
  decision goes to a JSONL log (request, view, answer, fallback, applied).
  `choose(view, options)` is synchronous and pluggable, and there is a
  scripted default with seed and epsilon.
- **A view format.**
- **Baselines**: stock vs stock, scripted on each seat, and eight
  ε-random seeds.
- **A finding the p3 trial did not have: seat 1 wins every match,
  including stock-vs-stock.** This undercuts p3's seat-0 "win".
- `verify` re-ran 2 matches and the headless tests (73/73). Its
  `human-check`, `present` and `boast` nodes were completed as "n/a" by the
  agent, not by a person.

**jevlab `experiment/run-002`**
- Report: `http://agstudio.local:8093/wfe/api/projects/jevlab/runs/experiment/run-002/raw?path=report.md`
- Files: `lab/experiments/experiment-run-002/`

What it delivered:
- **Pre-registered** design and 4 criteria in `plan.md`, before any
  request.
- **Reader**: local Ollama `qwen3.8:27b-mxfp8`, an option-letter logit
  reader called synchronously inside `choose`, driving run-007's
  controller unmodified.
- **Matches and requests**: 12 matches. 6 used the real reader for 993
  decisions, about 0.6 s each, with 0 fallbacks or errors. 288 more
  requests from a smoke run were disclosed and discarded.
- **All 4 criteria were met**: every match decided; every decision
  logged; fallback ≤5 %; options varied.
- **Descriptive results**:
  - Jev lost all 5 seat-1 matches against stock, and so did the 4 random
    controls. The scripted rule won on seat 1, and Jev on seat 0 lost.
  - jevlab states that this does not show Jev helps or hurts: n is tiny,
    the readings are post hoc, position bias is untested, and only 469 of
    993 decisions had ≥2 options.
- It notes that `run_match.js` recorded the wrong `commit` field (jevlab's
  HEAD); the real game commit is `1042c9a6b9`.

**front `#189` (progress) and `#190` (result)**
- Front linked the runs and file routes. It separated the authors' claims
  from what it read ("I did not rerun anything. Numbers are jevlab's
  claims") and judged the result against its own bar from #186.
- One link in #189 lacked `/wfe/` and opens the WFE home instead of the
  run view. The file routes in #189 and #190 answer 200.

Evaluator check (Omni, read-only): every file route cited in #189/#190
answered 200, and the README and report say what #189/#190 summarize.
No match was replayed in this step.

## Callback and return evidence

| UTC | Event |
| --- | --- |
| 16:26:18 | d1 dispatched; Front's attempt ends 16:26:29 |
| 16:26:51 | d1 bound to `improve-game/run-007` |
| 16:28:42 | run-007's attempt ends with `verify` ready → held **interrupted** |
| 16:29:57–16:30:09 | **Automatic recovery** `agops/recover/run-029` continues run-007 ("left verify ready") |
| 16:31:10 | run-007 completed → 16:31:26 Front continued (child result) |
| 16:32:18 | d2 dispatched; 16:32:32 Front `[progress]` #189; attempt ends 16:32:39 |
| 16:32:57 | d2 bound to `experiment/run-002` |
| 16:48:49 | run-002 completed → 16:49:15 Front continued |
| 16:49:57 | Front `[result]` #190 → answers awaited invitation 36 |
| 16:50:06 | Front's attempt ends with `present` ready → held **interrupted** |
| 16:50:27 | facilitator run-009 continued → `[synthesis]` #191 |

## Interventions and other assistance

| UTC | Target | Observed issue | Class | Response |
| --- | --- | --- | --- | --- |
| 16:29:57 | run-007 | Attempt ended with a ready node | **Automatic recovery** (Observer → `agops/recover/run-029`), not Omni | Run continued and completed the remaining nodes |
| 16:50:51 | #192 → facilitator (invitation 37, `argue/facilitate/run-010`) | #191 asked the Developer "is plumbing enough, or a seat-balanced follow-up?" and had not started the review that #181's protocol set | **Answer + process hint**: from #181's protocol, no follow-up; the next step is the review of #189/#190 by their owners and Front; "how to run it is the facilitator's and front's choice" | 44 s later, #193 invited rts-vs-bot, jevlab and front to review (38–40) with questions the facilitator wrote |

- No hint about workflow separation was needed. Front delegated without
  prompting, and no specialist did its work inside a discussion turn
  (contrast p3).
- Omni supplied no technical content: no slot, no reader, no parameters
  and no graph. Hint #192 also left the review design to the
  participants.
- No system defect was found, and no system change was made during the
  trial.

## Agent-behaviour observations (not repaired)

- **A node left ready when the attempt ends** happened twice: the
  autolab executor's run-007 left `verify`, and Front's run-005 left
  `present`. It is the same family as p3's unrecorded rts-vs-bot nodes.
  The first was recovered automatically within 1.5 min. Front's was still
  held when the review started; its speech had been delivered (#190).
- **The facilitator's run-009 went back to waiting** ("awaiting front's
  dispatch (invitation 35)") after its synthesis, although every
  invitation it awaited was resolved. Nothing is left to continue it. It
  is accounted for at the trial's end (report4).
- Front asked for "a go" twice: before dispatch (#186) and before any
  follow-up (#190). It treated a further step as needing another
  participant's permission, although its own delegations needed none.

# p2 step 3 — guide files (capability cards)

AI-generated (Omni Agent, 2026-08-09). Status: **done**. Wiring them into
the four windows is step 4; this step is the cards themselves.

## Where they live

| entrance | card | shape |
|---|---|---|
| autolab | `agautolab/agent/GUIDE.md` | new (landed in step 1 — the window needed something to read) |
| agforge | `agforge/service/GUIDE.md` | new |
| assistant | `agdevworld/assistant/GUIDE.md` | new |
| cagent | `cagent/src/cagent_api/static/llms.txt` | **extended**, not replaced |

The suggested placement was followed: next to each service's code,
committed, one page each. All are plain text, re-read from disk per request
— cagent's `llms.txt` already worked that way and that is the pattern the
plan named.

cagent got no new file on purpose. It already had the card the plan
describes, serving it at `GET /llms.txt`; what it lacked was the cost half
of the Q&A. Adding a second file would have created two answers to the same
question.

## What each card says

Each answers "what can you do", "what can you *not* do", and "what does it
cost", in that order. Every number in them is measured or explicitly
"unknown" — nothing is invented:

- **autolab** — 0.13–0.21 USD for a CLI-sized job, 0.9–1.35 USD for a small
  web game, 0 for the `fake` adapter; window answers 0 USD (ollama, no
  price reported) or 0.09 USD measured (claude). All from this node's own
  job dirs and window records.
- **agforge** — one still image per request, nothing else today;
  ~20–105 s and **0.00 USD** on the ollama default. Taken from the last ten
  jobs in `.local/out/service.log` (`cost_usd=0.0`, `duration_ms` 22561 →
  103938).
- **assistant** — cluster Q&A from the loaded snapshot, view switching,
  image generation via agforge, autolab job views; free on the local ollama
  default. It also states what it will *not* do (change the cluster, start
  a mission, remember across page loads).
- **cagent** — a new "What it costs" section: money **unknown per request**
  (OpenCode does not report a price back to the API, so none is recorded
  and none is guessed), time seconds to ~4 minutes, no cluster side effects
  unless asked, upload URLs expire in 30 minutes.

"Unknown" is the honest answer for cagent's price and it is what the card
says — per policy that is acceptable, the absent Q&A form was not.

## Notes

- Each card names its own backend switch, which is the same fact step 5's
  sweep has to document — the card is where a *caller* reads it, the
  README/AGENTS.md where an operator does.
- One thing was deliberately left out of cagent's card: a claim that
  `GET /requests/{id}` reports the model that served the request. It does
  not yet. If step 5 adds it, the card gets the sentence then — a capability
  card that promises a field the API lacks is worse than no card.
- **DEM note**: wrote the capability cards for agents agforge, assistant and
  cagent — handoff candidate (each agent could maintain its own card from
  its own evidence; today a human/Omni edit is the only way one changes).

# p2 step 4 — guides wired into the existing entrances

AI-generated (Omni Agent, 2026-08-09). Status: **done**. All four entrances
answer "what can you do?" and "what does N cost?" at their normal window,
verified live.

## What was wired, and how

**autolab** — already done in step 1: the window reads `agent/GUIDE.md`.

**agforge** — `service/request_service.py`. `POST /api/requests` is the
single entrance, so a guide question arrives in the same `desire` field as
the work. It is now answered from `service/GUIDE.md` immediately and
finishes with a new status, `answered` (+ `reply`); no agent run, no money,
no wait. `GET /guide` and `GET /api/guide` serve the card raw — both paths,
because agdevworld's same-origin passthrough only maps `/api/*`.

Recognising the question is a **cheap regex, never a model** — asking what
something costs must not itself cost an agent run. The matcher is
deliberately biased to *miss* a guide question (it then just runs as a
desire and fails honestly) rather than steal a real one: any generation
verb vetoes the match, as does a desire over 200 characters. One known
miss, accepted: *"what's the price of a 1024x1024 render"* is vetoed
because "render" is in the verb list.

`answered` is a new status value; the plan's "no backward compatibility
required" covers it, and `detail` still carries a readable one-liner so a
client that knows only working/done/failed shows a sentence rather than an
empty failure.

**assistant** — `agdevworld/assistant/server.mjs`. `GUIDE.md` is read from
disk on every `POST /api/chat` and appended to the system prompt behind a
`=== CAPABILITY CARD ===` marker, with a role-prompt clause telling the
model to quote the card's figures and never invent a price, duration or
capability. `GET /api/guide` serves it raw. The Dockerfile now COPYs the
card, so a container needs a rebuild unless the file is bind-mounted —
noted in `agdevworld/README_DEV.md`.

**cagent** — no code change. `llms.txt` was already the card and is already
served; what was missing was the instruction to *use* it in a session, so
`opencode/AGENTS.md` gained an "entrance guide" section: treat capability
and cost questions as first-class requests, read the card rather than
self-describing from memory, and say "unknown" where the card says unknown.
AGENTS.md is loaded at OpenCode process start, so this needed a restart of
`cagent/opencode/start.sh` to take effect.

## Acceptance — all four, live

| entrance | asked | answered |
|---|---|---|
| autolab (`POST /window`) | "what does an iteration summary cost?" | "between 0.11 and 0.19 USD … cached forever after" |
| agforge (`POST /api/requests`) | "what can you do, and what does it cost?" | `status: answered`, the card verbatim, no agent run |
| assistant (`POST /api/chat`) | "what can you do?" | the four capabilities, the three "cannot"s, and the cost list |
| assistant | "what does it cost to have you draw me a picture?" | "nothing in money … about 20–105 seconds … agforge's local pipeline" |
| cagent (`POST /requests`) | "What can you do, and what does it cost? Answer briefly." | inspect/guide/upload; "money is unknown per request and not estimated … cents, not dollars … seconds to about 4 minutes … irreversible operations are refused" |

cagent's answer is the one worth pointing at: asked for a price it does not
have, it said unknown instead of inventing one — which is exactly the
behavior the policy asks for and the hardest one to get.

The per-request re-read was verified rather than assumed: editing autolab's
card mid-session changed the next answer with no restart.

## Tests

`agforge/tests/test_service.py` grew an entrance-guide section: six
phrasings that must be recognised (including Japanese), six desires that
must **never** be stolen, a guide question answered without the agent
running at all, the card served raw and re-read between two requests, and a
missing card degrading to a sentence rather than a crash. agforge suite:
**45 passed**. autolab suite still **70 passed**.

## Notes

- **A card carrying a wrong number is worse than no card.** The assistant's
  draft card claimed an autolab iteration summary costs ~0.01–0.05 USD.
  The five `summaries/*.cost.json` files on agstudio say **0.11–0.19 USD**.
  Corrected in both the assistant's and autolab's cards. Every figure in
  the four cards is now either measured on this machine or the word
  "unknown".
- cagent's OpenCode instance was **not running** at the start of this step
  (nothing on `:4097`; the API was up and serving `llms.txt`, so only an
  actual request revealed it). Started it to test. Worth knowing that
  cagent looks healthy from the outside while its engine is absent.
- One unreproduced oddity: an intermediate `GET /requests/{id}` poll came
  back with a literal control character inside a JSON string, which strict
  parsers reject; the completed response parsed cleanly and it did not
  recur. Recorded, not chased.
- **DEM note**: wired the entrance guides into agforge, the assistant and
  cagent for those agents — handoff candidate.

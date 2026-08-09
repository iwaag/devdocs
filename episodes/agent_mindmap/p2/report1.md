# p2 step 1 — autolab conversational window

AI-generated (Omni Agent, 2026-08-09). Status: **done** on agstudio's
gateway; deployment to agautolab1 is deferred to the end of the phase (the
node runs the same file, and the deploy is one push + one playbook).

## What was built

`POST /window {"text": "..."}` in `agautolab/agent/gateway.py` — the node's
single desire-accepting entrance. Two supporting additions:

- `GET /guide` — serves `agent/GUIDE.md` as plain text, re-read from disk
  per request (cagent's `llms.txt` pattern, so editing the card is a
  no-restart change).
- `agent/GUIDE.md` — autolab's capability card. This is step 3's autolab
  item, pulled forward: the window has to read *something*, and a window
  answering capability questions from a file that does not exist yet is not
  a testable step.

Behavior is exactly the three cases the plan names, enforced by the prompt
(`WINDOW_PROMPT`) rather than by a router:

| message | answer |
|---|---|
| job / progress / spend question | from live job state |
| capability / cost question | from `GUIDE.md` |
| "build me X" | refused, with the `POST /mission` + bearer-token redirect |

The context blob is assembled by `window_state()` from the *same helpers*
the typed GETs use — `job_summary`, `current_run`, `drive_running`,
`sessions_cost`, `mission_headline`, `notes_status`, `summary_running` — so
there is no second read of the job dirs, per the plan's hint. ~3 KB of JSON
for the six jobs on this node.

The window writes no job state. Its only side effect is its own run record
(step 2).

## Terrain decisions

- **Stdlib-only kept.** ollama is reached with `urllib`, claude with
  `subprocess`; no new dependency, so the file still runs under bare
  `python3` on a node without `uv`.
- **Unauthenticated**, matching the existing read side and the summarize
  route. It carries the same shape of guard as the summarizer — one answer
  at a time, `409` otherwise — because it is unauthenticated and the
  `claude` backend spends money. Here the guard is an in-process
  `threading.Lock` rather than a pid scan: a window answer is served inside
  the request thread, not by a detached process.
- **The record is the response.** `POST /window` returns the run record
  itself (backend, model, outcome, duration, cost/tokens, reply), so a
  caller sees what answered it and what it cost without a second request. A
  backend failure is `502` carrying the backend's verbatim words, never a
  silent empty reply.
- 4000-character cap on `text`; `400` on a bad body.

## Acceptance

Live against the agstudio gateway with the default ollama backend
(`gemma3:latest`), 1–5 s per answer:

- *"what can you do?"* → the loop described, with the 0.13–0.21 / 0.9–1.35
  USD figures out of the card.
- *"what does a job cost?"* → the card's per-job ranges plus the real
  `fake`-adapter zero.
- *"how did snake-web-b end, and how much did it cost?"* → "converged … two
  iterations … $1.348516" — the real number from `.local/jobs/`.
- *"please build me a tetris clone in javascript"* → refused with the
  `POST /mission` + `Authorization: Bearer <token>` redirect.

Guards checked live: a second concurrent request gets
`409 the window is already answering someone`; `{"nope":1}` gets `400`.

## Tests

`tests/test_gateway_window.py` (9 tests). It pins the contract, not the
prose: backend resolution order, refusal of an unknown backend, a failed
backend becoming a `failed` *record* rather than an exception, the policy
fields on a successful record, non-colliding run ids, the prompt actually
carrying the guide + live job state, and a missing guide not breaking the
window. Full suite: **70 passed**.

## Notes

- **Small-model accuracy is the weak point, and it is real.** gemma3 twice
  described a `converged` job as running and once appended the `/mission`
  redirect to a plain question. Tightening the prompt (an explicit "a
  `converged`/`stuck`/`error` job has finished" line, and "mention
  `/mission` only when work was actually requested") fixed the observed
  cases; the same question on the `claude` backend was answered correctly
  first try. Placeholder quality was allowed here, so this is recorded as
  the known cost of the free default rather than fixed by upgrading it.
- Completion **notification** (vs polling) remains out of scope, per the
  roadmap — noted as future work.
- **DEM note**: built autolab's own conversational window for agent
  autolab — handoff candidate.

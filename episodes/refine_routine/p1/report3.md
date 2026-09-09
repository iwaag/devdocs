# refine_routine p1 — step3 report: observation, and continue/end as Front's judgment

Date: 2026-09-10

## What was given to the run

- **`agbudget`** (`agfront/src/agfront/budget.py`, a console script in
  agfront's venv, so it is on every run's PATH like `agentchat`). It reads
  the agentroom relay's `GET /budget` (`ag.budget.v1`) — or a JSON file of
  the same shape named by `AGFRONT_BUDGET_URL` — and prints the observation
  as text: per harness, the plan, when the read was made, and per window
  the percent used as the vendor states it, the reset time absolute and
  relative, and a flag when that reset has already passed since the read.
  A failed read is printed as `READ FAILED` with the reason and the last
  good numbers marked **STALE**; a source that cannot be read at all is
  printed as such. Nothing in the text ever says "reached" or renders a
  failure as a number. `--json` gives the raw document.
- **`tools/budget.md`** is written at the start of every run serving: the
  explanation of how to read a window (percent of the *current* window,
  other work on the account counts, reset, stale, failed) followed by the
  observation at that moment. Only the `routine_run` role gets it, and only
  that role holds the `Bash(agbudget:*)` grant. A failed observation never
  stops a serving.
- **The guide** (`agent/guides/routine_run/guide.md`, section *Conditions
  about usage*) fixes the readings the plan asks for: "until the window is
  N % used" is the current window reaching N whatever consumed it, met at
  once if it already stands there; "consume N points from the start" is the
  start reading plus N, a different condition; a reset neither ends nor
  restarts a "reach N" condition; a failed or stale read is neither reached
  nor 0, and a hold must say what resumes it; reaching the condition mid-run
  stops new work and lets the in-flight work come back, with no strict
  ceiling promised. Every entry names the read it judged on. Ending the run
  and reaching the routine's goal are two different things: `achieved` in
  the finish block is the goal, `reason` is why the run ends. The requester
  guides (`front`, `character_talk`) say to write the reading of a usage
  condition into the opening post, so the interpretation is recorded at the
  start, and that the run judges from a read of its own.

## Verification

- Fixtures (`agfront/tests/fixtures/budget/`, frozen at 2026-09-09T15:00Z):
  `below` (31 %), `reached` (57 %), `reset` (72 % read before a reset that
  has passed), `failed` (expired token, stale 12 % three hours old).
  `tests/test_budget.py` pins each rendering, the file source, the CLI, the
  document, that a run serving is handed it and other roles are not, the
  grant, and the guide's wording. 91 tests pass across agfront.
- Live, from the run's own environment (agfront's venv on this host, the
  relay on loopback): `agbudget` answered with the codex and agy cards live
  and the claude_code card as **READ FAILED** — the CLI's stored token had
  expired and the relay never refreshes it — with stale numbers whose
  session reset had already passed, flagged as such. That is the failure
  case the plan asks to handle, met in the real environment before any
  fixture; the connection itself works.

## Not done here

- Whether Front actually judges these cases well is a live question for
  step5 (the boundary is proven by the fixtures, the connection live). The
  claude_code card will read live once a Claude Code run on this host has
  renewed the token.
- Nothing resumes a held run on its own: a hold is recorded with its resume
  trigger, and time-based resumption stays a later phase.

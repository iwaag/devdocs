# zero_auth — report, Step 5 (window gains mission-starting power)

AI-generated (Omni Agent). Backend: Claude Code / claude-fable-5.
Date: 2026-08-10.

## Design chosen

Both shapes the plan suggested, because they compose: the `/mission`
handler body is refactored into a module-level `start_mission(mission,
max_sessions)` — the one mission-starting seam — and the window gets an
explicit ability (Tool Giving, not intent detection): a structured block
in its reply,

    <<mission max_sessions=N>>
    ...mission text...
    <</mission>>

which the gateway executes after the answer (`apply_mission_block`). The
start's outcome (202/400/409 document) is recorded on the window run
record under `mission`; the block is cut from the reply the user sees.
Existing semantics kept as concurrency guards: 409 with the current run
while a drive is alive, `.local/agent/done` cleared on start, run id +
pid in the response. `POST /mission` now drives the same seam (until
Step 6 retires it).

## Prose updated

- `WINDOW_PROMPT` — the window is told it is the entrance and how to
  start work with the block, one-at-a-time semantics, and to only start
  when the user clearly asks for work.
- Module docstring — window can start work; `/mission` shares the seam.
- `agent/GUIDE.md` "Doors" — `/window` listed first as the entrance with
  the mission block; `/mission` described as the open deterministic
  start (this text is load-bearing: served at `GET /guide` and injected
  into every window prompt).

## Verification

- `py_compile` passes.
- Unit checks (scratchpad, `start_mission` mocked): block with
  `max_sessions=3` parsed and executed, outcome recorded, block stripped
  from the shown reply; reply without a block untouched; default
  max_sessions 12; real `start_mission` refuses empty mission and
  out-of-range max_sessions with 400 without spawning anything.
- Gateway restarted on agstudio with the new code: `/healthz` ok; a
  harmless window question answered normally (window/run-0054, backend
  claude/claude-sonnet-5, $0.115, no mission block — none expected).
- The full conversational-start E2E is Step 8's proof, per the plan.

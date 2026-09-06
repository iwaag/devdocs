# ex2 step A — the `done` confirm addendum was already built

The plan asks for `../confirm/plan.md` to be carried out. **That file does not
exist under `p2/confirm/`, and it does not need to: the same plan is
`p2/ex1/plan.md`, and it has already been executed and reported.** ex2 was
written against p2's report, before ex1 landed; the four items it names are the
four items ex1 delivered.

| what the ex2 plan asks | where it is |
|---|---|
| `POST /ops/confirm`, confirm-all | `agentroom/src/agentroom/server.py` — `WRITE_ROUTES = ("/ops/confirm",)`, dispatching to `Ops.confirm` |
| `done` rows only | refused in `Ops.confirm`, not only in the view |
| in-memory marks | `confirmed` counters in the `/ops` payload; nothing on disk |
| `done` excluded from "N rows open" | `agdevworld/src/views.ts` — `const open = count - done` |
| one full cycle proved in `#ops-testbed` | `p2/ex1/report3.md`, five screenshots |

So step A is a **verification step**, and it was run rather than assumed.

## What was verified, today, against the running relay

- `uv run --project . python -m pytest -q` in `agdevworld/agentroom` → **43
  passed**, the same 43 (34 + 9) ex1 reported.
- `GET /ops` on the live relay answers `"state": "live"` with
  `"confirmed": {"rows": 0, "topics": 0}` — the marks are empty because this
  process has never been asked to hide a row, which is what "in memory only"
  looks like from outside.
- `POST /ops/confirm` with `{}` → `200 {"confirmed": 0, "topics": [],
  "refused": []}`. There are no `done` rows on the board right now, so
  confirm-all is honestly a no-op; the endpoint is reachable and the write
  route is wired.
- `POST /ops/confirm` naming a conversation that is not on the board →
  `404 "no row for agents / intro-agping-agstudio1 is on the board"`. The relay
  is what decides, and it declines to invent a mark for a row it is not
  showing.

The `409`-on-a-live-state refusal is ex1 step 1's test and ex1 step 3's live
proof; it could not be re-driven here without manufacturing a fresh stall in
`#ops-testbed`, which would have been a re-run of ex1 step 3 rather than a
check of it. The board's single live row is `agping-agstudio1` in `unknown` —
step C's subject, and not a `done` row.

## Note for whoever writes the next plan

The ex2 plan's own pointer, `../confirm/plan.md`, resolves to nothing. A plan
step that names a file by relative path is worth resolving *when the plan is
written*: the cost here was small (the work was done, under a different name),
but the same broken pointer with the work *not* done reads identically.

Constraints 1–3 of ex2 are untouched by this step: nothing was written to
Zulip, nothing was persisted, and no credential or absolute path entered a
tracked file.

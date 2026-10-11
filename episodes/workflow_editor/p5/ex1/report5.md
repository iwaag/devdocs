# p5/ex1 step 5 — the observed backend in the run header

Plan section: [plan.md](plan.md) §5.

Commit: agdevworld `60861a5` (pushed; deployed in step 7).

## Change

The run header's Executor fact read `run.json`'s `executor.backend`. That value is null for runs requested through agdevworld, so the header said "backend unknown", while the attempts listed below it recorded the backend. The header now uses the attempt evidence the view already loads from the execution API (`GET runs/<wf>/<run>/exec`). It adds no new endpoint, does not rewrite `run.json`, and does not infer a backend from the agent's name.

`shared/backend.ts` → `backendFact(configured, attempts)`:

| Evidence | Header text after "autolab ·" |
| --- | --- |
| An attempt not yet ended | `backend <b> (attempt in progress)` |
| All ended | `backend <b> (latest attempt)`, by start time |
| That attempt recorded no backend | `backend unknown (latest attempt recorded none)`. It is never taken from an earlier attempt |
| Attempts used different backends | adds `· N backends across M attempts` |
| No attempt | `no attempt yet` |
| `run.json` names a configured backend | adds `· configured: <c>`, kept separate from the observed one |
| No execution data (a historical revision, or the API unreachable) | `backend not observed here` |

The header is re-rendered whenever the 3-second execution refresh brings new attempt data.

## Evidence

| Check | Result |
| --- | --- |
| `test/backend.test.ts` | The run-002 shape (three ended attempts, one backend) gives the latest attempt's value. An active attempt wins over the latest ended one, whatever the row order. A missing backend says unknown and is not borrowed. Configured and observed values are kept apart. No attempts, and no data, are handled |
| `checks/backend.ts`: private service, four runs with recorded attempts, the real browser UI | completed → `backend claude_code / anthropic/claude-sonnet-5 (latest attempt)`; active after another backend → `… (attempt in progress) · 2 backends across 2 attempts`; latest recorded none → `backend unknown (latest attempt recorded none) · 2 backends across 2 attempts`; no attempt → `no attempt yet`; no page errors |
| Editor `npm run check` | 135/135 tests, type check and build pass |

The live `improve-game/run-002` header is checked read-only after deployment (step 7).

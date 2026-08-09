# agent_mindmap — Phase 1 report

AI-generated (Omni Agent, 2026-08-09). Phase 1 of `roadmap.md`:
policies and the common record, docs only. Status: **done**.

## What was done

1. `devpolicy/policy.md` — added an "Agent guidelines (recommendations,
   not enforcement)" section with the four expectations, in the file's
   existing terse, recommendation-tone style:
   - Single Entrance
   - Entrance Guide
   - Agent ≠ Model (pointing to the new record policy)
   - Deus Ex Machina note (one-line episode note is the whole obligation)
2. `devpolicy/agent_records.md` — new, one page. Fixes the per-run
   *fields* (request/job id, backend model+harness, outcome
   done/failed/aborted, cost/time when reported, free-text failure
   report in the agent's own words) and explicitly leaves paths and
   formats to each workspace. Documents agforge and autolab as the
   precedents it generalizes.
3. `devpolicy/terms.md` — added `Entrance` and `Entrance Guide`
   (optional step; kept purely definitional to match the file's
   "just terminology, not rules" framing).

## Acceptance check

- Policy files read coherently with their surroundings (checked against
  the existing tone of policy.md and terms.md).
- **agforge conforms without code changes**: `service/agent_run.py`
  records request_id, `meta.backend` (ollama/claude/override), job
  status, `total_cost_usd`/`duration_ms`/`num_turns` when the backend
  reports them, a raw transcript per run, and agent-authored
  `problem.md` on honest failure (path-only rule).
- **autolab conforms without code changes**: verified against a real
  job (`agautolab/.local/jobs/snake-web`) — job name + `iter-NNNN` as
  id, `job.yaml` `adapter:` plus `adapter_result.json` `modelUsage` as
  backend, `state.json` status as outcome, `adapter_result.json`
  cost/usage/duration, and the agent's raw output preserved as
  `adapter_output.txt`.

## Notes

- Minor gap, acceptable under "cost/time when the backend reports
  them": agforge's ollama backend reports cost only when opencode
  emits `step_finish` cost events; nothing invented when absent.
- autolab's failure free-text is covered by the preserved raw agent
  output rather than a dedicated agent-worded report file; if a
  dedicated report is ever wanted, it is the trivial-field-addition
  class the roadmap allows — not needed now.
- Recording comparison/exploitation stays out of scope, per the scope
  decisions in `roadmap.md`.

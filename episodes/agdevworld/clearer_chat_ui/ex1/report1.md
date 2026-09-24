# ex1 step 1 — a handler's confirmation requirement survives the run's report

## The failure, reproduced

autolab's task serving (`agautolab.zulip_listener._serve_run`) ends a run
that wrote `report.md` without the requester's agreement with a
`TopicResult` whose `meta` is `response_request ask=confirmation`; the
`serve_run` wrapper adds the run's own words as `output`. In
`agag.topics.serve_topic` the meta was `split.meta or meta`, so a run
replying ```` ```ag-reply intent=report ```` replaced the requirement: the
post said "it closes when its requester agrees here" and carried
`intent=report`, and `read_requests` listed nothing.

The new regression test in agautolab
(`test_a_report_nobody_agreed_to_asks_its_requester_whatever_the_run_declared`)
fails on the previously pinned pyagag `932d3c6` for four of its five run
intents (`report`, `progress`, a question, a question to someone else) and
passes with the fix.

## The contract chosen

`agag.post.combine(declared, handler)` — combining by role, not strength:

| handler `TopicResult.meta` | run's fence | the post |
|---|---|---|
| `response_request` (a **requirement**) | anything / nothing | a request; `to` and `ask` are the handler's; what the handler left out comes from the run's request (`ask` only if it asked the same person); a still-missing `to` is the recorded requester |
| `progress` / `report` (a **default**) | an intent | the run's |
| `progress` / `report` | none | the handler's |
| none | anything | the run's |

- `re=` is the union (run's references first); `seen` is always the
  listener's processed-input boundary, never taken from either side.
- Failure paths: a reply with no usable mark after the repair counts as the
  run declaring `report`, so a requirement still stands over it; a repaired
  reply is combined like any other. A handler that raised reached no state
  and requires nothing — its failure line is a `report`.
- A request with nobody to address (no requester recorded, or only the bot
  itself) falls back to the run's own non-request intent (else
  unclassified) instead of dropping the run's `report`.
- When a requirement overrides a declared intent, the listener logs
  `the handler's response_request to=<id> stands over the reply's intent=…`.

The `TopicResult.meta` docstring and `pyagag/docs/post-intent-v1.md`
("Handler and run together") say this.

autolab now names the recipient explicitly: the requester read from the
processed input (a start note's named requester — Front in the trial-G
shape), as `to=<id>`; with nobody but itself to ask it states no
requirement.

## Verification

- pyagag `tests/test_required_request.py` (18): `combine` precedence
  (requirement over report/progress/none/re-only; handler recipient and kind
  over the run's question to someone else; filling from the run's request;
  union of `re`; `seen` never carried; defaults do not override declared
  progress/report/question); and the real `serve_topic` path: run `report`
  + requirement → post `response_request to=7 ask=confirmation seen=501`,
  listed `pending` by `read_requests` over the conversation as delivered;
  progress and a question become the confirmation; conflicting recipients
  keep the handler's; a repaired report and a failed repair keep the
  requirement; a failed handler posts a `report`; no requirement leaves
  `report`/`progress`/question as declared; nobody to ask leaves `report`.
- agautolab: the real `handle_workrun` → `serve_topic` → `serve_run` path
  with a task started by a start note naming Front (15) and a run replying
  under each of five fences: the delivered post carries
  `response_request to=15 ask=confirmation seen=<boundary>`, contains both
  the run's words and the "closes when its requester agrees" line, is
  `pending` in `read_requests`, and no result is recorded and nothing is
  integrated (`report`/`integrate` calls empty) — receiving a request is not
  accepting a task.
- Suites: pyagag 877 passed; agautolab 298 passed (on the new pin).

## Commits

- pyagag `cb4568f` — `combine`, serving path, docs, tests.
- agautolab `9456a98` — explicit recipient, regression test, pin
  pyagag `cb4568f`.

Other consumers were checked: agobserver's intake returns a request with no
run output (unchanged); cagent computes its front answer's meaning itself
and returns a `report` default (unchanged); every other handler meta is a
`report`/`progress` default, whose behaviour is the same as before.

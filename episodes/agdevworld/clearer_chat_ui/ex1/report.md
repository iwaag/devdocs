# clearer_chat_ui ex1 — report

**Goal**: fix the two review findings. (1) A task's required confirmation
must survive the run's own `report` intent. (2) Choosing "not an answer"
must stop the next post from settling a request implicitly.

**Outcome**: both are fixed, tested through the real paths, deployed, and
checked live. Step reports: `report1.md`, `report2.md`; this file also
covers step 3.

## Contract changes (`ag.post.v1`, `pyagag/docs/post-intent-v1.md`)

1. **Handler and run metas combine by role** (`agag.post.combine`). A
   handler's `response_request` in `TopicResult.meta` is a *requirement*.
   It stands over whatever the run declared, and the handler's `to` and
   `ask` win. The run's request supplies only what the handler left out,
   and `ask` only when the run asked the same person. Any other handler
   intent is a *default* that applies only when the run declared none.
   `re=` is the union of both; `seen` always comes from the listener.
   - When the reply fails even after repair, the run counts as having
     declared `report`, so a requirement still stands.
   - A handler that raised requires nothing.
   - If a request has nobody to address, the post falls back to the run's
     own intent when that intent is not a request.
   - autolab now names the task's requester explicitly (`to=<id>`,
     `ask=confirmation`).
2. **`answer=none`**: an explicit non-answer, the third correlation
   choice beside "none given" (next-post rule) and `re=`. Combining it with
   `re=` is malformed. `read_requests` never settles anything with such a
   post and never lists it as `unmatched`. It still counts as speech for
   `overtaken`.
   - Where it can be set: `agentchat send --not-answer`, on the
     `ag-reply` fence, and `not_answer: true` in the relay's post body
     (refused together with `answers`).
   - What readers see: "not an answer to any request".

## Delivered behaviour

- **autolab**: in a task whose run wrote its report without the requester's
  agreement, the post asks the requester to confirm. This holds whatever
  the run declared (`report`, `progress`, a question, a question to
  someone else). The request is listed as pending. Nothing is accepted or
  integrated because of it.
- **Front Desk and Arguing Room**: the composer has three choices:
  *automatic*, *answering #n* and *not an answer*.
  - The choice is shown before sending, both on the strip and in the
    placeholder.
  - It can be switched either way, and returns to automatic after a
    successful send, on a conversation switch, or once nothing waits.
  - It is kept together with the draft when a send fails or is uncertain.
  - The history labels a non-answer `↷ not an answer`.

## Regression evidence

- **Finding 1**: agautolab
  `test_a_report_nobody_agreed_to_asks_its_requester_whatever_the_run_declared`
  follows the real `handle_workrun` → `serve_topic` → `serve_run` path.
  It fails on the old pin (4 of 5 run intents) and passes now. pyagag
  `test_required_request.py` covers precedence and the serving path
  (progress, question, conflicting recipient, repair, failed repair,
  failed handler, nobody to ask).
- **Finding 2**: pyagag `test_non_answer.py` covers the wire format,
  validation, read model, mirror restart/rebuild, CLI and redelivery.
  Relay tests cover the Front Desk read-back and the Arguing Room through
  a real mirror. The browser check used the real relay over a FakeRealm
  for both rooms: switching modes, an aside leaving #108 pending across
  a reload, a failed send keeping the choice and then posting
  `answer=none` on retry, an explicit answer settling by reference, and
  a plain post settling by next post.
- **Suites**:

  | Suite | Result |
  |---|---|
  | pyagag | 892 passed |
  | relay | 342 passed |
  | agautolab | 298 passed |
  | agfront | 184 passed |
  | agforge | 265 passed |
  | agobserver | 117 passed |
  | archsage | 28 passed |
  | cagent | 202 passed |
  | frontend | type-check and build clean |

## Deployment

- `nctl drift` before touching anything: converged=46.
- Every pyagag consumer was moved to pyagag `7f6095b`.
- No agent run was in flight. The ten services were restarted one by one:
  Front, autolab listener and gateway, forge listener and service,
  Observer, archsage, cagent listener and API, and the relay. Each came
  back running the new code, and startup recovery only ignored old
  mentions.
- The `:8090` web image was rebuilt.
- The agautolab1 VM was not redeployed; it gets the new pins on its next
  deployment.
- comfynotify was left on its old pin because it does not read the post
  contract.
- The autolab confirmation path was proved by the controlled task fixture
  above rather than a live task, so no unrelated work was started.

## Live trial (`#front › front-desk-20260925-ex1-live`)

| Post | By | What happened |
|---|---|---|
| #11432 | Developer | Asks Front for one question. |
| #11434 | Front | Question (`to=8 ask=question seen=11432`); pending. |
| #11435 | Developer | Picked #11434, chose *not an answer* in the deployed room; stored `answer=none`. #11434 stayed pending. Front's chatlog header read "not an answer to any request". |
| #11437 | Front | "Your answer has not arrived yet, still waiting" — but posted as a **new** request without `re=11434`, so two were pending. |
| #11439 | Developer | Picked #11434 and answered "Blue." with `re=11434`. Only #11434 became `answered (reference)`; #11437 stayed pending, as the rules say. |
| #11443 | Front | Report with `re=11434`. |

Front runs `desk/run-0016`–`0018` cost $0.145 in total. The conversation
is left open.

## Limitations and findings

- **A duplicate question is a false wait.** When Front re-asked after the
  aside, it did not reference its earlier question, so #11437 still waits
  after the answer. The contract and read model behave as specified; the
  model did not supersede its own request. The reply guide already says
  to use `re=` when "an earlier question of yours no longer stands". One
  sample does not yet justify stronger wording; if it happens again, add
  a concrete "asking again? put `re=<old id>` on it" example there.
  Separately, Front marked its first question `re=11432`, a post that is
  not a request. This is harmless (it settles nothing) and is noted as a
  second judgement sample.
- Routine chat and the Project Room still offer only the next-post rule;
  they show the `not an answer` label but have no composer choice.
- The retry after an uncertain send uses a new submit token, as before.
  If the first attempt actually landed, a retry posts twice. The UI still
  tells the user to check before sending again.
- The agautolab1 VM runs the old pins until its next deployment.

## Commits

- pyagag `cb4568f`, `7f6095b`
- agautolab `9456a98`, `c67fa21`
- agdevworld `592dee6`
- agfront `23e9231`
- agforge `735fdcc`
- archsage `6b6d994`
- pj-clusterintent `f60295a` (cagent)
- pj-agdev `340aacb` (agobserver pin and submodule pointers)
- devdocs: step reports, `README_DEV.md` post-intent section, parent
  `report.md` contract.

Machine-specific facts (fixture relay, versions, restart time) are in
`pj-agdev/.local/devenv.md`.

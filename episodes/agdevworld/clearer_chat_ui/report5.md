# clearer_chat_ui step 5 — validated, deployed, tried live

## Checks

| suite | result |
|---|---|
| pyagag (contract, read model, lifecycle, the new end-to-end fixture) | 859 passed (`4ea6356`) |
| agentroom relay | 339 passed |
| agfront / agautolab / agforge / agobserver / archsage / cagent | 184 / 293 / 265 / 117 / 28 / 202 passed |
| agdevworld frontend | `tsc --noEmit` clean, `npm run build` clean (the usual chunk-size advisory) |

## The sequence, with faults, in a fixture

`pyagag/tests/test_clearer_e2e.py` runs the real listener and `serve_topic`
over a mirrored fake realm (3/3 repeat runs green):

1. human request → the run posts **progress** and asks **a question**;
2. the process **crashes** right before the question is sent → nobody is
   asked anything; **restart** → the prepared question is delivered once,
   no model run;
3. **another agent interrupts** → it is answered; the question still waits;
4. the human answers → next-post settles it; **input arrives during that
   run**; the run's second question carries `seen=` from before that input
   → `overtaken`, not waiting; the listener serves the new input, and the
   agent withdraws the second question with `re=`;
5. the human finishes → a **report**, nothing pending;
6. another restart → the same `as_dict()` from the same record, no run;
7. a **stale** source → the same states, said to be stale.

Browser checks against the real relay code over a fake realm and the demo
are in report3 (two pending questions, picking one, reload, both views,
history).

## Deployment

Before touching anything, `nctl drift` (pj-clusterintent): every host
converged, every agent `polling`, the agautolab1 VM `stale` as before. No
agent run was in flight (no harness grandchild under any listener). Each
service was kickstarted on its own: Front, autolab listener and gateway,
forge listener and request service, Observer, archsage, cagent listener and
API, the agentroom relay. All came back; startup recoveries only ignored
old ✔ mentions. The web image on `:8090` was rebuilt. No desired-state or
Nautobot change was needed. The agautolab1 VM is not redeployed (gateway
only; it picks the pins up with its next deployment).

## Live trial

One Front Desk conversation, `front-desk-20260925-ccu-live`:

- Developer asks Front for two questions → Front's reply **#11424 declared
  itself** `intent=response_request to=8 ask=question seen=11422`; the relay
  read `asking`, one request pending; the deployed room showed
  `❓ QUESTION FOR YOU · waiting for your reply` and the strip
  (`live1-asking`).
- Clicking the request chip and sending → the relay wrote `re=11424` into
  **#11426**; the request became `answered (reference)` at once, the strip
  emptied, the question kept its label (`answered in #11426`).
- Front's report **#11428** declared `intent=report re=11424` on its own;
  status `answered · nothing is asked of you` after a fresh page load
  (`live4-reported-history`). The run records carry the intent beside the
  run identity.
- Cost: two desk runs, $0.10, plus the renderings.
- Found on the way: the screenshot driver splits steps on commas, so the
  typed answer was cut to "Blue. And yes" — a driver limit, recorded in the
  local notes.

## Docs

`devdocs/README_DEV.md` has a new *What a post is for* section and the
changed meaning of `awaiting_human` / new `answered` in the trace section;
the wire format and the read model are documented beside the code
(`pyagag/docs/post-intent-v1.md`). Machine facts are in the ignored
`pj-agdev/.local/devenv.md`.

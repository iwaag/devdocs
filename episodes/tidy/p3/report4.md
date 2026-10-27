# tidy p3 — Step 4 report: evidence and limits through judgement and reporting

Date: 2026-10-27 (JST). Release: agdevworld `b7d554a`; pj-ponderer `395f3c4`.

## Compared with p2's premature acceptance

The Step 1 baseline (no change deployed yet) did not repeat p2's acceptance from a report's output: every count in
ponderer's judgements was its own run. So no further generic warning was added. The changes go where the boundaries
showed missing facts or a missing route:

| p2 gap | Evidence | Change |
| --- | --- | --- |
| A result reached by `wfe request` + `ask --run` carried no evidence | p2 B/C; Step 1 code reading | the awaited result is told with its evidence and route (Step 3) |
| Reopening required the person's words | p2 C correction 1 | `--reopen --evidence` (Step 2) |
| ponderer asked before returning a correction | p2 B and C; the baseline (explicit scope) did not ask | one fact in `delegate revise` help, the receiver questions and ponderer's guide: returning what the commissioned work left out needs no go-ahead; materially new work is the person's |
| The last summary dropped a failed correction and a limit | p2 C #466 | the facts below |

## Facts at the moment of the account

- `conversation reply` (any kind but progress) lists the commissioned results with news since the run's previous
  post. It now includes runs the conductor requested and awaited in that room, and each line says how many
  correction requests a result needed and whether an earlier satisfied judgement was set aside, e.g.
  `d2 → agops evidence-fixture/run-008: judged satisfied (event 5) after 2 correction requests` and
  `agops/evidence-fixture/run-0xx (requested): result at seq 12 after 1 correction (c1; an earlier result fell short once)`.
- `results-unreported` (at a standing conductor's rest) counts awaited runs' latest result or correction too, and its
  text says that it compares record times, not what a post says. It still clears on the next post; it cannot and
  does not judge whether the post was complete or truthful.

Nothing became a gate: no new refusal, no checkbox, no required field outside the two new optional routes
(`--reopen` needs `--evidence` because the evidence is what the reopening is).

## Tests

`test/resultEvidence.test.ts`: the account phrases now carry the correction history (`after 1 correction request`);
the rest checks are asynchronous (they read awaited runs) and keep their p2 behaviour: cleared by a post, not by a
progress note or another run's post, silent while not at rest; run-048's replay still gives
`conversation-unanswered` + `results-unreported` at its first rest. `awaitedStanding` is covered in
`test/requestedResult.test.ts`.

## Next

Step 5: fresh behavioural trials on the deployed release.

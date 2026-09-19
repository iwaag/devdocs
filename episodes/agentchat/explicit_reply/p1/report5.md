# Step 5 report — Integrate, exercise, and report

Plan: [plan.md](plan.md) step 5.

## Consumers of the shared contracts

| Consumer | Conversational roles moved to the reply contract | Kept their own contract | Pin |
|---|---|---|---|
| Front (`agfront`) | `front`, `desk`, `argue`, `routine_run` (steps 2–4); callback home located by anchor; listener owns the served mark | `present` (structured) | `d20720c` (`4c17a03`) |
| autolab (`agautolab`) | `workplan_superdirector` (covering note is `output`, deterministic response lines are `sections`, repair reruns the director), `bmining_director`; entrance guide sentence | `workrun_supercoder` (report file is the contract), `entrance_front` through `agag.entrance` | `d20720c` (`a2e632a`) |
| forge (`agforge`) | `assetplan_front` (question path as `output`; the early post before a generator run is the resolved reply); entrance guide sentence | `assetplan_generator`, `assetrun_generator` (files) | `d20720c` (`944b75b`) |
| cagent (`pj-clusterintent/cagent`) | `front` (the early post is the resolved reply) | `operator_read` (answer relayed verbatim, as before) | `d20720c` (`e8e746e`) |
| archsage | `archsage` (as `output`), `sage` (resolved, then the speaker header) | — | `d20720c` (`a6352f6`) |
| Observer (`agobserver`) | argue participation through `agag.argue.participate` (step 2) | `intake` (`decision.json`), `observe` | `d20720c` (`bdd3837`) |
| generated agents | `agag.entrance` (step 2); `guide.md.in` says "your reply is posted for you" | — | with pyagag |

Every consumer's tests were re-pinned to the contract (stubs return marked
outputs; `deliver` is what fixtures intercept for the ack and the reply;
one fewer `history` read per serving) and run on its **synced venv**, not
a `PYTHONPATH` override: agfront 181, agautolab 247, agforge 245,
agobserver 72, archsage 28, cagent 202 — all passing. `pj-agdev`
submodule pointers pushed (`d4b90f2`).

## Deployment

Service state before touching anything: relay `/healthz` live (mirror
revision 3971, 0 resyncs), `nctl status` ok (Nautobot reachable,
submodules clean), every listener's queue empty and its last serving
answered (autolab's `workplan-setup-worldtrend` reply #7238 confirmed in
its mirror), no harness process running. Then per consumer:
`uv lock --upgrade-package pyagag`, `uv sync`, tests, commit, and
`launchctl kickstart -k` of the listener.

**Defect found by the deployment, fixed twice** (`740f8cc`, `d20720c`):
the v1 queue file wrote no `schema` row, so the v2 check "an existing row
that differs" never fired and every executor died on `no such column:
next_at` at its first `take()`; the first fix handled the missing row, but
the broken start had already stamped `schema = 2` over the old table, so
the second fix judges the layout by the table's own columns and a test
pins both shapes. All six listeners are up on the v2 layout as of
17:23 UTC 2026-09-20 (`recovery (startup): 0 conversation(s) queued`,
`pending` has `next_at`/`failure`, `servings` present).

**Not updated**: the `agautolab1` VM (its own checkout, Ansible-deployed,
older pyagag). Its listener keeps the old contract until
`setup_autolab_node.yml` is run; nothing on this host depends on it.

## Exercised

- **Parser, lifecycle, routing, consumer tests**: pyagag 683 passed
  (`test_serving_lifecycle` 15, `test_reply` 19, `test_handoff_binding`
  11, `test_continuation` 12, `test_end_to_end` 1, the queue-layout test,
  and the re-pinned suites); consumers as above.
- **End-to-end over local fixtures**
  (`tests/test_end_to_end.py`): the developer asks; the bot delegates by
  posting into another agent's topic with a root note carrying the
  request's anchor; its first reply is the marked text only and its
  `ag-continue` note lands at home; the delegate answers with a mention;
  the bot dies mid-callback; the restart serves the callback once, sees
  the interrupted record and the continuation view ("answered, not yet
  dealt with", the recorded goal), answers the developer at home, and the
  listener writes the served mark bound to the delegate's post.

| Measure | Result |
|---|---|
| lost replies | 0 (2 requests, 2 delivered) |
| duplicate deliveries | 0 (the read-back on every ambiguous send; step 1 fixtures cover lost response, crash after send, retries) |
| unnecessary reruns | 0 completed runs repeated; 3 runs for 3 servings (the interrupted callback run is the one repeated, by design) |
| recovery delay after the restart | 0.22 s from the crash to the final response (fixture; the real bound is the listener's startup recovery plus one serving) |
| reciprocal loop | none: nothing posted back into the delegate's topic |

- **Rendering receives only public speech**: the posted text is the marked
  reply plus notices; `[selfnote][continuation]` and the served mark are
  selfnotes and `agfront.render` / `present` filter by `is_speech`
  (fixture: `present.plain_content(post) == post` for the #7222 shape).
- **Conversation status reflects outstanding replies**: the existing
  surfaces — the listener log (`FAILED after n attempt(s) … kept in the
  queue`, `redelivering the prepared reply …`, `destination … resolved`),
  the status file's `last_error`, and `listener.sqlite` (`pending` rows in
  `failed`, `servings` with `state`, `delivered_id`, `reply_marked`,
  `reply_failure`, `extra.destination`). No new dashboard.

## Live checks pending authorization

No live post was made in this phase. Proposed, for authorization:

1. `#front › front-explicit-reply-p1-<date>` (Front Desk conversation,
   Developer account): one ordinary request ("say hello and tell me which
   agents are on the board") — checks a marked reply, no preamble in the
   post, the `reply` object in `agfront/.local/agent/front/run-NNNN.json`,
   and a `[selfnote][continuation]` note if the run writes one.
2. The same conversation: a request that delegates ("ask autolab, in its
   own channel, what it is working on") — checks the root note carries
   `#<anchor>`, the callback is answered at home, and the served mark is
   written by the listener bound to autolab's post.
3. `#argue › argue-explicit-reply-smoke` via `agentchat argue open` from
   that Front conversation: one human post — checks the argue reply's
   notices follow the marked reply and the Arguing Room re-render of the
   #7222 shape against memo record `j46335f85a54417f5f8ec` shows one turn.

Each is one paid Front run (plus one autolab run for 2); destinations are
Front's own conversations and autolab's own channel.

## Docs

- `devdocs/README_DEV.md`: new section *Explicit replies and journaled
  servings* before *How agents remember each other*.
- `pj-agdev/.local/devenv.md` (ignored): pins, the restart ritual, the v2
  queue layout and how to read it, the log lines to look for, the
  deployment defect.

## Remaining gaps

- `agautolab1` (VM) not redeployed.
- The status file does not carry queue counts; `listener.sqlite` and the
  log are the surfaces.
- The root note's anchor is written only by runs under a listener.
- `agentchat reply` (mid-run marked reply into home) stays out, per the
  advice.

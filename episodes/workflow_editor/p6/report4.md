# p6 step 4 — durable Zulip intake

Plan: [plan.md](plan.md) §Step 4. Contract: [report1](report1.md), `workflow_editor/docs/intake.md`.

## What changed

Commits:

- agdevworld `82e00d9`: `server/intake.ts`, run reducer and run files, `wfe intake`, docs;
- agautolab `bef2c07`: `wfintake.py` and its thread in `wfexec`.

The split follows the plan:

- **The editor** keeps routing, classification and run state.
- **autolab's transport** reads Zulip with the existing `agag.zulip.ZulipClient` and decides nothing. It imports no legacy mission/task listener and is not a listener: it reads only the channels the editor routes.

### The inbox (`<area>/.local/intake/intake.sqlite`)

| Table | Holds |
| --- | --- |
| `cursors` | per realm and channel, the highest message id received |
| `messages` | every post received, keyed by **(realm, message id)**, together with: its stage; its route, classification and binding (stored once); the run entry it became (`applied`: run and sequence); the reply owed to the person (`notice`, `detail`); the failure (`stage`, `error`, `attempts`, `next_try`) |
| `requests` | each request topic: identity `topic_first` (unique per realm), current name, origin post, project/workflow/workspace, run, state `opening / bound / refused` |
| `relocations` | renames of a request topic, with the post that showed them |
| `transport` | the transport's last pass: ok, or the error |

`receive` stores a channel's posts **and** moves its cursor past `--through` in one transaction. A post that is already stored is not stored again and is not reprocessed. A refused receive moves nothing.

### Routing (explicit)

A channel routes to a workflow only when all three hold:

1. the **notifier recorded** its stream id (`channel:<project>/<workflow>`);
2. the workflow, as it is now in its workspace, declares `intake.zulip: requests`;
3. the host's `zulip.json` enables the project under `intake`, which also names the **requesters** (Zulip user id → the name their words are recorded under) and `since`.

When a project has several workspaces here, one must be named; otherwise that is a visible problem. No channel or topic name selects anything.

### Application to existing operations

| Classification | Run operation (receipt `zulip:<realm>:<id>`) | Follow-up (by receipt) |
| --- | --- | --- |
| `request` | find the run by its `run.create` receipt; otherwise `createRun` with `executor: {host}` (performed as the workflow's agent), the post's exact text as `braindump.md`, `input.source`, `author` = the requester's configured name | start job `request:<receipt>` |
| `answer` | `question.answer` with `from` = the person, `source`, `via: zulip` | `afterAnswer` (the existing resume, once per answer) |
| `instruction` | `instruction.receive` (new): `instructions.i<n>` with text, `from`, `source`, `during` (nodes running then), `ack: null`. No node changes | none: it waits for the next boundary (step 5) |
| `late`, `clarify`, `closed`, `ignored`, routing problems | none | the reply is recorded on the post for step 6 |

Refusals by the run as it is at application time are recorded and explained, never applied elsewhere:

- a question closed after classification becomes `late`;
- a cancelled run becomes `closed`;
- any other `422` refusal becomes a `problem` notice.

A request whose workspace is held by another unfinished run, or whose workflow cannot run, is **refused** with the reason. A later post in that topic is `closed: refused`.

### Crash points

- **Between receipt and classification:** the post stays `received` and is classified on the next pass.
- **Between classification and the run change:** the classification is stored; application repeats the operation.
- **After the run changed, before the inbox said so:** the receipt is found in the run, so `runOp` returns `duplicate` (an answer or instruction is recorded once). For a request, `findRunByReceipt` finds the run among the workflow's runs before anything is created.
- **After application, before the follow-up:** the queue is idempotent by receipt (`request:<receipt>`, `answer:<ws>/<run>/<q>/<n>`).
- **Ordering:** posts of one request settle in id order (`wait`), so an answer cannot overtake the request that creates its run.

### The transport (`agautolab.wfintake`)

Every 5 s on a thread of `wfexec` (`[intake] enabled = true`), independent of the agent and of the slot, each pass:

1. `wfe intake due`;
2. for each routed channel, reads `messages` after the cursor, oldest first, 100 per page (anchor = cursor, so **downtime recovery is the same read**);
3. adds `sender.bot` (from `users`; a sender the realm does not list is treated as a bot) and `topicFirst` (the topic's oldest message);
4. calls `wfe intake receive`, then `wfe intake process`.

A Zulip error stops the pass where it is. The cursor stays at the last stored post, and `wfe intake transport --error` records the failure for `wfe intake status`; the next pass retries. The transport never posts.

### People's commands

- `wfe intake status`: configuration, channels and cursors, the transport's last pass, posts by stage, requests and problems;
- `wfe intake messages [--request r<id>]`;
- `wfe intake retry [<id>]`.

## Checks

- **editor**, `node --test 'test/**/*.test.ts'`: **162/162**. New tests are `test/intakestore.test.ts` (8):
  - routes come only from recorded channels of declaring workflows in enabled projects;
  - bot, stranger and run-topic posts change nothing; the first eligible post becomes the only run, with exact text, source, author, receipt, agent identity and one start job; repeated delivery stores nothing;
  - restart after the run was created and before the inbox recorded it: found by receipt, no second run, one job;
  - an answer recorded once (crash between run change and inbox), a post written before the question was posted becomes an instruction, a further answer before take-up joins the same question, and the resumption is queued once;
  - an instruction changes no node (`during: [build]`, `ack: null`); a late `q1:` reply is applied nowhere; a post after completion is `closed` and no new run appears;
  - per-request ordering; a held workspace refuses a second request with the reason; a later post there is `closed: refused`;
  - a failure is kept with its stage, shown in problems and retried from that stage; a resolved/renamed topic keeps its binding and records a relocation; a reused name is a `topic-name-reused` problem;
  - the cursor moves only with stored posts; a transport failure is visible until a good pass.
- **autolab**, `uv run pytest -q tests/test_wfexec.py tests/test_wfnotify.py tests/test_wfintake.py`: **29 passed**. New tests are `tests/test_wfintake.py` (4, real CLI, fake Zulip):
  - only the routed channel and the configured person start work; repeated passes and a restarted transport store and apply nothing twice; one start job;
  - 130 posts and a request made while the transport was down are read through the cursor (paging);
  - an outage records the error, keeps the cursor at 0 and creates nothing; the next pass receives the post and creates the run;
  - an instruction to a running node leaves it running and records `i1` from the person via Zulip.
- `tsc --noEmit`: clean.

## Limitations and notes

- **Answers cannot bind from Zulip yet.** Binding needs the request-topic message that carried the question (`posted`), which step 6 delivers. Until then `NotifyStore.postedLookup()` answers "not posted", so every reply to a question is `clarify: not-yet-asked` or an instruction. The rule and its tests stand; step 6 plugs in the lookup.
- **Replies owed to the person** (`notice` on the post) are recorded but not yet sent; step 6.
- **Polling.** Each pass costs one `messages` call per routed channel, plus `users` only when an unknown sender appears and one call per new topic. The existing client's rate budget applies.
- **Not deployed.** Release and configuration (`zulip.json` `intake`, `[intake]` in `wfexec.toml`) are step 7.

## Next entry point

Step 5: `instruction.ack` and the server-enforced boundary on node start and final completion, including documentation steps. Pending instructions also go into `wfe run show` and the execution brief.

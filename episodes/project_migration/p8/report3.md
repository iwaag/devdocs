# p8 Step 3 — shared conversation continuity

Source: agdevworld `7b16e0c` (WFE, Front Desk, relay), agautolab `3ba5abc`
(executor). Contract: agdevworld `workflow_editor/docs/conversations.md`;
lanes in `docs/workers.md`. Deployment is Step 5.

## The conversation facility (WFE)

- **Store.** `<area>/.local/conversations/conversations.sqlite` keeps conversations, messages and links.
  - The person's message keeps their words verbatim, the author and the time. Its submit token makes a resubmission one message.
  - A reply of the entrance names the run and attempt that wrote it and the messages it answers (`re`). Its kind is reply, question, progress or result. A key (default: the text's hash) makes a retried command one message.
  - A link says what a message is to a run:
    - `turn` is written by the store;
    - the rest are written by the entrance's agent: `new`, `follow-up`, `correction`, `steer`, `answer`, `question`, `informational`, `cites`.
  - A later link supersedes an earlier one, and both stay. Messages are never rewritten.
- **A turn per message.** Each message of the person starts one run of the entrance workflow (`<area>/conversations.json`), with:
  - the words as the braindump;
  - `input.source = {kind: conversation, conversation, message, author, at}`;
  - request receipt `conversation:<conv>:<message>`.
  
  The message is stored first. A failed request, or a crash between the request and its record, is started later by `wfe conversation dispatch` or the executor's reconcile. The receipt finds the same run, so nothing starts twice.
- **Front decides, the store routes.** The turn run reads `wfe conversation show` (it defaults to its own conversation) and its linked runs, then answers, works, steers, answers another run's question or asks. Any number of turns may be open in one conversation.
- **Attribution.** A reply is attributed by its run and `re`, never by recency. A question is pending until a later message of the person names it, or follows it while it is the only one pending.
- **Steering with provenance.** `wfe run instruct|answer … --from <person> --source-message <conv>:<id>` records where the words were said. From Front's attempt it records `by: front-agent`, `operator` (Step 2). An instruction continues a run nobody executes (Step 2).
  - A parent's stop does not reach its children. That is the delegation contract, and Front's guide says it explicitly.
  - Delegated children are steered through the parent (`wfe delegate instruct`).
- **HTTP** (same origin as the site on the agdev port):
  - `GET api/conversations`;
  - `GET api/conversations/<id>`: messages, links, linked runs as their records say now with run-view URLs, pending questions;
  - `POST …/messages`;
  - `POST …/close` (closes the conversation; runs continue).
- **CLI**: `wfe conversation list|show|reply|link|post|dispatch` (`wfe conversation help`).

## The human surface

- The Front Desk scene keeps its look. Its adapter now reads and posts the WFE conversations.
  - Each reply that names a run shows a `work: <run> ↗` chip to the run view.
  - The health line reads "WFE live · you post as <person>".
  - Finishing closes the conversation and says that no run stops.
  - Re-voicing into character dialogue is retired. Posts are shown as written.
- Zulip is not on the person's path. Front's runs still have their ordinary run and IO topics as evidence.
- The relay's Zulip writes that only the legacy Front served now answer 410 with where to talk instead:
  - desk posts and re-voicing requests;
  - argue posts and creation;
  - routine chat and "Ask Front to run it".
  
  Their history stays readable.

## Responsiveness: worker lanes

Scheduler facts (Step 1 inspection):
- one slot per worker, FIFO;
- a parent waiting on a delegation holds no slot, but a turn would still queue behind any long attempt on the same worker.

Decision: **lanes**.
- A worker may declare extra slots per named lane (`workers.json`: `lanes: {"conversation": 1}`).
- A workflow (or project) names its lane (`execution.lane`). It is frozen into the run as `executor.lane`.
- A claim is made in one lane, and is idle while that lane's slots are held.
- A lane the worker does not offer is served by its default lane, so a job never sticks.
- The default lane is exactly the old single slot.
- Exec store v4 adds `attempts.lane`.
- Views add `slots` and `lanes`; nothing was renamed.

In the executor (`wfexec`), each extra lane is an executor thread of the same process. The default lane keeps:
- the heartbeat, which lists every lane;
- reconcile, holding every lane's attempt;
- notify and intake;
- adoption after a restart, which hands each live attempt to its lane.

Limits:
- A remote worker refuses extra lanes for now. Its single-writer state file would need per-lane receipts and locking.
- A lane executor runs one attempt at a time.

## Tests

- `test/conversations.test.ts` (5):
  - one run per message with its source; the same token is one message;
  - a failed dispatch is started after configuration, across a store restart, and once only after a simulated crash between request and record;
  - two concurrent turns, with replies attributed by run and `re`, retried replies, and pending questions;
  - links superseding a correction, with the message unchanged;
  - from a real attempt through the CLI: `show` finds its conversation, `reply` names its run, `link`, and `instruct --source-message` records the conversation source.
- `test/lanes.test.ts` (6):
  - a conversation job is claimed while a long default attempt holds the slot, a second waits, and the default claim stays idle;
  - an undeclared lane falls back to default;
  - no lanes means exactly one slot;
  - configuration and definition validation and round-trip.
- agautolab: default and conversation executors each claim their own job; heartbeat and holding cover both; adoption is by lane; each lane carries out its own stop.
- Totals:
  - WFE: 419/419, type check and build pass;
  - agautolab: 46 passed, 1 skipped;
  - relay: 268 passed (three route tests now expect 410 for the retired writes);
  - site: type check and build pass.

Race conditions were exercised on private fixtures only. The live
responsiveness measurement is Step 6.

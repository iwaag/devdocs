# p5 step 4 — Zulip integration

Plan sections: [plan.md](plan.md) §6–§8 and §10.4.

## What was done

agdevworld `0cb9855` (`workflow_editor/`) and agautolab `daeaada`.

The split follows the plan:

- **The editor** keeps the rules and the durable state: what is notable, the message text, names, the outbox, bindings and links.
- **autolab** is the transport, using the existing `agag.zulip.ZulipClient`. No legacy mission/task model is imported, and no listener is involved.

### Hierarchy and identity

| Zulip | Entity | Name (label) | Identity |
| --- | --- | --- | --- |
| channel folder | project | `wf-<project>` | recorded folder id |
| channel | workflow | `wf-<project>.<workflow>` | recorded stream id |
| topic | run | `<run id>` | stream id + topic, followed through its last message id |

- **Names.** Over 60 characters, a name is cut and gets a hash of the whole, so two long identities never share a label.
- **Separation from legacy surfaces.** `wf-` is distinct from the legacy surfaces (`pj-<slug>`, `work-m<id>`, agent channels). This was checked against the actual routing rules:
  - every listener built on `agag` sweeps its own channel plus its topic prefixes (`workplan-`, `workrun-`, `bmining-`, `routinerun-`, `front-`, `watch-`, `assetplan-`, `assetrun-`);
  - otherwise it answers only a mention of its bot;
  - `run-…` topics match none of them, and messages mention nobody.
- **Ownership.** An existing folder or channel with our name is adopted only when its description is this integration's (`[wfe] project …`). Anything else is a visible problem with its repair path; nothing is taken over by name. An archived channel of ours asks to be unarchived.
- **Run identity.** Project + Workflow + Run, bound with the workspace and creation time. A different run under the same identity (another workspace, a re-created run) is refused as ambiguous, with `wfe notify reset <key>` as the repair. Runs created before `since` are not announced (no backfill).

### Events and messages

- **Selection.** `notableEvents()` (shared reducer module) selects:
  - started
  - a question: who answers, the `plan.md` it asked about, where to answer
  - answered
  - a node's report: its short outcome and document
  - manual policy only: a node's completion
  - failed
  - held: stopped / interrupted / unknown, with the reducer's own reason replayed from the frozen graph
  - resumed
  - completed
  - decided
  - cancelled

  Ids are `<project>/<workflow>/<run>#<seq>:<type>`. Node and step starts, progress notes, attempts and other internal transitions are not announced.
- **Publication events** come from the checkpoints: a failure or attention, the recovery, and a single "details published" update for documents a message had to call unpublished.
- **Batching.** Adjacent routine events go out as one message. Questions, failures, holds, completion, decisions, cancellations and publication notices always go alone.
- **Text** (`shared/notify.ts`):
  - a ~900-character budget shortens free text, never the actionable lines (Answer, Next, links);
  - the completion message says "Not accepted yet: … reviews the result and decides";
  - delayed messages say when the event was recorded;
  - a question that is no longer open says no action is needed;
  - the last line is a marker, `ref <workflow>/<run> #<seq>:<type>`.
- **Links** are commit-addressed Gitea links (`<origin>/src/commit/<sha>/<path in the owning repository>`, credentials stripped). They work in both devdocs modes and are given only when a published checkpoint commit holds the file. Otherwise the document is called "not yet published: only on the execution host", or "published at <sha>" when the remote has no web address.
- **Waiting for publication.** A message about a document waits while the run's attempt is open or its checkpoint is pending: routine messages up to 20 min (so a report goes out with its link, together with its neighbours), questions and blockers up to 90 s.
- **Answer route.** With no reachable `viewBase` configured (today the agdevworld route is loopback-only), questions say to answer on the execution host and give the exact `wfe run answer … q1 …` command. A `localhost` viewBase is refused.

### Durability (`server/notify.ts`, `<area>/.local/notify/notify.sqlite`)

| Table | Holds |
| --- | --- |
| targets | folder and channels, with Zulip ids or problems |
| runs | binding, identity, scan cursor, problem / skip reason |
| outbox | one row per event id; state, attempts, next try, uncertain flag, message id, unpublished documents |

- **Rebuilt from history.** The outbox is rebuilt from run history on every scan. A crash between the record and the queue loses nothing, and rows are inserted once.
- **Order and retries.** Delivery is ordered per run: nothing passes a row waiting for its retry. Backoff is 15 s doubling to 15 min. A Zulip failure never changes run state or blocks answers in agdevworld.
- **Configuration.** `<area>/zulip.json` holds `{enabled, zulipUrl, viewBase, since, person}` and no credentials.
- **`wfe notify`:** `due`, `target`, `sent`, `failed --uncertain`, `moved`, `run-problem` for the transport; `status`, `outbox`, `reset` for people.
- **Run view.** It gains "Coordination (Zulip)": the topic link, pending and failed counts, last error, provisioning problems, and the missing-remote-address note.

### Transport (`agautolab/wfnotify.py`)

- **Loop.** Each pass asks `wfe notify due` and repeats while it makes progress: folder first, then channels, then one message per run at a time.
- **Provisioning.** It creates folders with the provisioner (realm admin) credential and channels with the autolab bot. Subscribers are the realm owners and the bot.
- **Before posting** into a topic it has used, it reads the run's last message and follows a rename or move (`notify moved`).
- **Lost answers.** A send whose answer is lost is recorded `--uncertain`. A send in flight when the process died is detected through `.local/wfnotify/inflight.json`. In both cases the next attempt searches the topic for the marker line before posting. Exactly-once is not claimed: a message deleted by hand in between would be posted again.
- **Refusals.** A refusal naming the channel marks the channel target as a problem with the repair path.
- **Where it runs.** On a daemon thread of `agautolab.wfexec`, enabled by `[notify] enabled = true` in `.local/wfexec.toml`. It is independent of the agent process and of the execution slot, so it keeps running during human waits and while the queue is idle. It also runs standalone: `python -m agautolab.wfnotify [--once]`.

## Verification

### Editor: `npm run check` — 129 tests passed

`test/notify.test.ts` has 10 tests:

| Topic | What is checked |
| --- | --- |
| Names | Cut names stay distinct |
| Budget | Actionable lines survive it; the marker is last; no mentions |
| Completion | Completion ≠ acceptance; delayed and stale wording; links |
| Provisioning order | Not configured → nothing; folder → channel → messages |
| Delivery order | Per-run order; critical alone; a restarted store; the record-before-queue crash; the uncertain flag and marker; the publication wait while an attempt is open |
| Publication | Failure, recovery and the details-published update with the Gitea commit link |
| Submodule devdocs | Links into the devdocs repository without credentials |
| Identity | No backfill; duplicate identity refused and reset |
| Manual policy | Selection under manual |
| Holds | Hold reason replayed after a later resume |

### Browser: `checks/docsteps.ts` — 35/35

It now includes the coordination panel. Report icons are now re-placed when a card resizes: one run of this check found an icon left behind by a late card-height change. Three consecutive runs passed afterwards.

### autolab: `uv run pytest -q` — 346 passed

`tests/test_wfnotify.py` has 6 tests against the real `wfe` CLI and a fake Zulip:

| Test | Shows |
| --- | --- |
| Hierarchy | Folder → two channels (owners and bot subscribed, filed) → topic `run-001`; the question links the published `plan.md`; a restarted transport posts nothing twice; no mentions; autolab's `topic_filter` and `mentions_bot` do not match; no legacy prefixes |
| Lost response | Posted once, recorded uncertain, found by its marker, not reposted |
| Crash in flight | The in-flight journal plus an existing post gives one message |
| Outage | The run is answered, resumed and completed by the stub agent while every Zulip call fails; afterwards everything arrives once, in order, ending with "Not accepted yet" |
| Taken name | Nothing posted; problem recorded; after the owner renames their channel and `reset`, ours is created |
| Renamed topic | Followed |

### Real Zulip smoke test (dedicated test destination)

A throwaway area in the scratchpad, project `p5smoke`, with one synthetic run waiting on a plan question:

- It created folder **#28 `wf-p5smoke`** and channel **#271 `wf-p5smoke.smoke`**, filed in the folder, with subscribers [8 Developer, 11 autolab bot] and our description.
- It posted message **15895** in topic `run-001`. Posting by channel id works.
- The message was then set to failed + uncertain in the store. The next pass found it by its marker in the raw topic history and recorded it as sent. The topic still holds exactly one message.
- The destination is kept for the step 5 rehearsal and is to be archived after it.

## Limitations

- **Other agents' prefixes.** Their routing is checked against the prefixes their instance modules declare today. Their packages live in other virtual environments and are not imported by the test.
- **Answering from outside.** No remote address of the agdevworld UI exists, because it is loopback-only by p4 design. Answering from a remote device is therefore impossible, and the messages say so. Remote hosting is outside p5.
- **Configuration and cursor.** The integration is configured per area (`zulip.json`). `since` is the no-backfill cursor and must be set when the integration is first enabled on a host.
- **Duplicates.** Exactly-once delivery is not claimed (see Transport).

## Next entry point

Step 5:

- rehearsal of both policies and failure paths on isolated data with the `wf-p5smoke` destination;
- system release deploy;
- `zulip.json` and `[notify]` on this host;
- the RTS definition update;
- a human-led real run.

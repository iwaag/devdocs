# p6 step 7 — failure paths and deployment

Plan: [plan.md](plan.md) §Step 7.

## Final suites

| Suite | Result |
| --- | --- |
| editor `node --test 'test/**/*.test.ts'` (source, before release) | 171/171 + 1 added in this step |
| editor `npm ci && npm run check` **inside the release** (`wfe-release.sh build`) | type check, **172/172** tests, production build |
| autolab `uv run pytest -q tests/test_wfexec.py tests/test_wfnotify.py tests/test_wfintake.py` | 30 passed |
| autolab `tests/test_wfconversation.py` (new) | 1 passed |
| browser `checks/exec.ts` | all passed (p4 execution panel, answers, stop/resume, publication, acceptance; no page errors) |
| browser `checks/runs.ts` | all passed (33: run views, questions, branches, outages, history) |
| browser `checks/conversation.ts` (new) | all passed (12) |
| browser `checks/docsteps.ts` | **fails** at "every report icon sits at its card's lower-right corner" (a null bounding box). It fails **identically on the released `78caec2`**, run from a temporary worktree of that commit, so it is pre-existing and not caused by p6. Not investigated further in p6 |

All failure-path acceptance uses isolated, deterministic tests: temporary areas and registries, real `wfe` CLIs, fake Zulip transports and injected crashes. The one live call went to the dedicated rehearsal channel (below).

## The required checks

| # | Check | Evidence |
| --- | --- | --- |
| 1 | Only configured channels, request topics and authorized senders start work; bot and run-topic posts do not | `intake.test.ts` routing and classification tests; `intakestore.test.ts` "bots, strangers and run topics change nothing"; `test_wfintake.py::test_only_routed_channels_and_requesters_start_work_once` (an unrouted channel, `run-001`, `chatter`, a bot and a stranger in the same pass: one run); live smoke (bot post ignored) |
| 2 | Duplicate receipt, restart after run creation and restart after answer application each give exactly one run operation | `intakestore.test.ts`: repeated `receive` stores nothing; a crash after `createRun` is found by its `run.create` receipt (no second run, one job); a crash after the run recorded the answer gives `duplicate` (one answer, one resume). `test_wfintake.py`: a restarted transport stores and applies nothing again |
| 3 | Downtime recovery reads missed posts without reapplying handled ones | `test_wfintake.py::test_posts_made_while_down_are_read_through_the_cursor` (130 posts across pages plus a request, read by a restarted transport through the cursor); `…outage_is_visible_and_loses_nothing`; live smoke: cursor 0 → 15915, then a second pass received 0 |
| 4 | Answers keep sender and question identity; ambiguous and stale replies cannot approve another question or plan revision | `intake.test.ts`: a post older than the question post never answers it; ambiguous replies become `clarify`; a late `q1:` reply never moves to the newer `q2`. `conversation.test.ts`: answers carry `from`, the Zulip `source` and `via: zulip`; a late reply applies nowhere; an ambiguous reply gets the clarification. `instructions.test.ts`: a revised plan is asked about with its own document evidence |
| 5 | An answer arriving before the asking attempt exits resumes through the existing lifecycle | `conversation.test.ts::an answer arriving before the asking attempt exits…`: the question is posted and answered from Zulip while the attempt is open, so no resume is queued yet. `finished()` then queues it, and the next claim's brief says "resume: answer 0 to q1 recorded" |
| 6 | Input during a node does not interrupt it but blocks the next start until taken up; the final-completion boundary; races with receipt | `instructions.test.ts` (start, last completion, documentation steps, 12 concurrent start/receive trials); `test_wfexec.py::test_instruction_during_a_node_is_taken_up_at_the_boundary` (real loop: the attempt acknowledges mid-attempt and ends `exited`) |
| 7 | Acknowledgment survives restart; a stale attempt cannot acknowledge or change execution state | `instructions.test.ts`: a stale attempt and a non-attempt are refused, and a fresh read replays the acknowledgment. Existing attempt-fencing tests (p4) cover the other execution operations |
| 8 | Zulip outages, uncertain sends and retry keep the notification guarantees across both destinations | `test_wfconversation.py`, with the real editor CLI and the intake, notifier and executor transports against one fake Zulip: a Zulip outage at the first intake pass loses nothing; the first post to the request topic loses its answer and is **found by its marker, not reposted**; a later outage still delivers once. Marker tokens are unique per topic, the order is kept, and no message mentions anyone. `conversation.test.ts`: an uncertain request-topic delivery is searched for only in the request topic. The p5/ex1 notifier tests (`test_wfnotify.py`, 19) pass on the new code |
| 9 | Planning agreement cannot become result acceptance; final approval records the correct evidence without duplicate human approval | `conversation.test.ts`: `decide --question q1` (the plan question) is refused as `not-result-confirmation` ("plan agreement is not acceptance of the result"); the confirmation records question, answer, sender, Zulip post and deliverable hashes; a second decision is `decision-exists`; changed deliverables are `subject-changed` |
| 10 | Published presentation and preview links are usable by the requester | Links are commit-addressed `src/` (view) and `raw/` (download) Gitea URLs (`notify.test.ts` asserts both, with no credentials). Anonymous `HTTP 200` on the LAN for `developer/pj-ppt` and a raw file of `developer/pj-rts-vs-bot`. The real deck's links are verified in step 8 |

## Live smoke test (dedicated destination)

A throwaway area in the session scratchpad (project `p6smoke`, workflow `smoke` with `intake: {zulip: requests}`) was routed to the p5 rehearsal channel **#271 `wf-p5smoke.smoke`**. Real project topics were not used.

- The Omni Agent account (user 9, a bot) posted **#15915** in topic `request-p6-smoke`.
- One `Intake.pass_once()` with autolab's bot credential:
  - read #271 from cursor 0 (8 posts, including p5's seven rehearsal messages), using the real `users` and first-message lookups;
  - recorded the seven old posts as `ignored (before-intake)` and #15915 as `ignored (bot)`.
- No run was created. A second pass received 0, and the cursor stood at 15915.

The scratch area went with the session; the post stays in the rehearsal channel.

## Deployment

**Preflight** (21:45 JST):

- the executor was idle with nothing queued;
- `rts-vs-bot` `improve-game/run-001` and `run-002` were both completed;
- `wfe notify deliveries` showed nothing sending, failed or possibly posted;
- no attempt was open.

**Backups** in `.local/agdev/.local/backups/p6-20261011-214513/`:

- `exec.sqlite` (user_version 1) and `notify.sqlite` (2), via `.backup`;
- `zulip.json` and `wfexec.toml`.

**Editor** `agdevworld/scripts/wfe-release.sh deploy .local/agdev` built and checked release `0c2c072` (172/172 inside the release). It switched `service/current` (previous: `78caec2`), restarted `com.agdev.wfe-service`, and reported "healthy: the service and bin/wfe run agdevworld 0c2c072…".

**Migration:** `notify.sqlite` is at user_version **3**, with the existing rows kept as `kind = run`. `intake.sqlite` was created (v1).

**Configuration**

- `.local/agdev/zulip.json` gained:

  ```json
  "intake": {"enabled": true, "since": "2026-10-11T12:46:04.000Z",
             "projects": {"ppt": {"workspace": "ppt"}},
             "requesters": [{"id": 8, "name": "iwaag"}]}
  ```

  User 8 is the realm owner "Developer", the person; their words are recorded under `iwaag`, the name p5 used.
- `agautolab/.local/wfexec.toml` gained `[intake] enabled = true`.
- `wfe intake status` routes **#275 `ppt/create`** and **#276 `ppt/study`**.

**autolab:** `launchctl kickstart -k gui/<uid>/com.agdev.agautolab-wfexec` started three things:

- the executor, with the new claim and prompt composition;
- the notifier, which took over as sender (the new pid replaced the old, whose process was gone);
- the intake thread (`intake: workflow requests: reading routed Zulip channels every 5 s`).

The first live pass read both channels: transport `ok`, 0 posts.

**Commits**:

- agdevworld `0c2c072` (released) and `1d87d56` (docs only: the `intake` key of `zulip.json`);
- agautolab `9a9aa98`;
- pj-agdev `f47311b` (submodule pointers).

The local `devenv.md` records the host facts, including the rollback below.

### Breaking changes and what they affected

| Change | Effect on existing data |
| --- | --- |
| Compiler `ag.wfe-compile.v2` | Existing runs (v1 graphs) are followed as frozen; `rts-vs-bot` `run-001`/`run-002` check usable |
| Executor `host` | Old runs record no host; `hostOf` falls back to the name (`autolab`), so ownership is unchanged |
| Notify schema 3 | Additive columns; no reset needed |
| Common contract guide | The RTS workflows are now launched with the agent-neutral contract (no RTS run is active) |

**Rollback is not one command.** The released `78caec2` refuses a notify store newer than its schema ("newer than this release understands"). A rollback is therefore:

1. `wfe-release.sh rollback`;
2. restoring the backup's `notify.sqlite`, which is safe only while nothing has been delivered since the backup (otherwise messages replay);
3. removing `[intake]`;
4. autolab back to `ca8e807`.

`test_wfnotify.py`'s rollback test still passes, but it rolls back to the pre-p5/ex1 release, which predates schema versioning.

## Limitations

- `checks/docsteps.ts` fails as described above, on the old and the new release alike.
- **The deliverable links have not been opened by the requester yet.** That is step 8, with the real deck.
- **The requester is one configured person.** Other Zulip users' posts in `wf-ppt.*` request topics are recorded and ignored (`unauthorized`), without a reply.

## Next entry point

Step 8: the person starts a `study` request in `wf-ppt.study` (a new `request-…` topic). Then a `create` request in `wf-ppt.create`. Both are conducted in their request topics.

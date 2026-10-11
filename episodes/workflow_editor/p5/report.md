# Workflow editor p5 — final report

## Result

P5's implementation and first real default-policy RTS trial are complete. The person requested an improvement through agdevworld, agreed to its plan, and reviewed the result. Autolab produced the plan, three per-node reports, and a final report. The execution host published them with the game changes to Gitea, and Zulip received concise coordination messages with commit-addressed document links.

The final review on 2026-10-11 independently checked the live run, a fresh Gitea checkout, document evidence, headless game checks, and the actual Zulip messages. This confirms the normal operating path. Delivery reliability is not fully closed: the review also identified a reproducible duplicate-notification case and an intermittent SQLite lock error, described below. Those findings remain unresolved and must not be read as passing every failure-path requirement in [plan.md](plan.md).

The person's approval is recorded in the `human-check` answer. The separate result-decision list is empty; execution completion does not imply an explicit `accepted` decision.

## Implementation delivered

| Area | Result | Implementation evidence |
| --- | --- | --- |
| Workflow contract | `ag.workflow.v2` declares `documentation.policy: default \| manual`; the policy participates in definition approval | [report1](report1.md) |
| Effective graph | Default generates one plan talk, one report per authored node, and one final report; dependencies wait for reports. Manual adds none | [report1](report1.md) |
| Run recording | The effective graph and instructions are frozen. Documents are checked for presence, nonempty content, confinement, hash, and size; plan completion is tied to a taken-up answer about that plan revision | [report2](report2.md) |
| Execution | Generated steps use the existing attempt, fencing, stopping, and resumption rules. Interrupted reporting can resume without repeating completed work | [report2](report2.md) |
| Views | Plan talk, attached report icons, final-report endpoint, split work/documentation counts, document inspection, and reporting state | [report3](report3.md) |
| Zulip | Project folder, workflow channel, run topic; durable bindings/outbox, batching, retries, publication links, and integration status | [report4](report4.md) |
| Rehearsal and rollout | Real agent on isolated default/manual tasks; system deployment, notification configuration, and explicit RTS workflow update | [report5](report5.md) |
| Request-form correction | Refresh the retained run-request card after project data arrives, so workflow options appear even when run data arrives first | agdevworld `32498e9` |

The editor remains agdevworld system code. No editor project, self-development run, or deployment from a managed project's repository was reintroduced.

Devdocs is the work-record authority. Zulip supplies coordination summaries. Queue, process, publication, and notification-delivery state remain local operational data. P5 does not receive answers or commands from Zulip; the person answers in agdevworld.

## Real RTS run

| Fact | Recorded value |
| --- | --- |
| Project / workspace | `rts-vs-bot` |
| Workflow / run | `improve-game/run-002` |
| Policy | `default`, frozen at creation |
| Person | `iwaag` |
| Request | Add purchasable unit-strengthening upgrades at the barracks |
| Agent | autolab |
| Backend recorded by attempts | `claude_code / anthropic/claude-sonnet-5` |
| Final state | Sequence 29; all 3 authored work nodes and all 5 documentation steps completed |
| Human answers | q1: `オッケー!`; q2: `オッケー！`, both through the browser and taken up by autolab |
| Explicit result decision | None recorded |

The reviewer did not create the request, answer the questions, or record acceptance for the person.

### Execution and publication

Times below are Japan Standard Time (UTC+9).

| Attempt | Window on 2026-10-11 | Result | Published root commit |
| --- | --- | --- | --- |
| `amv3kgoch-2fb3ae` | 17:33:51–17:35:14 | Wrote the plan, asked q1, exited for the human wait | `0b825c7` |
| `amv3kipjm-540a02` | 17:35:26–17:41:29 | Took up agreement, implemented and verified upgrades, wrote both reports, asked q2 | `8e21812` |
| `amv3kqsv8-7cff74` | 17:41:43–17:42:50 | Took up q2, completed human-check and its report, wrote the final report, completed execution | `22b65d3` |

All three attempts are ended. All three corresponding publication checkpoints are `published`.

The implemented game feature includes attack and armor upgrades, each costing 200 ore and taking 30 seconds, purchasable once per player at a barracks. They apply to existing and subsequently trained soldiers and heavy warriors. The bot does not purchase them in this run's scope.

### Durable documents

The final commit contains:

```text
devdocs/runs/improve-game/run-002/
  braindump.md
  plan.md
  report-implement.md
  report-verify.md
  report-human-check.md
  report.md
  definition/improve-game.yaml
  run.json
```

The recorded hashes and byte lengths of all five generated documents match their contents in the fresh checkout. `_plan.agreement` names q1, whose answer was taken up. All document steps have completed evidence.

[The final run report](http://agstudio.local:3000/developer/pj-rts-vs-bot/src/commit/22b65d3a28e1b37e983163d317241203d448dde1/devdocs/runs/improve-game/run-002/report.md) describes the delivered feature and its limitations. [The verification report](http://agstudio.local:3000/developer/pj-rts-vs-bot/src/commit/22b65d3a28e1b37e983163d317241203d448dde1/devdocs/runs/improve-game/run-002/report-verify.md) explicitly states that the executing agent did not perform browser testing of the game. The human answer does not establish which browser interactions the person tried.

## Zulip verification

The reviewer read messages 15907–15914 from the actual topic, rather than relying only on local sent flags:

- Folder: `wf-rts-vs-bot`.
- Channel: `wf-rts-vs-bot.improve-game` (stream 274).
- Topic: `run-002`.
- Nine selected events delivered as eight messages; the implementation and verification reports were combined into one post.
- All nine outbox rows are `sent`, with no delivery errors or retry attempts for this run.

| Message | Content checked |
| --- | --- |
| 15907 | Request and start, with planning as the next action |
| 15908 | Plan question q1, plan link at `0b825c7`, and instructions for answering on the execution host |
| 15909 | Person's answer to q1 recorded |
| 15910 | Implementation and verification summaries, links at `8e21812`, 23 passing checks, and explicit disclosure that game browser verification was not performed |
| 15911 | Human-check question q2 with the same verification limitation |
| 15912 | Person's answer to q2 recorded |
| 15913 | Human-check report link at `22b65d3`; uncertainty about the person's browser checks retained |
| 15914 | Execution completion, final-report link at `22b65d3`, and a separate reminder that the result decision has not been recorded |

The messages agree with the run's records. No duplicate messages were observed for this normal-path run. Documents referenced by the messages exist at the published commits. The final message distinguishes execution completion from acceptance, even though its quoted agent outcome describes the human-check approval.

The topic can be opened in [Zulip](https://agstudio.local:8543/#narrow/channel/274-wf-rts-vs-bot.2Eimprove-game/topic/run-002).

`viewBase` remains unset because the editor route is loopback-only. Messages accurately say to answer through agdevworld on the execution host and include the CLI answer command. They do not advertise a remotely usable localhost link. Replies in Zulip do not resume the run.

## Independent checks

| Check | Result and scope |
| --- | --- |
| Fresh authenticated Gitea clone, including submodules | HEAD `22b65d3a28e1b37e983163d317241203d448dde1`; no changes made to the operating registry |
| Generated document evidence in that clone | All five hashes and byte lengths match `run.json` |
| `node game/test/headless.js` in that clone | All 23 checks pass, including payment, duplicate purchase rejection, simultaneous distinct research, completion, existing/new unit HP, and increased attack damage |
| Installed `wfe run check improve-game/run-002 --workspace rts-vs-bot` | Usable, exit 0 |
| Live run view through `/wfe/` | Sequence 29; completed; 3 work nodes, 5 documentation steps, 3 attached report icons; no browser page errors |
| Actual Zulip posts and local delivery state | Eight posts checked against nine sent event rows and the run history |
| Request-form fix, before rollout | `npm run check`: 129 tests passed, type check and build passed; `checks/exec.ts` passed including forced runs-before-project ordering and preservation of request inputs |
| Installed system release | `32498e9`; activation checked that service and CLI report the same release |

The final review reused the recent code checks rather than rerunning the full suite without a new code change. Earlier default/manual, branch/join, interruption, and synthetic outage evidence is recorded in reports 1–5. The real RTS trial demonstrates a linear default-policy workflow and normal notification delivery, not every failure case or the desired 7:2:1 reading frequency.

## Open findings and limitations

### Notification deduplication when a retry batch changes

The earlier lightweight review reproduced this unresolved case using an isolated `NotifyStore`:

1. A routine event is posted, but the send response is lost.
2. The event remains uncertain in the outbox.
3. Another routine event arrives before retry.
4. `due()` combines both events, changing the marker from `ref ship/run-001 #3:node-completed` to `ref ship/run-001 #3:node-completed #5:node-completed`.
5. The transport searches for the new marker and cannot identify the original successful post, so the original event can be posted again.

Relevant code: `workflow_editor/server/notify.ts` batches pending rows on each delivery; `agautolab.wfnotify` reconciles uncertain delivery by comparing the complete marker. The previously tested single-message lost-response case does not cover a growing batch.

Follow-up: persist the attempted delivery batch and marker until reconciliation finishes, or reconcile individual stable event identities before constructing a new batch. Add a failure-path test where work progresses between a successful post with a lost response and its retry. This is a coordination reliability issue; it does not alter the run's work record or repeat the game work. Do not claim that all duplicate-delivery cases are solved.

### Intermittent SQLite lock error

The lightweight review ran `uv run pytest -q tests/test_wfnotify.py`: five tests passed and `test_long_outage_keeps_work_and_delivers_in_order` failed with `database is locked` while opening `ExecStore`. Rerunning that test alone passed. The failure occurred at the constructor's SQLite PRAGMA setup, which sets WAL before its busy timeout.

This is evidence of an intermittent concurrency problem, not a confirmed diagnosis of its full cause. Follow-up: investigate startup locking and contention across the service, executor, and notifier, and verify recovery under concurrent access. No such error was observed in the inspected real run.

### Remaining product limits

- `decisions` is empty. Human-check approval and a separate result decision are distinct; the reviewer did not create a decision for the person.
- The game's browser controls and rendering were not independently tested in this final review. The published reports disclose that omission.
- The run header still displays backend as unknown although attempts record the backend; this is the existing display gap from p4.
- `build-game` remains an unsupported v1 definition pending an explicit policy choice. `improve-game` is the valid default-policy path used here.
- Older browser checks had existing failures recorded in [report3](report3.md). They were not repaired by p5 or declared passing here.
- The isolated `wf-p5smoke` rehearsal destination remains available; archival is optional cleanup.

## Completion assessment

The real-run requirement that was pending in [report5](report5.md) is now fulfilled: actual human input and answers produced the planned documents and game changes, published evidence is readable from Gitea, and actual Zulip coordination reflects the recorded work and its limitations.

P5 has delivered its normal operating workflow and a human-reviewed RTS trial. Full delivery robustness remains open because of the retry-batch duplication case and the intermittent lock failure. Keep those findings as explicit follow-up work rather than marking every failure-path criterion unconditionally complete. Result acceptance remains unrecorded as a separate decision.

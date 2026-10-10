# workflow_editor p4 — Stage 3: execution by autolab alone

Stage 3 of [plan.md](plan.md) is implemented:

- pj-agdev `91be3bb` (`experiments/workflow_editor`)
- agautolab `984f128` (`agautolab.wfexec`)

All evidence here is synthetic. The executor's steps are driven by tests, or
by autolab's loop running a stub agent. No real agent run has happened yet;
that is Stage 4.

## Implementation

### Durable facts in run.json (`shared/run.ts`)

`run.json` gains a `control` block and four operations:

| Operation | Recorded by | Effect |
| --- | --- | --- |
| `attempt.begin {attempt, reason, backend}` | the execution host, when it claims a job | opens an attempt, recording which backend serves it (Agent ≠ Model) |
| `attempt.end {attempt, outcome, detail}` | the execution host, when the process has ended | outcome is `exited`, `stopped`, `interrupted` or `unknown` |
| `run.stop {reason}` | the person | a request while an attempt is open; otherwise an immediate hold |
| `run.resume {reason}` | the person | clears a hold; the reason is the instruction to the next attempt |

**Fencing.** While an attempt is open, the executor's reports must carry its
id. The executor's reports are start, progress, wait, complete, fail, ask,
take-up, withdraw and attach.

- A report from a stopped or replaced attempt is refused (`attempt-stale`).
- A report without an id is refused (`attempt-required`), so nobody else
  records execution meanwhile.
- The person's operations stay open: answer, decide, stop, resume, cancel.

**The reducer decides what an end means.** This rule is not in Python. An
`exited` attempt is recorded as `interrupted` when either:

- a node is still recorded as running; or
- work is ready and nothing waits on a person.

A stop becomes a `stopped` hold only when the attempt's end confirms the
process is gone. The hold names the person who stopped it. Cancellation stays
final: every later report, resume or attempt is refused.

**Receipts.** Every entry may carry a `receipt`. A retransmission with the
same receipt returns the record unchanged.

**Situation.** `situation()` derives one of: executing, stopping, awaiting a
person, stopped, interrupted, unknown, completed, cancelled. It never treats
`running` as proof that a process is alive.

**Notification boundary.** `notableEvents()` and `wfe run events` select
events from committed history entries:

- the types are acknowledged, question, answered, blocked, stopped,
  completed, decided and cancelled;
- each has a stable id `<project>/<workflow>/<run>#<seq>`;
- nothing is sent.

### Locking

Every run operation reads, checks the version, applies the transition and
writes atomically under the run's cross-process lock (`server/lock.ts`, a
SQLite exclusive lock). The CLI, the service and the executor therefore never
lose each other's records. `expectSeq` remains as the stale-view guard.

### Execution management on this host (`server/exec.ts`)

`<area>/.local/exec/exec.sqlite` holds the jobs, attempts (pid, times, exit,
log), checkpoints and the executor's heartbeat.

- **Requests.** `request()` creates the run with the person's words as
  `braindump.md` and queues its start under the request's receipt. The same
  receipt queues once.
- **Workspace holding.** A run of the executor that has begun and has neither
  completed nor been cancelled holds its workspace. Another request there is
  refused; another workspace proceeds.
- **One slot.** `claim()` takes the oldest queued job only when no attempt is
  open. It drops jobs whose run is cancelled, held or already has an open
  attempt. It notes the workspace's uncommitted paths as the attempt's
  publication baseline, then records `attempt.begin`.
- **Prompt brief.** The claim includes a brief for the prompt: the run, the
  workspace, why this attempt, how the last one ended, and the person's resume
  instruction.
- **Answers.** `afterAnswer()` runs after the answer is persisted and queues
  the resumption once per answer, by receipt. It does nothing for a held,
  executing or not-waiting run.
- **Process end.** `finished()` records `attempt.end`, publishes a
  checkpoint, and queues a resume if an answer arrived during the attempt.
- **Stop and cancel.** `stop()` and `cancel()` mark the open attempt for
  termination; the executor polls `stops`. `resume()` records
  `run.resume` and queues.
- **Recovery.** `reconcile()` runs after restarts:
  - an attempt whose process is gone ends as `unknown`;
  - a live process keeps the slot;
  - requests and answers whose queue entry was lost are queued again under
    their receipts;
  - nothing is completed, failed or rerun by itself.

### Publication (`server/publish.ts`)

Checkpoints happen after a process has ended and after a result decision.

1. The paths changed since the attempt's baseline are committed and pushed.
2. Submodules go first, devdocs first among them, then the root that records
   their gitlinks.
3. A gitlink is never recorded for a commit the remote does not have.

Changes are sorted as follows:

| Change | Treatment |
| --- | --- |
| Under the run's own folder | always the run's |
| Present before the attempt, unchanged since | left out (shown as pre-existing) |
| Present before the attempt, changed again | left out; the checkpoint needs attention |
| A push rejected because the remote moved | attention; nothing is forced |

A failed checkpoint is retried from its journal: committed steps are only
pushed, so the work is not repeated. Work completion and publication are
shown separately.

### autolab's loop (`agautolab.wfexec`)

The loop has one slot. Each tick:

1. Heartbeat; and every minute, `wfe exec reconcile`.
2. Carry out due stops: SIGTERM to the agent's process group, then SIGKILL
   after 20 s.
3. Report a finished process (`wfe exec finished`).
4. Claim and launch the next job.

The agent runs:

- built by pyagag's `build_argv` for the new role `wfrun` (sonnet,
  `claude_code`), or by the `agent_command` setting for trials;
- in the project's workspace, in its own session;
- with `WFE_ATTEMPT` set and `wfe` on PATH;
- with the prompt on stdin and its stream-json log in
  `.local/exec/logs/<attempt>.jsonl`.

The guide is `agent/guides/wfrun/guide.md`:

- **Head:** who autolab is in this role, who reads its records, what it can
  see.
- **Facts:** attempts and new sessions, the fixed definition, the node types,
  a talk node ends the session, record-as-it-happens, interruption,
  path-scope access, publication not being its job, refusals.

If the loop itself exits, the agent keeps running; a restarted loop finds the
slot held.

Legacy listeners: the agautolab Zulip listener serves only `workplan-` and
`workrun-` topics, and requests through agdevworld never reach Zulip. So no
legacy path can execute the same request.

### Browser (agdevworld → `/wfe/`)

- **Project Editor → "Request a run".**
  - Choose a workflow (delegate-holding ones are disabled), give your name
    and your words.
  - The request receipt is kept until the service confirms, so a retry after
    a lost response is not a second run.
  - A refusal because the workspace is held is shown.
- **Run view header:** a situation chip (Executing, Stopping, Awaiting a
  person, Stopped, Interrupted, Unknown). The note no longer refers to a VS
  Code conversation.
- **Run view "Execution" section:**
  - the situation, and whether this host sees the process alive;
  - the hold, with who and why;
  - the executor's availability;
  - Stop, and Resume with a required instruction;
  - the queued jobs and every attempt (outcome, exit, backend);
  - publication checkpoints per repository: commit, pushed, left out, needs
    you, errors; and Retry.

  This host's facts are re-read every 3 s; record changes arrive through the
  watcher as before.
- **Dashboard:** the execution host's availability comes from the heartbeat.

## Validation

| Command | Result |
| --- | --- |
| `npm run check` | 87 tests pass, 0 fail; tsc and build clean |
| `node --test test/exec.test.ts` | 9 pass |
| `uv run pytest -q tests/test_wfexec.py` (agautolab) | 3 pass; the whole agautolab suite passes (338) |
| `node checks/exec.ts` | 20 browser checks pass |
| `node checks/runs.ts --repeat 10`, `node checks/agdev.ts` | all pass after the changes |

The completion criterion is a synthetic task going from a request through a
human wait, process exit, answer, resumption and completion. It is met three
times:

- in `test/exec.test.ts`;
- by autolab's real loop with a stub agent (`test_wfexec.py`): the agent
  asks and exits, the person answers through the CLI, attempt 2 takes the
  answer up and completes, and the result is published to the remote;
- in the browser (`checks/exec.ts`).

The plan's §8 items covered, and where:

| §8 item | Evidence |
| --- | --- |
| 5. Concurrent answers and progress, stale versions, duplicates | Two OS processes write 10 progress notes each while 5 answers are written concurrently: 20 notes, 5 answers, one contiguous history. A stale `expectSeq` is refused. A duplicate answer receipt is recorded once; a duplicate request receipt is queued once. |
| 6. Talk: question, answer, take-up, completion, acceptance stay distinct | All flows above; the answer is not taken up until attempt 2 does it. |
| 7. No process runs through a human wait; other workspaces proceed | The attempt has ended and the situation is awaiting a person. A second request in the held workspace is refused, while one in another workspace is claimed and runs. |
| 8. Restart, abnormal exit, stop then immediate resume, late updates after cancel | A process killed after a side effect but before recording completion leaves an interrupted hold: build is still running, nothing is queued, and a smuggled job is dropped. An explicit resume continues from the working tree. A live process survives an executor restart without a duplicate launch; once dead it becomes `unknown` and is not rerun. A late report after stop, or after cancel, is refused. |
| 9. Retrying a root publication after a child repository was published | The root push fails while devdocs is already pushed. The retry pushes the same root commit with no new commit. A pre-existing file is left out, and a pre-existing file the run changed again needs attention. |
| Progress visibility (target about 2 s) | 10 recorded steps were visible in the run view within 221–1175 ms (median 504 ms), with the 1 s poll and the executor writing through the CLI path. This is one local measurement, not a guarantee. |

Screenshots: `pj-agdev/.local/workflow-editor-p4/screenshots/p4-run-awaiting.png`,
`p4-run-stopped.png` and `p4-run-accepted.png`.

## Remaining limitations

- **Not yet deployed.** No launchd jobs exist yet for `agautolab.wfexec` or
  for the operating editor service; `.local/wfexec.toml` is not written.
  Stage 4 deploys them, from a copy of the editor separate from the source
  being edited.
- **No real agent yet.** The `claude_code` launch has not run; the trials
  used a stub agent. The first real attempt is Stage 4's first improvement
  run.
- **Single host.** Liveness of an attempt on another host is shown as "on
  another host"; nothing crosses hosts.
- **Overlapping checkpoints.** If an earlier checkpoint stays failed and a
  later attempt changes the same files, the later baseline counts them as
  pre-existing. Retrying the earlier checkpoint publishes them. This is
  documented, not solved.

## Next stage entry point

Stage 4:

1. Inventory the RTS and editor experiment workspaces.
2. Settle each project's root.
3. Create the repositories on Gitea under `developer`, and register the
   projects with workflows.
4. Obtain fresh workspaces.
5. Deploy the editor service (launchd, from a deployed copy) and
   `agautolab.wfexec`.
6. Run one improvement task per project through agdevworld, with the
   person's answers and acceptance.

# p9 step 3 — Holds that reserve nothing, and who acted

Step 3 of [plan.md](plan.md). Committed in two places:

- agdevworld `be656f1`.
- agautolab, branch `p9`, `1593144`. This is the wfrun guide; with step 2 it
  goes out at deployment.

Nothing is deployed.

## What changed

### No run reserves its project

- `ExecStore.holder()` is gone. It was the workspace reservation from a
  run's first attempt to its end. Neither `wfe request`/the browser nor the
  Zulip intake asks for it any more.
- Every run works in its own copy (step 2). The host's one execution slot is
  the only thing requests share. A run has at most one queued or claimed job,
  as before.
- `run.stop` drops only that run's queued job and asks the executor to
  terminate only that run's attempt. Another run's queued job survives.
- A run without a copy of its own is no longer executed: its job is dropped
  with "migrate it first". Such runs are recorded before p9; step 6 migrates
  them. Only one execution mode remains.
- A resumed attempt continues in its own copy, from the run's base. Its brief
  says how many commits the project's targets gained since then and where to
  look ("none of them is in this copy"). Nothing is merged, and plan
  agreement is neither reset nor assumed.

### Who acted (`shared/run.ts`, `cli/run.ts`)

- `run.stop`, `run.resume` and `run.cancel` take `because`, which is the
  person's answer (`question`, optional index) or instruction that the
  recorder acted on.
  - The reducer checks that the answer or instruction exists, then records
    whose words they were.
  - The record goes in the hold (`control.hold.because`, carried through
    `control.stop`) and in the cancellation (`cancelled.because`).
- A control operation that carries an attempt id must be the open attempt's,
  so a stale agent cannot stop or resume anything.
- CLI rules:

  | Who | Recorded as | `--by` |
  | --- | --- | --- |
  | An executing attempt (`WFE_ATTEMPT`) | the run's agent, in every run command | Another name is refused, with a pointer to `--because` (stop/resume/cancel) and `--from` (answer/instruct). |
  | Anyone else, on a run the execution host performs | the name given | Required ("who records this? Pass --by <your name>…"). |
  | An IDE agent, on its own runs | the run's executor (default) | Optional. |

  Before, stop/resume/cancel defaulted to the literal name `person`, and the
  other operations to the agent's name.
- `wfe run help stop|resume|cancel` describe the effect:
  - A hold keeps the plan, the records and the work in the run's copy (saved
    at the stop's checkpoint), and other requests go on.
  - A resume continues from the run's base.
  - A cancel reverts nothing.
- The help no longer says `--by <person>`, which is how the incident's agent
  came to write `--by iwaag`.

### What the person sees

- **Simple** (focus card while held): "Held (stopped) — lab-agent (on Test
  Person's answer to q1): … Its plan and work are kept, and nothing of it
  runs until it is resumed with an instruction. Other requests in this
  project can run meanwhile." The advice now says to look at the run's files
  "(they are in its own copy of the project)".
- **Detail** (Execution and publication):
  - The situation reads "Stopped (held) — its work is kept; resumes only when
    you resume it".
  - The held line names who stopped the run and on whose words, and that
    other requests can run.
  - While stopping, it says "…waiting for the process to end, then its work
    is saved."
- **Stop** keeps its label, so drafts and focus handling are unchanged.
  - Its title says what holding does.
  - The browser prompt reads "Hold this run? Execution stops; its plan and
    work are kept and it can be resumed later. Other requests in this project
    can run meanwhile. Reason:".
  - The activity line's held note reads "its work is kept; nothing runs until
    it is resumed".
- **Zulip.** The held notice names who stopped the run and on whose words.
  It says that the plan and work are kept and that other requests can run.

### Agent guide (agautolab `p9`)

- The cwd is the run's own copy; the project workspace is the person's.
- Work is saved to the run branch only, and the agent keeps that branch
  checked out.
- A hold keeps the copy while other runs go on.
- The agent's records carry its own name, with `--because` naming the
  person's words. The incident is cited: jevlab `experiment/run-001`
  recorded a hold `--by iwaag`.
- A resumed attempt starts from the run's base, and newer commits are named,
  not merged.

`docs/runs.md` covers holds, human waits, the one slot (formerly "one slot,
one workspace") and who records.

## Verification

`test/holds.test.ts` is new. Its main sequence is **A held with its plan and
unfinished work → B runs in the same project → A resumes**:

| Case | Evidence |
| --- | --- |
| **The incident** | <ul><li>A's plan.md, an unfinished untracked `lab/exp-a/wip.py` and an ignored `.local/venv-marker` exist.</li><li>The person answers "保留" and the resumed agent takes it up. `wfe run stop --by "Test Person"` from inside the attempt is refused. `wfe run stop --because answer:q1` records `by lab-agent`, its attempt id and `because`. The hold is `stopped by lab-agent` with `because {answer q1 #0 from Test Person}`, and the stop's checkpoint is published.</li><li>B is requested and executed in the same workspace and changes the same path.</li><li>The person commits to the project's main. A resume in the browser is recorded `by Test Person`.</li><li>A is claimed in the same cwd with its plan.md, its own wip.py (not B's), its ignored file, and no NEWS.md. Its brief says "the project gained 1 commit on main … none of them is in this copy". `_plan` is still running with q1 taken up.</li></ul> |
| Answer wait | B runs while A waits on q1. The answer resumes A in place with its files. |
| Running process | <ul><li>A browser stop on an executing attempt is a request, not a hold.</li><li>B waits for the slot, not for A, and only A's attempt is due for termination.</li><li>After the end, the stop is confirmed and A's unfinished file is on its run branch.</li><li>B's queued job survived and is claimed. A resumes in the same copy.</li></ul> |
| Interrupted save | <ul><li>A's stop checkpoint fails (unreachable remote) while A stays held.</li><li>B publishes regardless, and A's files are untouched.</li><li>A's retry later publishes A's work.</li></ul> |
| Host restart | <ul><li>A is held. B's attempt loses its process, and a new `ExecStore` reconciles.</li><li>B is recorded `unknown`, held, and its partial work is saved. A stays held, and nothing is queued by itself.</li><li>A third request runs. A is resumed and claimed in place with plan, unfinished and ignored files.</li></ul> |
| Who records | <ul><li>Outside an attempt, `wfe run stop` and `wfe run answer` without `--by` are refused on a host-performed run. With `--by "Omni Agent"` the stop is recorded so.</li><li>A control op with a stale attempt id is refused, and so is `because` naming a missing instruction.</li></ul> |

Updated tests:

- `exec.test.ts`: "a run awaiting a person reserves nothing: another request
  in the same workspace runs; other workspaces too".
- `intakestore.test.ts`: a second request topic in the same workspace becomes
  `make/run-002`, and its later post belongs to it.
- `settings.test.ts`: a second request is a run of its own, and the first is
  claimed first.

| Check | Result |
| --- | --- |
| Editor suite | **274/274** |
| `tsc`, `vite build` | clean / ok |
| `checks/exec.ts` | all passed, including "the hold says what it keeps and that it reserves nothing (p9)" |
| `checks/simplerUi.ts` | all passed |
| `checks/displayMode.ts` | all passed |
| `checks/runInteraction.ts` | all passed. Drafts and focus are preserved; the Stop label is unchanged. |
| `checks/states.ts` | renders its gallery; it has no assertions |
| agautolab `test_wfexec.py` + `test_wfintake.py` (p9 branch, p9 editor source) | 11 passed |

## Notes

- Ending a stop now publishes the copy (the checkpoint after the attempt).
  The run view shows a failed save on that run with its retry; it does not
  touch other runs.
- A held run's copy keeps ignored files and environments: nothing resets
  or cleans it.

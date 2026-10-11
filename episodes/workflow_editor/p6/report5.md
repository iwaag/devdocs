# p6 step 5 — additional instructions at node boundaries

Plan: [plan.md](plan.md) §Step 5.

## What changed

Commits:

- agdevworld `d4fa076`: reducer, run CLI, brief, run view, docs and tests;
- agautolab `33469aa`: contract guide sentence and an end-to-end test.

### Run history

| Operation | Who | Effect |
| --- | --- | --- |
| `instruction.receive` (step 4) | the intake (`via: zulip`) or a person (`wfe run instruct`) | adds `i<n>` with text, `from`, `source`, `during` (nodes running then) and `ack: null`. It changes no node. It is refused on a cancelled run (`run-cancelled`) and on a completed execution (`run-completed`; the intake explains that a new request topic is needed) |
| `instruction.ack` (new) | the executor; an **executor operation**, so it is fenced by the open attempt | names the instructions and gives a `response` (`incorporate` / `clarify` / `new-run`) and a note saying what it will do. Each instruction is acknowledged once; unknown ids and invalid responses are refused |

### The boundary (server-enforced in the reducer)

Two transitions are refused with code `instructions-pending` while any received instruction is unacknowledged:

- `node.start` of a **pending** node: authored nodes and generated documentation steps alike;
- the `node.complete` that would **complete the execution** (the last node; under `default`, `_final-report`).

The refusal lists each pending instruction (id, sender, time, text) and gives the exact `wfe run ack … --response incorporate|clarify|new-run --note …` to run, so the agent can acknowledge within the current attempt and retry.

The following are **not** boundaries, so a running node is never interrupted and no finished work has to be abandoned:

- resuming a waiting node;
- progress notes and questions;
- completing a node that is not the last.

**Race boundary.** Receipt, acknowledgment, start and completion all take the run's lock. The history order is the truth:

- an instruction applied before a start blocks that start;
- one applied after it belongs to the next boundary;
- after completion, nothing: refused, and explained by the intake.

A post still in transit (not yet applied) is not claimed to have reached the run.

### Changed work and changed plans

An instruction that would change completed work or the graph is a question to the person (`clarify`); nothing reopens a completed node. For a renewed plan agreement, `wfe run ask <run> <node> --about-plan` records the revised `plan.md`'s hash on the question, using the same document-evidence contract as the plan step's own question. The braindump stays unchanged.

### Where the agent and the person see instructions

- **Execution brief** (`instructionBrief`): every unacknowledged instruction in full, plus how to acknowledge.
- **CLI:** `wfe run instructions <run>` lists them with state and response. `wfe run show` has an *Instructions* section saying how many are not acknowledged and what waits for them. History lines name the instructions.
- **Run view** (browser): an *Instructions* section with received or acknowledged state, sender, the node that was running, and the acknowledgment note.
- **Summary:** `RunSummary.pendingInstructions`.
- **Contract guide** (autolab): every boundary checks pending input; acknowledge with what you will do; changed work is asked about; a revised plan is asked about again with `--about-plan`.

## Checks

- **editor**, `node --test 'test/**/*.test.ts'`: **168/168**. New tests are `test/instructions.test.ts` (6):
  - received during node `a`: `a` stays running, `during: [a]`; starting `b` is refused with the listing and the ack command; bad responses, unknown ids and a second acknowledgment are refused; after the acknowledgment `b` starts and the acknowledgment records the attempt;
  - the last completion is a boundary; after it, `instruction.receive` is refused (`run-completed`) and the run stays completed;
  - documentation steps:
    - an instruction during `_report.b` blocks the `_final-report` start;
    - one during `_final-report` blocks its completion;
    - a revised plan asked with `plan: true` records the new `plan.md` hash;
  - a stale attempt cannot acknowledge; nobody can while another attempt is open; the acknowledgment survives a fresh read (replay matches);
  - **race**: in 12 trials, `node.start b` and `instruction.receive` run concurrently. In every trial, either the start succeeded and the receipt has a later sequence, or the start was refused with `instructions-pending` and no start was recorded. The receipt was never refused;
  - the brief lists pending instructions in full; a cancelled run takes none; `--about-plan` is refused in a run without a plan step.

  `test/intakestore.test.ts` was updated: the completed-run case now acknowledges first, because the last completion is a boundary.
- **autolab**, `test_instruction_during_a_node_is_taken_up_at_the_boundary`, with the real executor loop and a stub agent:
  - an instruction is given (`wfe run instruct`) while `scope` runs, and `scope` stays running;
  - the stub's `start build` is refused, then it reads `wfe run instructions`, acknowledges and continues;
  - the attempt ends **`exited`** (not interrupted), the run is completed, and the acknowledgment names that attempt and precedes `build`'s start.

  The executor, notifier and intake suites (`test_wfexec`, `test_wfnotify`, `test_wfintake`) passed (29) before this test was added; the new test passes.
- `tsc --noEmit`: clean (UI included).

## Limitations

- The guard covers `node.start` of pending nodes and the final completion. A failed node restarted with `start` (a resume with a reason) is not a boundary. It is the same node continuing after a person's decision.
- Whether the agent *acts* on an acknowledged instruction as it said is its own claim, like every report. The record keeps the response and note to check it against.
- The browser run view's new section was type-checked but not yet exercised by a browser check (step 7).

## Next entry point

Step 6: request-topic and run-topic destinations in the notifier, questions and replies in the request topic (and the `posted` lookup that lets Zulip answers bind), result confirmation mapped to the result decision, and PPTX/preview delivery links.

# fix p2: explicit conversation closing and truthful status

## Purpose

Address [problems.md](problems.md): conversations remain open after agents say
they are finished, and aghangout labels every open conversation as active work.
Make closing easy for the Developer and available to agents across projects,
including warden when asked to tidy abandoned conversations. Keep additional
agent guidance to one or two shared lines.

This plan records the approach agreed with the Developer on 2026-11-02.

## Intended behavior

- The room header offers **Close conversation / Reopen conversation**. The list
  also supports closing explicitly selected conversations together.
- List filters read **All / Not closed / Closed** (Japanese UI:
  `すべて / 未終了 / 終了`). Open no longer means `進行中`.
- Room lifecycle and work status are separate: an open room may be idle,
  working, awaiting an answer, failed or holding a continuation. A closed room
  may still have unfinished work. Failures and held work remain visible.
- Every project can inspect and close a conversation by its ID, whether or not
  that project participates in it. Warden uses the same tools as other agents.
- Successful close operations, rather than an agent's completion wording,
  establish that a conversation is closed.

Retain the existing close semantics: withdraw requests that have not started,
allow started work to finish and record its results, and hold automatic
continuations. Closing neither approves results nor asserts that work is done.
A person's subsequent post reopens the room; reopening does not replay
withdrawn requests or automatically release held continuations.

Do not add keyword-based closing, inactivity-based automatic closing, a new
conversation state machine, or a separate warden-only closing mechanism.

## Work

### 1. Verify and complete the shared close path

Owner: WFE (`pj-agdev/agdevworld/workflow_editor`).

- Inspect the deployed and worker-side CLI paths for `conversation list`,
  `state`, `close` and `reopen`. The current CLI and service accept an explicit
  conversation ID without a participant/conductor restriction; verify this
  from an actual unrelated project's attempt, including warden's execution
  path. Fix any transport or context restriction that prevents it.
- Keep UI and CLI operations on the same service implementation. Repeated
  closes should not duplicate the close event or repeat side effects.
- Extend close provenance to retain the actual actor, originating run/attempt
  when available, and an optional reason. Provide a reason argument in CLI
  help and preserve these facts in the archive. A human UI operation retains
  the human identity; an agent does not impersonate the requester.
- Make command output report the resulting room state and withdrawn requests.
  Keep outstanding work and held results inspectable through `state`.
- Keep detailed syntax and lifecycle explanations in `conversation help` and
  the existing conversation contract, rather than adding an agent procedure.

### 2. Make the UI operation discoverable and status accurate

Owner: managed project `aghangout`.

- Move the existing close/reopen control from the bottom of the members panel
  to the room header, with concise nearby wording that started work continues.
- Add selection and batch close to the conversation list. Operate on the
  selected IDs, account for each result, and retain failed selections for
  retry. Do not report the whole selection closed if only some calls succeed.
- Show success only after the service response, then refresh authoritative
  room/list data. Expose failures with a retryable action.
- Replace the open filter label with `未終了`. Render `待機中` only when there
  is no working/owed work, awaited answer, failure, stopped work or held
  continuation. Preserve specific status indicators and closed-room
  outstanding-work indicators without introducing new persisted states.
- Keep close and reopen accessible on narrow screens as well as desktop.

### 3. Keep guidance minimal

Owner: shared wfrun conversation guidance and ponderer's conduct guide.

Add at most two shared guidance lines, with this substance:

> When asked to close a conversation, use `wfe conversation close <id>`; the target may belong to another project (fix p2).
> Closing does not complete or approve work, or stop started work; see `wfe conversation help`.

Replace ponderer's existing person-only description of closing with a
consistent description; do not copy the shared instructions into each
project's guide. Keep this episode's reference with the shared addition.

### 4. Repair the recorded omissions

- Re-read the seven conversations identified in `problems.md` immediately
  before repair. Check the cited finish instruction, newer messages and
  current obligations; the original observation is not current-state proof.
- Close those whose recorded instruction remains applicable, using the public
  close path and a reason referring to the instruction and this episode.
- Record each disposition, including already closed rooms and rooms left open
  because subsequent work changed the situation. Do not close other rooms
  merely because they are old or idle.

## Verification

Use focused automated checks for changed lifecycle/provenance and UI status
behavior, then the affected repositories' required checks. Use isolated check
conversations for live operations; do not use unrelated real conversations as
fixtures.

- An idle open room reads `未終了` / `待機中`, not `進行中`; working, awaiting,
  failed, stopped and held cases remain distinguishable. A closed room with
  work still shows it.
- Header close updates both room and list after success. Reopen works, and a
  later human message follows the existing reopening behavior.
- Batch close accounts for successes and failures per ID and supports retry
  without duplicate effects.
- An unrelated project attempt can list, inspect and close a check room by ID;
  exercise warden's path and the remote worker path where applicable. Verify
  the recorded actor, originating run/attempt, reason and archived event.
- Closing with pending requests withdraws them. Started work can finish and
  its result remains readable; automatic continuation stays held and no
  result is implicitly accepted. Repeated close is harmless.
- Verify the final assembled guidance adds no per-project instruction copies
  and stays within the shared two-line budget.

## Delivery

Deploy the affected WFE, executor guidance and aghangout changes through their
existing release paths, updating worker installations where affected. Check
the deployed UI and actual agent command paths, not only source tests.

Write `report.md` in this episode with changed revisions, checks actually run
and their results, deployed verification links, the seven rooms' dispositions,
and any remaining limits. Update existing maintained contracts and local
deployment notes where their facts change. Keep machine-specific addresses
and execution evidence in ignored local notes.

The Developer should be able to verify the result by opening a supplied check
room, using the header button, and seeing the list state change; include a
completed warden cross-project close example in the evidence.

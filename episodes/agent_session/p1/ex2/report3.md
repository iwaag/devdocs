# Agent session p1 ex2 — step 3: the changed correctness boundaries

Date: 2026-10-15. Source: agdevworld `a642e27`, agautolab `9382b1b`. All
checks use fixtures and a stub harness; no model was called. The behaviour
of a real resumed transcript is checked on the step 4 comparison pair, as
the plan asks. No separate paid probe was run.

## Boundary → test

WFE tests are in `test/continuation.test.ts` (C) and `test/sessions.test.ts`
(S). The executor tests are in agautolab `tests/test_wfexec.py` (E). They
run the real executor loop against the working-tree WFE with a stub
harness.

| Case | Required result | Where | Result |
| --- | --- | --- | --- |
| Ordinary resume | Compact state and pending input; no mandatory history, no unchanged bootstrap | C "ordinary resume"; E `test_run_policy_resumes_one_explicit_session_across_a_wait`, `test_prompt_kinds_new_continuation_and_reconstruction` | The continuation has the answer's exact text, ids and author, the delta and the state. It has no `--history`, working-copy path, catalog or policy, and is under 2,000 chars. The real executor's resumed prompt has the answer and no `--history`. |
| New or replacement session | Complete reconstruction with the original requirement and current plan | C "documentation steps" (the brief names `plan.md` agreed in q1); E `test_missing_session_is_replaced_before_launch` | A replacement after a missing transcript gets the whole bootstrap, naming `braindump.md (author …)` and carrying the pending answer. A missing session is never resumed. |
| Multiple answers/instructions during a wait | All discoverable, taken up and acted on once | C "pending input survives …" | Two answers to q1 and one instruction all appear in full in both the continuation and the brief. After take-up and ack, the state read no longer lists them. |
| Input after the snapshot / during execution | Boundary refresh and write-time checks expose it; no stale completion | C "input after the snapshot" | `wfe run state --since <seq>` lists the instruction recorded after the claim. `node.start` is refused while it is unacknowledged. Completion refusal is covered by existing `instructions.test.ts`. |
| Launch failure or interrupted delivery | Nothing disappears because it was only claimed or sent | C "pending input survives …" | The attempt whose prompt carried the answers exits 1 without take-up. The run is held `interrupted`. After the person's resume, the next continuation carries both answers and the instruction again. |
| Hold, cancellation, acceptance, graph changes | The current record controls; memory cannot override | C "holds, cancellation, decisions …" | A person's stop and resume during a wait appear in the delta in record order, and the stop queues nothing. A person's rejection during a wait appears in the delta and in the state (`Decision: rejected by …`). A cancellation appears in the state read, and nothing more is claimed. A revision in force is a state line. Adjustments are the agent's own records, guarded by the reducer's existing tests (`adjust.test.ts`). |
| Interrupted command | Effects checked before a retry; no blind replay | C "pending input survives …" | After a non-zero exit the continuation says the effects are uncertain and names `git status`, `git diff` and the record before any repetition. Real-harness behaviour is p1 step 4's evidence (the killed command was not replayed). |
| Guide/config change, missing store, mismatch | Existing replacement, refusal and ex1 ownership rules intact | S (7 tests); E (21 tests: changed guide, retirement, missing store, early crash and restart, launch gate, unconfirmed stop, mismatch stop and retirement, attachment retry) | All pass unchanged. The contract or guide hash still forces a new session with the whole bootstrap. |
| Compacted conversation | Requirements survive, or explicit reconstruction | C "compaction" | A `compact_boundary` in attempt 1's log resumes the same session with the whole bootstrap and records `reconstruct: the conversation was compacted in attempt …` before launch. The next resume is an ordinary continuation, because the reconstruction restarted the span. A deleted log forces a reconstruction ("cannot be ruled out"). The executor's reconstruction prompt starts with the reason, then the contract and the whole brief. |
| Output size on a long history | Compact state bounded | C "size" | 150 progress notes and 60 instructions give `show --history` 51,230 chars against 4,302 for `state`, which is mostly the 60 pending instructions in full. The agent's own 150 notes are not in its delta. The delta is cut at 30 lines with a `--since` refresh. |

**Cache expiry.** No code path depends on time. `server/continuation.ts`
reads no clock, and the reconstruction rule looks only at logs. A resume
after any wait therefore gets the same continuation. No mock clock was
needed, because there is nothing to mock. The behaviour across a 65-minute
wait (a cold cache, not a lost conversation) is p1's measured evidence. It
was not recreated.

## Commands run (each once after the last change)

- WFE: `node --test test/continuation.test.ts` → **7/7**. Earlier, with
  `sessions`, `instructions` and `docsteps`: 34/34. `tsc --noEmit` clean.
- agautolab: `uv run pytest -q tests/test_wfexec.py` → **21 passed**
  (127 s).
- The full WFE suite (`npm run check`) runs once in the release gate (step 4
  preparation / step 6).

## Guides that could bring a full read back

- RTS `agent/review.md` names `wfe run show <run>` as one source for the
  review report's presentation facts. That is a targeted read, not a start
  routine.
- agops' recovery guide and Observer's recovery request ask for
  `--history`. Recovery is an investigation of someone else's run, where
  the full history is the right diagnostic.

Neither was changed.

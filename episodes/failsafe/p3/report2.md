# failsafe p3 — step 2: reply delivery and acceptance routing

## 1. The reply mark is a tag block (R1, R2)

**Owner: pyagag `agag.reply`** (`a686f22`).

A reply is now written between two tag lines:

~~~
<ag-reply intent=report>
The tests pass:

```
Ran 207 tests in 0.2s
OK
```

The change is committed as 41d5503.
</ag-reply>
~~~

- **Inside the mark, fences are ordinary Markdown.** A fence opens a code
  block, and a bare fence of its kind and at least its length closes it.
  A `</ag-reply>` line inside a code block is quoted text. The close is
  therefore never confused with a code fence, which was the whole of R1.
- **A code block the run forgot to close** does not swallow the reply.
  When the text ends inside such a block, the last `</ag-reply>` line is
  the close.
- **The fenced mark is retired, not tolerated.** Its ambiguity is in the
  text itself (report1, R1). A run that writes ```` ```ag-reply ```` gets
  a failed split with its own reason ("the reply is in a fenced
  ```ag-reply block, which is no longer read: put it between …"), and
  the repair asks for the tag form. It is never cut and never partly
  posted.
- **Attributes never cost the reply.** Unreadable words on the opener
  are dropped and named in `meta_error`, and the readable ones stand.
  This includes a `to=` that is not a user id. R2's opener now yields
  `intent=response_request ask=confirmation`, addressed to the recorded
  requester, and the log line says what was dropped.
- **The repair is told the exact reason.** R2's repair was told "no
  ag-reply block" when the block was there. That is why it wrote the
  same opener again.
- **The guide** (`REPLY_GUIDE`, appended to every conversational role
  once) shows the tag form, a reply containing a code block followed by
  text, and "`to=<user id>` (a number, never a name)".
- **Regression fixtures** are the captured p2 outputs:
  - `tests/fixtures/reply/12328.md`, `12338.md` (autolab, trial C);
  - `12509.md` (Front, trial D2).

  With only the opener and closer rewritten as tags, each posts its
  complete reply: code blocks, and the text after them. As captured, each
  is a failure with the retired-form reason, never a truncation.
- **Documentation**: `docs/post-intent-v1.md` and the pyagag README.
- **Consumers.** Every consumer with a conversational role moved to
  pyagag `b2cdf75`: agautolab, agfront, agforge, archsage and cagent.
  Their fixtures and autolab's fault instructions (`STOP_MARKED`,
  `STOP_UNMARKED`) and closing fallback use the tag. The relay and
  comfynotify keep `f5c4359`, because they use no changed module.

## 2. A failed reply stays owed (R2's second half)

**Owners: pyagag `agag.topics`, `agag.listen`, `agag.trace`** (`b2cdf75`),
and agobserver (pj-agdev `3296f72`).

The contract, from the journal:

| stage | what is posted | journal | receipts | next |
|---|---|---|---|---|
| run and in-run repair both unusable (attempt 1) | "(this run produced no reply: *reason*; …; the reply is still owed and is asked for once more)", `intent=progress`, **no `end=`** | `extra.reply_owed` = reason, attempt 1, the run's own output (last 12 000 chars) | **none**: the input it was given stays unanswered | the listener re-queues the same entry after `REPLY_RETRY_SECONDS` (20 s) |
| the re-serving | the handler runs normally. `prompt_with_guide(reply=True)` appends "Your previous run on this input produced no usable reply", with that output and "Everything that run did has already happened … do not repeat any of it" | `journal.previous()` returns the owed serving | written after a delivered reply | the debt is settled (`settled_by`) |
| attempt 2 also unusable | "(… the input stays unanswered and is reported)", `intent=report end=<ack>` | `reply_owed.final` | written (the input was served twice) | trace: the conversation is `failed` |

- **Bounded**: two servings of one input, each with its own in-run
  repair. That is at most four model runs, and never a third serving.
- **Survives a restart**: the re-queued entry keeps its time in the queue
  file. Startup recovery also finds the debt (`owed()` honours
  `reply_owed`), and the two are one serving.
- **No replay of completed side effects**: the re-serving sees the failed
  output and is told its actions happened. Front's continuation still
  shows an *interrupted* serving the old way; an owed one is described
  only by the reply notice, to avoid saying it twice.
- **Escalation.** After the last failure the request's own conversation
  ends in its agent's failure notice. `agag.trace.stall_candidates` now
  reports that as **`unanswered`** on the root. Before, it skipped the
  root for `failed`, which is why #12509 was nobody's business.
  - `unanswered` is a failsafe kind (60 s), gated on the rollout horizon
    like `unheld`.
  - Observer reports it to the owners at once, because asking the agent
    that failed twice would buy a third run of the same failure.
  - It then goes to a developer review like every reported incident.
  - Bound: the first failure, 20 s, the re-serving's run, the last
    failure, then at most 60 s + one look (60 s). So **about 2 min after
    the last failure** the owners are named.

Tests:

- pyagag `test_owed_reply.py` (4, through the real listener and journal):
  - the re-serving sees the output and answers once;
  - two failures are the last, with exactly two runs;
  - a callback keeps its receipt until the reply is made;
  - a restart before the re-serving serves it once;
- `test_reply.py` (+3): the first failure, the settlement, the last
  failure;
- `test_failsafe.py` (+2): the first failure keeps the conversation in
  hand; the last is `unanswered` after 60 s;
- agobserver `test_p3_failsafe.py`: two failed replies reach the owners
  once, with one review occurrence, and Front is not asked.

## 3. A task agreement posted in the plan (R3)

**Owner: agautolab** (`zulip_listener.py`, planner guide).

The choice is explicit correction, not routing. Routing would need the
plan to decide, from words, that a post agrees to a task, and then carry
evidence across topics into a close-out that is built on the task
topic's own input. The correction needs neither, and it cannot be wrong.

- **The plan serving reads where each task stands** (`status.md` in the
  registered files): state, and the result that waits for agreement in
  which topic (`shown_result`: the newest `report` or confirmation
  request of ours, with nobody else speaking there since).
- **When the requester spoke in the plan after a task showed its
  result**, the reply carries a literal line from the record:

  > task 1 is still open: it closes only when its requester agrees in its
  > own topic, #**work-m12379>workrun-task1-m12379**, to the result it
  > showed there (#12428). An agreement posted in this plan closes
  > nothing; please post it there.

  The line is a section, so it follows whatever the planner wrote. A
  wrong claim can no longer stand alone in the post.
- **The planner guide**: a task closes only in its own topic; never say
  a task is closed, accepted or done unless its status says `completed`.
- Cost: only a planned mission reads its tasks' topics (one read of the
  plan and one per task, no model).
- Tests: `test_agreement_routing.py` (4), built from trial D's own ids.

## 4. Resumption, result, agreement, integration and acceptance stay distinct

Nothing in this step merged them. Each is pinned where it lives:

| fact | record | pinned by |
|---|---|---|
| resumption asks for work | a resume closes nothing | agautolab `test_a_resume_request_resumes_the_work_and_leaves_the_task_open` |
| a shown result | our `report` or confirmation request in the task topic | `result_shown_before`, `shown_result` |
| task acceptance | the requester's post after the result, in the task topic → `[change] accepted` | `test_an_acceptance_after_the_resumed_result_closes_the_task`, `test_the_close_out_binds_the_accepted_change_before_integrating_it` |
| a refused close-out | nothing recorded, and the reply says why | `test_a_report_nobody_agreed_to_closes_nothing_and_starts_nothing`, `test_a_question_answered_is_not_a_result_agreed_to` |
| integration | `[change] integrated`, then `completed` | `test_a_completed_task_is_not_closed_or_integrated_again` |
| mission acceptance | `agentchat accept` / `accept.flag`, refused while a task is open | pyagag `test_every_refusal_writes_nothing`; agautolab `test_accept_flag_while_a_task_is_open_says_so_and_changes_nothing` |
| a misplaced agreement | the correction line; the task stays open | `test_agreement_routing.py` |

The live exercise of a misplaced agreement, a refused mission acceptance
and a successful one is step 6's trial "Agreement reaches the plan".

## Test totals (new pyagag everywhere)

| suite | passed |
|---|---|
| pyagag | 963 (the full run, plus `test_post` re-run after its two expected updates) |
| agautolab | 317 |
| agobserver | 151 |
| agfront | 180 |
| agforge | 265 |
| archsage | 37 |
| cagent | 204 |

## Rollout

- Committed and pushed:
  - pyagag `a686f22`, `b2cdf75`;
  - agautolab, agfront, agforge (submodules) and pj-agdev with
    agobserver;
  - archsage;
  - pj-clusterintent (cagent).
- The tests ran in scratch virtual environments. The services' own
  environments were synced only at rollout.
- Before the restart, no listener had a process below its interpreter,
  so no run was in flight.
- 04:05:25Z: kickstarted the autolab listener and gateway, Front,
  Observer, forge's listener and service, archsage, and cagent's listener
  and API. The recoveries ignored old mentions and started nothing. The
  relay was not restarted, because nothing it uses changed.

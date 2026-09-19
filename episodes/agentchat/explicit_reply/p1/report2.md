# Step 2 report — Separate public replies from model output

Plan: [plan.md](plan.md) step 2; design from [advice.md](advice.md).
Repositories: `pyagag` (`a0573f3`), `agfront` (`f7508c0`; the `pj-agdev`
submodule pointer and lock move in step 5).

## The contract

`agag.reply` (new). A run says what it wants said inside a fenced block:

    ```ag-reply
    what is said, verbatim
    ```

- **Inside the mark is what is said.** Several marks are one post, in
  order. A reply keeps the code fences it contains: the splitter is
  fence-aware (a four-backtick mark closes at four or more; inside a mark,
  a fence with an info string opens a nested block that a bare fence of
  its own length closes; tildes and backticks do not close each other).
- **Everything outside is the run's own** and is never posted. It is kept
  in the transcript and the run record as before.
- **Machine blocks are the handler's.** `ag-argue` and `ag-routinerun` are
  read from the whole output by their own splitters, applied once, and
  stripped *before* the output reaches the reply splitter, so their place
  relative to the mark does not matter and their effects are never
  re-applied by a repair.
- **System notices are separate.** `TopicResult` now carries three kinds:
  `output` (model output, under the contract), `sections` (literal text a
  handler wants posted as written — a status line, the canonical block
  echoed as the record) and `notices` (system-generated notes appended
  after the reply: the desire recorded, a completion checked, a block that
  could not be read). The post is reply, sections, notices.
- **Missing, empty or unclosed marks are failures, not silence.** The
  advice's "no mark, post it whole" fallback is retired. `resolve_reply`
  asks the run once more for the reply alone — `repair_prompt` puts the
  previous output in front of it and says every action and machine block
  has already happened — and if that fails too, posts one visible line:
  `(this run produced no reply: <reason>; its output is kept in the run
  record)`. The conversation is never left with the ack as its last post.
- **The outcome is recorded.** The serving record (step 1) gets
  `reply_marked`, `reply_blocks`, `reply_failure`; the handler files its
  run record path (`context.journal.record`) and, after delivery,
  `serve_topic` writes a `reply` object into that `ag.agent-run.v1` file
  with `marked`, `blocks`, `failure`, `delivered_id`, `posted_to` and the
  serving id. The listener log says when a reply came from the repair run
  and when it failed.

## Where it is described to the model

Once, in the shared prompt composition: `prompt_with_guide(lines, guide,
reply=True)` appends `REPLY_GUIDE` after the role's guide. It describes
the mark as a tool (what it is, that anything outside it is not posted,
how to carry code fences, that machine blocks are read anywhere, what
happens when it is missing). Structured-output roles — Front's `present`,
generators, `observe`, `workrun_supercoder` — call `prompt_with_guide`
without it and keep their own contracts.

Applied in this step: the shared entrance (`agag.entrance`), argue
participants (`agag.argue.participant_prompt` / `participate`, which posts
on its own and therefore resolves the reply itself), Front's `front`,
`desk`, `argue` and `routine_run` roles. The `guide.md.in` template and the
entrance's default guide say "your reply is posted for you" and nothing
about what not to write.

Removed prose prohibitions: the argue guide's "Write only the reply: no
`[Name #id]` headers … no notes to yourself"; the desk guide's "everything
you output is posted verbatim … no notes to yourself, no analysis, no
preface. The first character of your output is the first character the
developer reads"; the routine guide's "Your reply is posted as it is.
Begin with the first word of your entry"; the participant guide's "the
closing message of this run". Other consumers' entrance guides carry the
template's old sentence and are updated in step 5.

## Front

- `argue.py`: `split_block` strips `ag-argue` from the whole output and
  applies it; the rest is `TopicResult.output`, the desire/outcome lines
  are `notices`, and `repair` runs the argue role once more on the repair
  prompt. `EMPTY_REPLY` is no longer posted for an empty output — an empty
  output is a failed reply and is handled by the contract.
- `zulip_listener.py`: `run_front` takes the serving journal and files the
  run record against it; `serve` returns `output` + `repair` for front and
  desk; `finish_run` posts the marked entry as the reply and the canonical
  `ag-routinerun` block (or the error fence) as a literal section.

## Evidence

`pyagag`: **658 passed** (19 new in `tests/test_reply.py`; the argue
participant stubs and the entrance test re-pinned to marked outputs).
`agfront` against this checkout: **181 passed** (wiring stubs now return
marked answers; `deliver` is what tests intercept for the ack and reply;
one fewer `history` read pinned; the prompt equality includes the guide).

Fixtures are the real posts, read from Front's mirror store into
`tests/fixtures/reply/`:

| Fixture | Pinned |
|---|---|
| #7222 as observed (no mark) | not posted whole: a missing mark, repaired |
| #7222 with the reply marked | only "Is the following a fair statement…" is posted; the two thought paragraphs are `rest` |
| #7230 (archsage) | "The sage is defined. Here is my analysis" stays out; the analysis is the reply |
| #7262 (smoke argue) | one turn, not two |
| several marks | one message, in order; `rest` reads as written |
| code fence inside a four-backtick mark; `python` fence inside a three-backtick mark; tildes | reply intact |
| missing / empty / unclosed | explicit errors; unclosed keeps the text for the record |
| machine block beside the mark | stripped by the handler's splitter first, never posted |
| repair once, then the failure line; a repair that raises | visible failure, no crash |
| through `serve_topic` | reply → sections → notices; the failure line is a *delivered* answer; `reply` object lands in the run record |
| agfront: #7222 marked through `handle_argue` | the post starts at the question, ends with "— the desire is on record as message 92.", and `present.plain_content(post) == post` — the renderer receives only speech |

The Arguing Room re-render of #7222's shape against memo record
`j46335f85a54417f5f8ec` is a live check and is listed under step 5's
pending live checks; the fixture above is the static equivalent.

## Remaining gaps

- autolab, forge, cagent, archsage and Observer still return model output
  in `sections` (posted whole, as before); step 5 moves them to `output`
  with their locks.
- `agentchat reply` (posting a marked reply mid-run into `AGENTCHAT_HOME`)
  stays out, as the advice decided for p1.

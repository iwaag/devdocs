# Step 3 — argues and their interpretations through the relay

Date: 2026-09-18 JST. Plan: [plan.md](plan.md) step 3. The running relay is
still on the old code; nothing was posted to the realm. Deployment is step 5.

## Routes

| Route | What it does |
|---|---|
| `GET /argues` | Every argue, newest first: `anchor`, `topic`, `live_topic`, `resolved`, `speakers`, `desire`, `outcome`, `status`. |
| `GET /argues/<anchor>` | One argue: every participant's real posts (`kind` human / agent / ack, roster `agent`, `speaker` — `sage:arxiv` for a post under that header), a Zulip link, and `presentation`. |
| `POST /argues` `{stem?, text, token}` | Opens an argue **as the human**: the `[selfnote][argue] from -` anchor, then the text verbatim (`agag.argue.open_argue`). A stem in use is refused; without one a UTC stamp is minted. |
| `POST /argues/<anchor>/post` `{text, token}` | The human's next turn into the **source** argue. A ✔'d argue is un-resolved first. |
| `POST /argues/<anchor>/render` `{revision?, token}` | One `[selfnote][render] <revision>` in the source (the active settings revision by default). |
| `POST /frontdesk/<id>/render` | The same request for a Front Desk conversation. |
| `GET /frontdesk/<id>` | Now carries the same `presentation` block; `dialogue` / `dialogue_error` are gone. |

## Decisions

- **An argue is its anchor.** Every route but the list and the create names
  an argue by the message id of its `[selfnote][argue]` note;
  `presentation.locate` answers where that message is *now* (the open topic
  preferred over a ✔ twin). A renamed topic is followed; a topic that took a
  freed display name has another anchor and is another argue. An id that is
  not an argue's anchor note is refused.
- **Reads are the mirror's.** Source history, memo results and status are
  read from the relay's existing mirror on the Observer credential — no
  second reader, no sweep per refresh. While the mirror is not live the
  history is still returned, with `status.state = unknown` and
  `stale_state` holding the last known one.
- **Writes reuse the chat door**: the Developer's credential, its text
  guards (length, no selfnote typed by hand), and submit tokens
  (`SubmitTokens`, now shared by the argue routes and both render routes). A
  repeated token returns the first result with `duplicate: true`; a failed
  send is `uncertain` and is never retried.
- **Reopening resumes the discussion and nothing else.** The un-resolve is
  one rename of the argue topic; the project or study the argue ended in,
  its channel and its setup topic are not touched, and nothing is executed.
  A render request goes into the source's *live* name, so a finished argue
  can be re-interpreted without being reopened.
- **No second completion engine.** An argue ends by Front's checked outcome
  (p1) or through the existing `/complete` door; the room adds no finish
  route of its own.
- **Status comes from the memo alone**: `result`, `failed`, `refused`
  records, and — derived — agent speech with no fresh result at the active
  settings revision (`pending`; `overdue` after 15 min, which makes the
  renderer `unavailable`: slow, off or down look the same from here and the
  payload says so).

## `presentation` (shared helper, `agentroom/presentation.py`)

`memo` (the topic whose `memosource` note names the anchor),
`active_revision`, `interpretations` (one per settings revision × renderer,
with counts), `renderings` (`{source message id: [{settings_revision,
renderer, job, stale, turns, memo_message_ids}]}` — each turn keeps its
`sources`, so a rendered line leads back to its posts; a `plain` turn is a
speaker without a character), `pending`, `failed`, `refused`, `requests`,
`renderer {state, reason}`. A result is `stale` when the fingerprint
recomputed over the source as it stands differs or a cited post is gone. A
multi-part result is shown only when every part is held.

The three things writer and readers must agree on moved into pyagag
`agag.memo`: `DIALOGUE_SCHEMA`, `fingerprint`, `RENDER_TAG` /
`render_request_note`. agfront uses them instead of its own copies.

## Verification

`agentroom/tests/test_argueroom.py` (12) and two new desk tests:

| Plan item | Test |
|---|---|
| Creation | `test_creating_an_argue_is_the_anchor_note_then_the_humans_words_once` — exactly two posts by the human, the name refused when taken or unusable, a minted stem. |
| Duplicate submit | the same test, `…goes_into_the_source_once…`, `…reinterpretation_request…`, `test_a_failed_post_is_uncertain_and_never_retried`. |
| Reload | every read is a pure function of the mirror; `test_the_list_and_one_argue…` reads list and detail from a freshly built room. |
| Anchor-following after rename | `test_an_argue_is_followed_by_its_anchor_through_a_rename_and_a_reused_name_is_another_argue`. |
| Resolved history | `test_a_resolved_argue_keeps_its_history_and_is_resumed_in_place` — five posts and the outcome under `✔`, then one rename and one post, all in `#argue`. |
| Unavailable renderer | `test_a_failed_job_an_unavailable_renderer_and_a_stale_result_are_said`. |
| No Zulip calls on repeated reads | `test_repeated_reads_cost_no_zulip_call` — five list + detail reads, the mirror's call ledger unchanged. |
| Provenance | `test_every_agent_post_is_related_to_its_interpretations_with_status` — two interpretations of one post, two turns for one post, the source id on each turn, the sage pending. |
| Stale mirror | `test_a_stale_mirror_marks_the_state_and_keeps_the_history`. |
| Front Desk adapted | `test_a_desk_conversation_carries_its_presentation_and_no_block_is_parsed`, `test_asking_for_another_interpretation_is_one_selfnote_in_the_source_once`. |
| HTTP | `test_the_routes_answer_over_http`. |

agentroom: **294 passed**. pyagag `tests/test_memo.py`: 8 passed. agfront:
179 passed on the shared contract.

## Commits

| Repository | Commit |
|---|---|
| pyagag | `7beeec5` |
| agdevworld | `6c6958f` |
| agfront | `66d4dd4` |
| pj-agdev | `472b6c6` |

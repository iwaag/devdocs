# clearer_chat_ui step 3 — the Front Desk shows it

Front produces the contract, the relay carries it, and the Front Desk shows
it. Code is pushed; the running services are **not** restarted yet — that is
step 5's deployment (agdevworld `cb7440f`, agfront `ed3d15b`, pj-agdev
pointers pushed).

## Front (agfront `ed3d15b`, pyagag pin `932d3c6`)

- Every conversational role already gets `REPLY_GUIDE`, so the desk, front,
  argue and routine_run roles learn `ag-reply intent=…` from the pin alone —
  no role guide was edited.
- `format_evidence` (Front's chatlog and threads) puts each post's meaning
  and request id in its header and never shows the line.
- The presenter's snapshot (`present.plain_content`, `sources_markdown`)
  strips the line and tells the presenter what each source post *is*
  ("this post is: asks Developer to answer (question)"). The rendering does
  not classify anything: the room reads the label from the source post.

## Relay (agentroom)

- `presentation.shown_content` strips the line; `post_payload` and the desk's
  posts carry `meaning` (`{intent,to,ask,re,seen}` or `{error}` or null).
- `requests_payload` = `agag.outstanding.read_requests` over the messages the
  relay already holds (engine history or mirror) — no second model. Request
  text is shown without the hand-off mention.
- The desk conversation carries `requests` and `viewer_id` (the Developer
  credential's user id). `status` gains **`asking`** (Front answered last and
  a request is pending); an overtaken request reads as `received` ("Front
  owes that post a run"). Board rows carry `asking: n`.
- `POST /frontdesk/<id>/post` takes `answers: [ids]`; the relay writes
  `ag-post re=…` into the post, refusing an id that is not a request of that
  conversation (nothing is sent).

## Room (agdevworld)

- `postMeaning.ts` is the one label function: icon + words + tone. Progress
  `⏳` restrained grey; report `📄`; a pending request for you `❓ QUESTION /
  CONFIRMATION FOR YOU · waiting for your reply` in bold on amber; a settled
  one keeps `❓ question for you` but dim and says `answered in #113`,
  `withdrawn`, `replaced by #n` or `unanswered when closed`. Colour is never
  the only signal.
- The dialogue box shows the reply's label beside its speaker, in both the
  dialogue and the original view (`PlayReply.meaning` comes from the source
  post).
- A strip above the dialogue: `❓ n waiting for your reply`, one chip per
  pending request (clicking goes to it and makes the next post answer it:
  `↩ answering #112`, the composer says `Your answer to #112…`, `✕ not an
  answer` undoes), a "pick the one your next post answers" hint when several
  are pending, and a note for overtaken requests. It is built only from the
  relay's current states, so an old question in the history never appears
  there.
- History rows carry the label (and `↩ answers #n` on the Developer's posts);
  rendered turns show their source post's label. Conversation chips show
  `❓n`. The status line says `❓ 2 questions for you — reply below` or
  `answered … · nothing is asked of you`.
- The demo (`?view=frontdesk&demo=1`) scripts progress, reports and two
  simultaneous questions, with a miniature of the read model's rules.
  `&probe=1` exposes the scene to a driver outside the demo.

## Evidence

- Tests: agentroom 338 (7 new: meanings, asking, reports, a reply updating
  the wait while the question keeps its label, overtaken, `answers` written
  as `re=`, refusal of a non-request id); agfront 184 (2 new: labelled
  evidence, presenter snapshot); `tsc` and `npm run build` clean.
- Browser, through the **real relay code over a fake realm**
  (`agdevworld/.local/ccu/fixture_relay.py`, a scripted Front answering each
  Developer post with `agag.post.compose`; vite on :5176 with
  `VITE_AGENTROOM_URL`), screenshots in `agdevworld/.local/shots/ccu/`:
  - `b1` Front's progress and question → `asking`, strip `1 waiting`;
  - `b2` "GitHub trending." → the question answered by next post; report
    labelled; status `answered · nothing is asked of you`;
  - `b3` two questions at once → strip lists #111 and #112 with the hint;
  - `c1` **a new page load** shows the same two pending (state is the
    relay's, not the page's); `c2` clicking #112 → `↩ answering #112`;
    `c3` "Weekly." → the relay recorded `re=[112]`; #112 `answered
    (reference)`, #111 still pending; the #112 reply keeps its question
    label, dim, `answered in #113`;
  - `d1` original view with history: every label, `↩ answers #112`;
  - `e1–e3` the demo in the dialogue view: rendered lines carry the source
    post's label; two pending; history under the dialogue.

## Known limitations

- In the demo, where no settings revision gives Front a character, Front
  speaks from the upper box and its label chip can run under the
  "re-voice" button. With real settings Front stands in the lower box.
- Nothing here is live yet (step 5).

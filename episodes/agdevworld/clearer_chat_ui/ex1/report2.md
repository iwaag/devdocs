# ex1 step 2 — an explicit non-answer, from the composer to the read model

## The failure

"✕ not an answer" in the Front Desk / Arguing Room only cleared the picked
request (`answering = null`), so the post went out plain; with one request
pending, `agag.outstanding`'s next-post rule read that plain post as its
answer and the wait disappeared.

## The contract chosen

- **Wire**: `ag-post answer=none` — the post answers no request, not even
  the one the next-post rule would give it. It may stand alone or beside an
  intent. `answer=` accepts only `none` (an answer names its request with
  `re=`); `answer=none` together with `re=` is **malformed** (removed from
  the text, read as unclassified, `compose` refuses it). `PostMeta.answer`,
  `.not_answer`; `describe` → "not an answer to any request".
- **Three correlation choices**: nothing (the reader correlates by the
  next-post rule) · `re=<ids>` (answers exactly those) · `answer=none`
  (answers nothing).
- **Read model**: a recipient's `answer=none` post settles nothing and is
  never listed as `unmatched` (a Zulip quote inside it included — the
  declared choice wins). It is still their speech, so it overtakes a
  request composed before it. The convenience rule is unchanged: with one
  pending request and no choice, the next post answers it.
- **Producing it**: `agentchat send --not-answer` (refused with `--re`);
  a run may put `answer=none` on its `ag-reply` fence; `combine` carries it
  unless a reference is present. The line is in the message, so the
  journal's prepared text, redelivery and read-back keep it (tested with a
  dropped send and `resume_prepared`).
- **Agents reading**: chatlogs and `agentchat read` show "(not an answer to
  any request)"; the shared reply guide says such a post leaves the
  questions it follows open.
- **Relay**: `POST /frontdesk/<id>/post` and `POST /argues/<anchor>/post`
  accept `answers: [ids]` or `not_answer: true`; both at once are refused
  ("not both"), a non-boolean `not_answer` is refused.
- **Browser**: `Correlation = auto | answer(id) | none` (`roomState.ts`),
  one body builder for both adapters and the demo. The strip shows, before
  sending: in automatic mode "your next post answers it" / "pick the one…"
  and `↷ not an answer`; in answer mode `↩ answering #n`, `↷ not an answer`,
  `↺ automatic`; in non-answer mode a bold `↷ NOT AN ANSWER — #n stays
  waiting` chip, `↺ automatic`, and the request chips to switch to
  answering. The placeholder follows the mode. The choice resets after a
  successful send and on a conversation switch, falls back to automatic
  when its request (or every request) stops waiting, and is **kept** on a
  failed or uncertain send along with the draft. History labels a
  non-answer `↷ not an answer`.

## Verification

- pyagag `tests/test_non_answer.py` (15): round-trip, malformed
  combinations, reader words; one pending + aside stays pending; the next
  plain post and an explicit `re=` still answer; two pending + aside is not
  `unmatched`; a quoting aside answers nothing; an aside still overtakes;
  the same history through a mirror, a restart and a rebuilt mirror gives
  the same answer; `--not-answer` and its refusal; one message on send;
  `read` shows the meaning; a reply declaring it survives a dropped send and
  redelivery once. pyagag suite: 892 passed.
- Relay (342 passed): Front Desk writes `answer=none`, and the conversation
  re-read through the relay's mirror path still lists the question as
  pending with nothing unmatched; both-at-once and a non-boolean are
  refused; in an Arguing Room over a FakeRealm and a real mirror, the aside
  leaves the question pending and a following explicit answer settles it.
- Frontend: `tsc` and `npm run build` clean.
- Browser (the real relay over a FakeRealm with a scripted Front, both
  rooms; `agdevworld/.local/ccu/ex1_relay.py`, screenshots
  `agdevworld/.local/shots/ex1/`):
  - Front Desk: question #108 pending → picked (answer mode) → not an answer
    → automatic → not an answer (all visible in the strip and placeholder)
    → "Unrelated: the staging box is back" posted as `answer=none` (#109);
    after Front's reply #108 is still pending, the mode is back to
    automatic.
  - Reload: #108 still pending; #109 read back as `{answer: none}`.
  - Failed send (realm refusing): status "uncertain", draft and non-answer
    mode kept; retry after the realm recovered posted #112 with
    `answer=none`; #108 still pending.
  - Explicit answer: pick #108, "GitHub trending" → `re=108` (#115), #108
    `answered` by `reference`, strip empty.
  - Arguing Room: not an answer (#118, `answer=none`) leaves #104 pending;
    a plain post (#121) then answers it by `next_post`.

## Commits

- pyagag `7f6095b` — `answer=none`, read model, CLI, reader words, guide,
  spec, tests.
- agdevworld `592dee6` — relay (both rooms, README), composer, demo,
  labels; pin pyagag `7f6095b`.

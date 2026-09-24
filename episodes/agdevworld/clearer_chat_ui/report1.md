# clearer_chat_ui step 1 — a post says what it is for

Every post can now say whether it is **progress**, a **report**, or a
**response request** addressed to one user id, and which request it
answers. Nothing is deployed yet: consumers still hold the old pyagag pin
until step 4 bumps them.

## The contract (pyagag `b292f5d`, `docs/post-intent-v1.md`)

One line at the very end of a post's own Markdown, outside any code fence:

```
`ag-post intent=response_request to=8 ask=question`
```

- `intent`: `progress` | `report` | `response_request`.
- `to`: the addressed **user id**; required for a request, refused otherwise.
- `ask`: optional `question` | `confirmation`, requests only.
- `re`: the request(s) the post answers, by message id; may stand alone
  (a human's reply carries only this).

**Why a trailing line in the content.** The plan preferred one delivery
holding text and metadata. `agag.delivery` already settles an ambiguous
send by matching the bot's own post on its *whole content*, and the journal
already stores the prepared text before the send — so a line in the content
is prepared, journaled, read back and redelivered with the words, with no
code of its own in the retry path and no second write to lose. The checked
alternatives were worse: a `[selfnote]` beside the post is a second write
that can be lost independently (and needs the post's id, known only after
the send); a fenced machine block renders as a large code box in Zulip;
HTML comments are escaped by Zulip. The line renders in Zulip as one small
code span, which the Zulip UI user can still read; the rooms (step 3) and
every agent rendering replace it with its meaning.

Semantic intent stays apart from the rest: acks carry no line (transport),
the journal still owns execution, a mention still decides who is served
next. **No line = unclassified, and unclassified never means waiting.**

## What changed

- `agag.post` (new): `PostMeta`, `parse_post` (fence-aware; only the last
  non-blank line; a malformed line is removed from the text and reported,
  the post read as unclassified), `compose` (replaces a line, refuses what
  it could not read back), `merge`, `describe`/`label`, and `quoted_ids`
  for Zulip's own quote-and-reply links (a reference a person makes in
  Zulip without knowing this contract — used in step 2).
- `agag.reply`: the `ag-reply` fence takes `key=value` attributes
  (```` ```ag-reply intent=response_request ask=question ````). Any other
  extra word still means "not the mark". Several blocks → the strongest
  intent. A misspelt attribute keeps the reply and posts it unclassified
  (`meta_error`, logged) — never costs the answer. The repair prompt asks
  for the intent again.
- `agag.topics.serve_topic`: writes the line into the prepared text; a
  request without `to=` is addressed to the requester recorded from the
  processed input (`requester_of`), or posted unclassified when there is
  none. Handler failures and "this run produced no reply" are `report`s.
  `TopicResult.meta` lets a handler classify literal sections. The intent
  is kept in the serving record's `extra.intent` and in the run record's
  `reply.intent`.
- Agents reading a conversation see meaning, not the line:
  `format_chatlog` → `[Front] (asks Developer to answer (question)) …`,
  `agentchat read` → `(message 12, asks Developer to answer (question))`
  in each header.
- `agentchat send --intent … --to <id|exact name> --ask … --re <id>`; a
  request without `--to` is refused before anything is posted. `--help`
  explains the three intents with examples, once.
- `REPLY_GUIDE` (appended to every conversational role by
  `prompt_with_guide(…, reply=True)`) teaches the fence attributes once,
  with two examples; no role guide repeats it.
- `selfnote.is_progress` (and trace's copy) is true for
  `intent=progress`; a declared other intent wins over the `🔧/💬` shape.

## Evidence

`tests/test_post.py`, 49 tests: every intent round-trips; missing line;
eleven malformed shapes; a line inside an unclosed fence, after a closed
fence, and inside a four-backtick block holding a three-backtick one;
fence attributes, merge and misspelling; serve_topic addressing and
failure classification; the repair prompt; and the retry cases — a dropped
send resumed with the same text, a lost answer found on read-back (one
post), a crash before the send redelivered after a restart **without
running the model** (the second harness would have posted `WRONG`), and a
crash after the send settled by read-back. Three existing tests now expect
failures to carry `intent=report`; one help test forbids the word "wait"
in `agentchat --help`, so the help says "asking". Full suite: **824
passed**.

## Left for later steps

- Nothing reads outstanding requests yet (step 2), and `trace`'s
  `awaiting_human` is still the old inference.
- No consumer is on this pin, no room shows labels (steps 3–4).

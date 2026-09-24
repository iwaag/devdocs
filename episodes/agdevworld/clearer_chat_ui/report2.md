# clearer_chat_ui step 2 — which requests are still open

pyagag `932d3c6` adds `agag.outstanding`, the shared read model every room
and consumer will use, and rebases `trace`'s human states on it. Still not
deployed (step 4 bumps the pins).

## The model (`ag.outstanding.v1`)

`read_requests(messages, complete, closed, stale, is_ack)` — a pure
function of one conversation's history. No store, no call: a restart, a
second process or a mirror rebuilt from nothing concludes the same.

- **Identity** is the request post's message id. Callers locate the
  conversation by its anchor (the relay's desk/argue ids, `agag.zulip.locate`)
  and hand the history found there, so renames and ✔ do not move it.
- **The post keeps its intent.** State is separate and derived:

  | state | when |
  |---|---|
  | `pending` | the recipient has not answered |
  | `overtaken` | the recipient spoke after the asker's `seen` boundary and before the request landed |
  | `answered` | the recipient answered — `how` = `reference` / `quote` / `next_post` |
  | `withdrawn` | the asker's own later `re=` names it |
  | `superseded` | the asker asked the same person again with `re=` to it |
  | `closed` | still open when the conversation was ✔'d (reopening restores it) |

- **Correlation.** Explicit first: `re=` in the recipient's `ag-post` line,
  or Zulip's quote-and-reply link (`/near/<id>`). Otherwise the recipient's
  next post answers their pending request **only when exactly one is
  pending**; with two or more it answers none and is listed in `unmatched`
  (for the room to ask "which one?"). An `re=` naming no open request is
  about something else and settles nothing.
- **Only the recipient's speech settles.** Progress (either shape), acks,
  selfnotes, system notices, third parties and the asker's own reports do
  not. An answer is a receipt — approval, acceptance and completion stay
  with the agents' own workflows (`agentchat accept` etc.).
- **The processed-input boundary.** A request now carries `seen=<id>`,
  written by `serve_topic` from `processed_up_to`. A recipient post between
  `seen` and the request means the question was written without it; the
  listener already owes a serving for that input, so the request is
  `overtaken`, not "waiting for you". It stands again once the asker speaks
  after it without withdrawing it. This is a wire addition to step 1's
  format (`seen` key; documented in `docs/post-intent-v1.md`).
- **Uncertainty.** `complete=False` → next-post answers are
  `certain=False` and `uncertain` says the history is incomplete;
  `stale=True` → said once. Edits are read as the post is now (a request
  edited to lose its line is gone; an answer edited to gain `re=` counts);
  a deleted request is absent; a deleted answer returns its request to
  `pending`.

Agents see request ids in their chatlog (`(asks Developer to answer
(question); request #9120)`) and the reply guide says to put `re=<id>` on
the fence when a reply answers or withdraws one.

## trace

`awaiting_human` was "the agent answered last in a human's conversation" —
true of every report. Now, in that situation:

- an explicit pending request → `awaiting_human`, detail
  `#<id> Front asks Developer (question)`, evidence the request ids;
- an overtaken one → `queued` (the input is owed a serving);
- nothing asked → the new state **`answered`**.

The recorded p3 fixture (`trace_p3.json`, pre-contract) now reads
`answered`; its test was renamed to say why. `agentchat trace --help`
lists the new state. The Observer's monitor, which lists `awaiting_human`
among its "moved on" states, is updated in step 4.

## Evidence

`tests/test_outstanding.py` (33) over `fixtures/outstanding/desk.json`, a
Front Desk conversation recorded as the realm would hold it: states after
each of ten points; two questions pending with an unreferenced reply
(`unmatched {107: [103, 104]}`); autolab's interruption, progress and acks
settling nothing; reference vs quote; overtaken → still overtaken after
the next ack → pending after the asker's report; withdrawal; and the same
history read directly, through a mirror, a **reopened** mirror (no network)
and a mirror **rebuilt from an empty store** — identical `as_dict()`.
Inline cases cover supersession, next-post, recipient progress, a third
party's `re=`, an `re=` to something else, closure/reopen, incomplete and
stale sources, edits, deletion, unclassified questions and self-addressed
requests; four `trace.classify` cases cover the three human states and an
unlabelled answer. Full pyagag suite: **858 passed**.

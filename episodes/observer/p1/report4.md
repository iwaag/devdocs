# observer p1 — step 4: deployed, proved, written down

> **Read with `../ex1/` beside it.** Three claims below were fixed in `p1
> ex1` (2026-09-13): the renamed-destination test used a message id from the
> outset, so it proved nothing about a destination *written as a name*; "an
> unreadable channel is not a cancellation" was true while a renamed watch
> topic was nonetheless uncancellable; and the deleted/closed destination
> being terminal was implemented so that an unreadable one was terminal too.
> The demonstrations and numbers here stand.

## Tests

`pytest`, **36 passing**, against a Zulip stand-in whose every method is a
real `agag.zulip.ZulipClient` method — a fake that answered a call the client
does not have would let a test pass over code that cannot run.

- `test_notify.py` (7, step 3): delivered once; repeated polling posts
  nothing; a failed send is retried and stays visible; an **ambiguous** send
  (accepted, then the connection drops) is recognized on read-back and not
  repeated; a deleted destination is terminal and opens nothing; a ✔
  destination is not reopened; a renamed destination is followed by id while
  the conversation that took its old name gets nothing.
- `test_lifecycle.py` (13): the anchor id is the identity; a second anchor
  never moves it; notes written by somebody else are not ours; a request
  awaiting an answer is not scheduled; a lost store loses memory and not
  work; recovery needs no fresh post; a finished watch is not recovered;
  resolving stops the schedule **before** any model call; a ✔ landing *during*
  an evaluation still cancels; an unreadable channel is **not** a
  cancellation; an empty queue costs no inference; a failed look keeps the
  watch and says so once at the third; a nonsense interval is the default.
- `test_destination.py` (16): the parser that caught the local model filling
  a missing destination from the nearest phrase.

## Deployment

- `devenv/launchd/com.agdev.agobserver-zulip.plist.in`, installed and
  bootstrapped; `AGOBSERVER_INTERVAL_SECONDS=60` lives in its
  `EnvironmentVariables`, so changing the interval needs `bootout` +
  `bootstrap` and not `kickstart -k`. Noted in the launchd README because that
  distinction has cost this realm time before.
- Nautobot desired state: `desired_service agobserver`, its placement
  (`manual_toolchain`, `process_pattern: agobserver\.listener`),
  `desired_workspace pj-agdev` and `desired_agent agobserver-agstudio1`
  (zulip user 23). `nctl drift` went **converged=43 → 46**, error 0;
  `nctl agents observe` lists the instance with its two channels.
  `liveness=unobserved` stays an `info`, not massaged into a clean result.
- **A `desired_agent` cannot reference a placement created in the same
  batch** — the reference resolves against what is already stored, so a single
  document comes back `conflict: unresolved desired_service reference`. Two
  passes; recorded in the ignored cluster memo.
- The published contract is the introduction in `#agents`, with its roster and
  `agag-exec` blocks, generated from the running instance.

## The demonstrations

All judgments by `qwen3.8:27b-mxfp8` through `agcode` on this host's ollama.

**1. A file watch with an explicit completion signal** — `w6676`. The producer
renames `release.zip.part` to `release.zip`. Four `not_met` looks with the
`.part` present, then `met` on the fifth, 39 s after the rename. The model
honoured the *how to tell* clause every time rather than treating any file in
the directory as completion.

**2. A natural-language conversation watch** — `w6706`. Condition: *"the Omni
Agent has asked the Developer a question or made a request there that the
Developer has not yet answered; progress reports and anything already replied
to do not count."* Target: a Zulip conversation, read with `agentchat read`.

| look | conversation state | verdict |
|---|---|---|
| 1 | two progress posts | `not_met` |
| 2 | **an unrelated progress post arrives** | `not_met` — stayed quiet |
| 3 | a request aimed at the Developer, unanswered | **`met`** |

The evidence it delivered quoted the message id, the sender, the recipient,
and the three reasons it qualified. This is the whole braindump's hard case
and it was right on all three looks.

**3. A request made by an in-system agent** — `w6718`. Front was told there is
a new agent, to read its introduction, and to use it. Front did: it opened
`watch-second-zip` in Observer's channel with its own `[selfnote][rootchat]`
note, a condition, a target and `Notify: front/front-observer-p1`, then
reported and ended without waiting. Observer accepted it; the file was
completed; Observer posted the notification into `front-observer-p1` at
04:13:28, **and Front's ordinary listener served that conversation the same
second** and reported the completion to the developer.

**The acknowledgement did not wake Front.** agfront's log holds exactly two
servings of that conversation — the request and the notification — with
nothing between 04:11:40 and 04:13:28, across Observer's ack, its acceptance
post and its notes. `serve_topic(handoff=False)` is what buys that, and it is
now the load-bearing line it was designed to be.

**A thing worth knowing before anyone adds a mention route.** Front's reply to
the notification *names* Observer (`@**agobserver-agstudio1**`) — its own
`serve_topic` prefixes the last other speaker. That buys Observer no run today
only because `listener_main` is called **without** `on_mention`, so Observer
has no mention route at all. Adding one without thinking about this creates a
two-agent loop out of one delivered watch.

**4. Restart with a pending watch, and after delivery.** `w6734` was accepted
at 04:14:18 and the service was `kickstart`ed at 04:14:44. The watch resumed
at 04:15:00 with **no fresh post anywhere** — the retained request, condition
and destination all intact — and delivered at 04:15:57. The
after-delivery restart was step 3's: the worker came up, found nothing to do,
and the destination still held exactly one notification.

**5. A question, answered, becoming a watch.** `watch-missing-destination`
had been left holding one question since step 1. Answering it with a
destination turned the same topic into active watch `w6734` — the anchor was
written then, not at the question.

## Numbers

| | runs | median | range | tokens (in / out) | paid |
|---|---|---|---|---|---|
| intake | 7 | 14.8 s | 10.8–28.0 s | 23.3k / 3.6k | none |
| observe | 19 | 15.2 s | 11.0–22.2 s | 114.5k / 9.2k | none |

Every run: `profile local`, `harness agcode`, `model
ollama/qwen3.8:27b-mxfp8`, `outcome done`. **Zero paid tokens for the whole
phase**, which is the braindump's first requirement and the reason a
one-minute interval is not a question anybody has to think about.

Zulip cost per tick: one channel listing, plus one listing per active watch
for the cancellation check. An empty queue is one call and no inference.

## Incorrect judgments

**None from the observe role**, across 19 looks covering a file rename, a
permission-denied path and three readings of a conversation. Specifically not
seen: a permission error read as absence, a `.part` file read as completion,
an unrelated progress post read as a request, or a question read as
outstanding after it was answered (step 2's probe).

**One from the intake role**, in step 1, and it is the one to remember: given
"Tell me when …" it answered `accepted: true` with `"destination": "Tell
me"`, filling the field from the nearest phrase rather than reporting the
gap. The deterministic parser caught it and the requester got the right
question. The guide gained one sentence, but **the code is what protects
this**, and that ordering was deliberate before the failure rather than after.

## Remaining unproven

- **A watch that waits for hours or days.** The longest here was minutes.
  Nothing prunes `.local/topics/`, so one directory per look means roughly
  1400 generations a day per watch; 252 KB after 26 runs, so it is disk-cheap
  but not free, and nobody has watched the local model's accuracy over a very
  long previous-observation chain.
- **More than a handful of concurrent watches.** Evaluation is sequential, so
  N watches make the effective interval N × ~15 s. At 60 s that is about four
  before a watch is looked at less often than it was promised. Nothing warns
  about this yet.
- **A destination in a channel Observer is not subscribed to.** `agentchat`
  posts to public channels unsubscribed and both live destinations were
  reachable, but no private-adjacent case was tried.
- **The `sonnet` execution option** is published and never exercised.
- **Two watches on the same target**, and **a watch whose target is Observer's
  own channel** — no reentrancy thinking has been done.
- **A ✔ landing between the second cancellation check and the send.** Covered
  by test, not by a live race.

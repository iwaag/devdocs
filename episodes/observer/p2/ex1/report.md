# observer p2 ex1 — report

## What was asked

Can Observer wake an **existing** coding session (Claude Code, Codex), rather
than a realm agent's next run? Asked from inside an Omni Agent session: have
Observer wake *this* session about a minute from now.

The answer found: not by pushing into it — there is no supported way to inject
a prompt into a live interactive session from outside — but the session can
hold a cheap background wait whose end re-invokes it, and Observer supplies
the "when". That worked on the first real try, and the first try also showed
the wait itself was the weak half. ex1 fixed the weak half.

## Try 1: it wakes, and the first wake was false

A topic in `#sandbox` (no agent listens there, so the notification buys
nobody a run) was the destination; a watch asked Observer to notify when
`date -u` reached a time a minute ahead; the session backgrounded a loop around
`agentchat read --since <anchor>` that ended on "any output".

It ended **12 seconds** after it started, before Observer had even accepted
the watch. `agentchat read --since` prints `nothing newer than message N …`
and exits 0 when there is nothing new, so "any output" was always true. The
wake looked exactly like the mechanism working; only reading what woke it
showed otherwise.

The loop was patched to skip that sentence and restarted, and the real
notification woke the session:

| time (UTC) | event |
|---|---|
| 11:45:44 | watch requested |
| 11:46:02 | Observer accepts `w6917` |
| 11:47:26 | Observer's look judges `met` |
| 11:47:35 | notification posted, watch topic ✔ |
| 11:47:44 | the session is re-invoked with the notification in hand |

The patch was not general. It keyed on a human-facing sentence, and
everything else — an error, an auth failure, an unrelated post — still read
as arrival.

## What ex1 changed

**The notification's first line is now fixed and written by code.** It was
already assembled by code with only the model's evidence free-form; what it
lacked was a line meant for a program. `agag.notice` (pyagag `24e122b`)
defines it:

    [notice][watch] <watch name> <outcome>

`notice_line` builds it, `parse_notice` accepts only an exact first line, so a
notice quoted in prose is not one. agobserver's `notify.message` puts it
first in every `met` post (pj-agdev `87281a2`); the human line with the
backticked watch name stays, so the read-back that makes an ambiguous send
safe is unchanged. The model still never posts — putting the CLI in its hands
would have traded a deterministic send for one it can forget.

**`agag wait <channel> <topic> --watch <name>`** blocks until that notice is
in the conversation (exit 0, the message printed) or times out (exit 3).
Other posts and failed reads do not end it. It follows the ✔ rename, like
`read --since`.

It is in `agag`, **not** `agentchat`, on purpose. `agentchat wait` existed and
was removed in `2890a29` because a run that blocks is absent when its answer
arrives; the realm's agents are woken by being served. This wait is for what
has no listener to serve it: a session outside the realm.

pyagag 613 tests (598 + 15), agobserver 63, green. agobserver was re-pinned
and its launchd service restarted; nothing else was deployed.

## Try 2: live, with a decoy

| time (UTC) | event |
|---|---|
| 12:13:37 | anchor posted in `#sandbox` › `omni-wake-test2` (6925) |
| 12:13:46 | watch requested (6926) |
| 12:14:00 | Observer accepts `w6928` |
| 12:14:03 | **decoy** posted in the destination: the "nothing newer" sentence, and `[notice][watch] w6926 met` quoted mid-line |
| 12:14:17 | `agag wait --watch w6928 --since 6925` starts; its first read already holds the decoy |
| 12:14:35 | look 1 `not_met` (14.4 s) |
| 12:15:35 | look 2 `met` (14.6 s); notice delivered as 6933 |
| 12:15:38 | `agag wait` exits 0; the session is re-invoked |

The decoy did not end the wait; the notice did, three seconds after it was
posted. Observer's post began with the fixed line, the evidence after it.

## What it taught

**A watch's name cannot be known when it is requested.** The first waiter of
try 2 was started with `--watch w6926`, the id of the request. The watch is
named after Observer's own acceptance note (`6928`), which exists only once
Observer has accepted. A `--watch` waiter therefore has to read the
**Watching** reply before it can start. The wrong waiter would have timed out
harmlessly; it was stopped. Either the introduction says so, or the notice
also carries the request's message id so a requester can wait on what it
already knows.

**"One minute" is 1–2.5 minutes.** Observer's interval is 60 s and a look
costs ~15 s; a timer lands on the next look after the condition holds. Both
tries woke 74 s and 38 s after the threshold. A timer does not need Observer
at all — the harness can schedule its own wakeup — and Observer earns its
place only for conditions that need judging.

**The session has to be there.** The wait lives in the session's process; a
closed editor is woken by nothing, and the notification then sits in the topic
unread. This is the boundary of the approach, not a defect in it.

**What the fixed line does not fix.** It makes *arrival* deterministic. A
wrong `met` (p2's `w6866`) is still delivered with a perfectly formed first
line.

## What remains, if this is taken further

1. **Wait on something the requester knows.** Carry the request's message id
   in the notice (or accept it as `--watch`), so `agag wait` can start in the
   same step as the request.
2. **The other realm clients do not use the line yet.** Realm agents are
   served by the post itself and need nothing; only waiters outside the realm
   read it today.
3. **Headless resume (`claude -p --resume`, `codex exec resume`)** — the other
   route discussed — was not built. It wakes a transcript, not a window, and
   racing an open session for the same transcript is the two-sessions problem
   the realm already knows.

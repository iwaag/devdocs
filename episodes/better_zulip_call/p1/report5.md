# better_zulip_call p1 — step 5: listener intake separated from execution

Date: 2026-09-14. pyagag "agag.listen: intake feeds a durable queue, one
executor serves it; the sweep loop is gone"; agfront, agautolab, comfynotify,
agobserver (docstrings), cagent commits; pj-agdev pointers bumped.

## What changed

**`agag.listen`** replaces `agag.zulip.sweep_serve`. A listener is now three
things over one credential:

- **a mirror** (`agag.mirror`, step 2) of the realm's public conversations,
  registered for all public channels, persisted in the instance's
  `.local/mirror/`;
- **an intake thread** that follows the mirror's change feed. A `message`
  change whose conversation the bot *owns* (`topic_filter`) becomes an
  `owner` entry in a durable queue (`.local/mirror/listener.sqlite`); one
  that names the bot — Zulip's `mentioned` flag on the bot's own queue, or
  `@**name**` in the content — in a conversation it does not own becomes a
  `mention` entry. Entries coalesce per conversation; an event that lands
  while that conversation is being served re-arms the entry, so the executor
  looks once more when the serving ends. A topic un-resolved or renamed into
  a name the bot owns is queued too. Nothing is read on intake: the mirror
  holds the topic;
- **one executor** that takes the oldest entry (owners before mentions) and
  **evaluates the conversation as it stands now** off the mirror — the open
  topic of that name, its last real speaker (`is_speech`: no selfnotes, no
  Zulip notices), and for a mention the served mark from the bot's own
  `[served]` notes in the index — and only then calls the handler.

**Recovery reads the index, never Zulip.** At startup and after every resync
the mirror makes (a queue expiry is downtime by another name), every open
topic the bot owns whose last real speaker is somebody else is queued, and
every open topic that names the bot past its served mark; then the
`on_recover` hook runs — the obligations no last-speaker check can see.
Entries a crash left `running` are judged by conversation evidence: a reply
of ours after the ack means the run finished and the entry is dropped;
anything else is served again. At-least-once, duplicate suppression by
evidence, no promise of exactly-once external effects.

**Deleted:** `sweep_serve`, `sweep_topics`, `sweep_mentions`,
`sweep_rootchats`, `SWEEP_BUDGET_RESERVE`, and 38 tests that drove them.
`listener_main` takes `on_recover` in place of `on_sweep`. The DM route
(`agag.zulip.serve`, its own thread and client) is untouched: a direct
message is account-specific and the mirror is public conversations only.

**Adopted:**

- *agfront*: `recover_runs` (unstarted runs, delivered-report continuation)
  is the listener's `on_recover` hook, so it runs after the startup recovery
  and after every resync rather than once before the loop; `opened_runs`,
  `recover_unstarted_runs` and `pending_continuations` ask a `MirrorReader`
  when a listener is running (root notes from the notes index, histories from
  the store — no Zulip call) and a `ZulipReader` otherwise.
- *agautolab, agforge, agobserver, arxivsage*: through `listener_main`,
  unchanged at their call sites.
- *cagent* (`zulip_window.main`): a `Listener` over a mirror on cagent's
  credential (`.local/cagent-window/mirror/`), the DM-to-window thread as
  before.
- *comfynotify*: the command intake registers an event queue for all public
  channels and handles a mention the moment it lands; the `is:mentioned`
  narrow is read once at startup and after a queue expiry (`catch_up`). The
  ticket loop keeps its five-second local sweep; Zulip is no longer in it.

Generated listener templates (`agag init`) call `listener_main` and needed
no change. Deployment configuration is unchanged (same commands, same
plists); the dependency pins move in step 7 with the push.

## Verification

### Fixtures (`pyagag/tests/test_listen.py`, six tests over a mirrored fake realm)

| fixture | what it proves |
|---|---|
| intake during a long run, a burst, a later post | a handler blocked on a gate; five posts on one topic and two on another arrive meanwhile and are **two** pending entries; when the gate opens they are served in order, once each; a human's post after the reply is served again and the bot's own reply is not |
| a post during the serving | the entry is re-armed (`again`), the second look finds the bot's reply made it quiet, one serving in all |
| restart | three topics queued; the crash lands with one finished, one running without its reply, one pending; the new listener drops the finished one by evidence, re-queues the running one, serves it and the one posted while down, and **resumes the mirror's queue with no resync** |
| a mention | served once through `on_mention`; the served note written at home keeps a restart quiet; a newer naming post is served |
| recovery and the hook | an old awaiting topic is served from the index at startup; answered and resolved ones are not; the hook runs at startup; after a forced queue expiry the mirror resyncs, a topic posted while the queue was dead is served, the hook runs again; the serving client made **one** Zulip call in the whole test (`whoami`) |
| the queue | coalescing, owners first, re-arm while running, the checkpoint |

Every consumer suite passes against the new pyagag: pyagag 593, agfront 157,
agautolab 242, agforge 241, agobserver 63, arxivsage 16, cagent 198,
comfynotify 24, agentroom 274.

### Live (Front's listener, log-only mode, this host)

| event | what the log shows |
|---|---|
| first start 05:37:15 | mirror registered, **62 calls, 1.0 s** to fill; recovery from the index found nothing awaiting Front (the realm is quiet); DM queue registered as before |
| stop, restart 05:37:43 | the mirror's queue resumed — no resync line, no realm read; recovery again from the index; live at +2 s |

The old listener's restart was a full sweep: 84 calls for Front, 69 for
autolab, and (`front` log, 2026-09-12) a paid run served on a restart when
the sweep's recovery and the served marks disagreed. The new startup reads
the realm once ever per store, and a restart reads it not at all.

### What a serving still reads

This step separated *listening* from *serving*; a serving's own reads are
unchanged: the threads beside the chatlog (`remotes_for_home`, two own-note
searches and one read per remote), the `#agents` harvest, the post-run
re-check. Step 1 measured a Front serving at 13–25 calls and an autolab one
at ~60; those figures stand and are step 6's and step 7's to revisit where
the mirror can answer them.

## Limitations, stated

- A handler that posts nothing after its ack (a crash between ack and reply)
  is served again after a restart, as before — at-least-once, deliberately.
- The `mentioned` flag is authoritative only on the bot's own queue, which
  the mirror's poller is; the content match is the fallback and is what the
  index recovery uses.
- Intake follows one mirror, so a listener whose mirror is stale keeps its
  queue and its executor but sees no new posts until the mirror recovers;
  the status file says so (`last_error`).

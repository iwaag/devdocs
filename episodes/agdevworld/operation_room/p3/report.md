# operation_room p3 — routines, their sessions, and a chat that works

The braindump asked for three things: a list of routines, a chat with Front
about the selected one carrying its history, and the running sessions of that
routine — with the selected one watched more closely. All three are on screen,
as agdevworld's seventh view, and the relay behind them made **no new Zulip
call** to do it.

Steps: `report1.md` (the routine read), `report2.md` (session trees),
`report3.md` (the write door), `report4.md` (the view and the chat panel),
`report5.md` (verification, including one paid round trip).

## What exists now

- **`GET /routines`** — one row per routine: its standing request, its
  schedule, its last fire, and whether that fire was answered.
- **`GET /routines/<name>`** — the last three fires as **session trees**, plus
  the fire topic as a chat log.
- **`GET /inflight/<name>`** — the only signal here that is not Zulip: this
  host's own directories, which is why it is the only thing polled.
- **`POST /chat`** — agdevworld's first write to anything: one post into a
  routine topic of `#front`, as the Developer, on a credential of its own.
- **The routines view**, and `chatPanel.ts` finally doing what README_DEV
  promised when the embedded assistant was deleted three weeks ago: a thin
  wrapper over one Zulip topic.

## The four decisions that shaped it

**Everything reads the engine that already exists.** A routine's topics are in
`#front`; the `/ops` sweep walks every public channel and its event queue
delivers every post in them. So the routine board is a second *reading* of
memory the relay was already holding, and every state word on the new screen is
the ops board's own verdict, lifted unchanged. p1's 66 phantom stalled rows are
what a second opinion costs.

**Selfnotes link and are never rendered.** The session tree needs two edges and
each is blind where the other sees: a `[served]` note in the fire topic is the
only edge that survives a child topic being resolved (a ✔ topic is never
swept), and a `[rootchat]` note in the remote is the only edge an *in-flight*
session has, because nothing has been answered yet. Both are read; neither
appears anywhere on screen, and the chat log cannot leak one because the engine
drops selfnotes before the history exists.

**The write door is narrow, in the relay.** Its own credential with no fallback
either way, `#front` routine topics only, a length guard (this realm truncates
silently past 10000), no selfnote by hand, and no retry — because a post there
starts a paid Front run, and a retry buys a second one for a request made once.
The rules are in the relay and not the view: a screen that hides a button is a
habit, a relay that refuses is a rule.

**High frequency means the non-Zulip half.** The realm is already live on the
event queue, so watching it harder would spend the agents' quota to learn
nothing. What is polled is the host: a role workspace with no run record newer
than it is a run in flight (p1 step B, 99% of agfront's 545 runs). An instance
whose workspace root is not on this host says `known: false`, never `idle`.

## What the realm turned out to look like

Three findings the screen produced on its first live read, none of them about
the code:

- **`mediagen` has never been fired by the dispatcher.** Its fire topic is the
  busiest of the eight — 164 posts — and carries no trigger line at all. Every
  run of it was started by hand. The row says `unknown`, which is the honest
  word, and the tree shows all 28 conversations as one session.
- **The standing request is not always the latest post.** `trigger.sh` tells
  Front it is; on `ghtrends` the latest post is a Front run report filed into
  the request topic, and `rtnotes` has fourteen such posts. The board reads the
  newest post by the topic's *author* and shows the strays.
- **An ack is not an answer.** Front acks everything it is served, so the
  question "was the last fire answered" has three answers, and `acked` for
  longer than the stalled threshold is its own kind of trouble.

## Proof

- **One real round trip**, deliberately: message 4952 from the panel, ack in the
  same second, Front's answer 8 s later, rendered in the panel with no reload.
  One run: sonnet‑5, 5.6 s, **$0.084**.
- The same run left the in-flight signal's own evidence: **6.9 s** in which the
  workspace was newer than every run record.
- **91 tests**, `npm run build` clean, six screenshots, and three defects that
  only the screenshots could have found — the fifth consecutive phase in which
  looking at the screen caught something no payload showed.

## Not done, on purpose

The braindump's seventh line — aggregating every live topic into one Zulip post
so the tracking is in one place — was declined in the plan and the reason held:
the relay reconstructs that aggregation from events already, and a post would
be a second copy that can go stale. `GET /routines/<name>` is the one place the
braindump was asking for, and it costs no post.

Left for later: the process and backend tiles (p4), and the `trigger.sh`
wording that disagrees with where Front actually files a report.

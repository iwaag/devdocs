# Step 5 — verification, including the one paid round trip

## The chat round trip, once, on purpose

A short question typed into the routine chat panel of the running app, in
`#front` › `front-routine-papers`, and Front answered it. Nothing was mocked
and nothing was retried.

| | |
|---|---|
| message 4952, Developer | 15:22:00 — sent from the panel, not from a Zulip client |
| message 4953, Front | 15:22:00 — the ack, drawn as a chip and not as speech |
| message 4954, Front | 15:22:08 — *"Received here in operation_room p3 — confirming the round-trip test. No action taken; nothing pending."* |
| `agfront/.local/agent/front/run-0546.json` | claude_code · sonnet‑5 · 5.6 s · **$0.084** · 2 turns · `done` |

Eight seconds from send to answer, and the answer **appeared in the panel
without a reload**: the relay's event queue carried it back and the panel's own
five-second refresh rendered it from the relay's memory. The screenshot
`.local/shots/p3-chat-reply.png` is the whole exchange as the human sees it.

That run cost $0.084 and was the point: a post into a `front-` topic starts a
paid Front run, which is what a chat with an agent *is*. Nothing in the send
path retries, and the automated tests never touch Zulip — one of them raises if
a refused message reaches a client at all.

## The in-flight signal, proved by the same run

The run left its own evidence on disk:

```
generation dir  .local/topics/front/front-routine-papers/37/front   15:22:01
run record      .local/agent/front/run-0546.json                    15:22:08
```

**6.9 seconds in which the workspace was newer than every run record** — the
window in which `/inflight/papers` answers `in_flight: true` for
`front-agstudio1`, and it matches the run's measured 5.6 s plus the harness's
own startup. Before and after it, the endpoint says *"every workspace this host
has is older than a finished run"*, which is the sentence on screen now.

## The read side, on real data

| checked | result |
|---|---|
| `/routines` under launchd | 8 routines, schedule `configured: true, ok: true`, 239-call sweep of 119 topics |
| `schedule.json` path from the plist | read, 14 requests / 3 events — the launchd job's own environment, not a shell's |
| `/routines/papers` | 3 sessions, 82 chat posts |
| `/routines/mediagen` | 1 session (no fire), 28 nodes, 164 chat posts |
| `/inflight/papers` | 4 topic rows, 2 agents, all `known: true` |
| `POST /chat` guards | wrong topic → 403, another agent's topic → 403, 5000 chars → 403, empty → 403 |
| `npm run build` | `tsc` then vite, clean |
| `pytest` | **91 passed** |

## Screenshots, and what they caught

Five, in `agdevworld/.local/shots/`: the board, the empty chat panel, an opened
routine, its session tree, `mediagen`, and the round trip. Visual verification
has now found defects in **five consecutive phases**, and this one found three
that no test and no payload showed (report4): a chat panel blank at startup, a
`chat` key that meant two things and silently kept the wrong one, and two
sentences run together into `no run in flightgeneration 1`.

## What was not done, and why

- **`/ops` is unchanged in behaviour** except that a routine's two topics are
  now read through a ✔. Every state on the routine screen is the ops board's
  own verdict, lifted; the plan's "do not implement the state calculation
  twice" is the reason there is no second engine here.
- **No Zulip polling was added anywhere.** The browser's five-second refresh
  reads the relay's memory; the only true poll is the host's filesystem.
- **The braindump's seventh line was not implemented** — aggregating every live
  topic into one Zulip post — and the plan's reason held up all the way
  through: the relay already reconstructs that aggregation from events, and a
  post would be a second copy of it that can go stale. What the braindump
  actually wanted (one place that shows a routine's whole conversation tree) is
  `/routines/<name>`, and it costs no post at all.

## One defect for somebody else

`#front` › `routine-ghtrends` carries a Front run report, and `routine-rtnotes`
carries fourteen. `trigger.sh` tells Front the standing request is "the latest
post" in that topic, so on `ghtrends` the sentence now points at a report about
the routine rather than the request for it. The board works around it (the
request is the newest post by the topic's *author*) and shows the strays, but
the trigger's wording and Front's habit still disagree — that is a fix for the
routine's own episode, not for a screen.

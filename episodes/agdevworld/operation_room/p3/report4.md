# Step 4 — the routines view, and a chat panel that is finally a chat

agdevworld's seventh view. Eight cards, one per routine, each saying when it
was last fired and whether anything answered; clicking one opens its session
tree in the popup and its **fire topic** in the right-hand panel, as a chat you
can answer in.

## `chatPanel.ts` is what README_DEV promised

The file has been dead since `modernize_agdevworld` p1 deleted the embedded
assistant on 2026-08-17: it has been posting to `POST /api/chat`, which nothing
has answered for three weeks. README_DEV said it would come back as "a thin
wrapper over the Front agent's own Zulip conversation in a later phase". This
is that phase, and the wrapper is thin on purpose:

- it renders `#front` › `front-routine-<name>` exactly as the realm holds it —
  real posts, oldest first, nobody's summary, and no local history of its own;
- Front's ack is styled as a **chip, not a bubble**, because it is the
  transport agreeing to carry a message, and a panel that draws it like speech
  turns a quiet topic into a busy-looking one;
- the fire line gets its own style, so a run's boundary is visible in the
  scrollback;
- the send button says **"Send · buys a run"**, and plain Enter inserts a
  newline — ⌘/Ctrl+Enter sends. A key that spends money on the way past is not
  a convenience.

"Ask agent" in the detail popup is now **"Ask Front", and it composes rather
than sends**: it writes a sentence into the box and stops. A run is bought by a
human pressing Send, never by a click that landed near a card. On every other
view the button is hidden, because there is no assistant in this application to
ask any more — a button that quietly does nothing is worse than no button.

## The high-frequency half is the non-Zulip half

The plan's rule, kept literally. The realm side is already live on the relay's
event queue, so the browser reads it at a slow five seconds **from the relay's
memory** — no Zulip call is made by any of it, for the selected routine or
otherwise. The only thing that is genuinely a *poll* is `/inflight/<name>`, and
every answer in it is a `stat` of this host's own directories:

> a role workspace directory with no run record newer than it is a run in
> flight (p1 step B: 99% of agfront's 545 historical runs, and it names the
> channel, the topic, the generation and the role)

Two honesty rules came out of p1's own measurements and are in the payload:

- **`known: false` is not `idle`.** An instance whose workspace root is not on
  this host — every agent on `agautolab1` — says so. A filesystem absence is
  read the same way the realm's absences are.
- **A coarse signal beside the attributed one.** autolab's `coding` and
  `director` roles run in no topic workspace at all (100% of them, p1), so the
  panel also asks "is this instance running *anything*", which is the answer
  that keeps a screen from calling an agent idle exactly when it is busiest.

Who gets looked at is decided by the **roster**, not by who owes a reply: an
agent with no open row is exactly the one a human wants to know is still
running.

## Three defects the screenshots caught

Visual verification has now found a defect in five consecutive phases. This
time three, and none of them was visible in a test or a payload:

1. **The chat panel came up blank.** `select(undefined)` is the panel's own
   first render, and its "same routine, do nothing" guard made
   `undefined === undefined` a no-op — so the panel had no header and no
   explanation at all, which is precisely the "empty for a reason nobody
   stated" this application is built against.
2. **`chat` meant two things and one of them won.** The relay's payload carried
   the fire topic's history under `chat`, and the server then added a `chat`
   block saying whether posting is possible at all — silently replacing the
   history with the status. The panel sat on *"reading the fire topic…"*
   forever. It is `chat_log` and `chat` now.
3. **A missing separator**: `no run in flightgeneration 1 is older than…`.

## What it renders, live

- `papers`: three fires, each with its workplan topic (`quiet`, linked by a
  rootchat note) and autolab's workrun topic (`unknown`, known only from a
  served note because a ✔ topic is never swept), plus one depth‑2 child.
- `mediagen`: one session, 28 nodes, the note *"no fire from the dispatcher;
  every run of this routine was started by hand"* — and a chat that is a real
  conversation with Front about mission M‑46.
- The host section, for both: *"every workspace this host has is older than a
  finished run"* — nothing is running, said in a sentence that cannot be
  confused with "nothing is known".

`npm run build` passes (`tsc` then vite). Nothing has been posted to the realm
yet; that is step 5.

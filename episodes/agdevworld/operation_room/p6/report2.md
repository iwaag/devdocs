# Step 2 — New session

The relay gained one write route, `POST /routines/<name>/start`, beside the
existing `/chat`. It uses the same Developer credential and the same
`#front`-only rule; the body is an optional `instruction` string. The fire
text is `routines.fire_line()`, the trigger's own sentence with the mark
"started by hand from the operation room" and, when given, a trailing
"Instruction for this run:" paragraph. It passes through the same length and
selfnote checks as ordinary chat, so nothing new can reach the realm that
chat could not. Refusals come back as 409 with the relay's sentence: a
retired routine (standing request under ✔, with the topic to un-resolve
named), a routine with no standing request, or a name this relay does not
know. A routine with no fire topic yet is not refused: the first post creates
the topic, and the answer says `first_fire: true`. A failure after the post
left the relay is answered 502 with `uncertain: true` and a note to read the
fire conversation before starting again; nothing is retried, because a second
post starts a second paid run. Starting by hand touches neither
`schedule.json` nor the dispatcher, so the next scheduled event is unchanged.

The dashboard's session pane has a "New session" disclosure showing the
standing request (topic and message ID, cut at 280 characters), an optional
instruction box and one button, "Start new session · buys a run". The reason
the button is disabled is written beside it before any click: relay not live,
chat read-only, retired with the topic to un-resolve, no standing request, or
sending. On success the relay's message ID is selected immediately as a dashed
pending card, the chat span note names it, the graph says it is waiting for
the event queue, and the box is cleared; when the relay lists the fire the
pending card gives way to the real one. A refusal or an uncertain result is
shown in place and the instruction text is kept. The wait for a pending fire
gives up after one minute and falls back to the relay's list. Session cards
now say "Manual start", "Scheduled start" or "Origin unknown". Ordinary chat
is unchanged and still posts into the fire topic as a continuation.

Verification: six new relay tests cover the marked fire line, the first fire
with no topic, the three refusals before anything is posted, the unconfigured
relay, the uncertain result and the over-long instruction; 110 tests pass.
The launchd relay was restarted (one sweep, live within 45 s) and the route
answered 409 for `manual` (retired) and for an unknown name, and 400 for a
non-string instruction. The UI was driven over CDP against `:5173` with
`window.fetch` mocked for the start route only: retired disabled with reason,
`papers` enabled, refusal and uncertain results shown with the instruction
kept, button disabled while sending, pending fire selected and surviving a
refresh, chat still bound to `front-routine-papers`. Screenshot at agdevworld
`.local/p6/step2/newsession.png`. No real fire was posted: a real start runs
the routine's actual work and is left for step 6 as a deliberate decision.
`npm run build` passes.

# Step 5 — verification of the assembled dashboard

Verified the deployed `:8090` build over CDP with `.local/opsshot.mjs`, driving
the DOM dashboard through evaluated check scripts (ignored under agdevworld
`.local/p5/step5/`). Every assertion passed on the first run; no source change
was needed, so no relay redeploy (`launchctl kickstart`) was required.

Selection binding: choosing `mediagen` pressed its routine card, moved the chat
header to its fire topic, selected its single manual session and drew the real
28-conversation outlier as 29 cards with the within-bounds line. Choosing
`papers` and then its second recent fire (#4615) pressed that card, named the
fire in the chat note, highlighted exactly the seven chat messages inside the
fire window, drew the graph of that session's nodes as reported by the relay,
and marked the host observation latest-only. After waiting past the 5 s refresh
the routine, fire, pressed cards and an unsent draft were all retained. The
resource trace held only relay reads (`/ops`, `/routines`, `/routines/<name>`,
`/inflight/<name>` for the selected routine only).

Degradation: a browser-level fetch outage (chosen over stopping the launchd
relay, whose respawn costs a 241-call sweep) turned the health line, every state
chip, provenance line, session card, graph card, chat note and host observation
to unknown and disabled the composer; restoring reads recovered live health
without losing the selected fire. Truncation: injecting `truncated: true` into
the outlier's session made the graph show the amber "Truncated — reconstruction
reached nodes bound" annotation while still drawing every returned node; the
real outlier remains untruncated.

Inspected five screenshots: the outlier session, a 390-pixel viewport with
stacked panes and a horizontally scrolling routine list, the injected truncation
marker, the relay-down degradation, and the chat round trip. No visual defect
was found this step.

One real chat round trip was made through the assembled pane, deliberately and
once: Developer post 4955 in `#front › front-routine-papers`, ack 4956 in the
same second, Front's answer 4957 (`run-0547`, claude_code/sonnet-5, 8.0 s,
$0.139) rendered in the pane 15 s after the send. The draft was cleared and no
error bubble appeared. `npm run build` passed (the existing Phaser chunk
advisory remains) and all 95 relay tests passed. No credentials or local paths
were committed.

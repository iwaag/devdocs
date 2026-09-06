# Step 2 — refresh lifecycle

Selected a five-second, non-overlapping frontend refresh of `/routines`,
`/ops` health and the selected routine detail. These reads use the relay's
existing reconstruction; no Zulip polling was added. Reads time out after
eight seconds so an unresponsive connection cannot freeze refresh forever.
Page exit stops the loop and returning from the browser page cache restarts it.

Host-directory observation uses `/inflight/<selected>` at most every fifteen
seconds during normal refresh. Selection can request a fresh observation.
Historical fire selection suspends these reads and explicitly says the signal
is latest-only. Missing configuration and unread topics remain unknown.

Routine and fire identity survive refresh and recovery. Unchanged graph data
does not recreate its DOM; changed data preserves scroll position. Draft text,
chat scroll, and the standing-request disclosure survive routine refresh.
Unavailable board/detail/ops health masks current state across panes and
disables chat. Last-known evidence remains identified as such.

TypeScript/Vite build passed. CDP selected a real historical fire, waited beyond
the automatic refresh interval, checked the selected routine/fire, highlighted
chat span and unsent draft, simulated fetch failure, checked unknown routine
chips and disabled send, and verified recovery without losing the fire.
The resource trace contained only the expected relay reads and no historical
in-flight polling. Evidence: ignored `.local/p5/step2/lifecycle.png` and
`.local/p5/lifecycle-check.js`. No real chat post was made.

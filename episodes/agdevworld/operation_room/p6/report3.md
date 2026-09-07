# Step 3 — hide resolved sessions by default

`GET /routines/<name>?resolved=hide` makes the relay drop positively resolved
sessions before it takes the last three, so hiding one finished run surfaces
the run before it; the default without the query is unchanged for every
other caller. The payload's `filter` echoes the choice, `history` counts
`hidden_resolved`, and `latest_fire` still names the routine's actual newest
fire. Only `resolved` hides: `reopened`, `open` and `unknown` stay listed.

The session pane has a "Show resolved" checkbox, off by default, persisted in
the browser under one `localStorage` key with every read and write wrapped so
a browser without storage still renders. Toggling re-reads only the session
list: the routine, the chat draft and any pending manual fire stay. A
selected session that becomes hidden falls back to the newest visible one; a
routine whose every listed session is resolved shows "Every listed session is
resolved and hidden (n)" in the list and the flow pane, and the host
observation line says no latest fire is selected. Each card carries a
resolution chip with a symbol as well as a colour (✔ resolved, ↺ reopened,
○ open, ? unknown) whose tooltip is the relay's evidence sentence. A history
line under the list discloses the relay's window: fires found, posts kept,
whether the 200-post window was full ("older runs not searched"), and the
hidden count.

Chat highlighting now uses the session's own `start_id`/`end_id`, and the
span note prints them, so the highlighted messages are the fire's window even
when the neighbouring card is not the neighbouring fire. Host observation is
attached only when the selected session is the relay's `latest_fire`; any
other selection says it is latest-only.

Verification: an engine-level test feeds two fires and a resolve notice
through the event path and checks the filtered and unfiltered detail
(`hidden_resolved: 2`, `latest_fire` 20); 111 relay tests pass. The launchd
relay was restarted for the query parameter (one sweep). Over CDP on `:5173`
with the real relay: `rtnotes` hides its resolved latest fire #2201 and shows
#2163, #2129 and #2067 with "1 resolved hidden" and the full-window note; its
selected older fire is marked latest-only and the chat span is exactly
[2163, 2201); a chat draft survives the toggle and the fallback; showing
resolved lists #2201 first with the ✔ chip and the preference is in
`localStorage`; selecting #2201 attaches the host observation; hiding again
falls the selection back to #2163; `mediagen`'s single resolved manual session
yields the named empty state. Screenshot at agdevworld
`.local/p6/step3/filter.png`. `npm run build` passes.

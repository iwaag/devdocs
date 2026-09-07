# Step 1 — per-window state and zero-cost latency

Setup: Chrome for Testing 152, **not headless**, two top-level windows of
960×1040 side by side on the one 1920×1080 display, driven over CDP
(`agdevworld/.local/p8/twowin.mjs`, script `.local/p8/step1/run.mjs`).
Window A opened `/`, window B `/?routine=ghtrends`, both on the deployed
build at `:8090`. The run used is `front-routine-ghtrends-2026-09-07T11:02Z`
(fire message 5104). Resolve / un-resolve is a `PATCH messages/5104` renaming
the topic (`.local/p8/toggle.py`, the Developer, the realm's own certificate
pinned rather than verification switched off); the wall clock is taken when
the PATCH returns. Nothing was posted; no Front run was bought.

## Per-window state

- B preselected `ghtrends` from the URL and had the newest visible run
  selected with no click (`01-start-B.png`).
- A then selected `imgprompt`; seven seconds later B was still on
  `ghtrends` / the same run (`02-selection-*.png`). Selection is per window.
- **Default preferences hide a ✔'d run.** With Show resolved off (the
  default), the ✔ made B's session list read *"Every listed session is
  resolved and hidden (2). Turn on Show resolved to see them."* 4.5 s after
  the PATCH returned (`03-default-hidden-B.png`); the conversation flow pane
  said the same. The card came back the moment Show resolved was ticked. A
  monitoring window therefore needs Show resolved on, or it loses the run
  it is watching at the exact moment somebody closes it — by design, but
  worth saying in the readme.
- Preferences are per window until reload: A switched Show resolved on and
  the graph to Detailed; B kept Compact seven seconds later
  (`04-prefs-B.png`), and adopted Detailed after a reload
  (`05-prefs-reloaded-B.png`) — same origin, same `localStorage`. (B's own
  Show resolved was already on by then, so the Detailed adoption is the
  evidence.) For a monitoring window this is the right behaviour: nothing
  moves under the viewer until they reload.

## Latency, eleven toggles

`relay` = `/routines/ghtrends` on `:8094` polled at 200 ms; `browser` = B's
resolution chip polled at 250 ms over CDP. Both measured from the PATCH
returning.

| | min | median | max |
|---|---|---|---|
| relay reflects the ✔ / un-✔ | 8 ms | 9 ms | 16 ms |
| B's chip shows it | 1.63 s | 1.64 s | 3.65 s |

The relay is effectively *ahead of* the PATCH response: Zulip's event queue
delivers the `update_message` to the relay's long-poll before the HTTP reply
to the caller lands, so the relay figure is the poll interval, not a delay.
Everything a human sees is therefore the dashboard's 5 s tick. Eight of the
eleven browser figures sit at 1.63–1.64 s because the trial cadence (3 s
pause plus the measurement) phase-locked with the tick; the 3.65 s and
1.83 s trials show the spread when it did not. Expected worst case is the
tick itself, ~5 s plus a fetch; nothing observed exceeded 3.7 s.

## Files

`agdevworld/.local/p8/step1/` — `step1.json` (every check and trial), the
screenshots named above, `run.mjs`. The run topic was put back to ✔ at the
end; both windows' preferences were reset to defaults.

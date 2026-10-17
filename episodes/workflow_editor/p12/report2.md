# p12 Step 2 — Project Editor surface

agdevworld `54bf7ce` (pushed). Source only; deployment is Step 4.

## What was built

`src/boastPanel.ts`, wired into `src/projectView.ts`:

- A full-width **Boast** card below the project heading, in Simple and
  Detail. Controls: Published / Preview workspace, Reload, Open in new tab.
  The chip names the source (`main @ <sha>`, or `preview · <workspace> @
  <sha> · uncommitted`); a stale fetch is shown with its time and error.
- **Continuity.** The project view now renders into two slots around the
  panel (`display: contents`), so the iframe is never detached by
  re-renders, live events or mode switches.
- **Height**: `clamp(360px, 62vh, 760px)` by default; an optional `height`
  message sets it (120–4000 px).
- **Messages** (`wfeBoast: 1`): `height`, `copy`, `resolve-shared`, accepted
  only from this frame with origin `null` and the documented shapes; text
  ≤ 64 KiB. Copy tries `navigator.clipboard` (secure contexts only), then
  `execCommand('copy')`, then a dialog with the text selected for ⌘C/Ctrl+C.
- **States.** Absent: a compact neutral note naming `boast/index.html`, no
  frame. Unreadable: a distinct error banner; the project stays editable. A
  new published commit (60 s poll, visibility, Git events) is offered as
  "A newer version of this page is published — Show it"; nothing reloads by
  itself. Reload re-reads the state (fresh fetch) and reloads the frame.
- No mandatory format, no SDK: the page is any HTML.

## Browser check

`checks/boast.ts` builds its own fixture (three projects, an ffmpeg video,
a share) and serves the built UI on this host's LAN name over plain HTTP.
All 24 checks pass in **Chromium and WebKit**:

- same-commit stylesheet; opaque origin; no access to the editor DOM; write
  API refused; reported height applied;
- the frame keeps its document and a playing video (1.5 s → 5.5 s) across a
  mode switch, Refresh, a Git event and a project.yaml change; seeking to
  9 s works (ranges);
- copy on the HTTP LAN origin answered `execCommand` in both engines (the
  clipboard API is absent there, as expected); the manual dialog is the
  remaining fallback and is exercised only when execCommand fails;
- `shared://` resolution and an inline shared image;
- a new published commit is offered without reloading; showing it loads it;
- preview shows uncommitted CSS and is labelled;
- open-in-new-tab targets the commit URL and is still an opaque origin;
- absent is neutral, unreadable is an error, settings stay reachable;
- no horizontal scroll at 390 px; no unexpected console errors.

Not checked here: the `/wfe/` nginx route and the deployed service
(Step 4). Whether the person's own Safari copies via execCommand from a
real click was checked only with Playwright WebKit.

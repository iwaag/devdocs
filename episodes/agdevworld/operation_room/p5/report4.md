# Step 4 — dashboard entrance and deployed web build

Promoted the dashboard to `/`, with `/?parts=shell` retained as an alias.
It is the primary routine workflow and directly exposes the four coordinated
panes, so hiding it behind an implementation workbench URL would make the
finished screen unnecessarily difficult to find. Phaser is now loaded only
when entering a world view, via `worldViews.ts`.

Retained the seven-view cycle at `/?view=nodes`, the compact Phaser routines
board at `/?view=routines`, and the distinct stalled-first board at `/?view=ops`.
The dashboard links to Ops board and World views; each world view links back.
No eighth scene was added. The graph workbench remains at `/?parts=graph`;
the superseded placeholder shell implementation was removed.

Updated agdevworld `README_DEV.md` for entry points, shared selection,
refresh/host-scan lifecycle, health and reconstruction boundaries, verification,
and deployment. Corrected obsolete descriptions of the chat door and relay.
The relay README documents the new session truncation metadata.

`npm run build` and `docker compose up --build -d web` passed. The production
web service returned HTTP 200. CDP on the rebuilt service confirmed the default
DOM dashboard, direct Ops entry, and the Ops-to-routines cycle. Inspected all
three screenshots in ignored `.local/p5/step4/`. The existing large Phaser
chunk advisory remains, but that chunk is no longer loaded by the dashboard.
No real chat post was made.

# Step 5 — recognisable routine names

Every routine row now carries a `display` block from the relay: an icon, a
title, and the source of each. The metadata's home is the standing request
itself: an optional line `display: 📰 Papers digest` anywhere in the request
text, read by `display_of()` in `routines.py`. The standing request is already
the document the Developer edits for the routine, so a title travels with the
request's version, needs no file, no environment variable and no
provisioning, and Front reads past the line like any other. When there is no
such line, a markdown heading on the request's first line is the title; when
there is neither, the title is the internal name. A leading emoji on the
chosen line becomes the icon. Otherwise the icon is assigned from the name
by FNV-1a over a 24-symbol palette, with assigned icons stepping to the next
free slot in name order so no two routines on one board wear the same symbol;
chosen icons are never displaced. The payload says `icon_source: assigned`
so a card can say so on hover. No standing request was edited: posting a
display line as a new message would replace the request Front reads, so the
line is something the Developer adds in the next version of a request.

The routine card leads with the icon and the title; the internal name is
secondary monospace text and is omitted when it is the title. The state chip
and the next scheduled time share one line; overdue counts and the overdue-ack
annotation stay as annotations. Time since fire, the title and icon sources,
and the provenance line appear only on the selected card, so the list stays
identity, state and next time. Retired routines are dimmed with a struck
title and the word "retired" under the name; the accessible name of every
card carries title, name, retired and state.

Verification: four new relay tests cover the display line, the heading and
name fallbacks, the row carrying display without a mapping, and icon
uniqueness with a chosen icon kept; 115 tests pass. The relay was restarted
and answers eight distinct icons where the first hash had three collisions.
Over CDP with the board read mocked to add a ninth routine with metadata and
a deliberately long title and name: nine unique icons, the title prominent
and the name secondary, the long card not overflowing horizontally, `manual`
marked retired, timing only on the selected card, and a card reachable by
focus and selectable by keyboard activation. That last check found a defect:
the five-second refresh rebuilt the cards and dropped keyboard focus. Card
redraws now remember the focused routine or session and refocus it. The
check also exposed a duplicate `data-routine` attribute on the New session
block, renamed to `data-for`. Screenshots at agdevworld
`.local/p6/step5/{names,narrow}.png`; the 390-px shot scrolls the list
horizontally with the icon and title readable. `npm run build` passes.

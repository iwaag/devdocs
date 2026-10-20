# workflow_editor p16 — Step 5: the discussion experience

aghangout has meeting rooms for discussions, and WFE Home has an entrance to
them and to their records. Both are deployed and checked against the fixture
(Chromium and WebKit, desktop and phone) and against the live, integrated API.

## aghangout (pj-aghangout `8c8b13d`, `7cd02c6`, `b050db8`; served `b050db8`)

| Part | Files | What |
| --- | --- | --- |
| Contract client | `src/api.ts` | Discussion types (participants, members, membership, messages with `participant`/`run`/`invitation`/`addressed`, invitations, archive status) and calls (list, view `?since`, create, post with `to`/`re`, members, close/reopen) |
| Data handling (tested) | `src/discussion.ts`, `test/discussion.test.ts` | Merge for `?since`; the speaker is the participant WFE recorded; addressable members; every invitation with its phase (waiting for a slot, working, answered, unanswered, failed, withdrawn), run and answer; syntheses and hand-offs read apart from speech; whom a message reached (WFE's invitations, not inference); the archive in words |
| Meeting room | `src/meeting.ts`, `src/styles.css` | The lobby picture (`MEETING_ROOM`, the person's choice in p16-s3-room #130: no new backdrop) with every member seated, the speaker raised; each line under its participant with the agent, kind, `re`, `#id`; a synthesis drawn as "an interpretation" (dashed box); chips to the line's run and invitation, whom the person's words reached, "reply to #n"; status: who is working or waiting for a slot, failed invitations with WFE's reason ("Nobody speaks for … instead"), questions, closed state, archive failures |
| Composer | `src/meeting.ts`, `src/local.ts` | **To:** Everyone (the facilitator takes it up) or chosen participants; replying to a participant's message preselects it (a reply to its question goes back to the run that asked); drafts and an outbox per discussion, retried with the same token |
| New discussion | `src/meeting.ts`, `src/lobby.ts`, `src/main.ts` | Lobby section *Discussions* (recent ones, members, open invitations, last line) and **+ New discussion** → `?discussion=<new id>`: title, who joins (unavailable projects cannot be chosen), the brief; creation is idempotent by id |
| Panels | `src/meeting.ts` | **History** (original speech verbatim, synthesis marked), **Invitations** (state, run, horizon, question, outcome), **Members** (current, join/leave history, add/leave, close/reopen), **Records** (archive state, the archived revision's Gitea link, repository, syntheses, hand-offs) |
| Facilitator | `src/characters.ts` | `argue`: name, "facilitator", role; no portrait (initials), as no art was drawn |
| Fixture | `fixture/server.ts` | A scripted `api/discussions`: the facilitator answers and invites the members, each answers, then a synthesis; direct answers; `sage` fails as unavailable; archive behind then current; failing posts and an unreachable WFE as before |

Deployment: `scripts/deploy.sh` (clean export, `npm ci && npm run check`),
served at `/hangout/`. aghangout-agent's own release `148a8aa` (the
hand-off of Step 4) was built on top of `8c8b13d` and kept it.

## WFE Home (agdevworld `947d0b9`, deployed)

A **Discussions** card in Home's glance row (both display modes): recent
discussions with state, member count, messages, open invitations and the
archived revision (linked to the Gitea tree at that commit), each opening its
meeting room in aghangout (`/hangout/?discussion=<id>`), plus a link to start
one there. `api/discussions` now lists each discussion's archive state.
Also: a discussion whose open/closed state is not published yet reads
`behind`.

## Checks

**Fixture** (`npm run check`: 12 unit tests; `npm run browser-check`, both
engines: all pass, 60+ checks including the conversation ones from p15):

- setup: unavailable participant not choosable, the chosen sit at the table,
  a title is required and the words stay;
- the facilitator speaks under its own name; the participant's line under
  its own name and agent; the speaker raised; a synthesis drawn as an
  interpretation and marked so in History;
- invitations listed with outcomes; an addressed message invites only those
  named (checked in WFE's record); a failed invitation shown with its reason;
- Records: archive link and the synthesis;
- **long contribution** scrolls inside the box, nothing overflows sideways,
  **← / →** move between lines;
- **failed post** stays with its reason, the **draft survives a reload**,
  **Retry** is one message in WFE;
- the lobby lists the discussion; **narrow**: the meeting fits a phone and
  the dialogue sits above the composer.

A narrow-layout defect was found by the checks and fixed: four tabs made the
header, and with it the grid column, wider than a phone; the column now
never grows past the screen and the tabs scroll.

**Integrated API, read-only** (`.local/p16/step5/deployed.mjs`, run on this
host; Step 6 runs it from another machine): the lobby lists p16-s3-room and
p16-s4-provenance; the meeting room opens at its address with Argue, Forge
and Archsage seated; every line under its participant and agent; syntheses
marked; addressed words show the addressee; 8 invitations with outcomes;
"Archived through #139 (7217f17)" and the archived revision opens (200);
the hand-off `aghangout/change/run-002` is linked and its run view opens;
Home lists the discussions with their archived revision and links into the
meeting room; the room fits a phone; no page errors. A first run caught a
moment where only the facilitator was seated before WFE answered; fixed in
`b050db8` (nobody is drawn until the view arrives).

## Deus Ex Machina note

The meeting rooms and Home's entrance were built by the Omni Agent —
handoff candidate for aghangout-agent (meeting rooms) and for an agdevworld
change (Home).

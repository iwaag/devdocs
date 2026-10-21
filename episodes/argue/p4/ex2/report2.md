# argue p4 ex2 — Step 2: metadata, overviews and status in WFE

## Implemented contract

agdevworld `8485dcd` contains the implementation, contract and tests. Its
contract section is `workflow_editor/docs/discussions.md`, *Reading a
discussion*. pj-ponderer `612a7b6` contains the guides.

### Store v9 (conversation store)

| Addition | Purpose |
| --- | --- |
| `conversations.category` (`discussion` \| `check`) | purpose category; the migration sets every existing discussion to `discussion` explicitly |
| `conversations.display_title` | a readers' title, only when recorded |
| `discussion_meta` | append-only history of category and display-title changes: field, value, by, run, at and note |
| `overviews` | append-only overview revisions: author, run, purpose, outcome, unresolved, `through`, sources, note, at and token |

### Writes

- **Category at creation**:
  - `wfe discussion create … --category check`;
  - `POST api/discussions {category}`.

  The default is `discussion`. The creation is recorded as the first
  `discussion_meta` row.
- **Corrections**:
  - `wfe discussion meta <id> [--category …] [--title …|--no-title] --note <why>`;
  - `POST api/discussions/<id>/meta`.

  Only the two fields and their history change. Messages, invitations,
  runs, the open/closed state and `updated` stay as they were, so the list
  order does not move.
- **Overviews**:
  - `wfe discussion overview [<id>] --purpose … [--outcome …] [--unresolved …]… [--through n] [--title …] --note …`,
    or `--file <json>`. Without either, the command shows the overview;
  - `POST api/discussions/<id>/overview`.

  **Validation**:
  - every part must cite at least one message as `[#n]`;
  - each cited message must belong to the discussion and be no newer than
    `through`;
  - from a run, only the facilitator's or the archive project's workspace
    may write (ponderer). A participant gets 403 `not-facilitator`.

  An overview never sends or starts anything. An identical overview is the
  same revision.

### Reads

- **The list**: `GET api/discussions?category=&state=&limit=&offset=`
  (CLI: `wfe discussion list --category … --state …`) returns:
  - `counts` per category and state, and `total` for the filter;
  - for each row:
    - `category`, `displayTitle`;
    - `participants` (project members only);
    - the `overview`, with `after`, the count of messages newer than its
      horizon. Without one, a labelled `brief`: the earliest person
      message or conductor announcement;
    - `status`.

  These are computed only for the rows of the requested page, and the
  client receives no transcript. `last` (the last-message preview) is
  removed.
- **`status`**:
  - `room`;
  - `working`: open invitations, awaited results, live runs and
    delegations;
  - `owed`: every obligation;
  - `awaiting`: each open question with its receiver. A `talk` question
    goes to the requester, either a person or a run with its agent. Other
    questions go to the configured person. `you` is true only when the
    receiver is the configured person;
  - `failed` invitations;
  - `held` runs.

  All of it comes from WFE's existing `obligations()`, `openAsking()`,
  activities and invitations.
- **The room view** also has `category`, `displayTitle`, `overview`,
  `overviews` (every revision), `brief`, `status` and `meta`.
- **Archive**:
  - transcript records `meta` and `overview`;
  - horizon fields `meta` and `overview`, each counted as 0 when absent,
    as in older manifests;
  - manifest fields `category`, `displayTitle` and the current `overview`.
    The live `after` count is left out, because the horizon states the
    coverage;
  - a metadata change or a new overview makes the discussion due;
  - the commit trailer gains `t<meta> o<overview>`.
- **WFE Home**: the Discussions card lists ordinary discussions only, by
  display title, with counts for both views and an "owed" or "waiting for
  you" note.

### ponderer's guides (`612a7b6`)

- `guide.md`:
  - the tool list adds `overview` and `meta`;
  - *Records are kept apart* says what an overview is and is not.
- `conduct.md` states as facts:
  - the overview is ponderer's;
  - readers pick discussions by it and read Japanese;
  - it cites the original speech and keeps disagreement and open work;
  - a new revision follows a change of purpose, outcome or open points.
    Not every turn needs one;
  - category and display title are corrected with `meta`.
- `synthesize.md`: a summary that changes the takeaway also revises the
  overview.

None of this adds a mandatory procedure.

## Migration (applied to the live store)

### Steps

- **Backup**: `.local/agdev/.local/backups/argue-p4-ex2-predeploy-20261021-151205/`.
  It holds:
  - the conversation store at v8 and the exec store;
  - `conversations.json` and `registry.json`;
  - the archive writer's clone;
  - a mirror of the pj-ponderer workspace.

  No attempt or job was active (573 attempts ended; jobs: 573 done and 2
  dropped).
- **Dry run**: the six overviews and one correction, run with the new code
  against a copy of the live store, which the dry run migrated to v9 in
  scratch space. All citations validated.
- **Deploy**: WFE `8485dcd` (from `153bdad`), with
  `scripts/wfe-release.sh deploy`.
  - The build runs the full check: 488/488 tests, `tsc` and the vite
    build.
  - The service restarted healthy, and the store migrated to v9 on first
    open.
  - Step 4 covers the aghangout deployment and LAN verification.
- **Apply**: `pj-agdev/.local/argue-p4-ex2/step2/apply.sh`, as
  `Omni Agent (for Developer)`.
  - Each `meta` command carries the note "migration map (argue p4 ex2
    report1)…".
  - The six overview files are in `step2/overviews/*.json`.

### Results

- **Categories**:
  - 6 `check`: `p3-s2-fixture`, `p4-s3-check`, `p4-s5-a`,
    `archsage-topic-line`, `forge-media-gap`, `20261020-233458-r93d5v`;
  - 6 `discussion`;
  - 0 uncertain.
  - The counts read `discussion {open 2, closed 4}` and
    `check {open 1, closed 5}`.
- **Overviews and display titles** for all six ordinary discussions. Each
  is written in Japanese and cites the original messages. Each keeps what
  is unresolved, for example:
  - `argue-p4-trial`: the review the brief asked for was not held before
    the discussion concluded;
  - `p16-s4-provenance`: the promised report that the knowledge revision
    landed never came;
  - `argue-p3-ex1-trial`: the later assessment of the "not on main
    (checked)" claims, marked as outside the speech.

  The display titles keep what distinguishes the three RTS×Jev
  experiments: 1回目・実行委員会方式, 2回目・Omniの助言つき and
  3回目・ponderer進行.
- **Nothing was sent or started**:
  - messages: 247 before and 247 after;
  - invitations: 70 before and 70 after;
  - jobs: 575 before and 575 after;
  - the list order is unchanged.
- **Archive**: `wfe discussion archive` published 1–2 records per
  discussion. All 12 are `current` in pj-ponderer.
- **Live status checks**:
  - `p16-s3-room` shows one failed invitation: lowerpoly, "no workflow
    `discuss`", WFE's recorded reason;
  - no discussion is waiting for the person;
  - nothing is owed.

> Deus Ex Machina note: did the historical category classification and
> the initial overviews and display titles of six discussions for agent
> ponderer — handoff candidate.

## Focused tests

These are new in `test/discussionReading.test.ts` (7 tests).

| Test | Covers |
| --- | --- |
| citations | `[#n]` and `[#n #m]` are parsed; a bare `#3` is text |
| category and meta | creation default and `check`; a bad category; correction with a reason; history; **no message, invitation, job, reopening or reordering**; idempotent value; removing the display title |
| overview | refused when there is no message; each part needs a source; foreign and too-new sources; a participant's run refused, ponderer's allowed; no invented outcome; `after` = 3 after three messages (stale but current); a new revision resets it and the earlier one stays; nothing sent; the brief fallback |
| status | a talk to the person gives `you: true`; a talk to an agent requester (a front run) gives a `run` receiver and `you: false`; a failed (unavailable) invitation; a **closed room with work still owed** (`working`/`owed` > 0); a conductor-opened discussion's brief is its announcement |
| list | 25 + 3 discussions; counts; page 20 + 5 reaches all 25; state filter; the status shape; bad filters refused |
| archive | meta and overview records; manifest fields with no live `after`; a change makes the discussion due; **replay publishes nothing twice** |
| store v9 | a real v8-shaped store migrates to `discussion` without writing history; category, title and overview survive a restart and **a reopening** |

These existing tests were extended:

- `discussions.test.ts` HTTP: `meta` and `overview` routes, a refused
  overview with no source, no job started, list filters and counts, and
  the CLI `list`, `overview` (write and show), `meta --no-title` and
  `show`;
- `conversations.test.ts`: the downgrade fixtures drop the v9 objects.

The full suite has 488 tests: 488 pass and 0 fail
(`.local/argue-p4-ex2/step2/wfe-check.log`).

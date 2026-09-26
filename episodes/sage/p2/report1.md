# sage p2 — step 1 report: reusable project and routine tools

## What changed

- **`agproject` moved into pyagag** (`agag.project`, pyagag `67a7abe`) and
  agfront's copy is gone (agfront `1f86ac7`; Front's `argue` imports the
  constants from pyagag). Any consumer of pyagag now has the command, so
  archsage can be granted it without depending on agfront.
- **`open` creates or continues.** It reads what exists first — channel,
  members, document, setup request, autolab's answers, and the internal
  `main` repository on the host's Gitea — and writes only what is missing.
  It never creates a second channel, document or setup request. A study set
  up by hand (its repository exists, but it has no setup topic) is `ready`
  and gets no setup request.
- **`agproject status <slug>`** is the read-only inspection. It reports one
  of these states: `absent`, `channel-only`, `document-posted`,
  `setup-pending`, `answered`, `ready` or `archived`. It also gives the
  channel id, the document and setup message ids, the conversation that
  autolab's answer returns to, the repository and revision, and what
  remains. `--json` prints the same as data.
- **Subscribers are resolved explicitly**, by why they belong:
  - the realm's human owners;
  - the routine runner and the workspace agent, read off the `#agents`
    board by the prefixes their introductions answer (`routinerun-`,
    `workplan-`);
  - the caller;
  - the board reader (`AGAG_BOARD_READER`, default `Opsroom Observer`).

  The old helper assumed the caller was Front. Since the runner is now
  resolved from the board, Front is subscribed even when archsage calls.
- **The setup request is machine-readable.** It carries a fenced
  `ag-setup` block (`ag.project-setup.v1`): pattern, slug, channel,
  document by message id, `knowledge: main`, `publish: none`, `about` and
  origin. It also carries a root note naming the caller's conversation.
  `status` says `ready` only once autolab's answer contains
  `study layout established: main/ = <repo> at <commit>` (that line is
  added in step 2), or once the repository exists on Gitea for an older
  setup. Front's argue now uses the same check for a `study` outcome.
- **`agroutine create | update | show | list`** (`agag.routine`):
  - `create` puts the `routine-<name>` channel in the `routine` folder,
    looked up by name. It uses the exact description text and subscribes
    the owners, the runner and the board reader.
  - Each guide revision is posted as a complete post by the caller and
    read back by id. A stored text that differs from the file, or holds
    `[message truncated]`, is a failure (exit 1).
  - A repeated `create` posts nothing. `create` with a different guide is
    refused and points at `update`. A retired (✔) guide is refused.
  - A channel that exists but is incomplete is completed: missing members,
    folder, description, or the first guide.
- Both tools explain their purpose, inputs, outputs, states and recovery in
  `--help`. Channels are created with `AGAG_PROVISIONER_ENV`; every post is
  made as the caller (`AGENTCHAT_ZULIP_ENV`).
- **`TopicResult.quiet_progress`** (pyagag): when a handler sets it, a reply
  that declares `intent=progress` names nobody. It exists for step 2: a
  run that ends while it waits for a delegated setup should not buy its
  requester a run just to read "still waiting".

## Verification

- **Tests.** pyagag `tests/test_project.py` and `test_routine.py` (14),
  plus a reply test for the quiet progress reply. They cover:
  - new creation with every member and a structured request;
  - a repeat that writes nothing;
  - a failure injected after the plan was posted, then continued with no
    duplicate channel, plan or request;
  - a hand-made study that is completed rather than replaced;
  - a study set up by hand that is ready by its repository;
  - pending → answered → ready as the answers arrive (across a ✔);
  - refusals;
  - routine create, repeat and update;
  - a channel completed in place;
  - truncation and retirement.

  Full pyagag suite: 919 passed. agfront suite against the new pyagag:
  179 passed.
- **Live, existing-artifact inspection** (Omni Agent credential):
  - `pj-worldtrend`: `ready`, plan #7233, setup #7235 (prose, no block),
    returning to `argue/argue-20260918-124701`, main at `50e841c`.
  - `pj-protoprey-research`: `ready` at `89c5240`.
  - `pj-studyindustry`: set up by hand, with no plan or setup topic. The
    first reading said `channel-only`, so the rule "repository exists,
    no setup topic → ready" was added.
  - `pj-aisvgs`: absent.
- **Live routine probe** (`routine-sagep2probe`, stream 218):
  - Created in folder 13 with Developer 8, Front 15 and Opsroom Observer
    22 (plus the caller).
  - Guide #11491 read back intact.
  - Within 20 s the relay's `/routines` listed it with its `display:`
    title ("sage p2 probe").
  - The guide post was already in the mirrors of the **already running**
    Front, autolab, archsage and Observer listeners. No restart was
    needed; the older advice to restart after a subscription no longer
    applies.
  - `update` posted #11492 intact, and a repeated `create` posted nothing.
  - A 15,226-character guide was stored as 10,000 characters (#11493),
    and `update` reported the truncation with exit 1.
  - Afterwards the probe guide was ✔ and the channel archived.
- **Not exercised live in this step:** creating a new `pj-` channel and a
  retry after a real failure. A new study's setup request would already be
  served by the running autolab, which does not yet recognize the block
  (step 2). Creation is exercised live in step 5 through the agents.

## Decisions

- **archsage holds the provisioner credential** through its process
  environment (`AGAG_PROVISIONER_ENV`). That is the plan's choice. It is
  one more holder of a realm-admin credential, and it is used only to
  create, file and subscribe channels.
- **Routine guides are signed by whoever posts them** (archsage for the
  ones it writes). The board reads the newest post whoever wrote it.

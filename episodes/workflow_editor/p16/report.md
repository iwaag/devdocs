# workflow_editor p16 — argues and shared memory: final report

A person, a facilitator and project agents with their own missions and
contexts now discuss in one shared place, and what they find is kept in one
system-wide repository, `pj-argue`. WFE owns the discussion's delivery and
execution, aghangout presents it as a meeting room, and argue owns the
archive, the syntheses and the cross-discussion knowledge. Everything below
is deployed and was verified from another LAN machine.

Step reports:

- [report1](report1.md): the boundary — a discussion as a conversation
  kind, invitations, facilitator continuation, the archive's one writer;
- [report2](report2.md): the WFE discussion surface;
- [report3](report3.md): argue, participant workflows, the first live
  exchange;
- [report4](report4.md): archive proofs, synthesis, knowledge, hand-off,
  reuse;
- [report5](report5.md): meeting rooms and Home's entrance;
- [report6](report6.md): deployment, trials and the evaluation.

## Review links

- Meeting rooms in aghangout:
  `http://agstudio.local:8093/hangout/?discussion=p16-s3-room` (the first
  exchange: two projects disagree, a specialist question, a late newcomer,
  delegated work, closed and reopened) and
  `http://agstudio.local:8093/hangout/?discussion=p16-s4-provenance` (a later
  discussion that reused and revised knowledge). Over Tailscale:
  `http://agstudio.tailab7641.ts.net:8093/hangout/`.
- WFE Home, *Discussions*: `http://agstudio.local:8093/wfe/`
- The archive and knowledge:
  `http://agstudio.local:3000/developer/pj-argue/src/branch/main/discussions`,
  `…/src/branch/main/knowledge/room-image-provenance.md`
- The hand-off to aghangout:
  `http://agstudio.local:8093/wfe/#/ws/aghangout/run/change/run-002`

## Delivered behaviour

- **Discussions in WFE** (store v5; `docs/discussions.md`; `wfe
  discussion`; `api/discussions`): members with history; verbatim messages
  under the participant WFE recorded, with agent, run, attempt, invitation
  and `re`; invitations as the only way a run starts (by receipt, with
  horizon and outcome); routing by addressee, the facilitator for
  unaddressed words, a reply to a participant's question to the run that
  asked; the facilitator continued once when everything it awaits is
  resolved; contributions start nothing; close/reopen without replays;
  unavailable participants visible and non-blocking.
- **The archive** in pj-argue (`manifest.json`, `transcript.jsonl`) by one
  writer in every reconcile, appending from the published horizon, with
  visible status; proven for concurrent discussions, an interruption after
  the push, failed and rejected pushes, and a fresh clone.
- **argue** (managed project): facilitation, syntheses that keep
  disagreements and open questions with sources, knowledge with revisions,
  hand-offs as ordinary requests pinned to `argue@<commit>`, linked back;
  `tools/argue` to read, validate and cite.
- **Participants**: `forge/discuss`, `archsage/discuss`,
  `aghangout/discuss`, with project guides and shared facts in the wfrun
  guide; real work goes to the project's own workflows (forge's `provide`
  by delegation).
- **aghangout** meeting rooms (members at the table, speaker under its own
  name, To:, History, Invitations, Members, Records, drafts and outbox) and
  **WFE Home**'s *Discussions* entrance.

## Deployed revisions

| Repository | Revision | What |
| --- | --- | --- |
| agdevworld | `7286d14` (from `ded957c`, `6625c65`, `947d0b9`) | Discussions, archive writer, API/CLI, Home entrance |
| agautolab | `c16f548` (`6c17e17`, `90f6215`) | wfrun guide: a run that speaks in a discussion |
| pj-argue (new) | main (`b2e20a0` created; guides `5d74f4d`; archive and runs since) | The argue project and its records |
| pj-forge | `c37161c` (`41c8436`) | `discuss` workflow and guide |
| pj-archsage | `d04c7bf` | `discuss` workflow and guide |
| pj-aghangout | served `b050db8` (`8c8b13d`, `148a8aa` by aghangout-agent, `7cd02c6`, `9ea7e7f`) | Meeting rooms, checks, `discuss` workflow; ASSETS.md provenance (hand-off) |
| pj-agdev | pins through `fa2b145` | agdevworld and agautolab |
| devdocs | this episode; README_DEV `e228249` | Reports, *Discussions and argue* |

Configuration: `conversations.json` `discussions` (facilitator
`argue/facilitate`; forge, archsage, aghangout; archive `argue`). Recovery
(code and data) and environment notes are in the ignored
`pj-agdev/.local/p16/notes.md` and `pj-agdev/.local/devenv.md`.

## Completion criteria

1. **A deployed discussion supports a person, a facilitator and at least two
   project agents speaking through their own contexts and runs** — p16-s3-room
   (forge, archsage, later aghangout) and p16-s4-provenance (forge,
   archsage), each turn a run of the project's own workflow.
2. **WFE owns delivery and execution; aghangout presentation; one argue
   repository synthesis and knowledge** — as built; no second live store,
   no transcript editing outside the writer.
3. **Original speech, participant history, reply references and run
   provenance survive retries, concurrent contributions and restart** —
   tests, a live replay and a live restart of the service and executor with
   no duplicates (report6).
4. **Logs and source-linked synthesis published in Git, with inspectable
   progress and proven recovery from publication failure** — report4.
5. **A project receives useful work with pinned provenance, and a later
   discussion reuses accumulated knowledge** — aghangout `change/run-002`
   (`148a8aa`, deployed); p16-s4-provenance revised the knowledge, keeping
   revision 1.
6. **Direct human participation and records through the agdev entrance,
   verified from another LAN machine** — agpc: read-only 15/15 and a
   discussion started through the page.
7. **Existing clients work; no compatibility machinery or approval gate**
   — Slack-to-Front and direct conversations checked after deployment; the
   store migrations are additive and nothing waits for approval.
8. **An evidence-based account of the experiment** — report6, *Did the
   interaction produce something?*: one new option that neither opening
   position contained (canonical per fact, the hash as the test), one
   exposed assumption (no recorded prompt or seed), proposals changed by
   another project's constraint (aghangout's `cover` cropping), and the
   unproductive repetition recorded too. A small trial shows behaviour, not
   superiority over a single model.

## Findings

- The facilitator pattern works with the existing continuation: invite,
  wait without holding the slot, be continued once. Participants answered
  each other's actual words when they could read the original speech.
- The most valuable exchanges came from projects reading their own records
  (archsage's C2PA find, aghangout's CSS) — the "different context" part of
  the experiment, more than different opinions.
- Three gaps were found only by running it and were fixed: the reply to a
  participant's question, a synthesis that did not report back, results
  without links.

## Remaining limitations

- **One conversation slot**: participants answer one after another (up to
  68 s queued seen). Long work leaves the slot; capacity unchanged.
- **Delegation answers travel through Zulip** (53 s seen across a restart);
  a discussion turn that delegates waits on that path.
- **A requested synthesize run reports to the discussion only if it says
  so** (guided since `5d74f4d`; p16-s4-provenance's gap was left as found).
- **A failed invitation still makes the invitee a member** (lowerpoly stays
  listed in p16-s3-room); the failure is shown, but the seat is too.
- **Characters**: argue and aghangout have no portrait (initials and the
  project name); new artwork was out of scope.
- **Citations drift** in places (an agent cited a file at a commit where it
  did not exist yet); `tools/argue cite` exists for transcript citations
  but nothing checks knowledge citations.
- Not built (out of scope): Slack group discussions, importing the retired
  Zulip argues, multi-user accounts.

Deus Ex Machina notes (each also in its step report): the participant
workflows and guides of forge, archsage and aghangout, aghangout's meeting
rooms and WFE Home's entrance, and corrections to argue's guides were made
by the Omni Agent — handoff candidates for those projects' agents.

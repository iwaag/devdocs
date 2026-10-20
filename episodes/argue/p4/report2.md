# argue p4 step 2 — workflow activities and reliable continuation

WFE agdevworld `fe2fbdb` (from `978a5ed`). Not deployed yet (step 5).
Checks: `npm run check` — tsc, **475 tests pass**, vite build.

## What was built

| Plan item | Implementation |
| --- | --- |
| Activity representation | Node field `activity: announce \| ask \| ponder \| talk` (`shared/model.ts` `ACTIVITIES`), validated (`activity-kind`, `activity-type`: talk on a talk node, the others on do nodes), in the definition digest, compiled graph, adjustment diff and YAML round trip. Canvas and run side panel show it on every card |
| Correlation and recovery | Store v7: `activities` + append-only `activity_events` (each event keeps the activity as it was), `invitations.activity`, `messages.node`, `conversations.conductor` / `requester`, `annotations` |
| `announce` | `wfe discussion announce --node` — message kind `announce` (key `activity:<run>:<node>`), then the node completes with its id; answers the conductor's own invitation; waits for nobody |
| `ask` | `wfe discussion ask <participant>… --node` (or `--run` to await a commissioned result): one invitation per participant in one activity, node waits on `discussion:<id>:activity:<a>`. When all are resolved and no attempt is open, **WFE resumes and completes the node with the outcomes**, then sends one instruction (or, if the run completed, a new conductor run gets them). `settle` completes early; later outcomes arrive as late outcomes of that ask |
| `ponder` | `wfe discussion ponder --node --decision --source … [--next <activity>:<node>]… \| --definition` — decision and sources recorded; `--next` appends a chain of new activity nodes; the revision and node completion are one write (`run adjust --completing`) |
| `talk` | `wfe discussion talk --node` — the run's WFE question to the requester (person, or the requesting run's agent via the existing delegation/request relay), also posted as a question message |
| Routing | A person's untargeted message goes to the **open conductor run**: answer to the question it names, else to its only open question here, else instruction. No second run. Without a conductor the facilitator is invited and becomes conductor |
| Requester | Recorded at creation: the person, or (from a conductor's attempt) the delegating/requesting run with its agent |
| Obligations | Derived: open invitations, awaited results, questions to the person, discussion runs still working and their open delegations — each with owner and later receiver (the awaiting run, a new conductor run, the delegating run, or "recorded only: closed"). `announce --conclude [--close]` snapshots them |
| Progress reads | `wfe discussion state` (conductor graph with activities, open invitations, questions, obligations, annotations); `show` and `GET api/discussions/<id>` carry `conductor`, `requester`, `activities`, `annotations`, `obligations`, message `node`, invitation `activity` |
| Delivered speech / unfinished node | `contribute --completing <node>` speaks and completes in one command. Reconcile completes a node whose message was delivered when the attempt ended without it — messages with `node`, and a participant's reply in a one-node discussion workflow (the p3 rts-vs-bot case) — and lifts a hold caused only by that node (`run.resume` by `discussion`) |
| Annotations | `wfe discussion annotate --kind intervention\|recovery\|evaluation\|correction\|note` — not speech, starts nothing, archived apart |
| Archive | Transcript records `activity`, `annotation`; manifest `conductor`, `requester`, `activities`, horizon `activity`, `annotation` (absent in older manifests = 0); commit trailer extended |
| Archive project as participant | `argue/synthesize` can be invited (an ask to `argue`); its runs may contribute `synthesis` even when requested rather than invited |
| Published integration evidence | `wfe projects run <p> <wf>/<run> --integration`: run.integrate records, published default branch (Gitea, fetched with time) contains/not, local workspace head contains/not and same/behind/ahead |

## Tests (new: `test/activities.test.ts`, 8 tests)

1. Activity metadata validation (`activity-kind`, `activity-type`).
2. ponder → revision r2 adding announce → ask → ponder; announce once
   (retry posts nothing); parallel ask: one answer continues nothing, the
   second completes the node with both outcomes and one instruction and
   one queued job; repeated reconcile changes nothing.
3. Ask resolved during the attempt (unavailable participant fails at once)
   is completed only after the attempt ends; settle with one open
   invitation; the late reply is told once, without repeating the told one.
4. **ex1 orphan**: talk to the person; the person's reply addressed to
   nobody is recorded as the answer on the conductor's question, no
   second conductor run; without `re`, the only open question is answered.
5. Conductor requested as work (Front-like requester run): talk goes
   `--to front-agent`; an agent requester does not join as a person;
   after the conductor completed, a late outcome starts a new conductor
   run which becomes the conductor.
6. **p3 delivered-speech case**: a discuss turn spoke and ended with its
   node running (held interrupted) → reconcile completes it with the
   message and lifts the hold, once; `--completing` does it in one command.
7. Conclusion with `--close` lists the outstanding invitation, its owner
   and "recorded only: the discussion is closed"; annotations dedupe and
   are not messages; archive exports activity/conductor/annotation records,
   manifest horizon and activities; a second export publishes nothing.
8. argue's synthesize run is invitable and may speak a synthesis; a
   participant cannot announce.

Existing discussion tests (11) pass unchanged; two store-downgrade tests in
`conversations.test.ts` now also drop the v7 objects.

Read-only check of the integration read against live records:

```text
Integration of rts-vs-bot improve-game/run-007
  decision 38 integrated 2026-10-20T16:31:54Z by autolab: study/study-rts → main already 303ce45d3d; . → main fast-forward 1042c9a6b9
study/study-rts (integrated commit 303ce45d3d):
  published: developer/study-rts main at 303ce45d3d — CONTAINS it (fetched …)
  workspace rts-vs-bot: detached at d2734dba5c — does not contain it; behind relative to published
. (integrated commit 1042c9a6b9):
  published: developer/pj-rts-vs-bot main at dc777a4752 — CONTAINS it (fetched …)
  workspace rts-vs-bot: main at dc777a4752 — contains it; same relative to published
```

The study submodule's local checkout is still the kind of stale view that
produced ex1's "not on main"; the published observation says otherwise.

## Choices and limits

- No new primitive node type: the reducer's states, waits and fencing
  already carry each activity; what was missing was correlation and
  completion evidence, now in the discussion store. A run's own `wfe run
  complete` of an activity node is still possible (it is its record), but
  a waiting ask cannot be completed without resuming it.
- WFE records the ask's resume/complete as `discussion`, only when no
  attempt is open; during an attempt the command result says what came.
- The delivered-speech repair is bounded to messages of the last two days.
- Catalog templates for the activities were not added: the person's
  catalog holds the person's own declarations; `ponder --next` makes the
  nodes.
- `talk` to an agent requester relies on the existing delegation/request
  relay (Zulip for delegations; 53 s seen in p3).

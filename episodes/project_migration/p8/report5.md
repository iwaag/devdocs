# p8 Step 5 — deployment and migration

Machine details, logs and the recovery procedure are in pj-agdev's ignored
`.local/project-migration-p8/notes.md`.

## Deployed revisions

| Component | Revision | How |
| --- | --- | --- |
| WFE (service and `wfe`) | agdevworld `7b16e0c` (was `ea09c14`) | `wfe-release.sh deploy`; the service and launcher both report it. Exec store migrated to v4 |
| Executor | agautolab `3ba5abc` | restarted with `lanes = ["conversation"]`; it reports "one slot and lanes conversation" |
| Site (Front Desk on WFE conversations) | agdevworld `7b16e0c` | web image rebuilt |
| Relay (legacy Zulip writes answer 410) | agdevworld `7b16e0c` | restarted |
| Front project | pj-front `b20cb6f` | created through `wfe create`, Gitea `developer/pj-front` |
| agfront (retirement introduction) | `befc06d` | — |
| pj-agdev pins | `5d1d14a` | — |

Configuration:
- `workers.json` gives the coordinator's worker a `conversation` lane (capacity 1);
- `conversations.json` names the entrance (`front/converse`) and the person;
- `zulip.json` no longer lists the legacy Front bot as an intake agent.

Before the release, every WFE store and setting was backed up. Recovery
reverses code and data together: the previous release *and* the exec store
backup, because the old code refuses the v4 store.

## Cutover

There was nothing in flight to hand over (Step 1):
- no queued servings;
- no open routine runs, argues or Front Desk conversations;
- the last legacy serving was 2026-09-28.

The order was:
1. The new entrance went live: deployment, Front project, first turn answered (report4).
2. Every route into the old listener was closed at the relay. Desk posts, re-voicing, argue posts, routine chat and starts now answer 410 with where to talk instead.
3. The listener `com.agdev.agfront-zulip` was booted out. Both plists are archived in the agfront checkout. No process remains.
4. The retirement introduction was posted in `#agents` (#19888) and its `intro-` topic resolved, so the boards show Front as retired.

Exactly one consumer owns each path:
- conversation messages: the WFE store, then the executor's conversation lane;
- callbacks of Front's delegations: the delegate relay of WFE;
- Zulip `#front`: nobody, by design.

## Legacy responsibilities

| # | Responsibility | Outcome |
| --- | --- | --- |
| R1/R2 | Direct chat, Front Desk | migrated: WFE conversations, `front/converse`, the same Front Desk scene |
| R3 | Routine runs | retired. No run was open; channels and guides stay readable. A recurring need is a Front request or a project workflow |
| R4 | Argues | retired. None was open; the Arguing Room reads history only |
| R5 | Re-voicing into `#memo` | retired. Posts are shown as written; old memos stay readable |
| R6 | Settings/characters | kept for the scene's portraits; no run reads them |
| R7 | Budget observation | the relay's `/budget` stays; Front reads it with `curl` |
| R8 | Introduction | retirement introduction posted; `intro-` topic resolved |
| R9 | Execution options | WFE execution settings |
| R10–R12 | Trials, evidence rendering, `agrun` | stay in the agfront checkout as history |
| — | Preferences/memories | none existed to migrate |
| — | Cagent intake grant | removed |

**Historical access.** These stay readable:
- the agfront checkout with its 1,700 run records, mirror and serving journal;
- every Zulip conversation, including the ten unresolved `front-*` topics from September about the retired mission system. They were left unresolved rather than closed: nothing consumes them, and a write would only add noise;
- the relay's read routes (desk history, routines, argues, progress);
- the old memos.

Nothing was restarted to be migrated.

## Placement and observation

- Nautobot:
  - `agfront` service retired;
  - placement `agfront-agstudio` disabled;
  - agent `agfront` retired (3 updates).
  
  The desired export from before the change is kept.
- Fresh `nctl observe` of the coordinator host after the change: drift is converged, with `agfront-agstudio` `inactive_by_intent`. The checkout (workspace) stays as history.

## Verified on the deployed system

- `GET /wfe/api/conversations` answers on the agdev port, with the entrance and the person configured.
- A posted message started `front/converse/run-001` in the conversation lane. Its reply came 50 s later and named that run (report4).
- A desk post to the relay answers 410.
- `wfe exec workers` shows the coordinator's worker with lanes `{default: 1, conversation: 1}`.

## Known gaps after this step

- The Front Desk's side panels (progress, contexts) still read the relay. Progress lists only the old `#front` requests; the conversation's own runs appear as run chips instead.
- The operation room's "Ask Front to run it" and the agent-room "Ask Front" buttons now get the relay's 410 message. They are not hidden.

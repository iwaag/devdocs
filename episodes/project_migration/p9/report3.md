# p9 Step 3 — Front onboards lowerpoly

## The request

One message in a new Front conversation `p9-lowerpoly`, #69, posted
2026-10-20 05:08Z by `Omni Agent (for Developer)` (it says that it is Omni's
request, not the Developer's words). Text: pj-agdev's ignored
`.local/project-migration-p9/step3/request.md`. It named:
- the repository;
- preservation of history and local areas;
- trustedagent for the preparation push;
- register/obtain/freework;
- one bounded trial (pipeline explanation, one representative case, outputs
  on the share, recipe, limitations);
- the facts nothing tracked records: where the inputs and the original
  toolchain live, and the suitable worker;
- pending human visual judgment;
- standing authority;
- the LAN links wanted.

Turn: `front/converse/run-032` (completed 05:15Z, three nodes).

## What Front did (from its session record and published state)

| Step | Evidence |
| --- | --- |
| Read help: `wfe help gitea`, `wfe project --help` | session commands |
| Checked `wfe gitea whoami --as trustedagent`, `wfe gitea repo …` | session commands |
| Preparation checkout: `wfe gitea git --as trustedagent -- clone …` into an ignored scratch folder of pj-agdev's `.local/` | session commands |
| `project.yaml` (ag.project.v2, id `lowerpoly`, areas `study`/`wedo`/`lab`, intent, goals, guides) and `agent/guide.md` | commit `a8c03aa`, author `front-agent` |
| Push as trustedagent | `wfe gitea git --as trustedagent -- push origin main` |
| `wfe project register trustedagent/pj-lowerpoly` | registry: project `lowerpoly`, root gitea:144 |
| `wfe project obtain lowerpoly` | workspace `lowerpoly` |
| `wfe workflow basics --dry-run`, then placed. Set `execution.agent: lowerpoly-agent`, profile `sonnet` | `wfe validate freework`: valid |
| Commit and push of the workflow as trustedagent | `1dbd2f8`, author `front-agent`; `git ls-remote` = `1dbd2f8` |
| Discovery | `wfe projects show lowerpoly`: three local areas with descriptions; freework bindings `project=.`, `study`, `wedo`, `lab`, all editable |
| Delegation `d1` to `lowerpoly/freework` | `converse/run-032` delegation d1 → `freework/run-001` |
| Replies #70 (onboarding done, trial dispatched), #71 (progress), #72 (result) | conversation `p9-lowerpoly` |

History preserved: `a31efb8` → `a8c03aa` → `1dbd2f8` → run commits. No
force push, no new repository. The original 21 commits are unchanged.

## Identities

| Write | Gitea principal | Git author |
| --- | --- | --- |
| Front's preparation pushes (`a8c03aa`, `1dbd2f8`) | trustedagent (`wfe gitea … --as`) | front-agent |
| Registration / workspace clone | developer (service; reads) | — |
| freework/run-001 checkpoints and integration (`535f57b` … `efafced`) | developer (WFE service) | the coordinator host's Git identity (`iwaag`) |

The run commits carry the host's Git user name. That is the executor's
configured identity, not a claim that the Developer wrote them. The run
record names `lowerpoly-agent` as the actor.

## Interventions

Omni intervened only by posting the request. Omni answered no questions and
fixed no infrastructure during the turn. Front and lowerpoly decided the
rest:
- the execution agent and profile;
- the trial case;
- the toolchain reuse;
- the review page.

## Observed so far (Step 4 continues)

- Front finished the whole onboarding in about 2 minutes, without a
  question.
- The trial result (`freework/run-001`) was delivered within the same turn.
  Step 4 checks it.
- **A gap appeared at once.** The child served its review page "until you
  answer". Front answered its question right after relaying the link, so
  the page stopped before anyone could look. Reply #72 says so, and offers a
  new page. Front's guide already holds this fact from p8 trial b ("a review
  page stopped before they looked leaves them nothing to open"), and it was
  not applied.
- The preparation checkout remains under pj-agdev's ignored `.local/prep/`.
- The workspace `lowerpoly` is at `1dbd2f8`, behind the integrated `main`.
  That is normal after integration.

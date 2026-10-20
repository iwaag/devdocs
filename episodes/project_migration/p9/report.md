# project_migration p9 — Front onboards lowerpoly: final report

Step reports: [report1](report1.md) (capability boundary),
[report2](report2.md) (named Gitea principal), [report3](report3.md)
(Front onboards), [report4](report4.md) (useful trial and delivery),
[report5](report5.md) (reusable workflow). Machine facts are in pj-agdev's
ignored `.local/project-migration-p9/` and the environment notes.

## Delivered

- **lowerpoly is a managed project**, onboarded by Front from the existing
  `trustedagent/pj-lowerpoly`:
  - history kept: `a31efb8` is an ancestor of `main`;
  - no second repository on Gitea;
  - `study/`, `wedo/`, `lab/` declared as local areas;
  - intent, goals and guide written by Front from the existing material.
- **Front can act as another Gitea principal** through a discoverable
  operation: `wfe gitea whoami|repo|git --as <name>` (agdevworld `075500c`,
  deployed). Its guide names the tool and `wfe project register|obtain`
  (pj-front `7b890f2`).
- **A useful result.** `lowerpoly/freework/run-001`:
  - reproduced the Phase 2 eagle_beak LOD chain (all recorded measurements
    match);
  - delivered on the share with the pipeline explanation, the exact recipe
    and its limits.
- **A reusable workflow.** `lowerpoly/lod-chain`, derived node by node from
  the trial and published. It was reused for eagle_beak (lod-chain/run-001),
  and for wolf_tongue from a request that did not name it (lod-chain/run-002;
  all 35 recorded cells match).

## Completion matrix (checked by Omni against deployed and published state)

| Criterion | Evidence | State |
| --- | --- | --- |
| Existing project preserved | `a31efb8` ancestor of `main` (36 commits); Gitea search finds only `trustedagent/pj-lowerpoly`; no force push (the tool refuses one) | met |
| Front performed onboarding | converse/run-032 session: clone, project.yaml/guide, push, register, obtain, basics, validate; commits `a8c03aa`, `1dbd2f8` authored `front-agent` | met |
| Local knowledge supported | editor Detail view lists `study`, `wedo`, `lab` as "local area" (screenshot, ignored step6/); runs read them; `lab/` gained the run's tools, integrated into root `main` | met; no substantive `study`/`wedo` finding arose, so none was written |
| Authentication works | Front's two preparation pushes ran `wfe gitea git --as trustedagent`; `wfe gitea repo` shows trustedagent and developer with admin/push/pull | met; later principals stated below |
| Managed operation works | registry project `lowerpoly` (root gitea:144); `wfe status` and `wfe validate`: both workflows valid; `wfe projects show` and the editor agree; 4 lowerpoly runs `usable` by `wfe run check` | met |
| Useful freework completed | freework/run-001: input set hash, command, 1:47 wall, 163 output files on the share, comparison; Omni's check: fresh files, L0 byte-identical to Phase 2, L1–L4 freshly built | met |
| Workflow extracted | `lod-chain.yaml` on `main`, provenance per node in its intent; reused twice | met |
| Delivery is usable | review pages `:8300`, `:8301`, `:8302` answer from the LAN address; downloads on WFE's share route | met after two corrections (below) |
| Authority is exercised | no human approval gate was opened; Front and lowerpoly chose the agent, profile, case, toolchain reuse and pages | met |
| Claims remain accurate | every reply separates agent measurements from the Developer's visual verdict; Omni interventions recorded here | met |

## Authentication identities

| Write | Principal | Git author |
| --- | --- | --- |
| Front's preparation (project.yaml, guide, freework) | trustedagent | front-agent |
| Registration, workspace clone | developer (reads) | — |
| Run checkpoints, integration, published run records | developer (WFE service, unchanged) | the coordinator host's Git user (`iwaag`); the run record names `lowerpoly-agent` |
| Front's own guide/knowledge (pj-front) | developer | front-agent / Omni |

## Omni interventions

1. Step 2 (before Front's trial): built and deployed `wfe gitea --as`, named
   the trustedagent credential in the host setting, and added two tool lines
   to Front's guide. This is common capability, not lowerpoly work.
2. Step 3: the request (#69), as Omni for the Developer.
3. Follow-up #73: the first review page had stopped before anyone looked.
   It also gave Omni's byte check.
4. Follow-up #75: the link Front promised had no run to carry it.
5. Step 5: the extraction request (#77) and a plain reuse request (#80, new
   conversation).

Omni relayed no link, answered no question and changed no run record.
Front and lowerpoly made each correction themselves. Front recorded both
lessons in `knowledge/review-pages.md`, and both held in the later turns
(run-035 and run-036 used delegate nodes and kept the pages up). Follow-ups
3 and 4 are gaps in Front's autonomous delivery, not autonomous successes.

## Review entry points

- Front Desk: `http://agstudio.local:8093/?view=frontdesk&conv=p9-lowerpoly`
  and `&conv=p9-lowerpoly-reuse`
- Project: `http://agstudio.local:8093/wfe/#/ws/lowerpoly`
- Review pages (original / L0–L4, dark-lighting toggle, side-by-side):
  - eagle_beak `http://agstudio.local:8300/` and `http://agstudio.local:8301/`
  - wolf_tongue `http://agstudio.local:8302/`
- Outputs:
  - `http://agstudio.local:8093/wfe/api/shares/agshare/runs/lowerpoly/freework/run-001/o1/`
  - `…/lod-chain/run-002/o1/`
- Trial report: `http://agstudio.local:8093/wfe/api/projects/lowerpoly/runs/freework/run-001/raw?path=devdocs/runs/freework/run-001/report.md`

## Remaining limitations

- **Pending human evaluation.** Whether the LODs look acceptable is the
  Developer's call: holes at L1–L4, dark lighting. The open questions are:
  - freework/run-002 q1;
  - lod-chain/run-001's review question (Front answered only "no verdict,
    keep open");
  - conversation message #82 (when to close the wolf_tongue page). This
    keeps front/converse/run-036 and lod-chain/run-002 waiting.

  Three review pages stay up until then.
- **The toolchain lives only in the original working copy** on the
  coordinator host. lowerpoly runs only on `studio`, and moving that copy
  breaks them. Rebuilding it reproducibly is future work.
- **Questions of a run started with `wfe request`.** A linked run's open
  questions show as its status in the conversation, but not in its
  "Waiting for the person" line, which counts only questions posted into
  the conversation.
- **A scratch folder is left.** Front's preparation checkout is in pj-agdev's
  ignored `.local/prep/`.
- **The original repository's early commits contain a few `.pyc` files**
  (tens of KB). Not rewritten.
- **Omni could not read the trustedagent credential.** Its own session
  classifier refused direct reads of the file. The tool reads it by
  reference, and Omni verified it only through `wfe gitea whoami --as`.

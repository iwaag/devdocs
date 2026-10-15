# p10 step 6 — Verification, deployment and report

## Deployment (2026-10-15, agstudio)

- **Before**:
  - nctl/Nautobot: `nctl status` ok. autolab-agstudio, agobserver-agstudio1
    and agdevworld-web are converged and polling. An `agobserver` service
    aggregate shows `service_missing` (pre-existing, unrelated).
  - Installed WFE `e1e6be3`. Only jevlab `experiment/run-001` was open,
    held by iwaag; no attempt was running.
- **Recovery copies**: `.local/agdev/.local/backups/p10-predeploy-20261015-071843/`
  (exec v2, notify v3, intake v1, present v1 via `.backup`, integrity ok;
  registry, zulip, system, present json, node catalog).
- **Editor**: `wfe-release.sh deploy` built and checked agdevworld
  `394def3` (335/335) and switched from `e1e6be3`. The service and `bin/wfe`
  both report `394def3`. The notify store migrated to v4 on the transport's
  first pass. No automatic rollback was added.
- **Executor**: agautolab `f66162a`. The wfrun guide is read at each claim,
  live since step 3; the Python is unchanged and was not restarted.
- **Observer**: pj-agdev `9de0696`, `com.agdev.agobserver` kickstarted.
- **`zulip.json`**: `viewBase http://agstudio.local:8093/wfe/` (was null),
  and `io {enabled, since 2026-10-15T07:21:30Z, secretFiles: 15 host
  credential files}`.
- **agops**: a disposable `delegate-check` workflow
  (delegate → `forge/provide`, `accepts: parent`, then `use`), pj-agops
  `41fa1d3`, pushed with the editor's Gitea module.

## The real round trip (agops `delegate-check/run-001` → forge `provide/run-007`)

Requested by the Omni Agent for the Developer: "a short English tagline
for our RTS-vs-bot game, one line, at most 12 words, as a UTF-8
plain-text file; I need the file itself back".

| Time (JST) | What happened |
| --- | --- |
| 16:22:24 | parent created; IO channel #300 created; "Started" posted with `Detailed IO:` and `Run:` |
| 16:22:26–23:02 | parent attempt 1: dispatches `d1` (request.md written in its own words, the original kept), ends `exited` with `asset` waiting |
| 16:23:04 | request post #16200 in `wf-forge.provide › request-agops.delegate-check.run-001.d1`; intake: `request → provide/run-007`; `delegation.bind` on the parent |
| 16:23:05–23:54 | forge attempt 1 in the freed slot; forge's **mechanical receipt** #16202 with Progress, Detailed IO (#301) and Run links |
| 16:23:57–24:21 | the plan question (q1, plan.md by hash) relayed; parent attempt 2 takes it up and agrees; answer post #16216 → `answer q1 → provide/run-007` |
| 16:24:31–35:46 | forge makes and delivers `tagline.txt` (asset put), presents it (p2), asks q2 confirming the result. q2 was relayed while forge's attempt was still ending: the parent's job waited and started only after that attempt ended |
| 16:35:50–36:37 | parent attempt 3: reads `tagline.txt` (hash matches), accepts within scope |
| 16:36:42–37:17 | forge takes the answer up and records `accepted` from q2 (its own decision); cleanup; completed; its integration ran by forge's rules |
| 16:37:25–38:36 | parent attempt 4: takes up `completed` (manifest `4c02bee0…`), re-checks the artifact hash, disposes **satisfied**, completes `asset`, does `use` → `received.md`; the run completes |

What came back: "One commander versus the machine. Only one side wins."
(9 words, 54 bytes, `8fb05165db17`).

- 7 attempts, all `exited`, never two at once; **$2.58** in all (parent
  $0.96, forge $1.62).
- Observer opened **no incident**: the waits were healthy.
- **58 IO messages** over two channels, with no failed delivery and no gap.

**LAN links**: the parent and child run views, an IO whole-entry page and
the workflow page were opened at `http://agstudio.local:8093/wfe/` (not a
secure context) in Chromium and WebKit. All rendered with no page errors.
The workflow page's approval digest uses the new plain-JS SHA-256, where
the pre-existing `reading 'digest'` error was before.

## Verification matrix

| Scenario | Evidence |
| --- | --- |
| Normal delegation | real round trip above: forge request, question answered, usable artifact received (hash-checked), the parent continued and completed |
| Decisions | plan agreement (forge `_plan`, bound to plan.md's hash), receipt (take-up), the child's acceptance (forge-agent's decision from q2, `accepts: parent`), integration (forge's), and the parent's disposition `satisfied`: separate records with their own actors. Tests cover the `accepts: person` refusals on both sides and `subject-changed` |
| Recovery | tests: lost send acknowledgment → found by marker, one child; bind failure after creation → one child, bound once; repeated answer application → one answer; lost parent continuation → re-queued by reconcile, once |
| Scheduling | step 3 executor trial (one slot, restarts, 0 overlaps); the real round trip (7 attempts, one slot) |
| Controls | tests: parent hold (facts kept, nothing queued, even through reconcile), stop/cancel never touches the child, an explicit child cancel, a child failure woken and explicitly replaced, late facts after a parent cancel not recorded |
| IO | `test/io.test.ts`, plus channels #300/#301 in production: stable across attempts, whole-entry links, gaps, an outage retained, navigation parent → child |
| Existing intake | forge's receipt #16202 is mechanical; the intake and notify suites pass; normal request topics are unchanged |
| Observation | 7 Observer fixtures from real observations; no incident during the round trip |

## Checks

- agdevworld `npm run check`: **335/335** (also in the release build);
  `checks/delegation.ts` 22/22.
- agobserver: 114/114. agautolab `-k wf`: 45 passed, 1 skipped.
- `checks/simplerUi.ts` and `checks/runInteraction.ts` fail one check each,
  exactly as on `e1e6be3` (pre-existing; see report5).

## Commits

- agdevworld `037091d`, `11af38a`, `21a5cce`, `56c6133`, `f8c16e8`,
  `394def3` (deployed).
- agautolab `f66162a`.
- pj-agdev pointer commits and agobserver `9de0696`.
- pj-agops `41fa1d3`.
- devdocs: reports 1–6 and `report.md`; `README_DEV.md` section *Delegation
  and detailed IO*; local notes in `pj-agdev/.local/devenv.md`.

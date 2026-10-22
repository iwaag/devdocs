# Challenge p1 — Step 6: verification, deployment and report

## Automated checks (the delivery contracts)

| Plan item | Check | Result |
| --- | --- | --- |
| Concurrent edits conflict without losing either draft | `test/challenges.test.ts` *concurrent edits* (stale base refused before journaling; remote change after acceptance → `conflict`, draft kept; discard); aghangout browser *a conflict … keeps the draft* | pass |
| Repeated requests and restart recovery do not duplicate | *create, read, edit…* (same request = one commit); *a failed push … after a restart*; CLI derived request identity (`challengeApi.test.ts`); fixture/browser: one record per save | pass |
| Push failure, and a crash after a successful push | *a failed push keeps the content unsynced; a retry publishes it once*; *a crash after a successful push: the next try finds it in the remote history* | pass |
| Index rebuilt from a fresh clone keeps records and history | *the index is rebuilt from a fresh clone*; on the live system: the recovery drill from the agpc mirror | pass |
| An attachment still resolves the original after edits | `attachments.test.ts` *a conversation message carries a challenge…*; `challenges.test.ts` pinned read | pass |
| Chat and discussion runs receive their attached revision | `attachments.test.ts` (turn, invitation's new run, a reply to the run that asked); deployed check below | pass |
| Reads and saves create no unsolicited runs | `attachments.test.ts` *reading and attaching start no run*; browser checks (fixture write counters); deployed check (job counts) | pass |
| Resolved records stay discoverable (fixture) | unit, browser and deployed checks (fixture `ch-0002`) | pass |

WFE: the whole suite (531 tests) ran in each release build (`npm run check`
inside `wfe-release.sh build`, for `0bffa86` and `aadee43`). aghangout:
`npm run check`, and the browser checks in Chromium and WebKit.

## Deployment

Pre-change state: `nctl drift` converged=46, drifting=2 (agobserver,
cagent-snake-e2e — pre-existing); the same after. Live stores were backed up
(SQLite `.backup` of the conversation, exec and challenge stores, the
configuration files and the current release names) before activation.

| Component | Before | After |
| --- | --- | --- |
| WFE (studio, `wfe-release.sh`) | `25c74c4` | `0bffa86` → `aadee43` (store v10 → v11) |
| aghangout (`scripts/deploy.sh`) | `b28f432` | `6e311e4` |
| wfrun guide (live checkout) | `40f32e7` | `227ef1c` |
| agpc worker (Nautobot placement `agautolab-agpc` → render → `setup_wfe_worker.yml`, ok=22 changed=4) | wfe `0d49456`, executor `a01564a` | wfe `aadee43`, executor `227ef1c` |
| Front guide (pj-front) | `27b740f` | `cfca3e5` |
| `challenges.json` | — | Gitea `developer/pj-challenges`, mirror on agpc over ssh |

## Verification through the deployed UI (studio, Chromium)

Driven by a script as the explicit test actor **"Omni Agent (challenge p1
check)"** — every page write carried that author; nothing was attributed to
the Developer.

1. The lobby links 「課題を見る（未解決 1件）」; the page filters by
   gimmickimg and opens `ch-0001`: title, the Developer's words, evidence
   images, history naming "Omni Agent（iwaagのため）" and the source
   message `20261022-230244-zlcew6 の #310`. No job was created.
2. **From another project's view**: forge's room → 課題 → `ch-0001`
   (gimmickimg's) → 添付 → sent. The message carries `ch-0001@8176fd6`; the
   turn `forge/converse/run-008` has `input.attachments` with that commit and
   `attachments/ch-0001@8176fd622497.md` in its folder; forge-agent answered:
   "ch-0001 … found as an attachment to your message, in
   devdocs/runs/converse/run-008/attachments/ch-0001@8176fd622497.md".
3. **A discussion started from the challenge** (category check): the
   meeting room opened with `ch-0001` attached and its title; forge was
   addressed; the invited `forge/discuss/run-013` received the same revision
   and answered with its id and revision; the archive transcript's message
   record carries the attachment.
4. **A fixture challenge** recorded in the page (published, `ch-0002`),
   resolved with a note, listed under 解決済み; its history shows both
   revisions by the test actor. No job was created.

The conversation and the discussion were closed afterwards.

## From another LAN computer

The same script in read-only mode ran on agpc (Ubuntu, Chrome) against
the agdev origin: lobby, list, filter, `ch-0001` with its evidence and
history, the forge conversation showing 「送信時の版」 and opening that
revision, the discussion's attachment, and the record at phone width — 12/12.
On agpc's execution node, `wfe challenge show ch-0001` (forwarded to the
coordinator) reads the same record.

## Backup and recovery

- Every publication is also pushed to a bare mirror on agpc's local disk
  (`challenges.json` `mirrors`, ssh). `wfe challenge status` reports it at
  the head.
- Recovery drill: a fresh bare clone of the mirror, a temporary area naming
  it, `wfe challenge reindex` → both records, `ch-0001`'s history and all
  four evidence files.
- Recovery of code, pending publications, configuration and data, with the
  exact commands, is in the ignored local notes (`.local/challenge-p1/
  notes.md`). Rolling WFE back requires the pre-deployment store backup
  (v11 is refused by older releases).

## Known limits

- Evidence uploads through the agdev origin are capped by nginx's default
  request size; the CLI (local to the coordinator, or forwarded from a node)
  has no such cap.
- Direct conversations have no archive export yet; their attachments are in
  the store and in each run's input.
- `wfe challenge evidence --out` on an execution node writes on the
  coordinator.

# Step 1 report: migration specification against a fresh inventory

Inventory taken 2026-11-02 (JST), read-only. Machine details and raw scans are in the ignored
local notes (`pj-agdev/.local/rename-warden/`).

## Inventory

- Repository: Gitea id 138, default branch `main`, published head `2939e81`. The local workspace
  was 22 commits behind (`44df265`), so the rename publishes from the default branch, not from
  the checkout. Published `project.yaml`, `agent/guide.md` and `devdocs/README.md` carry the name.
- Deployed: WFE release `2951d02` (source `33b357d`, docs only since). One registry project, one
  workspace, one root repository. `warden` / `pj-warden` / `wf-warden*` collide with nothing.
- Runs: 105 copies, 105 durable records; 104 `ready`, 1 `retired` (`recover/run-036`). Execution:
  103 completed, 2 cancelled, no open attempt. 21 `evidence-fixture` start jobs stay queued on
  purpose (profile `hand`, offered by no host). They are kept.
- Writers: studio executor (notify/intake threads), WFE service, Observer and its watchdog,
  slack-front, the daily backup. The GPU worker holds no copies of this project and no attempt
  of it ever ran there, so no worker file changes.
- Delegations: 4 where it is the parent (closed), 15 where it is the child (open, ponderer).
- Presentations: 8 instance configs, 6 store rows, all stopped; 5 review usages.
- Notifications: 114 targets (1 folder, 113 channels), all with numeric Zulip ids; all 1227 outbox
  rows and deliveries sent, nothing pending. Intake: 84 requests, all bound.
- Observer: 7 incidents on these runs, 69 assignments naming the recipient, none open on a live
  incident. Its settings and `intake.agents` name `agops/recover`.
- Incoming references: published default branches of all registered projects were searched
  after a fetch. No workflow definition or context targets agops. Outside run records there are
  history in `pj-cagent/records/` and `pj-ponderer/conversations/`, and one current display
  entry, curator's `boast/projects/data.js`.

## Identity decision

Code reading settled what the name is attached to:

| Identity | Derived from | Decision |
| --- | --- | --- |
| Creation hash | hash of `history[0]` (`run.create`) | unchanged; the entry is never rewritten |
| Run UID | project/workflow/run#creation | keeps the creation project (`agops/...#...`) for existing runs |
| Run branch | `storage.branch`, manifest `branch` | keeps `wfe/agops/...`; nothing recomputes it after this change |
| Definition and source snapshots | frozen, hashed | unchanged |
| Current project of a run | `run.json` `project` | becomes `warden` through a recorded operation |
| Workspace keys in stores | `workspace`, `project` columns | rebound to `warden` |
| Notification keys | current project | rewritten with their event ids, so nothing is queued again |
| Backups | hashed generations | unchanged; a new generation describes the result |

Rewriting `run.json`'s top-level field alone would fail replay, so the transition is a new history
operation, `run.rename-project`, which sets the current project and records `createdAs`. UIDs read
`createdAs`, so delegation, review and Observer identities do not change and Observer does not open
new incidents or spend recovery budget on old runs. An alternative that leaves records on `agops`
was rejected: every notify, delegate and review path would need name translation for as long as
those records exist.

Old names remain readable only for existing records: a registry `formerly` list resolves stored
workspace and project values and committed snapshots. Routing, publication and new requests do not
consult it, so it is not a second entrance. New run numbers stay above existing ones because
allocation reads the existing copies.

## Field manifest

Operational means rewritten; historical means deliberately kept.

| Owner | Operational | Historical |
| --- | --- | --- |
| registry | project id, workspace id/label/path, repository name, description; adds `formerly` | |
| zulip.json | `intake.agents` workflow | |
| copy.json | `project`, `workspace`, repository `url` | `branch`, `source`, `receipt`, lifecycle contents |
| run.json | one appended `run.rename-project` | `history[0]`, definition, storage, earlier entries |
| tree `.git/config` | `origin` url | run branches |
| exec | `workspace` in jobs, attempts, checkpoints, integrations | receipts, step JSON, responses |
| intake | `requests.project`, `requests.workspace` | messages, topics, classification |
| notify | target key/project/name/description; run key/project/workspace/channel; outbox id/run_key/event id; delivery run_key/events; delivery_events; io_attempts | delivered content, `tried`, channel on sent rows, `request` (a UID), `from-agops...` topics of other projects |
| delegate | `parent_ws`, `target_project`, `target_ws` | `uid`, `child_uid` |
| conversations | invitations participant/workspace/run; workflow_refs project/target/run; activities awaits | message text, tokens, events, `before`/`after`, hashed snapshots |
| present | `workspace` columns, config folder path | titles, review UIDs |
| Observer | `incidents.workspace`, settings, recipient of unclosed assignments | UIDs, evidence, events, posted text |
| contexts | `projects/agops` folder name | snapshots, revision text, archives |
| sessions | transcript folder names (they follow the working directory) | transcript contents |
| Zulip | folder, workflow and IO channel names and descriptions, by numeric id | topics, messages |
| Gitea | repository name, same id | `wfe/agops/*` branches, commits |

Kept unchanged: the retention repository `pj-agops-bff8c33a5c2f.git` (its key is recorded in the
retired run's contents) and all existing backup generations.

## Interfaces that need code

- `shared/run.ts`: the operation, `createdAs`, a shared UID helper.
- UID sites (`observe`, `notify`, `delegate`, `reviews`, `conversations`): use the helper.
- `runs.ts` reader: accept `createdAs` and `formerly`, because old copies still hold their own
  `project.yaml` and committed snapshots still say `agops`.
- `registry.ts` and the stored-id lookups: `formerly`.
- `branchCleanup.ts`: use the recorded branch, not one recomputed from the project.
- `runBackup.ts`: resolve a historical workspace id during a live reinstate.
- A new migration command with preview, preconditions and a journal.

## Open at this step

Nothing was changed. The rehearsal (step 3) tests these claims; none is verified yet.

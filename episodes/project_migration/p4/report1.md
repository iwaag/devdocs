# project_migration p4 — step 1: contracts, gaps and migration inventory

Written 2026-10-15. This step read:

- the installed workflow-editor release and its source;
- the legacy archsage checkout, its sage store, services and records;
- the managed projects' workspaces;
- the cluster's registered state (nctl).

Nothing was changed. Host details (paths, process ids, message ids) are in the ignored step notes, `pj-agdev/.local/project-migration-p4/step1.md`.

## State found

| Item | State |
| --- | --- |
| Cluster (nctl) | `status` ok. Drift: 45 converged, 1 drifting (agobserver `service_missing`, known since observer p3). archsage has four Nautobot rows: service `archsage`, placement `archsage-agstudio` (manual, `agag_agent`), workspace and agent `archsage-agstudio1`. All are converged. |
| Workflow editor | Installed release agdevworld `a642e27`, equal to the source `HEAD`; the service and CLI agree. |
| Managed projects | 6: rts-vs-bot, ppt, agsfx, jevlab, agops, forge. **All use `devdocs: directory` and all have a workspace on this host.** Some workspaces are behind their remotes after integrations (agops 5, forge 3). |
| Shared repositories (registry) | study-rts, study-ppt, wedo-ppt, wedo-sfx; each is used by one project. |
| Root-local knowledge | agsfx `study/study-sfx` and `lab`; jevlab `study` and `lab`; forge `catalog`, `requests`, `tools` |
| Legacy archsage | Checkout `fd89eef`, clean, equal to origin. The listener `com.agdev.archsage-zulip` is running and idle. Its last serving was 2026-09-28 (`study-aisvgs-round3`), and nothing has been pending since. |
| Sage store | `autodev/archsage-sages` `4612ea6`, clean. It has five sages; each knowledge tree equals its source's current `main`/`publish`. |

## Migration inventory and dispositions

| What | Found | Disposition |
| --- | --- | --- |
| Listener `com.agdev.archsage-zulip` | Running. No caller since 2026-09-28. Every restart replays three old mentions, which are ignored. | Retire in step 5: bootout, archive the plist, post a retirement introduction in `#agents`. |
| Scheduled synchronization | None. There is no timer, plist or cron entry; `service/sync_knowledge.sh` is a manual wrapper. | Nothing to retire. Refresh becomes an explicit index operation (below). |
| Sage definitions (5) | aisvgs, arxiv, growbox, protoprey, worldtrend: a `guide.md` domain description and a `sage.toml` naming project, source and repository | Import the domain descriptions into the archsage index with provenance (`archsage-sages@4612ea6:<name>/guide.md`). The store repository stays as history. |
| Study queues (`tostudy/`) | 2 notes: growbox project start and worldtrend round-1 writing rules. Later reports appear to answer both. | Import into `questions/` with provenance, marked "probably answered; verify against the reports" rather than resolved by guess. |
| Open study work | The aisvgs round-3 plan is parked by the Developer's decision. That plan also flags a personal e-mail address in `autodev/aisvgs` `reports/strand7-local-kit.md`, which nobody removed. | The aisvgs study is autolab's project, not archsage's. Record both facts in the index as open items, and repeat them in the final report for the Developer. archsage does not edit that repository. |
| Studies archsage knew | aisvgs, growbox, worldtrend, protoprey-research, papers/study-arxiv-trend. All are **autolab projects** (`autodev/*` on Gitea; arxiv's `publish` is on GitHub). None is a managed project. | Register the four Gitea study repositories and `autodev/papers` in the WFE registry as shared `study` repositories. The common surface then reads them like any shared repository, with no archsage configuration. GitHub `study-arxiv-trend` stays outside Gitea and is the live **inaccessible** source case. |
| Unresolved finished topics in `#archsage-agstudio1` | 6 topics, all finished but never marked ✔ | Mark them ✔ at retirement (step 5). The conversations are kept. |
| Front's guides (`argue`, `shared/requests.md`, the presentation speaker header) | They invite `@**archsage**`, use the `sage:<name>` selector, and ask archsage to establish studies | Front's episode. Recorded as a remaining Front dependency. The retirement introduction says where requests go now. No compatibility shim. |
| Routine guides `routine-study-{aisvgs,growbox,worldtrend}` (v3, in Zulip) | Each ends by asking archsage to refresh its sage | Step 5 updates their last step (with `agroutine update`) so a run no longer mentions a retired account. The refresh is replaced by the archsage index's own upstream reads. |
| pyagag (`agag.argue` selector, `routine-study-`/`sagesync` progress rules, fixtures) | Shared-library history readers and tests | Unchanged: they read history and do no harm without a listener |
| agdevworld (agentroom progress reader, speaker labels), the settings manifest's archsage character | History and presentation | Unchanged. Recorded for the Front/agdevworld cleanup. |
| agentroom plist `AGENTROOM_AGENT_ROOTS` | Still names retired **arxivsage**, not archsage | Unrelated to this migration; noted only |
| Nautobot | 4 archsage rows | Step 5: a prepared batch (service and agent `retired`, placement `disabled`, workspace kept or retired with them), previewed and then applied. In p3 the Omni Agent's apply was refused, so the Developer may need to run it. |

## What WFE already offers, and what is missing

Capability against the plan's five needs. "Local" means the operation needs the project's workspace on this host.

| Need | Existing operation | Gap |
| --- | --- | --- |
| Discovery | `wfe dashboard` / `GET /api/agdev`: every project, purpose, root, workspaces, ongoing runs, shared repositories and their users | Provenance is a free string. A definition is read from a **working tree** whenever a workspace exists, with uncommitted edits and no commit. Projects without a workspace are read from Gitea at the branch tip only. |
| Structure | `wfe status`, `GET /api/workspaces/<ws>/project`: repositories, areas, gitlinks, workflows, validation | Local only. Working tree only. Nothing at a revision, and no upstream tip of a submodule (only `behind` against last-fetched refs). |
| Content | `GET …/file?path=&rev=` for **run files only**; `server/devdocs.ts` for devdocs at a commit, non-recursive | No list, search or read of arbitrary project files at a revision. Git output is read as UTF-8 text, so binary content is not safe. No bounds outside run files. |
| Execution | `wfe run list/show/state`, `GET …/runs[/<wf>/<run>]`, `wfe observe`, `wfe runs index`. The live resolver is `runSpace`: a run's copy, else the project tree. | Local only. A published snapshot (the run folder on `main`) is not offered beside the live record. `wfe run …` opens the executor's store even for reads. |
| Provenance | The Gitea cache (`readFiles`): shallow branch tip, `<area>/.local/gitea-cache/` | No commit-pinned fetch, no lock, no state vocabulary. Absent, inaccessible, uninitialized and fetch failure are mixed into `error`/`reason` strings. |

The write path:

| Need | Existing | Gap |
| --- | --- | --- |
| Write a shared repository | A run's copy puts every initialized submodule on the run branch. An accepted result integrates the submodule (merge or fast-forward, plain push), then the root's gitlink. | None for the shared case. The consumers' gitlinks are not touched, which is what the plan wants. |
| Write root-local knowledge of another project | Delegation (p10) to a target workflow with a **fixed** target per delegate node. A run of the owning project can be requested through its intake. | The target-owned run cannot use archsage's guide: `execution.guides` must be files of the run's own workspace. |
| Detect a target changed since preparation | A moved target is **merged by design**. Only a conflict, or a merge that changes the run's own files, stops integration. | No precondition. A new file in the target, or an ownership change in a file the run did not edit, merges cleanly. |
| Keep general workflows from writing knowledge | Bindings are declarations. `freework` placement (`wfe workflow basics`) grants each path the most permissive access any workflow grants. | archsage's `freework` would become editable over every destination. It must be written with explicit bindings. Access is not enforced at publication; this is a known contract fact (bindings are self-checks), not changed here. |

## Proposal: the common project access surface

**A read-only command group `wfe projects` and the routes `GET /api/projects…`.** Both use one server module (`server/access.ts`). The name avoids `wfe project register|obtain`, which write the registry, and `wfe run access`, which is about bindings. No archsage name, index or configuration appears in it.

| CLI | HTTP | Result |
| --- | --- | --- |
| `wfe projects list` | `GET /api/projects` | every managed project: id, name, intent, root repository (Gitea id, owner/name), devdocs mode, workspaces here with availability, observation time; plus the registry's shared repositories |
| `wfe projects show <id> [--rev R]` | `GET /api/projects/<id>` | project.yaml at the revision, workflows (id, name, intent, bindings, intake, validity), declared local areas, submodules (path, category, registered repository, **pinned** commit at the revision, **upstream** tip of its default branch), devdocs mode and owner |
| `wfe projects tree <id> [<path>] [--rev R] [--upstream] [--depth N]` | `GET …/tree?path=` | entries with type, size and blob id, bounded |
| `wfe projects read <id> <path> [--rev R] [--upstream] [--max-bytes N]` | `GET …/file?path=` | text with sha256 and size. Binary content gets metadata only, unless `--out <file>` (CLI). |
| `wfe projects search <id> <pattern> [<path>] [--rev R] [--upstream]` | `GET …/search?q=` | `git grep` matches as path/line/text, with a result count and a truncation flag |
| `wfe projects runs <id> [<workflow>]` | `GET …/runs` | runs, each with where its record was read: **live** (this host's resolver: copy or project tree) or **published** (the run folder in the committed source) |
| `wfe projects run <id> <wf>/<run> [--published]` | `GET …/runs/<wf>/<run>` | the record (state, nodes, questions, decisions, integrations), its reports and files, and the published snapshot's commit when there is one |

Shared repositories are addressed through a project that uses them (`study/study-rts`). A registered repository that no project uses (the autolab studies) is addressed as `repo:<owner>/<name>` in place of a project id.

**Every result carries the same provenance block.**

```json
{"project": "rts-vs-bot",
 "repository": {"key": "gitea:130", "fullName": "developer/study-rts", "path": "study/study-rts", "category": "study"},
 "revision": {"selector": "pinned", "commit": "<sha>", "pinnedBy": {"repository": "gitea:131", "commit": "<root sha>"}},
 "path": "notes/economy.md", "source": "committed",
 "observed": "2026-10-15T…Z", "fetch": {"state": "fresh", "at": "…"}}
```

- **Revision selectors.**
  - The default is the root's default branch on Gitea, which is committed and authoritative even when a workspace is behind.
  - `--rev <commit|branch>`.
  - Inside a submodule path the default is the gitlink the root revision **pins**. `--upstream` reads that repository's default-branch tip instead.
  - `--require <commit>` checks that the source contains a commit, and fails otherwise.
- **Working tree.** `--source worktree` reads the workspace's working tree. It is explicit only and labelled `"source": "worktree"`. It lists only files Git does not ignore.
- **States.**

  | State | Meaning |
  | --- | --- |
  | `absent` | the path or commit does not exist at that revision, or the repository is gone from Gitea |
  | `inaccessible` | outside the configured Gitea (for example GitHub), refused by the operating boundary or a retired repository, or authentication refused |
  | `uninitialized` | worktree source only: the submodule is not checked out |
  | `fetch-failed` | Gitea could not be read; the cached commit is reported with `"fetch": {"state": "stale", "cachedAt": …}`, never silently |
- **Caches.** Full bare caches per repository at `<area>/.local/access-cache/gitea-<id>.git`, fetched with a lock. Credentials come from the existing `gitEnv()`. Nothing is fetched into or written in any workspace. The cache location and credentials appear in no output.
- **Bounds.** Default limits, each overridable up to a hard ceiling: 256 KiB per read, 2000 tree entries, 200 search matches. Truncation is always said.
- **Runs.** `live` uses the existing resolver (`runSpace`) and needs this host's workspace. `published` reads `devdocs/runs/<wf>/<run>/` from the committed source and works without a workspace. A live read never falls back to a snapshot.
- **Through agdevworld (LAN).** These are GET routes over managed project content only, so they are offered through `/wfe/` like the other reads. Nothing takes a host path. `.local/` is never content.

## Write-back routes

**Maintenance policy (smallest contract).** The root folder of each knowledge target holds one `knowledge.yaml`. The target is a shared repository's root, or a root-local area's folder.

```yaml
schema: ag.knowledge.v1
kind: wedo                 # study | wedo
maintenance: supplemented  # project | supplemented
maintainer: rts-vs-bot     # the project responsible for it
supplementer: archsage     # only with supplemented
```

A missing file, or any other value, means **unknown**, and unknown is not eligible. The policy travels with the knowledge, so the integration guard below also covers a policy change.

| Destination | Concrete workflow (owner) | Route |
| --- | --- | --- |
| Shared study/wedo repository | archsage's own `study` / `backfill-wedo`. The repository is a submodule of `pj-archsage` (`study/<name>`, `wedo/<name>`), bound editable in exactly that workflow. | The run's copy holds a run branch in the shared repository. Acceptance and the existing integration publish to the repository's `main`, and set **pj-archsage's** gitlink. Consumers' gitlinks are left as they are. archsage reports both facts, and project access shows the consumer's pinned commit against the new upstream tip. |
| Root-local knowledge of project X | A workflow **owned by X** (for the trial: rts-vs-bot `backfill-wedo`), with `execution: {agent: archsage-agent, guides: ["archsage:agent/…"]}`. It binds only X's knowledge area editable, with the guard. | The task reaches it through X's ordinary request intake (its `wf-<x>.backfill-wedo` channel, the editor, or `wfe request`). The result is confirmed in that run's own conversation and integrated into X's `main` by X's ordinary integration. archsage sees it through project access afterwards. Only one run owns the deliverable; there is no parent run that also claims it. |

**Two WFE additions follow from this** (step 2):

1. **Guides of another managed project.** An `execution.guides` entry `<project>:<path>` is read at claim from that project's committed default branch, through project access. It is recorded with path, sha256 and commit. A source that cannot be read refuses the job, as a missing local guide does today. Validation accepts the form.
2. **An integration guard.** A binding may declare `integrate: if-unchanged`. Before integrating that binding's repository, the target's changes since the run's merge base with it are compared under the binding's path (the whole repository for a submodule).
   - If there are any, the integration stops as `target-changed`, lists the files, and pushes nothing.
   - A run that brought the current target into its branch passes.
   - Reassessment is new work: a successor run (`--predecessor`) bases on the current target, rechecks ownership and content, and presents a revised result or none.
   - The field is part of the definition digest only when declared, like `execution` in p6.

No dynamic delegate target, no cross-project job lifecycle, and no change to acceptance are needed.

## archsage project (to build in step 3)

- `developer/pj-archsage`, project id `archsage`, `devdocs: directory`, agent `archsage-agent`, profile `sonnet`. Old archsage used Fable 5.1, and its quality needs are not shown to require it. The frontier profile is not offered by this executor.
- Local areas:
  - `index/` holds the knowledge index: domains, source references, applicability, observed revisions, refresh outcomes.
  - `questions/` holds proposed research questions and imported queue notes.
  - `agent/` holds the guides.
- Submodules (destinations): `study/study-rts` for the shared write-back trial, and `wedo/wedo-ppt` for the project-maintained (no-edit) case.
- Workflows:
  - `consult`: talk, then read and write `advice.md`, then confirm. Policy `manual`. Every source is readonly, and only the run folder is written.
  - `study`: plan agreement on scope and destination, research into the bound shared study, confirm. `index` is editable.
  - `backfill-wedo` for shared wedo, with the guard.
  - `freework` with explicit bindings: the root editable, every knowledge submodule **readonly**.
- No documentation step after `confirms: result`.

## Trial cases (step 4)

| # | Case | Target / route |
| --- | --- | --- |
| 1 | Cross-project advice citing study and wedo, with revisions and an honest unknown | `archsage/consult`. Asks how bots and economies were approached across rts-vs-bot (`study-rts`) and agsfx/ppt wedo, and about a GitHub-only source (`study-arxiv-trend`) that cannot be read. |
| 2 | Requested study → acceptance → integration; a later consultation sees the integrated revision | `archsage/study` into `study-rts` (shared), then `archsage/consult` reading `--upstream` |
| 3 | Wedo supplementation from execution records for an eligible project | rts-vs-bot `wedo/`. A new root-local area with `knowledge.yaml` `supplemented`, because rts-vs-bot keeps no wedo. Route `rts-vs-bot/backfill-wedo`, from its accepted `improve-game` and `ex3` runs. |
| 4 | Repetition produces no duplicate; a project-maintained target produces no edit | A second `rts-vs-bot/backfill-wedo` over the same evidence. Then `archsage/backfill-wedo` on `wedo-ppt` (`maintenance: project`). |
| 5 | A target update or ownership change between preparation and integration | A third rts-vs-bot backfill. After it presents, an operator commit to `wedo/` moves the target. The guard stops integration, and a successor reassesses. |
| 6 | Write-back for shared and root-local knowledge | Covered by 2 (shared) and 3 (root-local) |

Access trials, run without archsage:

- every live project (directory mode, local areas, shared submodules);
- a disposable registered project `accesscheck` with `devdocs: submodule` and **no workspace** here, read from Gitea only;
- GitHub `study-arxiv-trend` as an inaccessible source;
- a fetch failure, simulated in tests.

**Budget.** About 8 runs on `sonnet`, capped at $30 of agent cost. No GPU. The Developer gives every plan agreement and result decision in the browser. Step 4 stops to ask for them, with links prepared.

## Next

Step 2 implements `server/access.ts`, `wfe projects`, the `/api/projects` routes, cross-project guides and the integration guard in agdevworld, with focused tests, then runs `npm run check`.

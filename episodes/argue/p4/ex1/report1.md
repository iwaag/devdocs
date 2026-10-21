# argue p4 ex1 — Step 1: inventory and migration map

Observed 2026-10-21 04:20–04:40 UTC on agstudio. Raw listings are in
`pj-agdev/.local/argue-p4-ex1/step1/` (ignored).

## Is the braindump's concern valid?

Yes. Knowledge about *how* discussions among a person and differently
situated agents work is split today:

- ponderer `knowledge/` holds three facilitation findings
  (asking-the-requester, owed-work, artifact-claims).
- argue `episodes/` holds the comparison index and the analyses of
  argue-p3, argue-p3-ex1 and argue-p4. These evaluate the same practice.
- argue's analysis of p4 found the omitted review. ponderer's `owed-work.md`
  still says "to be checked against the p4 trial". The evaluator and the
  owner of the practice are different projects, and the loop is not closed.
- Nothing is sourced from outside literature. No `study-*` or `wedo-*`
  repository covers discussions.

## Published revisions inspected

| Repository | Published main | Notes |
| --- | --- | --- |
| developer/pj-argue (Gitea 148) | `4fd89fd` (140 commits) | workspace equal to origin |
| developer/pj-ponderer (Gitea 149) | `eb87712` (21 commits) | workspace was 11 behind; fast-forwarded (clean) |
| study-argue / wedo-argue | absent | not visible under developer, trustedagent or autodev to either principal; not in `registry.json`; no local clone |
| WFE (agdevworld) | `890b554` installed | store v7 |
| aghangout | `a573f09` served | |
| agautolab | `f37b0dd` | wfrun guide |

## Artifact map

Each retained artifact has one owner and one destination.

| Artifact (today) | Class | Owner → destination |
| --- | --- | --- |
| argue `discussions/<id>/manifest.json`, `transcript.jsonl` (11 discussions) | original record | ponderer → `pj-ponderer/discussions/<id>/` (same path); WFE stays the only writer |
| argue `discussions/{p16-s3-room,argue-p3-ex1-trial,p4-s5-a}/summary.md` | individual summary | ponderer → same path, beside the archive |
| argue `episodes/index.json`, `episodes/argue-p3*.md`, `argue-p4.md` | evaluation | ponderer → `pj-ponderer/episodes/` |
| argue `tools/argue` (list/show/validate/cite/episodes) | archive-reading tool | ponderer → `pj-ponderer/tools/discussions` |
| argue `devdocs/workflows/synthesize.yaml`, `agent/synthesize.md` | capability | ponderer → `synthesize` workflow and guide (rewritten for ponderer) |
| argue `devdocs/workflows/freework.yaml`, `agent/guide.md` | agent identity | retired (ponderer has its own) |
| argue `devdocs/runs/facilitate/*`, `synthesize/*` (run records) | original record (run identities `argue/…`) | stay in history (reachable in pj-ponderer through the imported commits); not copied under ponderer's `devdocs/runs/` |
| argue run branches `wfe/argue/*/run-*` (17, none merged into main) | original record | pushed to pj-ponderer as tags `history/argue/<branch>` so their commits stay reachable |
| argue live run store `pj-argue/.local/wfe/runs` (18 MB, ignored) | runtime evidence | moved to `.local/agdev/.local/retired/pj-argue/` with the workspace (Step 4) |
| argue `knowledge/room-image-provenance.md` | domain knowledge (asset provenance) | forge (making facts and `asset put` records are forge's; it already applies the rule in provide/run-013) → `pj-forge`, as an adopted conclusion with no implementation request (#145 "no handoff"); ponderer keeps the discussion and its summary |
| argue `knowledge/index.md`, `README.md` | index | retired; the provenance map (below) explains where each item went |
| ponderer `knowledge/*.md` (3 findings) | reusable discussion knowledge | `wedo-argue` (new), with trial sources and revisions; removed from `knowledge/` |
| ponderer `devdocs/runs/conduct/*` | run records | stay |
| External research on discussion practice | external research | `study-argue` (new), written through archsage's `study` workflow |

## Affected components and active references

- **WFE configuration** (`.local/agdev/conversations.json`):
  - facilitator `ponderer/conduct`;
  - participant `argue/synthesize`;
  - `archive.workspace: argue`.
- **WFE code** (`agdevworld/workflow_editor`):
  - `discussionArchive.ts`: the header names pj-argue. Archive rows hold
    `commit_id` and `horizon` but no destination. `origin()` reads the
    archive workspace's origin on every call. An existing clone is
    re-pointed with `remote set-url`, so after a switch the old
    commit/horizon would be shown as current under the new repository URL.
  - Format id `ag.argue.discussion.v1`: a schema name stored in every
    manifest, not a dependency on the argue project.
  - `homeView.ts` link title "in the argue repository".
  - `cli/discussion.ts` help: "pj-argue discussions/…", "the archive's
    (argue)".
  - `docs/discussions.md`: configuration example, archive section,
    hand-off and knowledge paragraphs.
  - `discussions.ts` comments: `argue/facilitate`, "argue's synthesize".
- **Participant routing assumes one workflow per workspace.** A participant
  is a workspace id. `contribute` derives the participant from the run's
  workspace, so any ponderer run speaks as `ponderer`. Delegation to
  another workflow of the same project is allowed: only `delegate-self`,
  the same workflow, is refused. ponderer can therefore get a synthesis
  through an ordinary delegate node to `ponderer/synthesize`. That needs
  no second participant entry and cannot start a conductor.
- **agautolab wfrun guide**: already names ponderer as conductor; no argue
  dependency.
- **Front** (`pj-front`): `agent/converse.md` and `discuss.md` name
  ponderer. Its argue mentions are trial references only.
- **archsage**:
  - `agent/discuss.md` points to "the argue project's knowledge/". Needs an
    update.
  - Its index has no argue source.
  - The argue mentions in `index/domains/worldtrend.md` and `questions/`
    refer to the legacy Zulip "argue" sessions (argue p1). They are history.
- **aghangout**:
  - `characters.ts` presents `argue` as "the facilitator of discussions".
    It is stale, and there is no ponderer character.
  - The fixture server and browser check use `argue/facilitate`.
  - README calls argue the facilitator and pj-argue the archive.
  - `ASSETS.md` cites `argue@e7174609f1c0:knowledge/…` (a historical
    citation).
- **ponderer guides**: `guide.md`, `conduct.md`, `README.md`,
  `project.yaml` and the knowledge files assign preservation and subject
  findings to argue.
- **Zulip notify**: channels `wf-argue.*` (facilitate, freework, synthesize,
  and per-run IO). All 6475 deliveries are `sent`, none pending.
- **Schedules**: none reference argue. There are no context blocks for
  argue (`.local/contexts`); only snapshots of earlier runs mention it.
- **Other projects' "argue" matches** (jevlab, agsfx, playprey studies) are
  the English verb, not references.
- **devdocs README_DEV**: *Discussions and argue*, *ponderer conducts
  discussions*.

## Outstanding work at cutover time

| Item | State | Receiver through cutover |
| --- | --- | --- |
| Discussions `p16-s3-room`, `p16-s4-provenance`, `20261020-233458-r93d5v` | open, 0 open invitations, facilitator member `argue` (pre-p4) | Untargeted person messages route by the *configured* facilitator (`cfg.facilitator`), so ponderer conducts any continuation. Step 4 verifies this live. Addressing `argue` will be refused after retirement, as for any non-configured participant. |
| Other 8 discussions | closed, archives `current` | none needed |
| argue runs (facilitate 001–012, synthesize 001–005) | completed or cancelled. synthesize 002–005 show a `ready: _final-report` node in a completed run (documentation nodes that were never needed). | none; history only |
| ponderer conduct 001–006 | completed | none |
| Invitations, awaits, delegations, questions involving argue | none open (`wfe discussion list`: 0 open in every discussion; executor idle, queue 0) | — |
| Archive writes | all 11 `current` (published horizon = live horizon) | final catch-up in argue before the switch, then re-verified in the new destination |
| Notify deliveries | none pending | — |

No discussion is closed for migration.

## Citation and history strategy

- **Import with history, without rewriting.** Merge pj-argue `main`
  (`4fd89fd`, 140 commits) into pj-ponderer with
  `--allow-unrelated-histories`. Choose the resulting tree by hand: argue's
  discussions, summaries and episodes are kept at their paths; its agent,
  workflows, knowledge and run records leave the tree. Every argue commit
  becomes reachable in pj-ponderer under its original id. A citation
  `argue@<commit>:<path>` therefore resolves in pj-ponderer at the same
  commit and path, for example `git show 04a807d:discussions/argue-p4-trial/manifest.json`.
  The 17 run branches go to pj-ponderer as tags `history/argue/…`.
- **No rewriting of speech or identity.** Transcripts, manifests and
  summaries keep their bytes. Run names `argue/facilitate/run-NNN`,
  participants `argue` and authors `argue-agent` stay as they were.
  ponderer does not claim argue's earlier work.
- **Provenance map** `pj-ponderer/HISTORY.md` records:
  - where each argue path went;
  - that `argue@<commit>` names a commit now also in pj-ponderer;
  - the new citation form, `ponderer@<commit>:discussions/<id>/…`.

  The map is documentation, not a runtime alias.
- Current guides and documents cite the new places. Historical episode
  reports and run records keep their literal citations.

## Archive cutover design (decided here, implemented in Step 3)

- Each archive row records its **destination**: the origin URL, without
  credentials, of the repository it was verified in. This is store v8,
  `archives.destination`.
- If the row's destination differs from the configured archive
  repository, the published status is `unverified`, never `current`. The
  discussion is due, and the next export reads the new destination's
  manifest. It appends only beyond *that* horizon and records the commit
  it finds there. Rows written before v8 (no destination) are `unverified`
  until the first export after deployment.
- The writer's clone is disposable. When its origin differs from the
  destination, it is deleted and cloned again, rather than re-pointed.
- There is still one writer, the same lock and the same configurable
  `archive.workspace`. No second publisher is added.

## Recovery plan (code and data together)

Before Step 4 deploys anything, the following are copied to
`.local/agdev/.local/backups/argue-p4-ex1-predeploy-<ts>/`:

- the conversation store (`.local/conversations/conversations.sqlite*`,
  v7);
- `conversations.json` and `registry.json`;
- the archive writer's clone;
- bare mirrors of pj-argue and pj-ponderer (all refs).

Rollback means all of the following:

- activate WFE `890b554` *and* restore the v7 store copy. An older release
  refuses v8;
- restore `conversations.json` and `registry.json`;
- reset pj-ponderer's `main` to `eb87712` from its mirror. The import is
  additive; argue's Gitea repository stays untouched until the very end;
- re-activate aghangout `a573f09`.

Gitea repositories created in Step 2 can stay; they are not used by the
old release. Retiring argue's Gitea repository is the last operation and
only archives it (read-only); it deletes nothing.

# argue p4 ex3 — Step 1: contracts, recording matrix, renderer, fixtures

I inspected the published revisions:

- WFE: agdevworld `5d6f511`, conversation store v9;
- aghangout: `c66bd02`;
- ponderer's guides: pj-ponderer `612a7b6`;
- the agautolab wfrun guide.

Before any service work I refreshed the Nautobot observation
(`nctl observe agstudio`). `nctl drift --host agstudio` reported
converged. The raw output is in the ignored `pj-agdev/.local/argue-p4-ex3/step1/`.

## What exists, and what it proves

| Fact | Where | What it proves |
| --- | --- | --- |
| A workflow definition is `<ws>/devdocs/workflows/<id>.yaml` in the project's working tree | `server/workspace.ts` `createWorkflow`, `saveWorkflow` | the bytes **now**. Create and save never commit. A definition an agent saved is a draft until a run publication or integration commits it |
| `saveWorkflow` returns `rev` = the first 16 hex characters of sha256(text), and refuses a stale `--base` | `workspace.ts`, `files.ts` | that the agent saved over the version it read. Nothing records the bytes it replaced |
| A run freezes its definition: `definition/<id>.yaml` (r1) and `definition/rN/<id>.yaml`, each with a byte `sha256` and a semantic `definitionDigest`, checked on every read | `shared/run.ts` `DefinitionRef`, `Revision`; `server/runs.ts` `loadBundle`, `loadRevisions` | the exact graph of every run revision, immutable and verifiable while the run's records exist |
| A run's history records every op with `seq`, `at`, `attempt`; `run.adjust` entries carry `change.revision: [before, after]` | `shared/run.ts` `HistoryEntry`, `replay()` | node states at any point (by replay), and which attempt made which revision |
| `semanticDigest('definition', w)` excludes id, name, approvals, layout and appearance | `shared/canonical.ts` | a layout- or style-only change can be told from a behavioural change |
| A run started by an invitation has `input.source = {kind: conversation, conversation, invitation}`; a conductor run is `conversations.conductor` | `discussions.ts` dispatch, `conductedBy` | the discussion and turn of a participant's or the conductor's run |
| A delegated or requested child has `entrustedBy {project, workflow, run}` and **no** `source` | `exec.request`, `delegate.ts` | the discussion context of a child is reachable only through its parent |
| A ponder activity records `decision.revision` (e.g. `r2`) beside its run, node and attempt | `activities` table | a proven activity → run-revision link for the conductor's own graph |
| `links` relate a message to a **run** (`turn`, `handoff`, …), never to a workflow version | `conversations.ts` | an executed child named by a `handoff` link is proven; its version is the child's r1 |
| The CLI knows its attempt from `WFE_ATTEMPT` (`attemptRow` → run) and calls the stores in process | `cli/discussion.ts` `own()`, `cli/run.ts` | the actor run, attempt and agent of every CLI operation |
| HTTP reads are unauthenticated on the LAN; writes need a same-origin page or no Origin | `server/api.ts` | an embed on `/hangout/` may read `/wfe/api/…` directly |

### Gaps, stated explicitly

- **Reads are not recorded.** No record exists of any `cat`, `git
  show` or `wfe workflow show`. No workflow reference before this
  episode is provable.
- **Saved drafts were not captured.** Before this episode, the bytes a
  save replaced are gone, and the saved bytes survive only while nobody
  saves again. A historical draft is never recoverable from WFE.
- **Children lose the discussion.** A requested or delegated child has
  no discussion context of its own.
- **No message link.** No operation names the message it belongs to.
  Only the attempt is known.
- **History lives with the run.** Historical node states exist only as
  long as the run's records do. Retired workspaces keep their run store
  in `.local/agdev/.local/retired/` (pj-argue). Nothing in WFE reads
  them.

## Recording matrix

`✓` = provable from records that exist today, `new` = recorded from
this episode on, `—` = not provable and not claimed.

| Case | Before this episode | From this episode |
| --- | --- | --- |
| Conductor revises its own graph (`ponder --next`, `ponder --definition`, `run adjust`) | ✓ activity + run revision (backfill) | new: recorded by the adjust itself, role **progression** |
| A participant run revises its own graph (`run adjust`) | history entry only (attempt + revision); not backfilled, because no discussion records it | new: recorded, role subject |
| Agent saves a workflow definition (`wfe workflow save`) from a discussion run | — | new: **edited**, before and after bytes captured as drafts |
| Agent creates a definition (`wfe workflow new`) | — | new: **created**, the new bytes captured (draft) |
| Copy of an existing workflow (new + save) | — | new: created, then edited. A `--from` provenance is recorded only when the agent declares it (`attach --relation created --from …`) |
| Child run requested/delegated from a discussion run (`wfe request --request`, `wfe delegate request`) | ✓ only when a `handoff` link names it | new: **executed**, version = the child's r1, with the time basis |
| A delegated project's own work (the child revises its graph, saves a definition) | — | new: the child's context is followed through `entrustedBy` (up to 4 levels). It is recorded with the child as actor and the parent's turn |
| Agent consults a workflow as discussion material | — | new: **explicit attach** (`wfe discussion workflow attach`), evidence `claim`. WFE reads and captures the bytes it names, which proves retrieval at that turn. It does not prove comprehension |
| Arbitrary shell read of a YAML file | — | — (not claimed) |
| An operation before any speech in the turn | — | new: kept with run, attempt and invitation (turn identity). Shown as a recorded activity, never moved under a message |
| Several revisions in one turn | — | new: one record per operation, ordered by record id, grouped per attempt |
| Editor (browser) edits during a discussion | — | — (no attempt, so no discussion context; stated as a limit) |

## Data contract (store v10)

WFE owns these records, in the conversation store beside the activities.
They are evidence about operations: not a workflow store, and not
execution truth.

### `workflow_refs`: one row per association, immutable once recorded

| Field | Meaning |
| --- | --- |
| `id`, `conv` | the association; its id is its order in the discussion |
| `relation` | `referenced` \| `created` \| `edited` \| `executed` |
| `role` | `subject` \| `progression`. Progression = the target is a run of the facilitator's workflow (`ponderer/conduct`); otherwise subject, unless declared |
| `project`, `workflow` | the workflow's identity (`ws`, workflow id) |
| `target` | `{kind: definition, file}` or `{kind: run, run}` (a definition vs one execution of it) |
| `after` | the version: `{kind: draft\|committed\|revision, sha256, definitionDigest, commit?, path?, revision?, run?, seq?}` |
| `before` | for `edited`: the replaced version, same shape; else null |
| `nodes` | node ids the record singles out (attach `--node`, or the added and changed nodes of an edit) |
| `note` | an optional explanation, given by the actor |
| `evidence` | `operation` (written by the operation's own command) \| `claim` (an authored attach) \| `backfill` (derived from proven records, with provenance in `note`) |
| `operation` | the source operation: `workflow.save`, `workflow.new`, `run.adjust`, `run.request`, `delegate.request`, `attach`, plus the run's history `seq` where there is one |
| `messages` | the explicit source messages (`--re`) |
| `activity`, `invitation` | the conductor activity, and the invitation of the actor's turn |
| `run`, `attempt`, `actor` | the actor run, its attempt and its agent; the originating run of a delegated child in `origin` |
| `state` | `pending` (journaled before the operation; never shown) \| `recorded` \| `unconfirmed` (an interrupted operation whose effect could not be verified) |
| `at`, `token` | time, and the idempotency key (`UNIQUE (conv, token)`) |

### `workflow_snapshots`: the captured bytes, content-addressed

`sha256` (the primary key, the same byte digest a run bundle uses),
`text`, `captured`, `origin`. A version a record names is captured
when it is recorded: a draft has nothing else to prove it, and a
committed file or run revision is captured too so that the archive
carries it.

**Retention**: as long as the conversation store, and exported to the
archive, which never deletes.

### `workflow_ref_events`

An append-only list of later message associations: `{ref, message,
by, run, at, note}`. `wfe discussion workflow link <ref> --message <n>`
adds one. The person can also add one from a client, with
`--note`. A recorded row is never rewritten; a correction is a new
event.

### Idempotency and interrupted operations

- **Token.** `op:<attempt>:<operation>:<target>:<after sha256>`. A CLI
  retried with the same result writes nothing new. An attach's token is
  `attach:<run or author>:<target>:<sha256>:<sorted messages>`.
- **Order.**
  1. Before the operation, a `pending` row is written with the
     `before` capture and the intended after bytes, where they are known
     in advance (a save renders its text first).
  2. The operation runs.
  3. The row becomes `recorded`.
  4. If the operation fails, the row is deleted.
- **Recovery.** The executor's reconcile, and `wfe discussion workflow
  repair`, resolve a `pending` row older than a minute from the
  operation's own durable evidence:
  - adjust: the run's revisions name the attempt and the sha256;
  - request: the receipt finds the child;
  - save or new: the file's bytes equal the intended bytes.

  Each outcome:
  - verified → `recorded`, with the note `recovered from <evidence>`;
  - not verifiable → `unconfirmed`, shown only in 記録.
- **Nothing is dispatched.** No association write posts speech,
  starts or continues a run, reopens a discussion or touches `updated`.

### Reads (HTTP, no model, no work)

| Route | |
| --- | --- |
| `GET api/discussions/<id>/workflows` | every recorded association with name, relation, role, version labels, change counts, turn identity, its messages, a preview layout and availability |
| `GET api/discussions/<id>/workflows/<ref>` | the pinned graph: `after` (and `before`) as parsed workflows with effective graphs, `changes`, `state` (historical node states with their basis, or `unavailable` and why), the source turn |
| `GET api/discussions/<id>/workflows/<ref>/current` | the current target, labelled current: a definition's working-tree bytes, or a run's active revision with current states. `unavailable` when the target is gone. It is never a substitute for the pinned graph |

A version resolves in this order:

1. the snapshot (`workflow_snapshots`);
2. then, for a revision, the run's own records;
3. then, for a commit, `git show <commit>:<path>`.

The byte sha256 is checked each time, and a mismatch is
`unavailable: digest mismatch`. Reading never executes the workflow:
the YAML is parsed by `parseWorkflow`, and the effective graph is
computed by `compile`, which is pure.

### Comparison

Definitions are parsed and compared by node id and edge endpoints:

- **Nodes**: `added`, `removed` and `changed`, with property changes
  (type, name, description, repositories, workflow, project, accepts,
  confirms, activity). `appearance` and the layout are **style**.
- **Edges**: `added` and `removed` by `from → to`.
- **Workflow fields**: intent, documentation, execution, intake,
  repositories.
- **behavioural** is true when `definitionDigest` differs. A style-only
  edit says so.
- A new id is an addition plus a removal. A rename is never inferred.
- **Generated steps** (effective nodes starting with `_`) are compared
  apart and labelled 自動生成.

### Archive

- The transcript gains `workflow` records (the association as
  recorded) and `workflow-link` records (events).
- The horizon gains `workflow` and `workflowEvent`, plus `snapshots`
  (the count). A manifest written before has neither, which counts as 0.
- Every captured version is written once to
  `discussions/<id>/workflows/<sha256>.yaml`. The manifest lists
  `workflows: [{id, relation, role, project, workflow, target, after,
  before, snapshot paths}]`.
- **A fresh reader** resolves a version from the archive alone: the
  snapshot file named by its sha256. This still works after the source
  branch moved or the project retired. The commit and run locators stay
  as provenance.

### Backfill (proven links only)

1. **Ponder activities with `decision.revision`**: progression,
   `edited`, conductor run. The before version is the parent revision,
   the after version the named revision, with the activity's attempt.
   The bytes come from the run's records when readable; otherwise the
   versions are recorded with `unavailable` and no snapshot.
2. **`handoff` links**: `executed`, the message given by the link, the
   child's r1.
3. **Children whose `entrustedBy` is a run of a discussion turn**:
   `executed`, subject, the parent's invitation turn, no message.

Every backfilled row is `evidence: backfill`, with a note naming the
record it came from and `argue p4 ex3`. Old speech is not touched, and
no row is placed by time.

## Renderer boundary

WFE's `Canvas` (`src/canvas.ts`) is a framework-free DOM/SVG class:
read-only mode, selection, focus framing, pan/zoom, the run-state
overlay and generated doc steps. Its import closure (`shared/model`,
`layout`, `compile`, `validate`, `dom`, `icons`) is browser-safe.

**Choice**: WFE builds a second bundle, **`/wfe/embed/graph.js`**, an ES
module that aghangout imports lazily (same origin). It exports
`mountGraph(host, options)`:

- it mounts into a **shadow root**. The editor's own `styles.css`
  (inlined, with `:root` → `:host`) is installed in it, so no style is
  copied into aghangout and no editor rule leaks out;
- it renders `Canvas` with `readonly` and no-op move/connect, plus the
  overlay and new `marks`: a per-node badge (追加 / 変更 / 削除 / 参照)
  with text and a pattern, and marked edges (dashed for added, struck
  for removed). These are added to `Canvas` as an optional context
  field the editor never sets;
- `update`, `reveal`, `fit`, `dispose`, and `onSelect`.

There is no navigation shell, no API client, no editor state and no
per-message instance. The panel holds one instance and switches graphs.

**Card previews** are not canvases. The list route returns a
**preview** (node boxes positioned by `shared/layout.positions`, edges
and marks), which aghangout draws as a small static SVG.

The full editor stays a secondary action: the run view
`#/ws/<ws>/run/<wf>/<run>` or the workflow view `#/ws/<ws>/wf/<file>`,
which show **current** state and are labelled 現在（エディタ）.

**Rejected**:

- copying `canvas.ts` into aghangout (two renderers to keep alike);
- an iframe of the editor's shell (navigation, global styles, no exact
  definition version);
- an `embed.html` iframe. It would also work, but it needs a message
  channel for selection and sizing, where a module needs none.

## Screens (fixtures)

The fixture discussion `fx-workflows` (Step 3) carries every case below.

```text
Desktop, panel open (≥ 1100 px)
┌ header: ← 議論  title …                            [進行][招待][…][ワークフロー] ┐
│ reader (overview, transcript)              │ ワークフロー panel                  │
│ #301 forge  返信                             │ 参考資料の取得手順  参照 · このターン時点  │
│   …speech…                                 │ [このターン時点] [現在]  [変更前|変更後]    │
│   ┌ card ───────────────────────────────┐ │ 出典: #301 forge のターン [発言へ移動]   │
│   │ ▣ 参照  画像の手配 (forge/provide)    │ │ ┌ graph (Canvas, fit/±/Fit) ─────┐ │
│   │ 定義 draft 3f2a… · 参照ノード2件      │ │ │                               │ │
│   │ [mini svg]           開く ▸           │ │ └───────────────────────────────┘ │
│   └─────────────────────────────────────┘ │ 変更: 追加 review「結果レビュー」 …     │
│ ⚙ 発言のない操作 · forge/discuss/run-012   │ ノード: (text list, selectable)         │
│   [card: 実行 forge/provide/run-031]        │ 詳細: selected node … 関連発言 #303     │
│ ▸ 進行の更新 3件 (ponderer)                  │ エディタで開く（現在）↗                 │
│ composer (draft kept)                       │                                     │
└─────────────────────────────────────────────┴─────────────────────────────────────┘

Narrow (390 px): cards full width; the panel is a full-screen sheet with
「閉じる」 (Escape) returning focus to the card; the composer is beneath.
```

| Fixture | Shows |
| --- | --- |
| reference card | 参照, draft version, selected nodes highlighted, explanation |
| creation card | 作成, 下書き (never 公開済み or 実行済み), provenance |
| edit card | 編集 · 変更 `結果レビューを追加`, or counts and names; before and after; a removal list; a style-only edit labelled 配置のみの変更 |
| execution card | 実行, the run and its state at that turn, basis "記録時点 (seq n)"; 現在 labelled current |
| progression group | 進行の更新 N件, collapsed, with the decisions behind them |
| operation with no message | its own row: 発言のない操作, turn identity, no speech |
| several workflows | two cards under one message; the panel switches between them, and the selected card is marked |
| unavailable | "この版は記録されていません" or a digest mismatch; never the current version silently |
| large graph | 40 nodes, focused on the relevant nodes, with fit/pan/zoom |
| long labels | Japanese node names of 80 characters |
| load failure | the text node and change list remain readable |

## Affected components

- **WFE** (agdevworld):
  - `server/discussionSchema.ts`: v10;
  - `server/discussionWorkflows.ts`: new. It holds the records,
    capture, resolution, comparison, preview and repair;
  - `cli/discussion.ts`: `workflow attach|link|list|show|repair`;
  - hooks in `cli/wfe.ts` (workflow new/save, request) and
    `cli/run.ts` (adjust); `discussions.ts` ponder adjust; and
    `cli/delegate.ts`;
  - `server/api.ts`: routes and MIME;
  - `server/discussionArchive.ts`;
  - `src/canvas.ts`: marks;
  - `src/embed/graph.ts` and a `vite.embed.config.ts` build;
  - `docs/discussions.md`, `wfe discussion help`.
- **aghangout**: `api.ts` types, `meeting.ts` cards and panel,
  `workflows.ts` (pure: grouping, placement, labels), `styles.css`, the
  fixture and the browser checks.
- **ponderer**: `agent/conduct.md` gets one fact. **agautolab**'s wfrun
  guide, *A run that speaks in a discussion*, also gets one fact.
- **devdocs**: README_DEV.

## Migration and recovery

- Store v10 is additive (three tables). v9 code refuses a v10 store, so
  a rollback restores the backup.
- Before deployment, the conversation store, the exec store, the
  configuration and the archive clone are backed up into
  `.local/agdev/.local/backups/argue-p4-ex3-predeploy-<stamp>/`.
- The backfill is a CLI command run once after deployment
  (`wfe discussion workflow backfill --note …`). It is idempotent, by
  token `backfill:<source record>`.
- The rollback order is written in the ignored
  `pj-agdev/.local/argue-p4-ex3/notes.md`.

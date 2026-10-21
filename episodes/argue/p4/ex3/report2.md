# argue p4 ex3 — Step 2: associations and immutable reads

Delivered in WFE agdevworld `024aaaf` (store **v10**), with guide facts
in the agautolab wfrun guide and in pj-ponderer `agent/conduct.md`.
`docs/discussions.md` gains *Workflows in a discussion*, the contract
summarised here.

## What was implemented

### Records (`server/discussionWorkflows.ts`, conversation store v10)

Store v10 adds four tables:

- **`workflow_refs`**: one immutable row per association, carrying:
  - the relation (`referenced`/`created`/`edited`/`executed`);
  - the role (`subject`/`progression`);
  - the project, workflow and target (`definition` file or `run`);
  - the `after` and `before` versions: the kind
    (`draft`/`committed`/`revision`), the byte sha256, the semantic
    digest, and the commit or run revision with its history `seq`;
  - the nodes, the note and the evidence (`operation`/`claim`/`backfill`);
  - the source operation, the messages, the activity and the invitation;
  - the run, attempt, actor and origin;
  - the state and the idempotency token.
- **`workflow_snapshots`**: the bytes of every version a record names,
  keyed by sha256.
- **`workflow_ref_events`**: later message associations, append-only.
- **`workflow_meta`**: when recording began. A derived operation older
  than that is `backfill`.

### How each operation is recorded

| Operation | Hook | Token | Recovery |
| --- | --- | --- | --- |
| `wfe workflow save` | `cli/wfe.ts`: journaled before the write, with the bytes it replaces and the bytes it will write (rendered as `saveWorkflow` renders them); recorded after | `op:<attempt>:workflow.save:<ws>/<file>:<sha>` | the file holds the intended bytes → `recorded (recovered from the file)`, else `unconfirmed` |
| `wfe workflow new` | the same | `…:workflow.new:…:new` | `unconfirmed` (the intended bytes are not known in advance) |
| `wfe request --request` | journaled by receipt, finished with the child's r1 | `request:<receipt>` | the job carrying `request:<receipt>` → recorded |
| `wfe run adjust`, `wfe discussion ponder` | derived from the run's own history after success | `adjust:<run>:<rN>` | every reconcile derives the same token |
| `wfe delegate request` | the child does not exist at dispatch; derived from the delegation record once it does | `delegation:<uid>` | reconcile |
| `wfe discussion workflow attach` | the explicit reference; WFE reads and captures the version | `attach:<run or author>:<relation>:<target>:<sha>:<messages>` | n/a |

**The discussion context** of an attempt comes from its run:

1. `input.source` (an invitation), else
2. the discussion it conducts, else
3. what entrusted it (`entrustedBy` run or delegation), followed up to
   four levels. The record keeps the child as actor and the parent run
   as `origin`.

**Failure handling.** A recording failure never fails the operation. It
is said on stderr, and the journal entry is left for repair. Repair runs
in the executor's reconcile every minute (`wfe executor reconcile`) and
on demand with `wfe discussion workflow repair [--all]`.

### Reads

- `GET api/discussions/<id>/workflows`: the cards' data, with name,
  availability, `changes`, a `preview`, `runUrl` and `editorUrl`.
- `GET …/workflows/<n>`: the pinned `after` and `before`, the
  comparison, and the node states replayed to the recorded `seq`.
- `GET …/workflows/<n>/current`: the target now, labelled current,
  with `same`.

A version is read from its snapshot, or from the run's frozen file. Its
sha256 is checked, then it is parsed and compiled. It is never executed.

**Comparison** is by node id and edge endpoints. It reports:

- property changes, with before and after;
- `restyled` (appearance or position only), and `behavioural: false`
  when the semantic digests are equal;
- workflow-level fields;
- generated steps, apart from authored nodes.

### CLI

`wfe discussion workflow attach|link|list|show [--current]|repair
[--all]`, and `contribute --workflow <record>`, all in `wfe discussion
help`.

### Archive

- Transcript records `workflow` and `workflow-link`.
- Horizon fields `workflow` and `workflowEvent`. Absent counts as 0, so
  older manifests stay valid.
- The manifest lists `workflows`, each with its snapshot paths.
- Every version's bytes are written to
  `discussions/<id>/workflows/<sha256>.yaml`.

A new record makes the discussion due for export.

### Backfill

`repair --all` derives the following from proven records only:

- conductor revisions, with their ponder activity;
- delegated children;
- `handoff` links, as executed records with the link's message.

Each row carries `evidence: backfill` and a note naming its source
("backfill (argue p4 ex3) from …"). No speech is touched, and nothing is
placed by time. It runs at deployment (Step 4).

### Rendering boundary (WFE side, used in Step 3)

- **The bundle.** `src/embed/graph.ts` builds as `dist/embed/graph.js`
  (`vite.embed.config.ts`, part of `npm run build` and `check`). It is
  109 kB, 26 kB gzipped.
- **What it does.** `mountGraph(host, options)` mounts the editor's
  `Canvas` read-only in a shadow root, using the editor's own
  stylesheet with `:root` → `:host`.
- **What the canvas gained.** `marks` and `edgeMarks`, which the editor
  never sets:
  - a text badge (＋追加, ～変更, －削除, ◎注目);
  - an outline pattern (dashed, dotted, solid; struck-through titles for
    removals);
  - dashed edges with a titled label.

  None of these relies on colour alone.
- **Serving.** `embed/*` gets `cache-control: no-cache` and is never
  answered with the editor's page.

### Guides

- agautolab wfrun guide, *A run that speaks in a discussion*. One fact:
  readers see a turn's workflows at the version the turn had; the
  commands record saves, revisions, requests and delegations; a
  consulted workflow is a reference only when attached; a name in prose
  is not a reference.
- pj-ponderer `agent/conduct.md`. One fact: the conductor's revisions
  show as progression; subject workflows are shown beside a turn when
  attached.

## Evidence-preservation checks (tests)

`test/discussionWorkflows.test.ts`, 9 tests:

| Case | Shown |
| --- | --- |
| Conductor revision | One `progression` `edited` record with its ponder activity, attempt, r1 → r2, both versions captured. A repair adds nothing. The added nodes and edges are named. The replayed state at that point (ponder completed with the adjust, the new nodes pending) does not move when the run moves on. Current is readable apart |
| Save from a turn | `committed` before, `draft` after. Added `review` "結果レビュー", `make` description changed, edge `make → review`. A definition reports "no execution state". **A later save leaves the record's graph unchanged**; current shows the later file, `same: false`. An unchanged save and an aborted save record nothing. Messages, invitations, jobs, `updated` and state are unchanged (**no dispatch**) |
| Crash recovery | The same operation retried is one journal entry. A save that wrote and died is recovered from the file. One that died before writing is `unconfirmed`, and journal rows are never listed |
| Explicit reference | Records the committed version and its commit, the nodes and the messages. It is idempotent. An unknown node or message is refused. After a working-tree edit the next reference is a draft. `--commit` reads the earlier bytes |
| Comparison | A new id is an addition plus a removal, never a rename. A move or colour change is layout only, `behavioural: false`. Documentation changes add generated steps, reported apart |
| Message link | It is an event. Another run's link is refused. Duplicates are one. The row is never rewritten |
| Missing or altered versions | Tampered bytes give "digest mismatch". Deleted snapshots and no source give "unavailable", never the current file. The list has no preview |
| Cross-project child, before speech | A child requested by a discussion run (another workflow, through `entrustedBy`) belongs to the parent's turn (`origin`, invitation). An interrupted request is recovered from the job's receipt. Its state at the request is all pending |
| Archive | The `workflow` transcript record, the horizon, the manifest's snapshot path, and the snapshot bytes whose sha256 equals the record's. A second export writes nothing |

Results:

- WFE `npm run check`: **497/497** (488 before, plus 9). The full log is
  in the ignored `step2/wfe-check.log`.
- Three migration tests that simulate older stores now drop the v10
  tables too.

## Not covered by tests here

- **The CLI glue.** The hooks in `cli/wfe.ts` and `cli/run.ts`, and the
  `wfe discussion workflow` subcommands, are not covered by these tests.
  The live trial in Step 4 exercises them.
- **`wfe delegate request`.** The child is created by the delegation
  intake. Its derivation is the same code path as the request case, but
  it was not driven end to end in a unit test.
- **Retired sources.** These are covered by snapshot resolution (the
  archive test, and the deleted-source case). A project actually being
  unregistered was not simulated.

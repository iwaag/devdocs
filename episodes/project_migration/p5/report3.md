# project_migration p5 — Step 3: `cagentctl` and its durable records

Written 2026-10-16. `cagentctl` is built and tested against fakes. It lives in `pj-cagent` (migration clone commit `9ee2f95`, not pushed until Step 4 creates the repository). One narrow common-system change was needed, report1's G4: agdevworld `38ebdff`, pushed and not yet deployed. No live effect was run.

## The tool

`tools/cagentctl/` is a uv project, run as `uv run --project tools/cagentctl cagentctl …`. Its documentation is `--help`. Every command prints one JSON document.

| Exit | Meaning |
| --- | --- |
| 0 | done |
| 1 | failed or still running |
| 2 | usage |
| 3 | refused, nothing dispatched |

| Command | Allowed in | What it does |
| --- | --- | --- |
| `read …` | anywhere | One read-only nctl call (`status`, `drift`, `relations`, `actual`, `workspaces`, `ops list/show`), with the pj-clusterintent and nctl revisions it ran at. The allow-list is the old cagent's `readonly_nctl.py`, copied with its provenance, plus `workspaces`. |
| `observe [HOST]…` | `scan`, `change` attempts | `nctl observe` with a pre-recorded operation id, as an `observation` entry with per-host outcomes |
| `snapshot` | `scan`, `change` attempts | `snapshot.json` (`cagent.snapshot.v1`, below) |
| `apply <batch>` | `change` attempts, agreed plan | Preview, then `nctl desired apply --yes` |
| `reconcile <host>` | `change` attempts, agreed plan | Dry plan bounded by the agreed hosts, then `nctl reconcile --yes --allowed-hosts … --operation-id …` |
| `playbook <name> --limit` | `change` attempts, agreed plan and allow-list | `ansible-playbook` in its own session |
| `person prepare / confirm` | `change` attempts | A prepared handoff; the person's own answer as its outcome |
| `ledger list / show / recover` | `scan`, `change` | Entries and states; outcomes from evidence |
| `records new / check / index` | the project | Record ids, relations, the advisory index |

## How a change is bound to the agreement (report1 §3)

- **Attempt.** `--run` plus `WFE_ATTEMPT` must equal the run record's open attempt (`wfe run show`). Mutations happen only in a `change` run.
- **Agreement.** The governing question is the latest one that recorded a document. All of the following are required:
  - it asked about this run's `plan.md`, and the file still has that hash;
  - it is `_plan`'s recorded agreement, or a taken-up `--about-plan` revision;
  - its taken-up answer is a configured approver's words.

  The ledger entry keeps the question, the plan hash and the answer with whose words it was.
- **Scope.** The scope comes from the one `cagent-plan` block in the agreed `plan.md`:
  - batch paths with sha256 and an optional preview `expect`;
  - reconcile hosts with `allowed_hosts`, an optional reconciler `expect` and `max_rounds`;
  - playbooks;
  - `kind: one-time | maintain`.
- **Prerequisites re-read before execution.**
  - A batch is refused if its bytes differ from the agreed ones, if its fresh preview conflicts, or if the preview differs from the agreed `expect`.
  - A reconcile is refused if its fresh dry plan reaches outside `allowed_hosts` or plans reconcilers beyond the agreed `expect`.
  - Each such refusal is `prerequisite-changed`: a question for the Developer, never a widened scope.

## Destroy-class and self-dependency (D4, D9)

- **What is refused.** Destroy-class work is refused whatever the arguments: the action set is read from nctl's own dry plan, so a `destroy_compute_instance` that reconciliation infers is caught, not only an explicit flag.
  - `cagentctl` never passes `--allow-destroy`, and nctl refuses a destroy without it in every round.
  - A host in `self_dependency_hosts` is refused as a reconcile target, an allowed host, a playbook limit, or a batch key.
- **What a refusal leaves.** It writes a `person` entry with the prepared command, its check, and `performed_by: person`. It stays `prepared`, never `ended`.
- **Confirming.** `person confirm` closes the entry only from a taken-up answer whose author is the person. It records their words, and it notes that a statement is not verification, so a fresh read follows.

## The ledger (D6, D8)

- **Where.** `devdocs/runs/<workflow>/<run>/effects/eNNNN.json` (`cagent.effect.v1`). The copy holds it for the run's life, and checkpoints publish it to the run branch.
- **Allocation and writes.** Each id is allocated by exclusive create. Every write goes to a temporary file, is fsynced, renamed, and the directory is fsynced.
- **Order.** Entry written → process started → `dispatch.state: spawned` → outcome. The process's stdout and stderr are kept beside the entry.

| State | Meaning |
| --- | --- |
| `claimed` | allocated, never written: nothing was dispatched |
| `pending` | written, dispatch not confirmed |
| `open` | dispatched, no outcome: **unknown** |
| `prepared` | a person handoff |
| `ended` | `succeeded`, `failed`, `partial`, `not-started` or `refused`, with its source (`direct`, `recovered:ops-show`, `recovered:preview`, `person`) |

**Long operations.** nctl runs in a session of its own with an operation id written to the entry first. When WFE ends the attempt and kills its process group, nctl finishes anyway, and `nctl ops show <id>` holds the result. The foreground wait is bounded (configured; `--wait`). Past it, the command returns `running` with the entry open.

**Recovery (`ledger recover`)** re-issues nothing. For each entry without an outcome:

1. With an operation id, it reads `nctl ops show`, then `result.json`.
   - A finished operation's outcome is recorded.
   - A running or stale operation stays unknown, with a note.
   - No record at all, with a dispatch never confirmed and older than 10 minutes, is `not-started`. Otherwise it stays unknown.
2. For a batch, it runs a fresh dry preview of the same bytes.
   - "Nothing left to apply" means `succeeded`.
   - Work still pending with an unconfirmed dispatch means `not-started`.
   - Otherwise it stays unknown.
3. Anything else stays unknown.

The agent then reads fresh drift; that is design D8's order.

## Snapshot (`cagent.snapshot.v1`)

The snapshot holds:

- the `drift`, `actual --detail` and `workspaces` envelopes;
- per source, its schema, `generated_at`, `ok`, exit and errors;
- the run's latest observation entry with its operation id and per-host outcomes;
- the nctl and pj-clusterintent revisions;
- `complete`: every read succeeded **and** that observation observed every scoped host.

An incomplete snapshot is written and says so.

## Records

- **Ids.** `records/{reports,scans,changes}/<workflow>.<run>.<n>.md`, YAML front matter. The run reference is in the id, and allocation is exclusive, so two runs cannot collide.
- **Reports.** A report carries reporter, source, time, node, service, endpoint, purpose, owner, `verification {outcome: reachable|placed|not-found|not-checked|contradicted, at, by, evidence}` and `relation {supersedes, duplicate_of}`.
- **Checks and index.** `records check` validates ids, relation targets and verification values. `records index` regenerates `records/index.yaml`: advisory, with the run's unknown entries as `open_effects`. The target lock stays deferred.

## Secrets

- **cagentctl reads no credential.** nctl and Ansible read their own.
- **Common-system fix.** WFE's IO redactor now treats a single-value file as one secret (agdevworld `38ebdff`; the documentation is updated). The test uses synthetic values: a bare token file and a vault-password file are redacted, and a multi-line file is not taken as one value.
- **Deployment.** The real credential files are added to `io.secretFiles` at deployment (Step 4/5).

## Tests (`uv run pytest -q`: 26 passed)

| Plan item | Test |
| --- | --- |
| refusal before dispatch | wrong attempt, missing attempt, `_plan` not agreed, a non-approver's answer, plan.md edited after the question, a mutation outside a `change` run: no nctl call at all |
| altered batches | changed bytes refused; a preview that moved since agreement refused |
| generated destroy actions | a dry plan containing `destroy_compute_instance` refused with a prepared handoff; no `--yes` call |
| self-dependency handoff | reconcile and batch refused; `person confirm` from the person's answer records `performed_by` and the quote; an agent's answer is refused |
| partial success | `non_converged` with progress becomes `partial` |
| interrupted ledger writes | an allocated empty file reads `claimed`; a cut temporary file is ignored; the next id is still unique |
| lost responses | apply waited 0.1 s: the entry is open; recovery from a fresh preview gives `succeeded`; one `--yes` call total |
| long operation, repeat recovery | the backend runs in another process group, survives the caller, reads "still running", then `succeeded`; a third recovery changes nothing; one `--yes` call |
| no evidence | an entry whose operation has no record stays unknown |
| scope selection | unagreed host, outside `allowed_hosts`, reconcilers beyond `expect` |
| observation and snapshot | a partial scan with an unavailable host gives a `partial` entry and `complete: false`; no reconcile call |
| duplicate records | distinct ids across and within runs; a dangling `duplicate_of` is reported |
| synthetic secret | nothing in the ledger or output contains a secret placed in the environment and a token file |

The external-wait route (`wfe run wait --external nctl:<id> --deadline` when the foreground bound is exceeded) is a guide fact for Step 4. It needs a real run record, and is exercised in Step 6 only if an operation outlasts the bound.

## Acceptance

- Each allowed operation leaves a ledger entry with its command, its plan binding and its outcome source.
- Every refusal was tested to dispatch nothing.
- An unknown entry stays unknown until evidence decides it, and recovery never repeats an effect.
- Long-operation ownership was shown with a fake operation.

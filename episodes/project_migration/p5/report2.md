# project_migration p5 — Step 2: observation-only collection, deployed

Written 2026-10-16. The scheduled observation job no longer runs `reconcile --yes`. It runs a new command, `nctl observe`, which collects and ingests and does nothing else. The command is tested and deployed, and one live cycle was audited from nctl's own records. Operation ids and host names are in the ignored notes (`pj-agdev/.local/project-migration-p5/step1.md`, *step 2*).

## Why, in one fact

Of 1616 recorded plans carrying the refresh job's forced observation, **14 also planned other actions, and `--yes` executed them**: `service_profile` playbooks and `dnsmasq_config` deployments, the latest on 2026-10-03. The job that the documentation called "observation refresh" repaired hosts unattended.

## What changed

| Repository | Commit | Change |
| --- | --- | --- |
| nctl | `8bb4d3b` | `nctl observe`; `reconcile --operation-id`, `--allowed-hosts`; docs `docs/observe.md`; tests |
| pj-clusterintent | pin to `8bb4d3b`, script and README | `devenv/observation/refresh-node-observations.sh` runs `nctl observe`, with no host list in the script |

### `nctl observe [HOST]… [--operation-id ULID] [--json]`

- **Only observation.** It reuses `observation.run_observation`, the same pipeline as the `observe_node` reconciler: collection playbook, report retrieval, identity and age validation, cache, and the *Ingest Nodeutils Inventory* Job. It runs under the reconcile lock and writes the same kind of operation record. It never computes drift, plans or dispatches.
- **Host resolution.** An explicit host must be exactly one desired-node slug that the bootstrap inventory exports. Anything else is a usage error (exit 2) before an operation exists, so an unresolved name neither widens nor silently shrinks the set. With no host, the scope is every exported node, and skipped nodes are listed with their reasons.
- **SSH trust.** The existing bootstrap key scan runs first, with strict trust unchanged. A host that is unenrolled, unreachable or offers a different key is `unavailable` and is not contacted. The other hosts are still observed.
- **Envelope `nctl.observe.v1`.** One row per host: `observed`, `unavailable`, `collection_failed` or `ingest_failed`. The run's `state` is `observed`, `partial` or `failed`, and `result.json` serves `nctl ops show`.

  | Exit | When |
  | --- | --- |
  | 0 | every host observed |
  | 1 | partial, failed, or desired state unreadable |
  | 2 | usage error |

- **Observing is not converging.** Drift does not enter the command at all.

### Two additions for Step 3 (report1 G2, G3)

- `--operation-id` on `reconcile` and `observe`: a fresh ULID chosen by the caller, so it can be recorded before the operation starts. A malformed or reused id is a usage error.
- `reconcile --allowed-hosts a,b`: before every dispatch in every round, an action whose contacted hosts (or, for ledger-only actions, node targets) fall outside the set is refused, and the round stops with `action_outside_allowed_hosts`. Plan mode lists such actions in `data.outside_allowed_hosts`. This keeps a re-planned round or a multi-placement `service_profile` off a host the caller did not name.

## Tests

| Gate | Result |
| --- | --- |
| nctl ordinary (`uv run pytest -q`) | **1349 passed** |
| compute conformance (`devtests/test_strategy/test_compute_conformance.py`) | 1 passed (the compute contract was not touched) |

The new tests (`tests/test_observe_only.py`, and five in `tests/test_reconcile_executor.py`) cover:

- observation of explicit hosts, with the operation id honoured;
- an input with **actionable drift**: a service placement on the observed node. The planner, drift and the action dispatcher are replaced by functions that fail if called. Only the collection playbook and the report retrieval run.
- an unknown host and an inactive host, refused before any record;
- an unavailable host that is not contacted while others are observed (`partial`); all hosts unavailable (no Ansible at all);
- an ingest skip as `ingest_failed`;
- the default scope with skipped nodes;
- malformed and reused operation ids;
- CLI exit codes;
- refusal outside `--allowed-hosts` for a ledger action and for a multi-host SSH action, dispatch inside it, and plan-mode listing.

The pinned CLI-surface and `ReconcileData` field tests were updated deliberately.

## Live evidence

1. A hand check of one host observed it.
2. The deployed job was started once through launchd. The script is read from the checkout at each start, so no plist change or reload was needed. That one operation covered nctl's default scope (six hosts), and **all six were observed**.
3. The operation's records:

   | Record | Content |
   | --- | --- |
   | Events | `started`, `preflight_completed`, `collection_started`, `reports_retrieved`, `job_started`/`job_poll`/`job_completed`, `observation_completed`, `finished`. No `plan_created`, no `round_started`. |
   | Ansible artifacts | the collection and the report retrieval only |
   | Jobs | the one ingest Job |
   | Ingested times | The controller dumps and Nautobot's actual `facts.collected_at` moved to the run's time for all six hosts. |

4. Drift was unchanged (45 converged, 1 drifting): nothing was repaired.
5. The run's envelope is retained as `history/<UTC start>-observed.json`, and the per-host lines in launchd's log carry the time and the operation id.

No host was unreachable during this cycle, so the unavailable path is shown by tests only. Step 6's T3 includes a host that is asleep or otherwise unavailable.

## Acceptance

- The scheduled path no longer invokes `reconcile --yes`.
- The live cycle's recorded actions are observation and ingest only.
- The README's recovery note says to run `nctl observe` by hand. It says not to put `reconcile --refresh-observation --yes` back, so a recovery cannot silently restore unattended repair.

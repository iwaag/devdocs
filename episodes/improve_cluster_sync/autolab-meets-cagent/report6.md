# Step 6 report — autolab-to-cagent end to end

Status: complete (2026-08-09).

The complete storyline now works: an agautolab1 mission produced and pushed a
new resident browser game, the mediator called cagent, cagent registered and
deployed it through desired state, and later cagent conversations stopped and
restarted it. Final drift reports the new service converged with no diffs.

## Mission and project evidence

Gateway run 4 created the job and project; after a recoverable mediator
allowlist repair, run 5 resumed from disk and finished exit 0 with
`STATUS: complete`.

- Public repo: `http://agstudio.local:3000/autodev/cagent-snake-e2e.git`
- Commit: `516be81a781393f41a86e3a5b4e20bd5ee4a2579`
- Job: `.local/jobs/cagent-snake-e2e`, converged in iteration 1
- Gates: `node --test` and exact `<title>Cagent Snake E2E</title>` check,
  both passed
- Durable job evidence on agautolab1:
  `.local/jobs/cagent-snake-e2e/evidence/iter-0001/`

The mediator's `agent/session.sh` allowlist had not included the
already-chartered `autolab-cagent` command. It was added without enabling
`--dangerously-skip-permissions`, published as agautolab commit `1e4aba7`,
and redeployed. agautolab's 61 tests pass.

## Registration and convergence

Mediator request `req_a67e05bb3e5f46c996ccf9d91c97099a` composed and
re-applied the service registration and ran reconcile
`01KZJ40NXZCX2S35EQ9WB99QY3`. Deployment and its local HTTP probe succeeded,
but the desired service endpoint had no address, so nodeutils correctly had
no endpoint URL and reported `service_missing`.

From agautolab1, follow-up request
`req_65fbaa8e294a44418c11d9ac6c71fbbe` made cagent preview and atomically add
the node's mDNS name while preserving the placement. Reconcile
`01KZJ46G8Z5DJ5ZKYKNT58EQRN` then converged; final service drift has no diffs.
Curl returned HTTP 200 with `<title>Cagent Snake E2E</title>`.

The first request of the resumed chain,
`req_1164bd63b73444b38224ea0956cfa44d`, was interrupted by the required
OpenCode restart after it entered a headless `external_directory: ask` while
diagnosing operation artifacts. cagent now permits those read-only external
artifact reads, while all destroy-class bash denies remain. Its 92 tests pass.

## cagent-only ON/OFF cycle

- Stop request `req_901dcf59c0a54cc0a56899d93e428c44` previewed one update,
  changed only `run_state`, and converged as operation
  `01KZJ4B74020BYGZDY6X84SD9W`. An independent curl failed to connect to
  port 8124, and stopped-state drift was converged.
- Restart request `req_930c973732dd48c79cb17938d938c045` previewed one update,
  preserved repo/port/endpoint/profile, and converged as operation
  `01KZJ4FDS30G4J329Y541EY7VC`. Curl again returned the expected title;
  final cluster drift reports `cagent-snake-e2e` converged with no diffs.

Private captures are retained at
`pj-clusterintent/.local/autolab-meets-cagent-step6-stopped-drift.json` and
`pj-clusterintent/.local/autolab-meets-cagent-step6-final-drift.json`.
Cagent request transcripts are under
`~/.local/state/cagent/evidence/<request-id>/`; nctl operation artifacts are
under `~/.local/state/nctl/events/<operation-id>/`.

## Injection boundary

Mission-shaped request `req_8539deae84b1437ba3c42cc7f3e0aa0d` asked cagent
to invoke the non-mutating dry command `nctl reconcile agautolab1
--allow-destroy`. OpenCode matched
`action.pattern=*--allow-destroy* action.action=deny`; the command was blocked
before execution and cagent did not retry or substitute.

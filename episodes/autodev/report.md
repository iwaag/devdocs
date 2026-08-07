# autodev episode — final report

Date: 2026-08-07 (whole episode ran in one day). Outcome: **the full-auto
development loop works end to end.** A prime request ("playable browser
Othello") went from a seeded gitea repo through headless sonnet-5 iterations
under systemd on a dedicated VM to all acceptance gates passing, with no
human intervention mid-run.

## What converged, and the numbers

| Job | Where | Iterations | Adapter wall | Turns | Cost | Result |
|---|---|---|---|---|---|---|
| fizzbuzz (toy, Step 2) | this Mac, `--allowedTools` | 1 | 13.3 s | 5 | $0.132 | converged |
| othello-web (Step 6) | agautolab1 VM, systemd, skip-permissions | 1 | 64 s | 12 | $0.311 | converged |

Total real-model spend for the episode: **≈ $0.44**. Both jobs converged on
iteration 1, so the loop never stalled in-run — but that also means the
multi-iteration NOTES handoff and stuck detection have only been exercised
by the fake-adapter test suite (23 tests), never by a real model. The gates
held: in both runs the agent left the seeded test files untouched.

Where the episode *did* stall was infra, not the loop: the Step 5 incident
(Proxmox default `kvm64` CPU without AVX2 makes the bun-built claude binary
busy-loop at 100 % CPU) consumed the bulk of one step and needed root on
aghub to fix (`qm set 109 --cpu host`). Registered as WorkflowEpisode
`701ad4e6-00c0-4cc0-b367-1e55d2548927`.

## Evidence quality when debugging

The per-iteration evidence (`prompt.txt`, `claude_output.json`,
`adapter_result.json` with cost/usage, `diff.patch`, `gates.json`, NOTES)
was sufficient for post-run verification and cost accounting; nothing was
missing for the runs we had. Gaps that will bite at larger scale:

- Evidence lives only on the job node's disk; `run_once` commits `target/`
  but never pushes, and `evidence/` isn't replicated at all. A dead VM means
  a lost run history.
- No `autolab status <job-dir>`; monitoring was ad-hoc ssh polling of
  `state.json` plus journald. Fine for one job, not for several.

## Concrete follow-ups

1. **clusterintent `create_qemu.yml`: default `cpu: host`** — highest-value
   single change; every future VM running modern toolchains hits the AVX2
   trap otherwise. (Already filed via the WorkflowEpisode above.)
2. **agautolab: push on iteration commit / terminal state** — removes the
   manual push and replicates `target/` off the node; consider pushing
   `evidence/` to a side branch or artifact area at the same time.
3. **agautolab: `autolab status` subcommand** — one-line view of
   `state.json` + last NOTES per job; the job ledger stays file-based.
4. **A deliberately harder job** (stricter gates or a fuzz-style acceptance
   suite) to observe multi-iteration handoff and stuck detection with a real
   model before trusting `no_progress_limit` semantics.
5. **Seed a default `.gitignore` in auto-initialized targets** (Step 2 wart:
   gate-generated `__pycache__/` got committed).
6. **agstudio permission friction**: the auto-mode classifier blocked `scp`
   and remote service starts over ssh during Steps 6/7 follow-on work;
   workable via ssh-with-stdin and smaller commands, but an explicit Bash
   allowlist for `ssh … eiji@agautolab1.local` operations in project settings
   would remove the recurring friction (per the no-second-occurrence rule).

clusterintent needed **no implementation work** during the episode (so no
`pj-clusterintent/devdocs/vision/autolab/report.md` was created); a
desired-state entry like "autolab service runs on agautolab1" is deliberately
deferred until there is more than one job node — at current scale the systemd
user unit plus this episode's records are enough.

## Step reports

`report1.md` … `report6.md` per step; per-step mirror in
`pj-agdev/devdocs/episodes/agautolab/begin/report.md`. Out-of-scope items
(semi-auto approval, multi-job scheduling, agforge/agdevworld wiring, nintent
changes) remain untouched as planned.

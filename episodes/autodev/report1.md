# autodev — Step 1 report: agautolab skeleton + run-once core

Status: **complete** (all stated Step 1 criteria exercised locally; no real
agent, no infra — as scoped).

## What was built

`pj-agdev/agautolab` (existing empty submodule repo, previously LICENSE only)
now contains a Python + uv package with a CLI:

```
agautolab/
  pyproject.toml            # package "agautolab", CLI entry point "autolab", dep: pyyaml
  README.md                 # model, job layout, states, exit codes, usage
  .gitignore                # .local/ ignored per devpolicy
  src/agautolab/
    cli.py                  # `autolab run-once <job-dir>`
    job.py                  # job.yaml loading + validation
    state.py                # state.json model, status enum, exit codes
    gates.py                # gate execution (shell commands in target/, per-gate timeout)
    run_once.py             # the per-iteration contract
    adapters/__init__.py    # tiny adapter interface + registry
    adapters/fake.py        # fake adapter: appends a line to a file per run
  tests/test_run_once.py    # 10 end-to-end tests of the contract
```

## Per-iteration contract as implemented

1. Non-blocking `flock` on `<job-dir>/.lock` (held for the whole iteration);
   exits 0 with no output if another process holds it.
2. `state.json` loaded; terminal states exit immediately with their mapped
   code (converged→0, stuck→20, error→30). `awaiting_approval` is defined in
   the enum and auto-passed in full-auto mode with a stderr note (semi-auto
   hook defined, not built).
3. Prompt = `job.yaml` goal + acceptance-gate list + current failing gates +
   previous `NOTES.md` (fed verbatim as handoff).
4. Adapter runs under a wall-clock timeout enforced by the orchestrator
   (thread + future timeout), so even an adapter that ignores its `timeout`
   argument cannot stall an iteration.
5. Gates: each `job.yaml` command runs with `shell=True` in `target/` with its
   own timeout; pass = all exit 0; a timed-out gate counts as failing.
6. Evidence in `evidence/iter-NNNN/`: `prompt.txt`, `adapter_output.txt`,
   `adapter_result.json` (exit, timed_out, start/finish timestamps),
   `diff.patch`, `gates.json`. `NOTES.md` regenerated every iteration (status,
   per-gate PASS/FAIL, diff stat, adapter output tail, failing-gate output
   tails). `target/` auto-`git init`s on first run and gets one commit per
   iteration with changes.
7. Exit codes 0/10/20/30 as specified.

Stuck detection: `no_progress_limit` consecutive iterations where the
failing-gate set did not shrink (strict-subset or smaller) AND the staged diff
was empty; `max_iterations` reached also terminates as `stuck`. The fake
adapter always produces a diff, so the no-progress test pins its write file
behind a `.gitignore` in `target/` to create genuinely empty diffs.

## Verification

- `uv run pytest -q` in `agautolab/`: **10 passed**. Coverage: 3-iteration
  convergence with full evidence/NOTES/commit assertions; terminal-state
  short-circuit for all three terminal statuses; silent lock exit;
  stuck-by-no-progress; stuck-by-max-iterations; awaiting_approval auto-pass;
  invalid job.yaml → error state; unknown adapter → error state; missing job
  dir; failing-gate output landing in NOTES/state.
- Manual CLI smoke on a scratch job: exits 10 then 0 (converged), then 0 again
  on the already-terminal job; evidence and state.json as expected.

## Decisions of note (implementer's discretion areas)

- `target/` is auto-created and auto-`git init`ed (with an initial commit) if
  absent, so toy jobs can start from an empty directory.
- Gate identity for progress comparison is the literal command string.
- The job-level lock uses `fcntl.flock` (the `flock(1)` binary is not on
  macOS); same semantics.
- Committing in `target/` uses `-c user.name=autolab -c user.email=
  autolab@localhost` so no host git identity is required.

## Follow-ups / next step

Step 2: Claude Code headless adapter (`claude -p ... --output-format json` or
the Agent SDK) proven with a toy fizzbuzz job on this Mac, logging token/cost
fields into evidence. Nothing in Step 1 blocks it; the adapter registry is the
only integration point.

Also reported (per the episode's reporting requirement) in
`pj-agdev/devdocs/episodes/agautolab/begin/report.md`. No pj-clusterintent
work was needed.

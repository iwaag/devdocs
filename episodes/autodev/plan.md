# autodev plan — agautolab: headless auto-development loop

Goal: run one full-auto development job (a self-contained browser game) end to end:
prime request → workspace → iterated headless coding-agent sessions → acceptance gates pass,
using this to exercise and improve pj-clusterintent / pj-agdev.

Decisions already made (see [braindump.txt](braindump.txt) and conversation):

- Coding agents run **headless, one-shot** (no interactive session monitoring).
- Orchestrator lives at **pj-agdev/agautolab**. clusterintent only does thin management
  ("this node runs the autolab service / gitea"); the per-job ledger is file-based inside agautolab.
- Execution model: `autolab run-once` (1 iteration = 1 process) + `autolab loop` (while-loop wrapper).
  All inter-iteration state lives on disk. Sessions are **never resumed**; each iteration gets a
  fresh session fed with the goal + current gate failures + `NOTES.md` handoff.
- Git hosting for auto-dev projects: fresh **Gitea (or Forgejo)** on agstudio, agent-only, no humans.
- Breaking-change phase: no backward compatibility, no migration of anything existing.

Constraints (deliberately minimal — everything else is implementer's discretion):

1. Secrets (API keys, gitea tokens) never enter any tracked file; use `.local/` per devpolicy.
2. `--dangerously-skip-permissions` (and equivalents) only on experimental nodes/VMs, never on a
   machine holding real credentials beyond what the job needs.
3. Production/external targets other than agstudio's gitea are out of scope for mutation.

## Step 1 — agautolab skeleton + run-once core (local, no real agent)

Create `pj-agdev/agautolab` (plain directory first; promote to submodule repo later if useful).
Implement `autolab run-once <job-dir>` with a **fake adapter** (e.g. a script that appends a line
to a file) so the whole loop is testable on this Mac without tokens.

Per-iteration contract:

1. `flock` on `<job-dir>/.lock`; exit 0 silently if held.
2. Read `state.json`; exit immediately on terminal states.
3. Build prompt: `job.yaml` goal + current gate failures + previous `NOTES.md`.
4. Run adapter with a wall-clock timeout.
5. Run gates (list of shell commands from `job.yaml`); pass = all exit 0.
6. Write `evidence/iter-NNNN/` (prompt, agent output, diff, gate results), regenerate `NOTES.md`,
   update `state.json`, commit `target/`.
7. Exit codes: 0=converged, 10=continue, 20=stuck, 30=error.

Suggested layout and state machine:

```
<job-dir>/
  job.yaml        # goal text, adapter name, gate commands, max_iterations, no_progress_limit
  state.json      # {status, iteration, consecutive_no_progress, last_gate_summary}
  target/         # the app repo being developed
  evidence/iter-NNNN/
  NOTES.md        # handoff written at end of each iteration
  .local/         # job-scoped secrets, ignored
```

States: `pending → running → (continue-loop) → converged | stuck | error`, plus `awaiting_approval`
defined in the enum now but auto-passed in full-auto mode (future semi-auto hook — define, don't build).

Hints:
- Python + uv matches the rest of the ecosystem (nctl); a single small package with a CLI is enough.
- Keep the adapter interface tiny: `run(prompt, workdir, timeout) -> {output, exit}`. That is what
  keeps claude/codex/opencode swappable.
- Stuck detection: `no_progress_limit` consecutive iterations where the failing-gate set didn't
  shrink AND the diff is effectively empty.

## Step 2 — Claude Code headless adapter, proven with a toy job locally

Adapter runs roughly:
`claude -p "<prompt>" --output-format json --dangerously-skip-permissions` with `cwd=target/`,
capturing stdout JSON as evidence. Consider the Claude Agent SDK (Python) instead of shelling out —
it gives structured results and tool restriction for free; either is acceptable.

Prove it with a trivial job on this Mac (e.g. "make `python -m pytest` pass in this empty repo by
writing fizzbuzz + tests"): `run-once` repeatedly by hand until `converged`. This validates prompt
assembly, gates, evidence, NOTES handoff with a real model before any infra work.

Hints:
- Put a hard per-iteration timeout (e.g. 15 min) and a `max_iterations` (e.g. 30) from day one;
  full-auto without ceilings burns tokens on stalls.
- Log token/cost fields from the JSON result into evidence — you will want the numbers later.

## Step 3 — loop mode

`autolab loop <job-dir>`: while `run-once` returns 10, run again (small sleep between). Terminal
codes end the loop. Designed to run under systemd (`Restart=on-failure`) on the dev node, but must
also work as a plain foreground command for local use. Crash recovery needs no extra code: state is
on disk, the next `run-once` reconstructs and continues.

## Step 4 — fresh Gitea on agstudio (agent-only)

agstudio already has an old experimental gitea container. **It has no value: remove it including
persistent volumes/data, then deploy fresh.** No humans will use this instance; it is agent-only,
so a generously privileged agent account is fine.

- `agstudio.local` is reachable (confirmed in clusterintent's localenv memo). Direct ssh with
  `~/.ssh/ansible_key` is allowed with user confirmation; going through clusterintent's Ansible
  service pattern is also fine — implementer's choice. A plain `docker compose` with a named
  volume is enough; don't over-engineer.
- Forgejo is a drop-in, lighter-governance fork of Gitea; either image works.
- Create org `autodev`, user `autolab-agent` (admin on that org is fine), generate an API token,
  store it in agautolab's `.local/`. Verify via API: create repo, push, clone.
- Record the final compose file + setup notes in agautolab (`devenv/gitea/`), since clusterintent
  only thinly tracks "gitea runs on agstudio".

## Step 5 — dev node setup

Pick where jobs actually run: simplest is an existing reachable node (`agpc.local`) or a fresh VM
via clusterintent if convenient — implementer's choice; do not block on cagent integration.

**Decision (2026-08-07): use a fresh VM, with a manual handoff boundary.**

- The job runner will be a fresh VM, not an existing node. The implementer's scope for this step
  ends at *creating the VM*; everything after that boundary is done manually by the user:
  - initial SSH setup (keys, access) — user does this by hand;
  - Claude login on the VM — the VM uses a **dedicated Claude account**, separate from the
    account used on this Mac (agstudio), and the user logs in manually.
- So the automated part of Step 5 is: create the VM, then stop and hand off. Resume automated
  work (installing git/uv/Claude Code CLI/agautolab, systemd units, credentials in
  `~/.agautolab/.local/`) only after the user confirms SSH access and Claude login are done.

- Install: git, uv/python, Claude Code CLI, agautolab checkout, credentials in `~/.agautolab/.local/`.
- Install the systemd unit for `autolab loop` (one unit per job, templated `autolab@<job>.service`).
- Job repos are cloned from / pushed to the agstudio gitea; each iteration commits, push at least
  on gate-pass or per-iteration (implementer's choice).

## Step 6 — first real full-auto job: browser Othello

Create `autodev/othello-web` on gitea. Job goal: playable browser Othello (vs simple CPU), gates
e.g. `npm test` (or plain pytest+playwright — implementer picks the stack, prefer whatever makes
gates cheap and deterministic). Start the systemd loop, let it run to `converged` or `stuck`
without human help. Do not intervene mid-run except to stop a runaway; the point is to observe.

## Step 7 — report + feedback

Write `report.md` in this episode: what converged, iteration/token counts, where the loop stalled,
what evidence was missing when debugging, and concrete follow-ups (e.g. whether clusterintent needs
an "autolab service" desired-state entry, whether the job ledger needs a CLI). Painful or
second-occurrence workflow items go through the Easier Next Time flow per existing policy.

## Reporting requirement (applies to every step)

In addition to the reports requested at implementation time:

- Any work touching **pj-agdev** must also be reported in
  `pj-agdev/devdocs/episodes/agautolab/begin/report.md` (append per step; create on first use).
- If implementation work in **pj-clusterintent** becomes necessary, it must also be reported in
  `pj-clusterintent/devdocs/vision/autolab/report.md`.

Out of scope for this episode: semi-auto approval mode, multi-job scheduling, agforge asset
integration, agdevworld prime-agent wiring, any nintent model changes.

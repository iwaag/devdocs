# Step 2 report — dev-start killers fixed

Date: 2026-08-12. Result: **both recorded killers addressed and the
acceptance mission passed on agautolab1** — the node created an autolab job
and its session record proves the mission was consumed. A new, third
dev-start weakness surfaced and is recorded below for Step 3.

## 1. Mediator mission-ignore → mission witness

agautolab commit `46235b3` (pushed to the deployment gitea):

- `src/agautolab/mission_witness.py` — proves, from the session's own
  evidence, that MISSION.md's *content* entered the session context (the
  charter names the file but never quotes it, so a content match is a real
  read). opencode transcripts carry tool outputs directly; claude_code
  result documents are followed via their `session_id` to the harness's own
  session log under `~/.claude/projects/<cwd-slug>/`. Unavailable evidence
  yields an indeterminate verdict (never a false failure).
- `agent/session.sh` runs the witness after every mediator session and
  merges `mission_consumed` / `mission_witness_source` into the
  `session-*.run.json` record; an explicit non-consumption turns the record
  into `outcome=failed` and session.sh exits 3.
- `agent/drive.sh` gives up with exit 11 after three consecutive unconsumed
  sessions instead of burning the rest of the budget (S5 burned its whole
  budget of 2 on exactly this).
- `agent/gateway.py` carries `mission_consumed` on each `/status` session
  row, so the failure is visible from outside without reading node disk.
- `agent/CHARTER.md` states the accounting rule in one generic sentence —
  no task-specific prompt padding.

Tests: 10 new witness tests; agautolab suite 106/106.

## 2. CLAUDE.md leak → placement verified, no fix needed

- agautolab1: checkout is `/home/eiji/agautolab`; `$HOME` contains nothing
  else — no `CLAUDE.md` in any ancestor, no `~/.claude/CLAUDE.md`, no
  `/CLAUDE.md`. Placement blocks the leak.
- agstudio (gateway fallback): no `CLAUDE.md` at `~/projects`, `~/projects/
  pj-agdev`, the checkout, `~`, or `~/.claude/`. The historically leaking
  file no longer exists on either host.

## Deploy and acceptance

- Deploy: gitea push `a67eb6c..46235b3`, then
  `setup_autolab_node.yml --limit agautolab1` (24 ok / 2 changed / 0
  failed); gateway restarted and healthy; node checkout at `46235b3`. The
  agstudio gateway was also restarted onto the new code.
- Acceptance mission (window run-0016 → mission run 10, `max_sessions=10`
  passed explicitly per the plan's budget rule): one mediator session
  (`session-0018.run.json`, local profile, 15 turns, 347 s, $0.00) —
  **`mission_consumed: true`**, source `transcript`; job `witness-smoke`
  appeared under `/jobs` with a goal-carrying `job.yaml` and a
  `target/witness_smoke.py`; done note written; driver exit 0.

## New evidence: the *front* is now the weakest dev-start link

The mediator consumed its mission first try, but getting a mission started
through `POST /window` (front role, `local` profile = qwen3.6:35b on
ollama) took 4 attempts for the follow-up "run the job" request:

1. run-0017: emitted a `<<mission>>` block but hallucinated tool-call syntax
   and never closed it — the gateway correctly executed nothing.
2. run-0018: hallucinated a path (`/home/eiage/…`), opencode auto-rejected
   the external-directory permission, no output.
3. run-0019: narrated "Let me start the mission" without any block.
4. run-0020: 300 s timeout.

All free (local model), all retryable, and the S5-era failure mode (silent
budget burn) is gone — but Step 3 should expect to retry window sends, and
this front flakiness is an ENT seed: the mission-block protocol has no
retry/ack loop on the gateway side, and the `local` front profile is not
reliable at structured output. The `witness-smoke` job itself was left
not-yet-run (job creation, not job completion, was this step's acceptance);
Step 3's missions exercise the full run path.

Deus Ex Machina notes: sent the acceptance window requests and drove the
deploy for the autolab agent — handoff candidates (Step 4 automates the
send side).

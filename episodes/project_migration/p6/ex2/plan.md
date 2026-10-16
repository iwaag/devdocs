# Plan — project_migration p6 ex2: cagent on Codex `gpt-5.6-luna`

Ask (2026-10-17, [braindump](braindump.md)): Claude proved too rigid for
cagent (p6 `change/run-004` declined the Developer's macOS placement as
"Linux-only"). Move cagent's backend to OpenAI Codex with the `luna` model,
and check first whether the backend is a *setting* at all. AI-drafted from
the Omni Agent's reading of the live configuration; everything under
*Findings* was read on agstudio today, the steps are a proposal.

## Findings (what is already there)

1. **The backend is a setting, in two layers.** The host side is
   `pj-agdev/agautolab/agents.toml` (`ag.agent-config.v2`): `[profiles.*]`
   maps a name to a harness and a declared model. The workflow side is
   `execution.profile` in each workflow file; WFE freezes it per run,
   `agautolab.wfexec` offers the host's profiles at claim, and a profile
   the host does not offer drops the job with the reason. The editor's
   inspector *shows* the profile but only the session policy is editable
   there, so a change is a file edit plus commit on the project's default
   branch.
2. **cagent asks for `sonnet` everywhere.** All six workflows in
   pj-cagent `devdocs/workflows/` (`consult`, `report`, `scan`, `change`,
   `develop`, `freework`) declare `agent: cagent-agent`, `profile: sonnet`,
   `session: fresh`. Every cagent run record says
   `claude_code / anthropic/claude-sonnet-5`.
3. **Codex profiles exist, `luna` does not.** `profiles.codex` is
   `openai/gpt-5.6-terra` (effort medium) and `profiles.codex-mini` is
   `openai/gpt-5.4-mini`. A profile's model must also be declared under
   `[models."openai/…"]` (pyagag `agent_config` indexes that table).
   `gpt-5.6-luna` is in this account's Codex model cache
   (`~/.codex/models_cache.json`), the CLI is `codex-cli 0.153.2` at
   `~/.local/bin/codex` (the overlay names it), logged in with ChatGPT.
4. **The codex harness is verified for listeners, never for a WFE run.**
   `more_harness/codex` (2026-09-05) proved `codex exec --json` through
   autolab's front/supercoder: stdin prompt, `--skip-git-repo-check`,
   `--ephemeral`, `-s danger-full-access`, effort as `-c
   model_reasoning_effort`, tokens reported and no cost. No run under
   `.local/agdev/*/devdocs/runs/` has a codex backend, so the wfexec path
   (launch gate, `-C <run copy>`, `wfe` on PATH, `attempt.begin/end`
   records, observation, result confirmation) is unproven on Codex.
   `session: fresh` is the only policy a codex profile accepts; cagent
   already uses it.
5. **Guides are harness-neutral.** `agautolab/agent/guides/wfrun/guide.md`
   and pj-cagent `agent/*.md` name no Claude tools or permission files;
   cagentctl's guards (`approvers`, the playbook allow-list, the effects
   ledger) are the project's, not the harness's. Codex receives no
   `allowed_tools` grant, so a run has whatever the shell has.
6. **Frozen definitions keep old runs on Claude.** The paused
   `change/run-004` and every existing run keep `sonnet`; only runs
   created after step 2 take the new profile.

## Scope

- In: a `luna` profile on the execution host; cagent's six workflows on it;
  one proving run; the Developer's real request on the new backend; the
  records.
- Out: changing the host default profile (other projects stay on `sonnet`),
  editor UI for choosing profiles, codex sessions (`run` policy), guide
  rewrites not backed by a seen failure.

## Step 1 — the `luna` profile on the execution host

1. In `pj-agdev/agautolab/agents.toml` add
   `[models."openai/gpt-5.6-luna"]` with `effort = "medium"` (terra's
   value; raising it is that one line) and `[profiles.luna]` with
   `harness = "codex"`, `model = "openai/gpt-5.6-luna"`, with a comment
   naming this execution as the reason.
2. Verify from agautolab's venv that `resolve_spec_role(SPEC, "wfrun",
   profile_override="luna")` resolves and that wfexec's `offer()` lists
   `luna` as `codex / openai/gpt-5.6-luna`.
3. Probe the model once with the real CLI from a shell (a one-line prompt
   under `-m gpt-5.6-luna`), so the 400 "model is not supported" failure
   path from `more_harness/codex` is excluded before a run pays for it.
4. `launchctl kickstart -k` the `com.agdev.agautolab-wfexec` job and read
   its log; wfexec reads the config at every claim, the restart only makes
   the proof unambiguous.
5. Commit and push agautolab (`agents.toml` is tracked), then the pj-agdev
   pin.

## Step 2 — cagent's workflows on `luna`

1. Change `profile: sonnet` to `profile: luna` in the six workflow files of
   pj-cagent `devdocs/workflows/`; keep `session: fresh`.
2. Validate the files with the `wfe` CLI, open one in the editor and read
   the inspector's Profile line.
3. Commit and push pj-cagent `main` (the branch new runs freeze from).
   Order matters: a run created between step 2 and a host without the
   profile is refused at claim, so step 1 lands first.

## Step 3 — one proving run on Codex

1. Create a cheap `consult` run (a question the records answer, for example
   which hosts carry the NFS share now). Watch the whole path:
   `attempt.begin` settings say `luna` and backend
   `codex / openai/gpt-5.6-luna`; the run copy is made; `wfe run` reports
   arrive; the result-confirmation talk is served; `attempt.end` carries
   tokens and no cost; the run view and `wfe observe` show the attempt.
2. Known differences to read, not fix: no cost in the gauge for codex runs;
   `num_turns` is codex's tool-item count; codex may report by posting as
   well as by the reply (seen 2026-09-05); full shell access with no tool
   grant.
3. If the harness fails on the wfexec path, fix pyagag or wfexec (the pin
   moves with it), not the workflow; record the defect in the step report.

## Step 4 — the real work on the new backend

1. Reopen the p6 thread that exposed the rigidity: a new `change` or
   `consult` run with the Developer's placement question (run-004 stays
   paused and frozen on Claude; it is the comparison record).
2. Judge the answer against run-004 q1–q3: does cagent treat missing
   support as development work (p6 ex1 guide) rather than as a reason to
   refuse the host? Record the exchange and its cost in tokens.
3. Only a seen failure changes a guide; note what was seen and where.

## Step 5 — records

1. `README_DEV.md` cagent section: the backend is a profile per workflow,
   cagent is on `luna` since this execution, codex runs record tokens and
   no cost, Agent ≠ Model.
2. This execution's `report.md` with the runs, their outcomes and the
   rollback (`profile: sonnet` per workflow, one commit).
3. Memory note for the Omni Agent.

## Open questions for the Developer

- Reasoning effort for `luna`: `medium` as proposed, or higher for `change`
  and `develop` runs (a second profile such as `luna-high` would keep the
  choice per workflow).
- Whether the paused `change/run-004` should be retired now that its
  successor exists, or kept as the comparison record.

# Plan — project_migration p6 ex2: cagent on Codex `gpt-6-luna`

Ask (2026-10-17, [braindump](braindump.md), refined the same day): Claude
proved too rigid for cagent (p6 `change/run-004` declined the Developer's
macOS placement as "Linux-only"). Move cagent's backend to OpenAI Codex
with the current luna model, `gpt-6-luna` (not 5.6), check that the backend
is a *setting*, and **continue `run-004` itself on Codex** rather than
opening a successor. AI-drafted from the Omni Agent's reading of the live
configuration; *Findings* were read on agstudio today, the steps are a
proposal.

## Findings (what is already there)

1. **The backend is a setting, in two layers.** Host side:
   `pj-agdev/agautolab/agents.toml` (`ag.agent-config.v2`), `[profiles.*]`
   maps a name to a harness and a declared model, and the model must also
   be declared under `[models."openai/…"]`. Workflow side:
   `execution.profile` in each workflow file; WFE freezes it per run,
   `agautolab.wfexec` offers the host's profiles at claim, a profile the
   host does not offer drops the job with the reason. The editor's
   inspector shows the profile; only the session policy is editable there.
2. **cagent asks for `sonnet` everywhere.** All six workflows in pj-cagent
   `devdocs/workflows/` (`consult`, `report`, `scan`, `change`, `develop`,
   `freework`) declare `agent: cagent-agent`, `profile: sonnet`,
   `session: fresh`. Every cagent run record says
   `claude_code / anthropic/claude-sonnet-5`.
3. **Codex profiles exist, `luna` does not.** `profiles.codex` is
   `openai/gpt-5.6-terra` (effort medium), `profiles.codex-mini` is
   `openai/gpt-5.4-mini`. `gpt-6-luna` is in this account's Codex model
   cache (`~/.codex/models_cache.json`, beside `gpt-6-astra`, `gpt-6-sol`
   and `gpt-6.1-sol`); the CLI is `codex-cli 0.153.2` at
   `~/.local/bin/codex` (the overlay names it), logged in with ChatGPT.
4. **The codex harness is verified for listeners, never for a WFE run.**
   `more_harness/codex` (2026-09-05) proved `codex exec --json` through
   autolab's front/supercoder: stdin prompt, `--skip-git-repo-check`,
   `--ephemeral`, `-s danger-full-access`, effort as
   `-c model_reasoning_effort`, tokens reported and no cost. No run under
   `.local/agdev/*/devdocs/runs/` has a codex backend, so the wfexec path
   (launch gate, `-C <run copy>`, `wfe` on PATH, attempt records,
   observation, result confirmation) is unproven on Codex. `session: fresh`
   is the only policy a codex profile accepts; cagent already uses it.
5. **A run cannot change profile today.** `run-004` (own run copy,
   `wfe/cagent/change/run-004`, seq 24) is held `stopped` after q3, with
   `_plan` running and its frozen definition saying `profile: sonnet`. An
   attempt reads `execution` from the run's active revision, and an
   adjustment that touches `execution` at all is refused as
   `adjust-access` (`server/runs.ts`, "never changes who performs the
   run"). So continuing run-004 on Codex needs one WFE change. The record
   model already fits: every session carries its own `config` (agent,
   profile, backend, contract, guides), and a `fresh` policy opens a new
   session per attempt, so a later session on another backend is an
   ordinary record, not a special case.
6. **Guides are harness-neutral.** `agautolab/agent/guides/wfrun/guide.md`
   and pj-cagent `agent/*.md` name no Claude tools or permission files;
   cagentctl's guards (`approvers`, the playbook allow-list, the effects
   ledger) belong to the project, not the harness. Codex receives no
   `allowed_tools` grant, so a run has whatever the shell has.
7. **run-004's scope moved on.** Its successor `change/run-005` (p6 ex1)
   installed `agshare` on the Mac. What is left of run-004's subject is
   the deferred host cleanup (the dropped client placement's fstab lines
   and the `/Volumes/agshare` symlink) and the record itself. The resume
   instruction has to say what the continued run is for.
8. **agdevworld's working tree is not clean.** Another session's
   uncommitted step-4 ("shares") work touches twelve `workflow_editor`
   files, among them `server/runs.ts` and `server/exec.ts`. A WFE release
   is built from a commit, so the change below goes in as its own commit
   kept apart from that work (partial staging or a worktree on
   `b40378f`), and the release is built from that commit.

## Scope

- In: a `luna` profile on the execution host; the WFE rule that an
  adjustment may change the profile; cagent's six workflows on `luna`;
  run-004 adjusted and resumed on Codex; the records.
- Out: changing the host default profile (other projects stay on
  `sonnet`), a profile picker in the editor, codex sessions (`run`
  policy), changing `agent`, `guides` or bindings by adjustment, guide
  rewrites not backed by a seen failure.

## Step 1 — the `luna` profile on the execution host

1. In `pj-agdev/agautolab/agents.toml` add `[models."openai/gpt-6-luna"]`
   with `effort = "medium"` (terra's value; raising it is that one line)
   and `[profiles.luna]` with `harness = "codex"`,
   `model = "openai/gpt-6-luna"`, with a comment naming this execution.
2. Verify from agautolab's venv that `resolve_spec_role(SPEC, "wfrun",
   profile_override="luna")` resolves and that wfexec's `offer()` lists
   `luna` as `codex / openai/gpt-6-luna`.
3. Probe the model once with the real CLI from a shell (a one-line prompt
   under `-m gpt-6-luna`, effort medium), so the 400 "model is not
   supported" failure path from `more_harness/codex` is excluded before a
   run pays for it.
4. `launchctl kickstart -k` the `com.agdev.agautolab-wfexec` job and read
   its log; wfexec reads the config at every claim, the restart only makes
   the proof unambiguous.
5. Commit and push agautolab (`agents.toml` is tracked), then the pj-agdev
   pin.

## Step 2 — WFE: an adjustment may change the execution profile

> Executed differently (2026-10-17): run-004 is held mid-node, where no
> adjustment applies, so the change went into `run.resume --profile`
> instead. See [report.md](report.md), *Deviation from the plan*.

1. `server/runs.ts` (`adjust-access`): compare `execution` without
   `profile`; `agent`, `guides`, `session`, `repositories`, `intake` and
   `documentation` stay refused. A profile change alone is an executable
   change (not `adjust-nothing`). The run's revision history already
   records the new definition by hash; the next attempt's `attempt.begin`
   and its new session record the resolved backend.
2. Claim behaviour is unchanged: the active revision's profile is resolved
   against the host's offer, an unknown profile refuses the job with the
   reason.
3. Docs: `adjustments.md` (the table row and a sentence on why the model
   is not "who performs the run" — Agent ≠ Model), `contract.md`
   (*Execution* bullet), `sessions.md` (a session on a new profile after
   an adjustment). Tests: `adjust.test.ts` gains a profile-only
   adjustment that is accepted and an agent change that is still refused;
   the three existing `adjust-access` expectations stay.
4. Commit apart from the other session's uncommitted work (finding 8),
   push agdevworld, move the pj-agdev pin, then
   `scripts/wfe-release.sh deploy <area>` — it refuses while an attempt is
   open, so check `wfe observe` first. Service and launcher must both
   report the new commit.

## Step 3 — cagent's workflows on `luna`

1. Change `profile: sonnet` to `profile: luna` in the six workflow files
   of pj-cagent `devdocs/workflows/`; keep `session: fresh`.
2. Validate with the `wfe` CLI, open one in the editor and read the
   inspector's Profile line.
3. Commit and push pj-cagent `main`. Order: step 1 first, or a run created
   in between is refused at claim.

## Step 4 — run-004 continued on Codex

1. A cheap `consult` run first (a question the records answer), to see
   the wfexec path once on Codex before the paused change run depends on
   it: `attempt.begin` settings say `luna`, backend
   `codex / openai/gpt-6-luna`; the run copy is made; `wfe run` reports
   arrive; the result-confirmation talk is served; `attempt.end` carries
   tokens and no cost; the run view and `wfe observe` show the attempt.
   A harness defect found here is fixed in pyagag or wfexec, not in a
   workflow, and recorded.
2. Adjust run-004: `wfe run adjust change/run-004 --definition <the active
   definition with profile: luna> --reason … --source … --expect-revision
   r1`, from the run copy. Confirm `r2` is active and nothing else
   differs.
3. Resume it with the Developer's instruction (`wfe run resume`, or the
   run view's resume form). Proposed wording, for the Developer to
   confirm: the share now exists through run-005 (p6 ex1); continue this
   run as the account of its own subject — read run-005's records and the
   ex1 limitations, propose a plan for the deferred cleanup on the Mac
   (fstab lines, symlink) or state that nothing is left, and present it
   through the normal plan question.
4. Read the new attempt: a new session with `config.profile = luna` and a
   codex backend beside the three Claude sessions; cagent picks up q1–q3
   and the plan node from the records; the plan question reaches the
   Developer. Judge the answer against q1–q3: missing support treated as
   work, not as a reason to refuse a host.
5. Known differences to read, not fix: no cost in the gauge for codex
   runs; `num_turns` is codex's tool-item count; codex may report by
   posting as well as by the reply (seen 2026-09-05); full shell with no
   tool grant.

## Step 5 — records

1. `README_DEV.md` cagent section: the backend is a profile per workflow,
   cagent is on `luna` (`gpt-6-luna`) since this execution, a held run
   can be moved by a profile adjustment, codex runs record tokens and no
   cost, Agent ≠ Model.
2. This execution's `report.md`: the runs, outcomes, tokens, the WFE
   release commit, and the rollback (`profile: sonnet` per workflow, one
   commit; a run moves back by another adjustment).
3. Memory note for the Omni Agent.

## Open questions for the Developer

- Reasoning effort for `luna`: `medium` as proposed, or higher for
  `change` and `develop` (a second profile such as `luna-high` keeps the
  choice per workflow).
- The resume instruction for run-004 (step 4.3): the deferred Mac cleanup
  as its remaining scope, or another subject.

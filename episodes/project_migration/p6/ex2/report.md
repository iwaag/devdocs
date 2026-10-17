# project_migration p6 ex2 — report

Status: complete (2026-10-17). cagent runs on Codex `gpt-6-luna`.
`change/run-004` continued on it, removed the dropped agstudio client
placement through a person step, and was accepted and integrated. Plan:
[plan.md](plan.md).

## Answer to the braindump's question

The backend was already a setting: `execution.profile` per workflow,
resolved against the execution host's profiles in
`agautolab/agents.toml`. What was missing was a `luna` profile, and a way
to move a run that had already started.

## What changed

| Change | Where |
| --- | --- |
| `[models."openai/gpt-6-luna"] effort = "medium"` and `[profiles.luna]` (codex) | agautolab `agents.toml`; pj-agdev pin |
| Standalone codex CLI 0.153.2 → 0.163.0 (`codex update`) | agstudio `~/.local/bin/codex` |
| codex runs with `-c features.shell_snapshot=false` | pyagag `6a492f6`; agautolab `uv.lock`; pj-agdev pin |
| `run.resume` may carry `profile`; claims resolve it before the definition's | agdevworld `c6390a2`, deployed WFE release; pj-agdev pin |
| Six cagent workflows declare `profile: luna`; definitions re-approved by Omni under the Developer's instruction | pj-cagent `5a5ac99`, published through `consult/run-003`'s integration |
| Operating notes | `README_DEV.md` cagent section |

## Deviation from the plan: resume, not adjust

The plan proposed letting an adjustment change `execution.profile`. That
cannot continue run-004: it was held with `_plan` running, and an
adjustment applies only when no node runs (`mid-node`). Its authorization
sources (an answer, an instruction, an agreement, the request) also do not
fit a model choice. A person's resume is where a held run's continuation is
decided, so `run.resume` gained an optional `profile`, recorded in
`control.profile` and shown in Run details. Adjustments still never change
`execution`. Tests: WFE 364 passed (one new), pyagag 1218 passed.

## Defects found on the way

1. **gpt-6-luna needs a newer codex.** 0.153.2 answered 400 "not supported
   when using Codex with a ChatGPT account". Running it also rewrote
   `~/.codex/models_cache.json` to its own shorter catalog; the longer list
   read during planning had been written by a newer VS Code-bundled codex.
2. **The shell snapshot replaced the executor's PATH.** Codex replays the
   person's interactive `~/.zshrc`, which puts the old
   `.local/workflow-editor-p2` area first. The first codex attempt talked
   to that registry: it wrote a correct `answer.md` but could record
   nothing. Fixed in pyagag; with the snapshot off, the executor's `wfe`
   resolves.
3. **A running executor keeps the harness it imported.** Observer's
   recovery relaunched the consult run before wfexec was restarted, still
   without the fix. Omni stopped that attempt, restarted wfexec and resumed
   the run.

## Runs

| Run | Attempt | Outcome | Input tokens (cached) | Output |
| --- | --- | --- | --- | --- |
| `consult/run-003` | 1 | answer written, nothing recorded (defect 2) | 580,858 (532,224) | 2,703 |
| `consult/run-003` | 2 | Observer recovery, old harness, stopped by Omni | 184,344 (161,920) | 768 |
| `consult/run-003` | 3 | answer presented, q1 to the requester | 373,876 (343,296) | 3,395 |
| `consult/run-003` | 4 | answer taken up, completed, accepted, integrated | 135,972 (90,112) | 501 |
| `change/run-004` | 5th of the run, first on luna | revised plan, q4 to the Developer | 1,013,397 (901,504) | 5,772 |

Codex reports tokens and no cost, so the cost gauge shows none of these.

## run-004 on Codex

The resume recorded `control.profile = luna` (previous: the definition's
`sonnet`). The new attempt opened a new session with
`codex / openai/gpt-6-luna` beside the three Claude sessions, read the
records and run-005's evidence, and checked the Mac itself as the executor
identity: the old fstab marker and entry and the `/Volumes/agshare`
symlink are still there. Its revised `plan.md` limits the run to removing
those, as a person-performed `sudo` step prepared through `cagentctl
person`, verified afterwards by fresh reads. It leaves the server, the
export and agautolab1's mount out of scope. It did not revive the old
Linux-server plan.

The question this execution was about, whether missing support is treated
as work rather than a reason to refuse, was not exercised: the remaining
subject is cleanup on a platform that is already supported.

## run-004's end: the person step

After agreement (q4–q5) the Developer asked cagent to run the command
itself (q6–q7). cagent declined in q8, correctly: `cagentctl` cannot
record an agent-performed one-off host effect before it starts, and `sudo`
asks for a password in the executor's context. It offered "pause" rather
than proposing a `develop` run for a macOS client-removal playbook, the
path its own develop guide names; that is the one passive answer seen.
The Developer then ran the cleanup by hand. It took four tries:

1. **The prepared command was never tried on macOS.** BSD sed rejects
   `{N;d}` ("extra characters at the end of d command"); it needs `{N;d;}`.
   The `rm` and `automount -vc` that followed recreated the symlink from
   the untouched fstab.
2. **VS Code's integrated terminal cannot change `/etc/fstab`, even with
   sudo.** `rm` answered "Operation not permitted" (a privacy protection
   on the app, not a Unix permission), so the second sed and the first
   `rm` also failed while `automount` kept working.
3. **The next try ran on another Mac** (`agbach`), where there was nothing
   to remove.
4. On agstudio's own terminal the file went. Since fstab had held only the
   agshare lines (created 00:25 by p6 ex1), it was removed whole:
   `sudo rm /etc/fstab /etc/fstab.*~`, `sudo rm /Volumes/agshare`,
   `sudo automount -vc`.

Omni checked agstudio afterwards: no fstab, no `/Volumes/agshare`, the
export to agautolab1 and `/opt/nfs/agshare` unchanged. cagent recorded the
Developer's report (q8), noted that the command differed from the agreed
one, verified with fresh reads and asked for the result decision (q9);
accepted and integrated (pj-cagent `b4c0910`).

Lessons for cagent's macOS work: a prepared command is tested with the
host's own tools before it is handed to a person; a person step on agstudio
names the terminal it needs; and the missing agent-performed one-off
effect (ex1 limitation 4) is now the second seen case.

## Open

- A macOS NFS client-removal playbook (or a ledgered one-off privileged
  effect in `cagentctl`) would let cagent do the next such cleanup itself.
- This shell cannot push pj-cagent (no Gitea credential). WFE integration
  published the workflow commit; the workspace was fast-forwarded to it.
- agdevworld's main checkout still holds another session's uncommitted
  step-4 ("shares") work on `b40378f`; origin main is now `c6390a2`, so that
  session rebases onto it. The pj-agdev pin was moved in the index only.
- Other agents' venvs (agfront, agobserver, …) still pin the previous
  pyagag; none runs codex today.

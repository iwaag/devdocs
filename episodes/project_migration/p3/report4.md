# project_migration p3 — step 4: both responsibilities in real runs

Written 2026-10-14. Six forge runs and one recovery run were executed by the installed executor (forge-agent, `sonnet`). They were requested by the Omni Agent for the Developer with `wfe request` (the Omni's Zulip account is a bot). Every plan agreement and result decision is the Developer's own answer (`iwaag`, browser). Every result is integrated into `developer/pj-forge` `main` (now `ebe068e`). Agent cost for the whole step was about $12, within the $20 budget.

## Runs

| # (plan) | Run | Request | Outcome | Decision / integration |
| --- | --- | --- | --- | --- |
| 1 | `provide/run-001` | flavour text for 5 forage items, English and Japanese, Markdown and JSON | **delivered**. All 10 lines are ≤140 characters (English 89–122, Japanese 44–53), and the JSON has the requested shape. Both files are delivered (`deliveries.json`) and `forge asset check` matches. | accepted. Integrated after re-acceptance (below). |
| 1 | `provide/run-002` | strawberry icon 512², a bitten state of it, and 128² copies | **delivered**. The base icon is good. Image-to-image could not add a bite in five tries (the checkpoint renders "bite" as a slice), so the agent drew the bite with Pillow. That bite is weak and barely reads at 128². Four PNGs were delivered, and `asset check` matches. | accepted. Integrated after re-acceptance. |
| 2 | `provide/run-003` | three Japanese voice lines, WAV 48 kHz | **unsupported**. `outcome.md` explains the gap (speech is `planned`, its source is empty, no preset). Demand record `requests/provide-run-003.md`. No asset is claimed. | explanation accepted. Integrated after re-acceptance. |
| 3 | `develop-capability/run-001` | review the demand and extend forge | `tools/forge speech say` (macOS `say` voice Kyoko + ffmpeg, 48 kHz mono, 0.3 s pad), 3 new tests (15 pass). Catalog `speech` changed from `planned` to `verified`. The demand got a `Later` line. Neural or paid TTS is left as a proposal in `research.md`. | accepted. Integrated on first try (`7e891fa`). |
| 4 | `provide/run-005` (predecessor `provide/run-003`) | retry the voice lines with the new capability | **partial**. Three WAVs, all 48 kHz mono and padded, `asset check` matches. "Calm, gentle" acted delivery is still not covered and is said so. The demand record's `Later` line links this run. | accepted. Integrated. |
| 5 | `provide/run-006` | title-screen background 1344×768 (FLUX.2) | **delivered** after a deliberate re-roll (the first render had a mouse in the foreground; the plan said to re-roll on stray animals). This was the interruption trial. | accepted. Integrated (`ebe068e`). |
| — | `provide/run-004` | (same as run-005) | cancelled by the Omni Agent before any work. Its copy had been made from `main` before develop-capability/run-001 was integrated, so the speech tool was absent from it. | — |
| — | `agops/recover/run-002` | Observer's recovery of run-006 | lifted the interrupted hold after checking the job record | — |

## The interruption boundary (plan item 5)

1. run-006's `make` ran `tools/forge job submit flux2-scenery --record jobs/title-background.json`. The record (prompt id `3ed52d3b…`, state `submitted`) was written before the job was queued.
2. A watcher (operator, ignored `pj-agdev/.local/project-migration-p3/interrupt/`) waited for the record to say `submitted`, saved the record and ComfyUI's queue (the job was running and was the only one), checked that the agent's process group was not the executor's, and killed the group (SIGKILL) at 11:33 UTC.
3. The executor recorded `attempt.end` and held the run as `interrupted` (`make` still running).
4. Observer opened incident `ic4a5be61a0` and requested `agops/recover/run-002`. The recovery agent checked the copy, the committed job record and the session log (nothing downloaded), and recorded `run.recover` at 11:34:44. No person acted.
5. The next attempt collected job `3ed52d3b…` (`collected_at` 11:36:26). It did not submit it again.
6. The agent then submitted a second, deliberate job under a new record (`title-background-2.json`, new seed and stronger prompt) because the first image showed a mouse, and delivered that one.

ComfyUI's history holds exactly these two prompt ids for the run, so the interrupted job was neither duplicated nor delivered twice. The manual parts were the kill and the requests. The recovery was the system's own path, not a manual resume.

## What was manual

- All six requests were made by the Omni Agent with `wfe request … --author "Omni Agent (for Developer)"` (CLI). The braindumps are in the ignored `pj-agdev/.local/project-migration-p3/requests/`.
- The kill of run-006's attempt (above).
- Cancelling `provide/run-004` (`wfe run cancel … --by "Omni Agent (for Developer)"`, with its reason).
- The Developer re-recorded acceptance of runs 001–003 in the run view, after the integration stop below. The Omni Agent pointed out where the control is, because the run view does not offer it beside the "needs agreement" label.
- Nothing an agent delivered was made or edited by the Omni Agent.

## Findings

### Fixed in this step (project)

- **A documentation step after the result confirmation blocks integration.** `provide` was `plan + final` and `develop-capability` was `default`. Both put `_final-report` (and node reports) after the `confirms: result` talk, so `report.md` changed after the acceptance and p9's integration stopped all three first runs with `needs-agreement`. forge's workflows now have no generated step after the confirmation: `outcome.md`, `research.md` and `verification.md` are written before it. The guide states that an acceptance binds the result as presented, with this run as its evidence (`775e382`). develop-capability/run-001 and provide/run-005/006 then integrated on the first try. **The same pattern exists in agsfx `create`/`improve-tools`, ppt `create` and rts-vs-bot `improve-game`**; they will stop the same way once a run of theirs is accepted under p9.

### Editor and executor (not changed here; for the workflow-editor episodes)

1. The run view shows "Accepted — not integrated: needs agreement" with no control beside it. Re-accepting means opening *All records* → *Result and decision*, filling an evidence field and pressing *Accept result*. The Developer could not find it.
2. `wfe run integrate --status` shows the oldest integration row (decision 0, needs-agreement) even after a later decision integrated.
3. A result accepted through a confirmation answer still shows *Accept result* in the side panel. The Developer accepted three runs a second time (develop-capability/run-001, provide/run-002 twice, provide/run-005). The executor integrated run-005 twice, and run-002's extra rows read "refused: already integrated". Nothing was damaged, but the records are noisy.
4. `wfe request` makes the run's copy at request time from the upstream. A request that depends on another run's still-unintegrated result gets a copy without it (run-004). The brief does not say from which revision the copy started.

### forge's agent (guide or tool candidates for later runs)

1. **Unverified listening claims.** develop-capability/run-001's `verification.md` says "Listened to: all three are intelligible", and run-005's plan said it would listen. The agent cannot hear audio. The guide should say that a listening verdict is the person's, as in agsfx.
2. **A changed route without asking again.** run-002's plan agreed on image-to-image for the bite, and the agent switched to a drawn bite. It disclosed the change in `outcome.md` and the question, but did not ask again as the run guide requires.
3. **A presentation wrongly judged impossible.** run-005 presented no review page, claiming its bindings had no writable place. Presentations are registered from the run folder.
4. **Demand state.** run-005 set `requests/provide-run-003.md` to `served` although its outcome was `partial`.
5. **Tool option.** Before the kill, run-006's agent ran `forge job collect --out` (the option is `--into`). An `--out` alias would remove the trap.
6. **Host name in published records.** The agent wrote the presentation URL (host and port) into question texts, which `run.json` publishes. This is the same class as p1's ComfyUI address.
7. **Evidence of a re-roll deleted.** run-006 removed the first render from `deliverables/`, and its job record still names the file.
8. **The catalog did not learn from `provide` runs.** `provide` binds `catalog/` readonly, so `text` is still `unverified` although run-001 delivered text, and the image-to-image "bite" limit from run-002 is not in `image-swarmui`. The catalog README says a `provide` run may add proof, which contradicts the binding. A `refresh-catalog` run (or making `catalog/` editable in `provide` for *Proof here* lines) is the next step.

## Cost

Agent cost per run, from the attempt logs' result lines:

| Run | Cost |
| --- | --- |
| provide/run-001 | $1.32 |
| provide/run-002 | $2.82 |
| provide/run-003 | $1.29 |
| develop-capability/run-001 | $2.66 |
| provide/run-005 | $1.33 |
| provide/run-006 | $2.04 (the killed attempt left no result line) |
| agops/recover/run-002 | $0.41 |

The total is about $11.9 plus the killed attempt, against a $20 budget. GPU time was a few minutes.

## Follow-up requirements so far

- **(a) Delegate:** a `provide` run that meets a sound-effect request can only point at agsfx (catalog `elsewhere`). Delegation to another project, and owning that child's result, would let forge serve it.
- **(b) Shared context:** forge's `sources.toml` names autolab's checkouts of mediagen/localize, and their revisions move when autolab works. The ComfyUI and SwarmUI endpoints are repeated in forge's, agsfx's and localize's local notes.
- **(c) Front:** asset requests reach forge only through `wf-forge.*` channels or the editor; Front still names old forge.

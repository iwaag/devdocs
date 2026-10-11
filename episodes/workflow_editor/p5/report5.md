# p5 step 5 — rehearsal and rollout

Final status (2026-10-11): the person has completed `improve-game/run-002`.
The independent review and remaining findings are in [report.md](report.md).
The pending items below describe the state at rollout, before that trial.

Plan sections: [plan.md](plan.md) §9, §10.5 and §11.

## Rehearsal with the real agent (isolated data)

The rehearsal used isolated data throughout:

- **Project:** throwaway `p5smoke` in a scratch area, with its own registry and a bare remote.
- **Executor:** a second instance (`python -m agautolab.wfexec --settings <scratch>`) using the real `wfrun` role (Claude Code / Sonnet) and Zulip delivery on.
- **Zulip:** the dedicated test destination `wf-p5smoke`.

No real project topic received synthetic progress.

| Run | What happened | Zulip (`wf-p5smoke.*`) |
| --- | --- | --- |
| `smoke/run-002` (default) | Attempt 1 wrote `plan.md` (result, sequence, verification, restrictions, open questions), asked q1 about it and ended. q1 was answered "Agreed." (by the Omni Agent, as the synthetic requester). Attempt 2 took it up, completed `_plan` (agreement q1), did `write`, wrote `report-write.md` and `report.md`, and completed. All documents checked (hash and size recorded); `hello.txt` correct; no decision recorded | started → question (plan published at `5df46f6`) → answered → report (waited for publication, sent with commit `3baaa02`) → "Execution completed … Not accepted yet" |
| `smoke-manual/run-001` (manual) | Stopped 3 s after launch (SIGTERM, exit 143, recorded *stopped*), resumed with an instruction, completed. No plan or reports were written or asked for | started → Stopped (with reason and who acts) → Resumed → completion |
| `smoke/run-001` | Cancelled before the rehearsal | Run cancelled |

**Defect found and fixed** (agdevworld `2f0873c`). In the manual run, the entry that completed the last node produced two messages ("Done — …" and "Execution completed"), sent in alphabetical order. Now:

- the entry that completes the execution is announced once, as the completion;
- events of one entry keep their selection order.

The test was updated.

## Rollout on this host

| Item | State |
| --- | --- |
| Open attempts / queued jobs before cutover | none (checked) |
| Editor release | `scripts/wfe-release.sh deploy .local/agdev` → **`2f0873c`**, health-checked (service and `bin/wfe`); rollback target `6fe2b61` |
| autolab | `daeaada` (guide, wfnotify); `com.agdev.agautolab-wfexec` restarted |
| `.local/agdev/zulip.json` | enabled; `zulipUrl` `https://agstudio.local:8543`; `viewBase` null (the editor route is loopback-only, so messages say to answer on the execution host and name the `wfe run answer` command); `since` `2026-10-11T07:57:04Z` (no backfill); person `iwaag` |
| `agautolab/.local/wfexec.toml` | `[notify] enabled = true` (autolab bot; folders via the provisioner credential) |
| Zulip | folder **#29 `wf-rts-vs-bot`**; channels **#273 `wf-rts-vs-bot.build-game`**, **#274 `wf-rts-vs-bot.improve-game`** |
| Existing RTS run `improve-game/run-001` | `wfe run check`: usable; shown as "Documentation policy: none (created before p5; the authored graph)"; not announced in Zulip (no synthesized plan, reports or messages) |
| RTS `improve-game` | now `ag.workflow.v2`, `policy: default`, committed and pushed to Gitea **`5f2ae5d`** (details below). The deployed editor previews it: plan talk, 3 reports, final report |
| RTS `build-game` | left at v1: shown as an unsupported schema with the fix it needs. It has not been run since p4, and choosing its policy is the person's call |
| Docs | agdevworld README_DEV §"Plans, reports and Zulip coordination" (`316e202`); `pj-agdev/.local/devenv.md` host section; pj-agdev gitlinks `cfed399` |

The `improve-game` change in detail:

- `confirm-scope` is removed: the generated plan talk replaces it.
- Its instruction — read the request, agree the scope and completion conditions — is now a sentence in the intent: agreed in `plan.md`.
- `implement`, `verify` and `human-check` are unchanged.
- The file carries no approvals (it never did), so intent and definition show as draft.

## Evidence against plan §11

| # | Evidence | State |
| --- | --- | --- |
| 1 | Default: one plan, one report per node, one final; no collisions or recursion. Manual: none | tests (step 1) |
| 2 | Branches and joins wait for reports; completion waits for the final report; delegate refused | tests (step 2) |
| 3 | Preview equals the frozen graph; edits don't mutate runs; policy invalidates definition approval | tests and browser (steps 1–3) |
| 4 | Plan before consultation; revision means a new question; work waits for agreement | tests (step 2); real agent (rehearsal) |
| 5 | Reports exist and are nonempty before completion; work and reporting observable separately | tests (step 2), browser (step 3), rehearsal |
| 6 | Interrupted report resumes; failure fabricates nothing; stale attempt refused | tests (step 2, wfexec e2e) |
| 7 | Manual generates and prompts nothing; explicit requests still work | tests and rehearsal |
| 8 | Icons attached, selectable, visible under fit/zoom, distinguishable without color; drafts kept | browser (step 3/4) |
| 9 | Hierarchy, restart, legacy separation, distinct identities | tests (step 4), real Zulip (smoke, rehearsal, production provisioning) |
| 10 | Crash before queue, lost response, outage — no repeated work | tests (step 4), real Zulip reconciliation (smoke) |
| 11 | Links in both devdocs modes; honest "not yet published" and no remote link | tests (step 4); rehearsal links are commit-only (local bare remote) |
| 12 | **Real RTS run** with `braindump.md`, agreed `plan.md`, per-node reports and `report.md` in Git/Gitea, and a truthful Zulip topic | **pending — needs the person** |

## Remaining

1. **The human-led real RTS run (§11.12).**
   1. The person requests an improvement in agdevworld: project → Request a run → `improve-game`.
   2. They answer the plan question in the run view.
   3. They answer the `human-check` question.
   4. They review the result and, if satisfied, record the result decision explicitly.

   Its topic will be `wf-rts-vs-bot.improve-game › run-002`.
2. **Archive the rehearsal destination** `wf-p5smoke` (#28, #271, #272) after the person has seen it, if wanted.
3. **Optional:** approve the revised `improve-game` definition; decide on `build-game`.

## Next entry point

After the real run, verify the following, then write `report.md`:

- the published records from a fresh Gitea checkout;
- the Zulip topic's content against the record.

# project_migration p3 — final report: forge as a managed project

**Outcome.** forge is a workflow-editor project (`developer/pj-forge`, workspace `forge`). Both of its responsibilities were proven through managed runs the Developer decided on.

- **Serving requests.** Text and an image with an edit were delivered with durable references. A request for spoken voice lines was explained as unsupported, and its demand was recorded.
- **Extending capability.** The Developer agreed a modest speech capability in `develop-capability`. It was built, verified and catalogued. A new request then served the original demand partially, linked back to the record, with what is still missing stated.
- **Interruption.** A ComfyUI job interrupted mid-attempt was recovered by the system's own recovery path and collected once, without being submitted again.

Old agforge is retired. Its Nautobot record waits for one command by the Developer (below).

Step reports: [report1](report1.md) (boundary and dispositions), [report2](report2.md) (project and validation), [report3](report3.md) (registration and local context), [report4](report4.md) (runs and findings). Plan: [plan.md](plan.md).

| | State |
| --- | --- |
| pj-forge | `developer/pj-forge` `ebe068e` (main), every run integrated |
| agdevworld (editor) | `75cdae7`, unchanged: no system change was needed |
| agforge | `ba5a7c3`, retired. Services stopped, checkout kept. |
| pj-agdev | pointer `agforge` → `ba5a7c3` |
| Agent cost | about $12 for step 4 (budget $20) |

## What was done

1. **Boundary** (step 1).
   - Old forge had no open requests: all 59 topics were resolved, and only a bookkeeping topic stayed open.
   - No external job was pending, and the delivered objects are durable in MinIO.
   - Its HTTP service had no caller. Its Zulip route was named only in Front's and autolab's guides.
   - All generation routes answered when checked directly.
2. **Project** (step 2).
   - Workflows `provide`, `develop-capability`, `refresh-catalog` and `freework`.
   - Guides for `forge-agent`.
   - A catalog of 10 capabilities that keeps sources (`<source>@<rev>:<path>`) apart from proof here.
   - A demand-record format: one file per originating run.
   - `tools/forge`, extracted from agforge `26dc1df` and mediagen-localize `31c0ba1`. It provides images, edits, durable delivery by content hash, ComfyUI jobs recorded before queueing, and the local source map.
   - Validated with the installed release in a scratch registry, including a hand-walked unsupported path and a live smoke test.
3. **Registration** (step 3).
   - Gitea, `wfe project register` and `obtain`; channels `wf-forge.*` were provisioned by the system.
   - Ignored local context in the workspace (`forge.env`, `sources.toml`, `comfy/`, `localenv.md`), checked from a real run-copy location with the executor's environment.
4. **Runs** (step 4): see report4.

| Plan item | Run | Result |
| --- | --- | --- |
| Supported: text | `provide/run-001` | delivered, accepted |
| Supported: image and an edit | `provide/run-002` | delivered (weak drawn bite), accepted |
| Unsupported | `provide/run-003` | explanation and `requests/provide-run-003.md`, accepted |
| Capability extension | `develop-capability/run-001` | `forge speech say`, catalog `speech` verified, accepted |
| Subsequent provision | `provide/run-005` (predecessor run-003) | partial (format met; acted delivery not), linked, accepted |
| Interruption boundary | `provide/run-006` + `agops/recover/run-002` | same job collected after SIGKILL and recovery; delivered, accepted |

5. **Settling the old forge** (step 5).
   - `com.agdev.agforge` and `com.agdev.agforge-zulip` were booted out, and their plists archived.
   - A retirement introduction was posted in `#agents` (#16181), pointing at `wf-forge.*`.
   - agforge's README now says where its work went.
   - `README_DEV.md` has a forge section, and the agforge section is marked retired.
   - The host notes (`pj-agdev/.local/devenv.md`) were updated.
   - The Nautobot batch was prepared and previewed. Applying it was refused to the Omni Agent, so it is left to the Developer.

## Reused and deleted

| Kept and reused | Retired (not carried over) |
| --- | --- |
| SwarmUI generation and image-to-image, transforms, presign/existence check (as content-addressed delivery), ComfyUI submit/history/fetch, the video and music presets, the FLUX.2 node settings | The Zulip listener, the role runner, anchors and selfnote records, `assetplan`/`assetrun` topics, retirement, introductions (except the final one), the entrance, `argue`, the HTTP service with `/api/resign` |

The old conversations, run workspaces, transcripts and the MinIO objects stay in place. forge's `tools/forge asset url <key>` signs old keys from the same bucket.

## Demonstrated limitations

- **SDXL image-to-image cannot add a bite.** It keeps the fruit whole or slices it. The drawn substitute reads poorly at 128 px.
- **System speech is flat.** Acted ("calm, gentle") narration is not covered. A neural model or a paid voice is a proposal (`develop-capability/run-001/research.md`).
- **Sound effects** are agsfx's. forge can only point there.
- **Background removal** has no model, so it is planned.

## Manual operating procedure (this phase)

1. **Request.** Use a `request-…` topic in `wf-forge.provide` (or another `wf-forge.*` channel), or *Request a run* in the editor. The Omni Agent requests with `.local/agdev/bin/wfe request <workflow> --braindump <file> --author "Omni Agent (for Developer)" --workspace forge`.
2. **Answer** plan and result questions in the run's topic or the run view. A result accepted with "ok" is recorded by the agent from that answer. Do not also press *Accept result*, which records a second acceptance (report4, finding 3).
3. **Results** are on the review page linked in the question, plus `deliverables/` and `deliveries.json` in the run folder. For a fresh link: `cd pj-forge && tools/forge asset url <key>`.
4. **An interrupted run with a ComfyUI job** is recovered by Observer, or resumed by the person. The next attempt reads `jobs/<name>.json` and collects (`tools/forge job status|collect`). Nothing is submitted again.
5. **A request that needs another run's result** is made after that run is integrated. A run's copy starts from `main` at request time (report4, editor finding 4).
6. **Host settings** live in `pj-forge/.local/`. A new workspace needs them copied before its first run.

## Remaining old dependencies

- **Nautobot.** To finish the retirement, the Developer runs: `cd pj-clusterintent && uv run --project nctl nctl desired apply -f ../pj-agdev/.local/project-migration-p3/retire-agforge.yaml --yes`. It makes 3 updates (service and agent `retired`, placement `disabled`), and the preview showed no conflicts. Until then, `nctl drift` will report the agforge placement as missing once the next node dump is collected.
- **comfynotify** runs from agforge's `.venv` and reads `AGFORGE_COMFYUI_URL`. The checkout is kept for it.
- **Front's guide** (`agfront/agent/guides/shared/requests.md`) still names forge as a destination, and autolab's supercoder guide mentions asking forge for images. The retirement introduction tells any reader where requests go now. Re-pointing them is Front's episode.
- **History readers.** The agentroom relay (`forge.py`, `AGENTROOM_AGENT_ROOTS`) and agdevworld's forge views read history and were left as they are.

## Next-episode requirements

**(a) Delegate and parent/child results.**
- A `provide` run met by a sound-effect request should be able to run agsfx's `create`, and own its result and acceptance.
- The same holds for a request that needs a capability another project serves.

**(b) Shared cluster context.**
- forge's source map names autolab's checkouts, whose revisions move with autolab's missions.
- The ComfyUI and SwarmUI endpoints, the bucket credentials and the workflow exports are copied by hand into each project's `.local/` (forge, agsfx, localize).
- A new workspace starts without them.

**(c) Front routing and operations.**
- Route asset requests to `wf-forge.provide`.
- Retire forge from Front's guide and roster.
- Have the Omni Agent (a bot) request through Zulip, or keep the CLI path documented.
- Decide whether results should come back to the requester's own conversation.

**Also for the workflow-editor episodes** (report4; the Developer's `p3/ex1` braindump covers 1 and 3):
1. "needs agreement" offers no control, and re-acceptance is hidden.
2. `integrate --status` shows a stale row.
3. A double acceptance is possible after a confirmation answer.
4. The copy revision at request time is not shown.
5. Workflows of agsfx, ppt and rts-vs-bot have documentation steps after `confirms: result`, which p9 integration will stop.

**For forge itself** (next runs):
- `refresh-catalog`: record run-001's text proof, run-002's image-to-image limit and the speech proof.
- Guide facts: a listening verdict is the person's; ask again when the agreed route changes; presentations come from the run folder; demand state follows the outcome.
- Add an `--out` alias to `job collect`.
- Keep the presentation URL out of question texts.

## Deus Ex Machina note

The Omni Agent made no deliverable. It requested the runs, interrupted one run on purpose, cancelled one mistaken request, and fixed forge's workflow definitions after the integration stop. Agent work was the agents'. The plan's handoff-candidate note does not apply.

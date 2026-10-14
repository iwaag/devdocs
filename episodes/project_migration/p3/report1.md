# project_migration p3 — step 1: the migration boundary

Written 2026-10-14. This step read the current forge code, guides and knowledge configuration and the installed workflow-editor contract. It also checked the real state of the services and inventoried forge's old work. Host details are in the ignored step notes (`pj-agdev/.local/project-migration-p3/step1.md`).

## State found

| Item | State |
| --- | --- |
| Cluster (nctl) | `nctl status` ok. Drift: ComfyUI, SwarmUI and MinIO are converged. agforge's placement on this host is active. The only drift is agobserver's retired listener pattern (observer p3), which is not this episode's. |
| Generation routes, checked directly | ComfyUI on the GPU node answers, and its queue is empty. SwarmUI opens a session. MinIO answers its health check. |
| agforge repository | `26dc1df`, clean, equal to origin |
| Knowledge sources | mediagen `main` `c415b0c` and `localize` `31c0ba1`, both equal to their remotes. localize has two verified capabilities: `hud_icons` and `flux2_scenery`. |
| Installed editor | agdevworld `75cdae7`. The contract docs and the `wfrun` guide were read. |
| Old forge services | `com.agdev.agforge` (HTTP, healthz ok) and `com.agdev.agforge-zulip`, both running and idle. The last forge run was 2026-09-24. |

## Old work and its disposition (proposed)

| What | Found | Proposed disposition |
| --- | --- | --- |
| Open requests | 59 forge topics. Every `assetplan-`/`assetrun-` topic is resolved (✔). The only open topic is `relations`, a bookkeeping note (failsafe p6 ex1). | Nothing finishes in place, and nothing is re-requested. All of it is historical. |
| External jobs | No `watching.json` in any old run workspace, and the ComfyUI queue is empty | None outstanding |
| Delivered objects | MinIO bucket `agforge` (keys `images/…`, `files/…`), recorded in the old conversations' `[selfnote][result]` notes | Kept. The new project signs links against the same bucket, so an old key stays retrievable. |
| Old run workspaces, transcripts, Zulip conversations | `agforge/.local/agentws`, `.local/agent`, `#agforge-agstudio1` | Kept as history. Not reinterpreted as new runs. |
| HTTP request service (`:8092`) | No caller since `modernize_agdevworld` p1. Its `/api/resign` is replaced by the new delivery tool (same bucket). | Retire in step 5 |
| Zulip listener (`#agforge-agstudio1`) | Callers: Front's request guide names forge as a destination, and autolab's supercoder guide mentions asking forge for images. Both reach it by Zulip. | Explicitly discontinue in step 5, with a final introduction saying where asset requests go now. Front's routing to the new project belongs to the Front episode and is recorded as that dependency. |
| agentroom relay `forge.py` and agdevworld's forge views | They read forge's conversation records (history) | Left as they are. They read history and have nothing new to read. Their removal belongs to the Front/agdevworld cleanup. |
| comfynotify | Runs from agforge's venv (`PATH`) and reads `AGFORGE_COMFYUI_URL` | The agforge checkout stays (it is not deleted), so the notifier keeps working. This is recorded as a remaining dependency. |
| Nautobot | Placement `agforge-agstudio` (process pattern) and the agent registration `agforge` | Update with the retirement in step 5 and verify with nctl before and after |

## Reuse decisions

The generation modules depend only on `requests`, `boto3` and `pillow`, not on pyagag or the listener. They are extracted into the new project's `tools/` with their history noted (provenance: agforge `26dc1df`):

| Reused (adapted) | As |
| --- | --- |
| `generate.py` (SwarmUI text-to-image and image-to-image, MinIO upload, presign) | image generation and edit |
| `transform.py` (resize and convert) | asset edit |
| `generate.py` `presign` / `object_exists`, `request_service` `/api/resign` | a delivery tool: upload, record key and hash, renew a link |
| `comfy_async.py` / `comfy_video.py` / `comfy_music.py` (submit, history, fetch) | external-job submit/inspect/collect with a saved job record (the interruption boundary) |
| `agent/toolsets/*.md`, mediagen/localize INDEX rows | catalog entries with source references |

Not carried over: the Zulip listener, `role_run`, `agent_run`, anchors and records, `assetplan`/`assetrun` topics, retirement, introductions, the entrance, `argue` and the HTTP service. The editor's runs, questions, acceptance and intake replace them.

Configuration moves from agforge's `.local/.env` to the forge workspace's ignored `.local/` (an `env` file and `sources.toml`). Tools find the workspace's `.local/` from a run copy, because a copy lives under `<workspace>/.local/wfe/runs/…`, or through an explicit variable. The run brief already names the project workspace.

Knowledge sources for the first catalog:

- mediagen `main` (general findings per model)
- mediagen `localize` (`hud_icons`, `flux2_scenery`)
- agforge's toolset documents, for image, video and music routes and the empty speech toolset
- the agsfx project, as the authority for sound effects, indexed but not copied

## Trial cases (for step 4)

| # | Case | Route | Why |
| --- | --- | --- | --- |
| 1 | Text asset (short game text in a stated tone and length) | the agent itself | Supported without external services |
| 2 | Image, then an edit of the delivered image (or of a supplied reference) | SwarmUI text-to-image, then image-to-image or a Pillow edit; MinIO delivery | Inexpensive (seconds of GPU), already verified in old forge |
| 3 | Unsupported: spoken voice lines | none | The catalog has no speech capability; forge's speech toolset was always empty |
| 4 | Capability extension (to agree with the person) | Proposed: a local speech procedure (macOS `say` voices, including Japanese, with WAV/M4A conversion and loudness checks). Alternative: transparent-background cutouts by a chroma-key procedure, since ComfyUI has no background-removal model. | Modest, local and testable, with no new infrastructure |
| 5 | Subsequent provision | Request the voice lines again through the new capability, linked to the demand record | Shows the improvement and what is still unsupported (for example, expressive character acting) |
| 6 | Interruption boundary | ComfyUI job via the new submit/collect tool (flux2 scenery image, about 40 s, or a short video) | A saved prompt id is inspected and collected after an interrupted attempt, without resubmitting |

**Budget.** The trial uses the `sonnet` profile with about 7 runs. Agent cost is capped at about $20 for the whole step. GPU time is a few minutes. Required resources are ComfyUI and SwarmUI on the GPU node, MinIO on this host, the local speech tools (`say`, `afconvert`, `ffmpeg`), the editor and executor, and the Zulip intake.

## Contract gaps

None blocks the trial. Each one is handled with local files or tools:

1. **Outcome kind.** A result decision is the same `accepted` record whether an asset or an explanation was presented. The project records the kind (`delivered`, `unsupported`, `partial`) in the run's outcome file, which the confirming question presents as its subject. So accepting an explanation is a decision on the explanation file, not on an asset.
2. **Local context from a run copy.** The brief names the project workspace. Tools resolve its `.local/` by the copy's location, or by a variable, so no system change is needed.
3. **External work beyond an attempt.** The existing `wfe run wait --external … --deadline` and a documented operator collect plus resume cover it. No scheduler or callback is added.
4. **An interrupted attempt invites Observer's recovery** (`agops/recover`). The interruption case must account for it, either by letting recovery collect the job or by resuming by hand before recovery acts. This is decided in step 4.
5. **The Omni Agent's Zulip account is a bot.** Omni requests use `wfe request` (known since p1).

Follow-up requirements so far, not implemented here:

- **(a) Delegate:** none yet.
- **(b) Shared context:** the source map is a hand-kept local file per workspace, and the ComfyUI endpoint is repeated in each consumer's local notes.
- **(c) Front:** Front still names forge as a destination, and old forge's discontinuation needs Front to route asset requests to `wf-forge.*`.

## Next

Step 2 builds `pj-forge` in an isolated migration checkout under `.local/agdev/migration/` and validates it with the installed release in a scratch registry.

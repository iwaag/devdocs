# project_migration p3 — step 2: the project, validated

Written 2026-10-14. `pj-forge` (project id `forge`) was built in an isolated migration checkout (`.local/agdev/migration/pj-forge`, two commits, nothing pushed yet). It was validated with the installed release (agdevworld `75cdae7`) in a scratch registry. No system code changed, and no delegate or shared-context infrastructure was added.

## What the project holds

| Path | Content |
| --- | --- |
| `project.yaml` | `ag.project.v2`, devdocs directory. Three local areas: `catalog`, `requests`, `tools`. |
| `devdocs/workflows/` | `provide`, `develop-capability`, `refresh-catalog`, and `freework` (placed by `wfe workflow basics` from the others' settings) |
| `agent/guide.md` + one guide per workflow | `forge-agent`: head, then facts. Tool usage stays in `--help`. |
| `catalog/` | Format and states (`README.md`), `INDEX.md`, and 10 entries |
| `requests/README.md` | The demand-record format: one file per originating run |
| `tools/` | The `forgetools` package and the `tools/forge` launcher, with 12 tests |
| `README.md`, `devdocs/README.md` | What forge is, how to ask, provenance |

### Workflows

| Workflow | Documentation | Nodes | Bindings (beyond the run folder) |
| --- | --- | --- | --- |
| `provide` | plan + final report | make → present → confirm (`confirms: result`) → cleanup | catalog, tools readonly; requests editable |
| `develop-capability` | default (plan, a report per node, final) | research → implement → verify → record → present → confirm → cleanup | catalog, tools, requests editable |
| `refresh-catalog` | plan + final report | read → update → confirm | catalog editable |
| `freework` | final report | talk | catalog, requests, tools editable |

- Every workflow runs as `forge-agent` on the `sonnet` profile and accepts Zulip requests.
- **The plan step is the assessment in `provide`.** `plan.md` proposes the outcome as *delivered*, *partial* or *unsupported*, and the person agrees before anything is made. `provide` never researches or builds a missing capability; it records the demand instead.
- **Outcome kind.** `outcome.md` begins with `Outcome: <kind>`, and the confirming question presents it (with the delivered files or the demand record) as its subject. Accepting an explanation is therefore a decision on `outcome.md`, and nothing in the record suggests an asset was delivered.

### Catalog seed

| State | Entries |
| --- | --- |
| `verified` (proof dated 2026-10-14, the migration smoke below) | `image-swarmui`, `image-edit`, `flux2-scenery` |
| `unverified` (method and route known; proof only historical, or none) | `text`, `hud-icons`, `video-minimax`, `music-acestep` |
| `elsewhere` | `sound-effects` (the agsfx project is the authority) |
| `planned` | `speech`, `background-removal` |

Every entry cites its sources as `<source>@<revision>:<path>`, using mediagen `c415b0c`, localize `31c0ba1`, agforge `26dc1df`, agsfx `e8d82d5` and protoprey-refs `6e017fb`. Each entry keeps *Sources* (knowledge) apart from *Proof here* (dated checks in this environment). Historical proofs are labelled historical, and the migration smoke is labelled as the Omni Agent's, not as an agent run.

### Tools (`tools/forge --help`)

| Command | From | Notes |
| --- | --- | --- |
| `local`, `sources` | new | Find the project workspace's `.local/` from a run copy's location (`<ws>/.local/wfe/runs/…`) or `FORGE_LOCAL`. List the knowledge sources with path and revision. Values are never printed. |
| `image generate`, `image transform` | agforge `generate.py`, `transform.py` | Writes to a given folder and prints the conditions (model, size, steps, seed, the reference's hash). Upload is separate. |
| `asset put`, `asset url`, `asset check` | agforge `generate.py` and `/api/resign` | Content-addressed keys `forge/<h2>/<sha256><ext>` in the existing bucket. The same bytes never upload twice or get recorded twice. A link is refused for a missing key. `check` re-hashes the stored bytes. |
| `job presets`, `job submit`, `job status`, `job collect` | agforge `comfy_*`, localize `flux2_scenery` | The record (with a prompt id chosen by the tool) is written before the job is queued. A second submit on a record is refused. A record left `submitting` is queued only if the server never got it, and with the same id. `collect` downloads once, and `--wait N` is a bounded foreground wait. |

Records hold relative paths only. The first test run caught an absolute path in a job record, which was fixed before the commit.

## Validation

| Check | Result |
| --- | --- |
| `wfe validate` (installed release, scratch registry) | 4 of 4 workflows valid, 0 warnings. Structure complete, and the three local areas are recognized. |
| Bindings (`wfe run access` on a scratch run) | catalog and tools readonly, requests editable, `outcome.md` in the run's own record folder, all as declared |
| Unsupported result path (scratch clone, recorded by hand) | `_plan` agreed (q1), then make writes `outcome.md` and `requests/provide-run-001.md`, then confirm asks with both as `--subject`, the answer is taken up and `decide --question q2` records acceptance. `_final-report` completes. `run check`: usable. Execution `completed`, and the decision is bound to the two files by hash. |
| Local-context lookup from a run-copy path | From `<clone>/.local/wfe/runs/smoke/run-000/tree/…` the tools resolved the clone's `.local/` and read `forge.env`, `sources.toml` and `comfy/` |
| Live smoke from that path (Omni Agent) | SwarmUI text-to-image, 768², about 25 s. Image-to-image changed the apple from red to gold and kept the composition. Transform to 256² PNG. `asset put` uploaded twice and recorded once, `asset url` fetched HTTP 200 with the right byte count, `asset check` matched hashes, and a missing key was refused. FLUX.2 job: the server kept the tool's prompt id, a second submit was refused, the first `collect` reported "running", `collect --wait 400` downloaded one PNG (3 min 53 s, model cold), and a third collect downloaded nothing. ComfyUI's history has the job once. |
| Tests | forge tools: 12 passed (local lookup, idempotent delivery, the interruption cases, the FLUX.2 preset). Reused agforge behaviour, as a baseline: 38 passed (`test_transform`, `test_comfy_async`, `test_init_image`, `test_comfy_video`, `test_comfy_music`). |
| Host details in tracked files | None found (grep for addresses, host names and absolute paths). One channel name with a host label was reworded. |

The supported path's records are the same as the walked unsupported path, apart from the subject files. The real supported runs are step 4's.

## Local context (ignored, migration clone only)

`.local/forge.env` was derived from agforge's settings with the keys renamed, mode 0600, and its values never printed. `.local/sources.toml` names five sources. `.local/comfy/` holds three workflow exports: FLUX.2 from localize, plus video and music from agforge's ignored resources. Step 3 prepares the same files in the obtained workspace, together with `localenv.md`.

## Notes for later steps

- A cold FLUX.2 job took about 4 minutes against 40 s warm. The catalog says so, and the interruption trial needs a `--wait` that covers it.
- `freework` placed by `basics` binds catalog, requests and tools editable (the most permissive of the other workflows) and names only `agent/guide.md`, as the contract specifies.

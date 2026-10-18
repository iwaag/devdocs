# p14 Step 4 — cagent publishes, a consumer receives

## Order of work

cagent publishing "through the supported interface" needs the block API on
the installed release, so the release was deployed first (agdevworld
`de014de`, Step 5's procedure: backup, `wfe-release.sh deploy`, 401/401
check, service and launcher healthy). The live p13 store converted on the
first read: the Developer's system instruction is the `developer` block
(manager Developer, last editor iwaag, all four revisions kept); the two
cleared agops overrides are removed blocks with their history.

## The entrance

pj-cagent (pushed to its Gitea main):

- workflow `context` (*Context refresh*, one `do` node `publish`, profile
  `luna`, channel `wf-cagent.context`): for the targets a request names,
  read what bears on them and cagent's current block, then publish, check
  or record a failed refresh; `published.md` names what was done;
- guide `agent/context.md` (facts, not procedure): who reads the blocks
  and how short the system summary is; cagent's block ids
  (`cagent-cluster`, `cagent-resources`), kind, manager, consultation;
  human edits are kept unless contradicted; the three times; what evidence
  supports (hardware ≠ free capacity, desired ≠ application test, model
  counts and health need their own observation); failed and partial
  refreshes; details instead of prompts; files outside the run copy; no
  scan required;
- `agent/guide.md` lists `wfe context`; `project.yaml` intent and a goal;
  README.

Tool syntax stays in `wfe context --help`; project intent comes from
`wfe projects show`.

## Trials (cagent on autolab/codex, gpt-6-luna)

| Run | Asked | What cagent did | Delivered fingerprint (forge freework) |
| --- | --- | --- | --- |
| context/run-001 | first system summary and forge contribution, no scan required | published `cagent-cluster` (5 body lines, `resources` detail, evidence: status/actual/drift reads with collection time 07:08Z partial; earlier scan) and forge `cagent-resources` (ComfyUI FLUX.2 and SwarmUI SDXL proofs on agpc, GPU from collection; model inventory, endpoint health, free capacity **unknown**; `generation` detail); no scan | `4d687024…` |
| context/run-002 | unchanged refresh | `check` on both blocks (bookkeeping, noted "does not claim a fresh observation") | `4d687024…` — unchanged |
| context/run-003 | refresh after the Omni Agent's `nctl observe` (agbach unavailable) | republished both with observation 12:26Z, agbach partial; unknowns kept | `3389cc4f…` — changed |
| context/run-004 | probe ComfyUI and SwarmUI over HTTP, try agbach | ComfyUI answered (24 checkpoints); SwarmUI API failed; agbach unavailable → kept content, recorded a **partial refresh** (delivered with its day) | `353a0dbb…` — changed (refresh line) |
| context/run-005 | publish what run-004 established | ComfyUI 200 + 24 checkpoints as observed 12:30Z, partial; SwarmUI API and agbach unknown with reasons; the refresh record cleared by the publication with evidence | changed |

Evidence-driven guidance: run-004 discarded a successful probe because
other probes failed. `agent/context.md` now says a partial refresh
publishes what it established (with the trial named); run-005 shows the
intended result — the braindump's "ComfyUI has N models; last connectivity
check at …".

Observed properties:

- old evidence never got a new time by republication; probe and
  collection times are the sources' own;
- a check changed no fingerprint; a failed/partial refresh kept the content
  and changed delivery; no empty replacement;
- cagent was recorded as manager and last editor with its runs
  (`cagent/context/run-00N`, from `WFE_ATTEMPT`); the producing run is in
  every revision;
- the blocks' content and details stayed in the area's store;
  `published.md` in pj-cagent (a private repository whose records already
  name hosts) summarises outcomes. Details carry the per-host facts.

## Consumer trial

forge `freework/run-002` (Claude Code, sonnet), asked where it would run
a small icon-generation job, without generating or asking anyone:

- its attempt snapshot (`wfe context run freework/run-002`) holds
  `system/block/developer`, `system/block/cagent-cluster` and
  `project/block/cagent-resources` (section hash `037971cd…`, revision 6),
  prompt delivered;
- its attempt log shows `wfe context detail
  context:project/forge/cagent-resources/generation`;
- its answer: agpc, SwarmUI SDXL for icons (FLUX.2 route is scenery-only),
  "ComfyUI answered 12:30Z (cagent run-004 probe) … SwarmUI API may be
  unhealthy", 24 checkpoints with age, GPU shared and free capacity
  unknown, sources named (catalog, local env, the context detail, the
  cagent summary), missing facts → ask cagent via `cagent/consult`.

The consumer identified the relevant resources from what it was given,
read the detail on demand, and named the consultation route; nobody was
asked first.

## Deus ex machina notes

- Omni ran `nctl observe` before run-003 to provide a fresh observation —
  handoff candidate (a scan-triggered refresh is the "after first
  delivery" trigger).
- Omni answered the consumer's confirmation as "Omni Agent (for Developer)".

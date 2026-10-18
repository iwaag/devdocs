# p14 ex1 Step 3 — cagent's overviews, with details behind them

## Before

| Block | Revision | What made it report-like |
| --- | --- | --- |
| system `cagent-cluster` | 4 (context/run-003) | a line of collection times and per-host ages ("six ordinary hosts collected at 12:26 UTC, agbach ~17 h old"), a line naming `cagentctl read status`, `actual` and `drift`, and a line repeating the consultation route the card already showed |
| forge `cagent-resources` | 6 (context/run-005) | the probe's run id, HTTP status and checkpoint count, the collection's time and install paths, which checks were not done, the failed SwarmUI/agbach probes, and "see the resource detail for evidence, methods and limits" |

Both details (`resources`, `generation`) already held the inventories,
methods and coverage; evidence held each observation with its own time.

## Guide (pj-cagent `agent/context.md`)

- `114208b`: **An overview, with details behind it** — a body says which
  resources are relevant and what they are good for, the uncertainty or
  constraint needed before acting, and where to read more; the reader
  already shows details, consultation, evidence and history, so the body
  does not repeat them or name commands; chronology, methods, status
  codes, per-host coverage, inventories, collection times and provenance
  go into details and evidence; length is judged by reading effort, not
  lines; brevity never makes a qualified observation unconditional. The
  failure reference is the Developer's screenshot of forge's block. The
  fixed "three to five lines" was replaced by the reading-effort measure.
- `d98e6bd` (evidence from run-006, below): brevity never drops a
  difference that changes which resource a reader picks — what was last
  seen working and what was last seen failing stay in the overview, each
  with its age.

Tool syntax stays in `wfe context --help`; no procedure was added.

## cagent's refreshes (autolab, `luna`)

| Run | Asked | Result |
| --- | --- | --- |
| context/run-006 | rewrite both as overviews, keep evidence, times and unknowns, no scan | system rev 5 and forge rev 7: short, evidence and details unchanged, no scan. **But** forge's overview dropped that ComfyUI answered at the last probe while SwarmUI's API failed: both routes read as equally unverified ("current endpoint health, model inventory and usable capacity remain unknown") |
| context/run-007 (predecessor run-006) | keep that difference, same evidence, no probe | forge rev 8: the two routes, then "In the 2026-10-18 probe, ComfyUI API returned 200 and listed 24 checkpoints, while SwarmUI API requests failed (root answered). These are probe-time results, not availability guarantees; current health, checkpoint inventory and usable capacity remain unknown."; system rev 6–7: the same distinction in one sentence, and the existing 12:30:11Z probe evidence added to the system block with its original time |

No new observation time was manufactured; no scan or probe ran. Both
bodies are one paragraph of short sentences (system 1564 bytes and forge
1744 bytes delivered, from 1297 and 1847). The forge overview keeps a
status code ("returned 200"), which is at the edge of the guide's
"methods belong in the details"; it was left as cagent's judgement rather
than rewritten by the Omni Agent.

## Details and access

- `generation` (2288 bytes) still holds both routes' capabilities and
  limits, the 12:26Z collection, the 07:08Z GPU facts, the run-004 probe
  results with their reading, and the unknowns; `resources` (3318 bytes)
  is unchanged.
- Read locally (`wfe context detail`) and from agautolab1 through the LAN
  route (`api/context/detail?…&raw=1`: 200, 2288 bytes; the cluster
  detail's heading) — log in the ignored `workflow-editor-p14ex1/step3/`.
  The UI link is checked on the installed release in Step 4.

## Consumer trial

forge `freework/run-003` (asked for 256 px app icons from a reference
sketch; no generation, nobody asked, no confirmation):

- delivered context (`wfe context run freework/run-003`): snapshot
  `ctx-fbccea7c…`, the Developer's block, `cagent-cluster` (`2b5c2f5f…`)
  and `cagent-resources` (`235f213d…`), prompt delivered;
- answer: **SwarmUI on agpc** — the only catalogued route that takes a
  reference image (FLUX.2 via ComfyUI is text-to-image) — with
  generate-then-downscale; verified on 2026-10-14 (4 days); **not
  verified**: SwarmUI health today, quoting the overview's "API GETs
  returning an HTML error and POSTs HTTP 400 … may block the route, and
  the catalog's verified predates it", native 256 px quality, icon
  specifics, free GPU capacity, agbach; sources named (catalog files,
  local notes, both blocks); missing facts → cagent via `cagent/consult`.
- It did not open the details and said so: the overview was enough to
  choose and to see the blocking uncertainty. The decisive fact (SwarmUI's
  failed API) reached it only because run-007 restored it; with run-006's
  text the same answer would have recommended SwarmUI without the risk.

## Semantics

- forge `freework/run-002`'s snapshots still hold the old sections
  (`3004a284…`, `037971cd…`); its next attempt now differs by both cagent
  blocks — cagent's deliberate publications follow next-attempt semantics.
- UI disclosure (Steps 1–2) changed no body, delivery or fingerprint
  (`contextBlocks`: opening and closing a block leaves its revision and the
  next-attempt fingerprint unchanged).

## Revisions

pj-cagent `114208b`, `d98e6bd` (pushed); cagent runs context/run-006,
context/run-007; forge freework/run-003 (records in their run copies).
Rollback for content: the store's history keeps system revisions 4–7 and
forge 6–8.

# scheduled_routine p1 — Step 4 report: first routine, image prompt exploration

## Standing text as posted

`#front` › `routine-imgprompt`, message 1445 (quoted in full in report1).
Differences from the plan's suggestion: the forge spec (1024x1024 PNG, one
image per prompt) is stated up front, and Front is told not to wait for the
Developer during a routine run. Unchanged since; nothing so far says it
needs editing.

## Run 1 — manual trigger, 2026-08-23 15:46Z (messages 1446–1497)

Timeline (all UTC, from `zulip-listener.log` of Front and forge):

| time | what |
|---|---|
| 15:46:39 | trigger posted; Front served within the second |
| 15:47:20 | Front → forge `assetplan-autumn-forest-imgprompt`: theme "autumn forest" (no history to vary from), 4 prompts, full spec |
| 15:47:36 | forge front role wrote `required_items.md`; **it copied Front's closing sentence "Please register this as a Work; once you open the run topic I'll trigger it" into it as "Action requested"** |
| 15:48:02 | forge generator role declined: "no tool here can register a Work or open a run topic" (its guide never tells it that writing `plan.md` *is* the registration) |
| 15:48:22 | Front reported to the Developer and **asked**: try a plain question in forge's channel, or hold? |
| 15:49:xx | Developer answer (posted by the Omni agent on the Developer's behalf): open a fresh `assetplan-`, describe only the asset, never tell forge what to do with Works/run topics; and in general don't stop to ask during this routine |
| 15:50:21 | Front → `assetplan-autumn-forest-v2`, spec only |
| 15:50:55 | forge registered F2-23, opened `assetrun-autumn-forest-v2` |
| 15:51:12 | Front pressed the button |
| 15:51:53 | forge: run failed, `agpc.local:7801` (SwarmUI) connection refused |
| 15:52:30 | Front retried on its own, citing forge's intro ("re-triggering is a legitimate retry") and the standing text |
| 15:53:16 | failed again, same cause; Front stopped retrying and reported |
| 16:00:03 | **scheduled fire** lands in the same topic; Front reads the stuck run and retries F2-23 instead of picking a new theme (see report3) |
| ~16:00 | Developer started SwarmUI (and ComfyUI) on agpc by hand |
| 16:03:42 | forge delivered: 4 files, one zip, `[S3KEY] files/2026-08-24/e425ba8e16f748118c12a012c2955e02.zip` |
| 16:04:01 | Front's final report in the run topic: URL, S3KEY, the four prompts, two-line take |

Front runs for this routine run: **7** (initial, 3 forge callbacks, Developer answer, scheduled fire, delivery callback). Forge runs: 2 plan + 3 run attempts.

Verified: the zip holds `autumn_01_dawn_wide.png`, `autumn_02_macro_leaves.png`,
`autumn_03_silhouette.png`, `autumn_04_aerial_canopy.png`, each
1024x1024 RGB PNG.

### What Front asked, what the Developer answered

One question (15:48:22), answered as above. The answer is the candidate
Evidence-Driven Guidance for the standing text: *"describe only the asset to
forge; it registers and opens the run topic itself"*. Not yet added — run 2
will show whether Front has learned it from the topic history, which is
the cheaper outcome.

### Findings

1. **forge-side, not routine-side**: forge's generator declines any
   `required_items.md` that contains an orchestration request, because its
   `guide_plan.md` does not say `plan.md` = Work registration. Its front
   role also leaks the requester's closing courtesy into `required_items.md`.
   Out of p1 scope; noted for an agforge episode. Also in forge's log:
   `unknown toolset 'toolset'; skipped` — the CSV header row is being read
   as a toolset name (harmless).
2. **Shape that emerged**: one `assetplan-` with 4 prompts → one Work → one
   `assetrun-` → one zip of 4 images. Front therefore reports **one URL and
   one S3KEY for the set**, not one per image as the standing text literally
   asks. Acceptable; the Developer judges per file name.
3. **Infra**: the SwarmUI backend on agpc was down (ComfyUI had been launched
   in its place on 2026-08-22). nctl reported `swarmui converged 0 diff(s)`
   while the port refused — reported to cagent by DM (message 1486) as an
   invalid cluster explanation, per README_DEV. Needed a human to start the
   service; nothing in the system can.
4. **Retry behaviour**: Front retried once on its own, stopped after two
   identical infra failures, and the next scheduled fire became the third
   retry. No loop, no stall.
5. **Tooling drift**: the `agentchat` installed in `agfront/.venv` has no
   `wait` subcommand although devenv.md describes one — the venv is behind
   the pyagag pin described there. Not needed for the routine itself.

## Run 2 — scheduled fire 17:00Z

(to be appended)

## Run 3 — scheduled fire 18:00Z

(to be appended)

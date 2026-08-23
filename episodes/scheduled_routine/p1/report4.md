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

## Schedule change during the step

After run 1 the Developer asked for faster observation than an hourly
fire; the launchd job was `bootout`-ed (16:41Z) and runs 2 and 3 were fired
by hand with `trigger.sh imgprompt`. The launchd path was already verified
by the 16:00Z fire (report3); the plist stays in the repo for re-enabling.

## Developer judgement on run 1 (message 1499, 16:31Z)

> どれも良い出来で90点とします。次のルーチン実行では海岸の風景をテーマとしてください。

Finding: a Developer comment in the run topic is itself a `front-` post, so
**Front served it** (message 1501: "feedback + standing instruction for the
next run — no action now, not contacting forge"). Harmless and correctly
judged, but every judgement costs one Front run. Nothing to change in p1.

## Run 2 — manual trigger, 16:41Z (messages 1502–1522)

| time | what |
|---|---|
| 16:41:40 | served; Front read the judgement, opened `assetplan-coastal-landscape` with the coastal theme, 4 prompts, spec only — no orchestration sentence |
| 16:41:59 | forge registered F2-24, opened `assetrun-coastal-landscape`, no questions |
| 16:42:31 | Front pressed the button (callback) |
| 16:45:25 | forge delivered; Front reported URL, `[S3KEY] files/2026-08-24/34b98ff4c3eb4e9e9742f36f7cbaf7b9.zip`, 4 prompts |

4 minutes end-to-end, **4 Front runs**, no question to the Developer.
Front applied the run-1 correction from topic history on its own: the
candidate guidance "describe only the asset" did **not** need to go into the
standing text.

## Run 3 — manual trigger, 16:46Z (messages 1524–1544)

No Developer comment between runs 2 and 3.

| time | what |
|---|---|
| 16:46:04 | served; Front chose **alpine lake** ("distinct from both prior runs"), `assetplan-alpine-lake` |
| 16:46:59 | forge registered F2-25, opened the run topic; Front triggered |
| 16:49:30 | delivered; `[S3KEY] files/2026-08-24/6a4f0d585dc74a7681c0cc6c4a26e7a8.zip` |

3.5 minutes, **3 Front runs** (the plan-registration and button press were
one callback this time).

### Finding: the theme varies, the prompts do not

Front wrote in run 3: "same structure: wide-angle painterly dawn shot, macro
photorealistic detail, minimalist silhouette, flat-vector aerial top-down."
All three runs use that identical four-slot template; only the subject
changes. The standing text says "clearly different prompts" per run and
"vary it from previous runs" about the *theme*, and Front satisfied both
literally. The Developer's single judgement ("90, all good") gave it no
reason to change the template either. If the routine is meant to explore
*prompting*, the standing text needs to say that the variation axis is the
prompt technique, not the subject — candidate Evidence-Driven Guidance for
the standing text (Step 5), not for Front.

Front's "two-line take" is also thin (it restates that the run went well);
with no negative judgement to react to, there is nothing for it to propose.

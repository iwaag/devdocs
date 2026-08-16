# Report 7 — End-to-end check and deploy

Status: **done, with one deployment step blocked by a pre-existing
environment gap** (see "Deploy"). The whole asset pipeline was driven live on
agstudio against real Zulip, Plane, gitea, MinIO, SwarmUI, and real agent
runs. `agautolab` 117 tests green, `agforge` 173 green.

## The live run

Fresh channel `pj-assetpipe1`, fresh Plane project `Assetpipe1` (`A`), on
2026-08-16 UTC. Participants: Omni Agent (9), Autolab Agstudio (11), Forge
(13).

```
14:29:05  autolab listener restarted; prefixes ('mission-','run-','create-')
14:29:51  mission-titlescreen swept
          init_project -> gitea main/direction/devlog clones
          direction/aesthetics.md committed and pushed  (be68409)
          front (local/ollama) -> new_mission.md
          superdirector (sonnet) in .local/projects/assetpipe1/
              -> plan.md + task1..task5.md; task3.md ends "[Asset]"
          A-1 created; A-2..A-6 sub-works; A-4 carries [asset]
          project folder cleared back to devlog/ direction/ main/
14:32:10  run- #1  -> A-2 (plain work) -> coding -> "work A-2: ... Done yes"
14:32:5x  A-3 -> Done by hand (Omni Agent; see the note below)
14:32:57  run- #2  -> A-4 is [asset], ledger "absent"
              -> order posted to pj-assetpipe1/create-asset_a099a8d4-...
              -> "asset ordered in ..."; no coding run; A-4 left unstarted
14:32:57  autolab swept its own order post and DECLINED:
              "the last message does not mention 'Autolab Agstudio'"
14:32:57  agforge swept the same topic -> front -> required_items.md
14:33:14  agforge generator -> plan.md -> A-7 registered in Assetpipe1,
              external_id pj-assetpipe1/create-asset_a099a8d4-..., src agforge
14:34:10  run- #3  -> ledger "working" -> "asset in progress in ..."
              no runcreate- emitted, no coding run
14:36:29  runcreate- fired BY HAND by the Omni Agent
14:39:xx  generator -> result/hero.png (78 588 bytes); delivery posted:
              "... temporary download (expires in 60 min): http://agstudio.local:9100/...
               [S3KEY] files/2026-08-16/dddf83da1c244c43b55e432545d82584.zip"
14:39:40  autolab declined the create- topic a third time (delivery has no mention)
14:40:09  run- #4  -> ledger "done" -> find_asset_key -> resign -> coding
              coding downloaded the zip and wrote main/assets/hero.png
              -> run aborted: turn_budget_exhausted (see "The one real failure")
14:43:36  run- #5  (coding on sonnet) -> resign -> coding
              "asset ready (files/2026-08-16/dddf83da...zip)"
              -> hero.png placed, report + success.flag
              -> "work A-4: commented yes, Done yes"
14:46:31  Forge asks, mentioning the bot -> gate opens -> superdirector
              -> answer.md posted -> Forge acks and resumes
```

### The delivered asset

`hero.png`, PNG 384×384 8-bit RGBA, transparent background: a pixel-art hero
with a sword, unmistakably "2D retro digital game art style". The stopgap
`direction/aesthetics.md` (Step 3) travelled through the superdirector's
task file, into autolab's order text, through agforge's front and generator,
into the SwarmUI prompt, and came back visible in the image. That chain is
the phase's thesis and it holds.

### Final Plane state

```
A-1  Build a Title Screen for a Browser Game  unstarted  [auto]          agautolab
A-2  Create the HTML Structure                completed  [auto]          agautolab
A-3  Design the CSS Layout                    completed  [auto]          agautolab   (by hand)
A-4  Hero Character Image Asset               completed  [asset, auto]   agautolab
A-5  Implement the START Button Behavior      unstarted  [auto]          agautolab
A-6  Test and Refine the Title Screen         unstarted  [auto]          agautolab
A-7  Plan: Hero Character Image Asset         completed  [forgeauto]     agforge
```

Two agents, two external sources, two label vocabularies, one project. The
`asset`/`AUTO` vs `FORGEAUTO` split kept each agent's work selection blind to
the other's throughout.

## What each step's live evidence was

| Step | Evidence |
|---|---|
| 1 | listener started clean on the renamed guides; every serving reached its agent |
| 2 | superdirector ran in the project folder; A-1's description **is** `plan.md` verbatim; `[Asset]` → the `asset` label; project folder cleared after registration |
| 3 | `direction/` git log: `be68409 Add aesthetics.md`, pushed to gitea |
| 4 | delivery and Plane comment both carry `[S3KEY] files/2026-08-16/dddf83da…zip`; `/api/resign` served the fresh URL two runs later |
| 5 | all three ledger states observed in order across four separate `run-` triggers; **no `runcreate-` ever emitted by autolab** |
| 6 | mention gate declined three times and opened once; `plan.md`/`task.md` recovered from Plane; answer posted; agforge resumed |

## Step 6's live check needed staging

agforge's generator decided it *could* plan the hero sprite, so it never left
`question.flag` and never asked. To exercise autolab's half against the real
listener, the question was posted **as the Forge bot** — which is exactly the
message autolab is built to react to, and which agforge itself ignores (its
sweep skips topics whose last poster is itself).

What that proved:

- `<1>/answer/` cut with `chatlog.md`, `plan.md`, `task.md`
- `plan.md` recovered **verbatim** from A-1's description, heading included
- `task.md` recomposed from A-4 **without** the `[Asset]` marker (`grep -c` = 0)
- the superdirector, running with cwd = the project folder, read
  `main/style.css` and answered:

  > The rendered hero sprite is **256px tall**. `main/style.css` sets
  > `.hero img { width: 256px; height: auto; }`, and `hero.png` is a square
  > 384×384 source image, so it scales down to 256×256 on screen.

  That answer is only possible because the run happens in the project rather
  than in the generation directory. It is the clearest justification for the
  cwd choice in Step 6.
- posting it made autolab the last non-forge poster and Forge immediately
  acked — the hand-back works, and the loop did not ping-pong.

### A race worth recording

A first attempt posted the mention as the **Omni Agent**. agforge acked within
seconds, so by the time autolab swept, the last message was Forge's ack and
the gate correctly stayed shut. The gate reads only the *last* message, so
**anything posted after agforge's mention hides that mention forever** —
autolab will not scan backwards for it. In the designed flow agforge's
mention *is* the last message, so this does not bite today. It is a real
fragility of a last-message-only gate and belongs on the ENT list, not in a
quiet fix.

## The one real failure

run- #4's coding run aborted:

```
role=coding profile=local harness=agcode provider=ollama
model=ollama/qwen3.6:35b-a3b-coding-nvfp4
num_turns=20  outcome=aborted
failure: agcode exited 2: turn_budget_exhausted: max_turns (20) exhausted
```

Everything under test worked — the ledger said `done`, the key was recovered
from the topic, `/api/resign` returned a live URL, and the agent downloaded
the zip and wrote `main/assets/hero.png`. It then ran out of turns before
writing `report.md`/`success.flag`, so `report_work` left A-4 `unstarted` and
the Work stayed selectable. **That is the retry path behaving exactly as
designed**, observed for free.

`.local/agents.local.toml` pins `coding` to the `local` ollama profile, and
this plan asks for real sonnet runs. The overlay was switched to `sonnet`,
`main/assets/` deleted, and the same Work re-triggered: 27 turns, 75 s,
0.51 USD, `outcome: done`, A-4 → Done. **The overlay has been restored to
`profile = "local"` and the listener reloaded**, so the developer's
deployment is exactly as it was found.

Failure-farming notes, agent behaviour rather than pipeline defects:

- A-2's coding run created `main/main/index.html` — one directory deeper than
  the task asked, because cwd is already the `main/` clone. A-4's run then
  consistently placed `main/main/assets/hero.png` beside it. Nothing in the
  flow noticed or cared.
- agforge's generator logged `unknown toolset 'toolset'; skipped` — the front's
  `toolsets.csv` had a stray line. Lenient parsing absorbed it, as designed.

## Deus Ex Machina notes

Per `devdocs/README_DEV.md`, the Omni Agent's interventions in this episode:

- **fired every `runcreate-` trigger by hand for agforge — handoff candidate.**
  This is the plan's deliberate design for this phase; autolab is explicitly
  forbidden from emitting `runcreate-` and a test asserts it never does. The
  one-liner, from any subscribed account:

  ```python
  ZulipClient.from_env(Path(".local/zulip/omni-agent.env")) \
      .send_to_channel("general", "runcreate-<anything>", "go")
  ```

- **moved A-3 to Done by hand for autolab — handoff candidate.** `next_work()`
  returns the oldest eligible Work, so reaching the asset task meant clearing
  the ordinary CSS task ahead of it. That task is not what this phase tests.
- **posted agforge's question as the Forge bot — handoff candidate.** See
  "Step 6's live check needed staging" above.
- **switched `coding` to the sonnet profile for one run — restored after.**

## Deploy

- `agautolab` pushed to GitHub, HEAD `fc6ae66`.
- `agforge` pushed to GitHub, HEAD `8806b58`; both launchd jobs kickstarted
  (`com.agdev.agforge`, `com.agdev.agforge-zulip`) and `/api/resign` probed
  live on `:8092`.
- Inventory re-rendered from Nautobot; both placements carry
  `autolab_node_repo_url: https://github.com/iwaag/agautolab.git`.
- **`--limit agautolab1`: succeeded** — `ok=21 changed=3 failed=0`, gateway
  restarted and health-probed. Verified on the node:
  `git -C /home/eiji/agautolab rev-parse --short HEAD` → `fc6ae66`, the same
  commit as local HEAD.
- **`--limit agstudio`: FAILED, and not because of this phase.** The
  `claude_code_agent` role asserts a user-scoped Node toolchain at
  `/Users/eiji/.local/node/bin/npm`; this Mac has npm 11.18.0 at
  `/opt/homebrew/bin/npm` and nothing under `~/.local/node`. The play stops at
  `ok=2 changed=0 failed=1` before touching anything autolab-related. This is a
  pre-existing baseline gap, and installing a user-scoped Node runtime on the
  developer's machine is outside this plan's scope — **left for the developer
  to decide.**

  In practice agstudio is already current: its "deployment" *is*
  `pj-agdev/agautolab`, the live working tree, at `fc6ae66`, and both launchd
  services (`com.agdev.agautolab-zulip`, `com.agdev.agautolab-gateway`) were
  reloaded onto that code and are what served this entire end-to-end run.

## Left as the plan left it

- The autolab Work blocks the queue while its asset is being made — every
  other Work waits. Accepted for this phase.
- The director check on a delivered asset is skipped; the coding prompt asks
  the coding run to compromise or fail instead.
- `runcreate-` is manual.

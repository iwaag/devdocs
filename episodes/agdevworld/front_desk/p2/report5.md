# Step 5 — integrated verification, deployment, and report

agfront `143908c`, `a446819`, `82c73ad`; pyagag `41cc5d5` (pushed);
agdevworld `1f8b785` (unchanged since step 4); agdevworld-settings
`4f3b55f` (pushed in step 1). Seven paid `character_talk` runs, $1.32.

## Tests and build

| suite | result |
|---|---|
| agdevworld `npm run build` | passes |
| agdevworld/agentroom `uv run pytest -q` | 169 passed (20 new this phase: settings sync/routes, dialogue split) |
| agfront `uv run pytest -q` | 61 passed (32 new: settings pinning, evidence with ids, dialogue block, preface rule) |
| pyagag `uv run pytest -q` | 465 passed (1 new: plain `read` follows ✔) |

Shared input/output examples: the same `ag.frontdesk-dialogue.v1` block
shape is pinned on both sides — agfront writes it (`test_dialogue.py`,
`test_zulip_listener.py`), the relay splits it (`test_frontdesk.py`) — and
the same synced-tree layout (`active.json`, `revisions/<sha>/manifest.toml`)
is written by agentroom's tests and read by agfront's.

## Deployment

Environment checked first: `nctl status` ok (Nautobot, worker, dumps),
`/ops` live with four instances, `/inflight` idle for front and autolab
before each restart. Then `docker compose up --build -d web` (`:8090`
200), `launchctl kickstart -k` on `com.agdev.agentroom` (step 4, `/settings`
served, live after the sweep) and on `com.agdev.agfront-zulip` (after steps
2, 3 and twice in this step: the pyagag upgrade and the preface rule).
No plist change anywhere.

## The live conversations

**Casual** — `front-desk-20260908-p2live`, 5281 → 5283 (`run-0019`,
$0.073): in character from the lore (姐さん, 親方 mentioned), revision
`4f3b55f6` stamped in the record, no dialogue block (small talk), as the
guide asks. It began with an English preface again.

**The bounded task** — 5284, *run ghtrends once and see it through*:

1. **5289** (`run-0020`, $0.35): Front read the threads it was given —
   both under `✔` names — found the morning's `work-g-17 ›
   workrun-task1-g-17` resolved but empty, and instead of starting anything
   asked autolab in its own channel (`autolab-agstudio1 ›
   status-ghtrends-g17`, 5287). Two-turn Front-only scene.
2. **5294** (`run-0021`, $0.17), the callback: three turns, Autolab's line
   citing `status-ghtrends-g17 #5290` — *G-17/G-18 never ran; start.flag
   alone does nothing; the earlier "done" report has no support; G-13 was
   the real completion*. Checked against 5290 as posted: faithful. Front
   then asked for a re-plan in `pj-ghtrends › workplan-trend8` (5292).
3. autolab re-planned (5299) and opened `work-g-17 ›
   workrun-rerun-task1-g-17`. **The callback went to
   `front-desk-20260908-164810`**, the conversation that had anchored
   `workplan-trend8` in the morning: a topic carries one root note and the
   earliest wins (`agent_standardize` p8). Not a p2 defect, but the
   developer at the p2live screen would not have seen the rest.
4. **5304** (`run-0022`, $0.25) in 164810: Front started the rerun (5302),
   scene with Autolab citing `workplan-trend8 #5299`. autolab ran the task
   and reported **5307** — `ayghri/i-have-adhd`, commit `fdf6d28`, pushed,
   `publish/` untouched — and resolved the topic in the same second.
5. **5310** (`run-0023`, $0.25): Front read the thread file the listener
   had placed (correct, read under the ✔ name, header saying so), then
   ran `agentchat read work-g-17 workrun-rerun-task1-g-17` itself, got *no
   messages*, and reported the completion as **fabricated** — for the
   second time that afternoon. Plain `agentchat read` only tried the bare
   name; `--since` and the listener followed the rename, and the tool's own
   docstring claimed both names were tried. Fixed in pyagag `41cc5d5`
   (`topic_messages`, one test), pushed, agfront's lock upgraded, listener
   kickstarted.
6. **5314** (`run-0024`, $0.14), after *fixed, read it again*: the
   completion scene — Front lower-left, `Autolab（親方）` upper-left saying
   *ayghri/i-have-adhd（トレンド#1位）を追加し、コミットfdf6d28でmainにpush済みだ。publish/は触っていない。*
   with the `📎 #work-g-17 › workrun-rerun-task1-g-17 #5307` chip and the
   API link chip; the readable reply carries the same facts.

Verified independently of Front's words: `agautolab/.local/projects/ghtrends/main`
HEAD `fdf6d28` "Add ayghri/i-have-adhd trending repo summary (2026-09-08,
#1 on GitHub trending)", `git ls-remote origin main` = `fdf6d28`; 5307 in
Zulip under `✔ workrun-rerun-task1-g-17` says the same; `main/index.md` has
the row. An ack and the "started" post were never reported as done.

**Settings change without a restart** — the fixture repository's Front lore
gained a catchphrase (「〜なのだ」) and its face was mirrored and greyed;
config pointed at it, `agentroom-settings sync` → active `1291050f` (the
relay served it at once). A casual post (5315) → 5317 (`run-0025`, $0.079):
`settings_revision` `1291050f…` in the record, `characters.md` carrying the
new line, and the reply using the catchphrase. The screen on `:8090` drew
the grey face for that plain reply. Config switched back, sync → `4f3b55f6`
active; the screen drew the original face again and
`front-desk-20260908-164810`'s scenes still read *scene at 4f3b55f654c8*
with their faces. No daemon restart, no frontend rebuild in any of it.

## Fixes made from what the live run showed

- pyagag `41cc5d5`: plain `agentchat read` follows the ✔ rename.
- agfront `143908c`: the guide says once more, at the end, not to begin
  with a sentence about the message. It leaked anyway in all three casual
  runs after that, so agfront `82c73ad` drops a leading paragraph that has
  no Japanese in it when a later one does, logs it, and pins it with a
  test — the smallest rule that removes exactly what was seen.
- agfront `a446819`: the pyagag pin.

## Remaining issues

- **Callbacks follow the topic's first anchor, not the latest requester.**
  A second Front Desk conversation asking about a mission another one
  opened gets its answers in the first one. Worth an `agent_standardize`
  note; nothing in p2 changes it.
- The preface rule is deterministic post-processing (a small shackle, by
  the terms doc); the guide alone did not hold on sonnet-5. The dropped
  text is logged per run so its frequency can be watched.
- agautolab's threads still use the shared renderer without ids, and the
  other agents' locks still hold pyagag `9c9e5a9`; their plain
  `agentchat read` has the same ✔ blind spot until upgraded.
- The `agautolab1` VM was not touched.
- Step 4's two cosmetic items (empty dimmed box on a collaborator-first
  scene, queue chip placement) stand.

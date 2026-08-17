# Plan — agforge music generation via ComfyUI (p1)

Goal: `agforge music generate --prompt "…"` generates music through ComfyUI
(same contract as `agforge video generate`), and the legacy ACE-Step 1.5
service path — the `music-gen` wrapper (workspace-root `music-gen/` project)
running on agpc plus its dependency service `ace-step` — is retired
everywhere: code, charter, desired/actual state, and — if agpc is reachable —
the on-node artifacts.

**Out of scope: the ACE Studio desktop-app integration stays.** The
`acestudio-cli` path (sung vocals), its `role_run.py` plumbing
(`ACE_STUDIO_ENV`, the `Bash(acestudio-cli:*)` grant), `.local/ace-studio.env`,
its charter/GUIDE sections, and its tests are all untouched.

Destructive phase. No backward compatibility required. Prohibitions are the
minimum listed per step; everything else is implementer's discretion.

## Known facts (verified during planning)

- Working API-format workflow:
  `agforge/.local/resources/comfywf/music/audio_ace_step_1_5_checkpoint.json`
  (git-ignored, like the video one). Node classes:
  - `TextEncodeAceStepAudio1.5` — carries `tags` (this is where the prompt
    goes; it still holds the test prompt), `lyrics: "[Instrunment]"`, `bpm`,
    `duration: 120`, `keyscale`, and its **own `seed`** input.
  - `KSampler` — also has a `seed`. Randomize both, or every run returns the
    same audio (the video module had the same trap with `RandomNoise`).
  - `SaveAudioMP3` → output is `.mp3`.
- `comfy_video.py` is the template: inject by `class_type` (node ids change on
  every re-export), `submit` → `wait_for_outputs` → `/view` download →
  `upload_and_presign`. Contract: presigned URL as last stdout line, `local:
  <path>` on stderr.
- `generate.py` `CONTENT_TYPES` has **no `.mp3` entry** — without one the
  upload becomes `application/octet-stream`. Add `.mp3: audio/mpeg` (add
  `.wav`/`.flac` too if you feel like it).
- Same ComfyUI server as video (`AGFORGE_COMFYUI_URL` in agforge
  `.local/.env`), shared GPU. The video model wants ~45 GiB; reuse
  `free_memory()` before submitting.
- `agent/toolsets/toolset-music.md` is already rewritten to describe
  `agforge music generate --prompt` (it has typos — "Video Tools",
  "senconds", "video is ready" — fix them in passing if you touch it).
- Legacy path remnants (music-gen / ACE-Step only):
  - `service/charter.md` — the "Music generation service" (`MUSIC_GEN_URL`)
    bullet. The ACE Studio CLI bullet next to it stays, including its line
    "For a lyrics request, do not fall back to the instrumental
    `MUSIC_GEN_URL` path" — reword that sentence to point at
    `agforge music generate` instead.
  - Local file: `.local/music-gen.env`.
  - (`role_run.py`'s ACE Studio plumbing and the ace-studio tests are the
    kept integration — do not remove.)
- Desired state (scratch Nautobot; operator input is
  `pj-clusterintent/.local/desired-state.yaml`) carries five rows to remove:
  `desired_service ace-step`, placement `ace-step-agpc`,
  `desired_service music-gen`, placement `music-gen-agpc`, and the
  `music-gen` `desired_endpoint` on agpc. All five already read
  `unknown / service_observation_stale` in `nctl drift` (agpc observations
  are stale), so nothing live depends on them.
- **Keep** `comfyui` and `swarmui` on agpc — comfyui is the new backend.

## Step 1 — `agforge music generate`

Add `src/agforge/comfy_music.py` (mirror `comfy_video.py`) and wire a
`music generate` subcommand into `cli.py` next to `video`.

- Workflow file constant: the music JSON above. Prompt → the
  `TextEncodeAceStepAudio1.5` node's `tags` input; fresh random seeds into
  both that node and `KSampler`. Leave `lyrics`, `duration`, `bpm`, etc.
  as exported — prompt is the only parameter for now, matching the toolset
  doc.
- Share code with `comfy_video.py` however you like: import its
  `submit`/`wait_for_outputs`/`free_memory`/`output_references` directly, or
  extract a small common module. Your call; don't over-engineer it.
- Music generation was observed at tens of seconds to a few minutes; the
  video module's 600 s poll timeout is fine.
- Add `.mp3` to `CONTENT_TYPES` in `generate.py`.
- Tests: mirror `tests/test_comfy_video.py` (misconfig exits with one clear
  line; prompt+seed injection by class_type; no-output and error statuses).
  The live path is checked by hand, as with video.

Verify: `uv run pytest`, then one real run
`scripts/agforge music generate --prompt "…"` — wait for the URL, download
it, confirm it is an mp3 that plays.

After code lands, reload the natively running consumers so they pick it up:
`launchctl kickstart -k gui/$(id -u)/com.agdev.agforge` and
`…/com.agdev.agforge-zulip`.

## Step 2 — retire the music-gen call path in agforge

- `service/charter.md`: delete the `MUSIC_GEN_URL` bullet; add one line for
  `agforge music generate --prompt` (same wording spirit as the toolset doc:
  foreground, wait for the last-line URL). In the ACE Studio CLI bullet,
  reword the "do not fall back to the instrumental `MUSIC_GEN_URL` path"
  sentence to name `agforge music generate` — the vocals/instrumental split
  itself is unchanged.
- `service/GUIDE.md`: update the instrumental-music description to the comfy
  path; leave the ACE Studio sung-vocals paragraph as is.
- Delete `.local/music-gen.env`.
- If any test pins the `MUSIC_GEN_URL` charter text, update it. Leave the
  acestudio-cli assertions and `test_agent_config.py` ace-studio tests alone.

Commit and push agforge when done (localrule: commit then push). agforge
itself runs from this working tree on agstudio, so launchd kickstart (step 1)
is the whole deployment.

## Step 3 — desired/actual state cleanup

Precedent: `devdocs/episodes/modernize_agdevworld/p1/report6.md` retired
devworld-assistant with one `op: delete` batch — deletes run in reverse
`KIND_ORDER`, all rows in one transaction.

1. Write a small batch file (e.g.
   `pj-clusterintent/.local/retire-agpc-music.yaml`) with `op: delete` for
   the five rows listed above (services, placements, and the `music-gen`
   endpoint on agpc).
2. `uv run --project nctl nctl desired apply -f .local/retire-agpc-music.yaml`
   to preview, `--yes` to apply. Run from `pj-clusterintent`.
3. **Also remove the same five upserts from `.local/desired-state.yaml`.**
   That file is operator input, not a mirror — leave them in and the next
   full apply resurrects everything you just deleted.
4. Actual state: stale observed rows should stop being reported once the
   desired rows are gone; if agpc is reachable, refresh its observation
   (`nctl reconcile agpc --refresh-observation` or a fresh `nctl drift`) and
   confirm the ace-step/music-gen rows are gone. If agpc is offline, stale
   leftovers are acceptable — note them in the report.

Verify: `nctl drift` shows no ace-step/music-gen rows (or only
honestly-stale ones with a note).

## Step 4 — delete the on-node artifacts on agpc (best effort)

"可能なら実体も削除": if agpc answers, kill any leftover processes matching
`ACE-Step-1\.5/.venv/bin/acestep-api` and `/projects/music-gen/.venv/bin/music-gen`
(both believed already stopped) and remove their install dirs
(`~/projects/music-gen`, the ACE-Step-1.5 checkout/venv). Do **not** touch
comfyui or swarmui on that node.

Access: direct ssh with `~/.ssh/ansible_key` is allowed per
`pj-clusterintent/.local/localenv_memo.md` but requires user confirmation
first — ask, or use an ansible ad-hoc command through the rendered
inventory. If agpc is unreachable, skip and record that; the desired-state
deletion in step 3 already makes the cluster honest.

The model checkpoints under ComfyUI (`ace_step_1.5_turbo_aio.safetensors`)
stay — the new path uses them. The workspace-root `music-gen/` repository on
this Mac also stays; only its deployed service dies. Archiving or deleting
that repo is a separate decision for the user.

## Wrap-up

- `devdocs/episodes/agforge/music_gen/p1/report.md`: what landed, live-run
  evidence (URL + local file), drift before/after, what was skipped on agpc
  and why.
- Steps 1–2 (agforge code) and step 3 (cluster state) are independent; order
  within them is free. Step 4 is opportunistic.

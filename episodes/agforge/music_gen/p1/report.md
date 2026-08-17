# Report — agforge music generation via ComfyUI (p1)

Executed 2026-08-17, all four plan steps. `agforge music generate --prompt`
works live; the ACE-Step 1.5 service path (music-gen wrapper + ace-step on
agpc) is gone from code, charter, desired state, and the node itself. The
ACE Studio desktop integration is untouched, as re-scoped.

## Step 1 — `agforge music generate`

- `src/agforge/comfy_music.py`, mirroring `comfy_video.py` and importing its
  `submit` / `wait_for_outputs` / `free_memory`. Prompt → the
  `TextEncodeAceStepAudio1.5` node's `tags`; fresh random seeds into both
  that node and `KSampler` (the seed lives in two nodes — the plan's trap
  was real). Everything else in the export is left alone.
- `music generate` wired into `cli.py`; `.mp3` / `.wav` / `.flac` added to
  `CONTENT_TYPES` in `generate.py`.
- `tests/test_comfy_music.py` (10 tests, mirroring the video suite).
  **182 passed** total.
- Live run:
  `scripts/agforge music generate --prompt "cheerful upbeat chiptune adventure theme, bright melody"`
  → prompt_id `955b7e34`, local `.local/out/2026-08-17-955b7e34.mp3`
  (`MPEG ADTS layer III, 64 kbps, 48 kHz, Stereo` per `file`), presigned URL
  delivered; ranged GET through the URL answered `206` with
  `Content-Type: audio/mpeg`. Wall clock well under the 600 s poll budget.
- `com.agdev.agforge` and `com.agdev.agforge-zulip` kickstarted.

## Step 2 — music-gen call path retired in agforge

- `service/charter.md`: `MUSIC_GEN_URL` bullet replaced by an
  `agforge music generate` bullet; the ACE Studio bullet's no-fallback
  sentence now names `agforge music generate`. ACE Studio path otherwise
  untouched.
- `service/GUIDE.md`: instrumental-music capability now describes the
  ComfyUI path; the sung-vocals paragraph stays.
- `agent/toolsets/toolset-music.md`: typos fixed ("Video Tools" → "Music
  Tools", "senconds", "video is ready").
- `.local/music-gen.env` deleted. No tests pinned the old charter text.

## Step 3 — desired/actual state

- `pj-clusterintent/.local/retire-agpc-music.yaml`: one `op: delete` batch,
  five rows (services `ace-step`, `music-gen`; placements `ace-step-agpc`,
  `music-gen-agpc`; the `music-gen` endpoint on agpc). Two format lessons
  over report6: the file needs the `operations:` envelope, and every
  operation — deletes included — must carry a `values:` key (`{}` works).
- Preview `delete: 5, conflict: 0`, committed with `--yes`.
- The same five upserts removed from `.local/desired-state.yaml`
  (backup: `.local/desired-state.yaml.bak-music-gen`). Full re-apply of the
  file is now a clean no-op: `unchanged: 85`.
- `nctl drift`: agpc's intent now lists only `comfyui-agpc` and
  `swarmui-agpc`; no ace-step/music-gen rows anywhere. Overall
  `converged=38 unknown=3`.

## Step 4 — on-node artifacts on agpc

Reached via ansible ad-hoc through the generated `hosts_intent.yml`
inventory (agpc is not in the production inventory).

- **The music-gen service was still running**, contrary to the braindump's
  belief: `uv run music-gen` since Aug 11, parented to PID 1 (manual
  start, no systemd unit, no crontab entry). Killed cleanly.
- Before deleting, verified the ComfyUI checkpoint is independent:
  `ace_step_1.5_turbo_aio.safetensors` lives under
  `~/StabilityMatrix/Models/StableDiffusion/`, and ComfyUI itself runs from
  `~/StabilityMatrix/Packages/ComfyUI` on `:7821`. Untouched.
- Deleted `~/projects/music-gen` (50M) and `~/ACE-Step-1.5` (28G).
  Confirmed no matching processes remain and both directories are gone.
- The workspace-root `music-gen/` repo on agstudio stays (plan note: its
  archival is the user's separate decision).

## Left alone / known residue

- agpc still reads `unknown / stale_actual_data` in drift for
  comfyui/swarmui: `nctl reconcile agpc --refresh-observation --yes` fails
  with `ssh_host_key_unreachable: agpc` — nctl's own SSH trust to agpc is
  broken, which is the pre-existing reason those observations were stale
  before this episode (ansible's `ansible_key` path works fine). Fixing
  that trust is a separate episode.
- ACE Studio integration (`acestudio-cli`, `role_run.py` plumbing,
  `.local/ace-studio.env`, its tests): intact by design.

Omni Agent note: did the implementation and the agpc cleanup directly —
handoff candidate for autolab/cagent workflows.

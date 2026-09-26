# give_context_easier p1 — step 6 report: verify, deploy and prove the workflow

## Focused tests

| suite | result | covers |
|---|---|---|
| pyagag `uv run pytest` | 905 passed (+ `test_init` grant check → 7 passed after `c462f73`) | discovery, refresh interval and unknown-name refresh, revision identity across consumers, archived, cached reads during an outage, unusable newest catalog, duplicates/bad entries, empty repository, unknown revision |
| agentroom `uv run pytest` | 352 passed (11 in `test_contexts.py`) | relay reads, interrupted creation retried with no duplicate, registration, rename/archive/reactivate, publication conflicts (panel and plain `git push`), unavailable/invalid input, routes, catalog init |
| cagent | 204 passed | the `contexts` tool, grants, run environment |
| agfront / agautolab / agforge / agobserver / archsage | 184 / 298 / 265 / 117 / 28 passed | grants and guides on the new pin |
| frontend | `tsc --noEmit` clean, `npm run build` ok | |

UI, headless Chrome over CDP (`agdevworld/.local/ctx/`): the demo scenarios
`s1` (20 checks: toggle, IME search, caret insertion of several references, a
file reference, Esc/focus, persistence, a failed request keeping the draft,
newer-version offer, narrow width), `s2` (narrow keyboard), `s3` (Arguing
Room, IME), `s4` (create, upload, publish, conflict, rename, archive).
Against the live relay: `s5`/`s5b` (report5), `s6`–`s9` below.

## Deployment (2026-09-26)

- pyagag `c462f73` in agfront `8c4ddc4`, agautolab `ae0ee6b`, agforge
  `978d859`, archsage `817ec07`, agobserver (pj-agdev), cagent
  (pj-clusterintent `47e0273`) and the relay (agdevworld `5eba6f4`).
  ansible_agdev `366d7ed`.
- Catalog `developer/context-catalog`; host setting `~/.config/agag/refs.toml`;
  per-agent `refs.toml` retired (report3).
- Listeners, cagent-api and relay kickstarted 08:34:49Z (Front again
  08:37:58Z), with no run in flight; every recovery queued nothing. Web image
  on :8090 rebuilt after the last UI change (HTTP 200). `/healthz` ok, mirror
  live.
- `nctl drift` at 08:50Z: **converged=46**, every agag agent `polling`,
  `autolab-agautolab1` `stale` as before. Node observation dumps are 23.4 h old
  (last collected before this deployment). Liveness is from the last agent
  observation, so launchd and the listener logs above are the fresher
  evidence for the restart.

## The complete workflow, live

1. **Created through the UI** — `context-trial` (report5), text and image
   published (`cceca3b`), brief edited (`3a72710`).
2. **Pinned reference inserted** (`s6-send`): in `#front ›
   front-desk-20260926-ctx-trial` the request was typed, then `context-trial`
   was clicked in the panel. The draft carried
   `context-trial@3a7271083a445cde3f97404095fbd9047462d990` with the typed text
   intact, and Enter sent it (#11450, 08:43Z).
3. **Front read it** (desk run, $0.235). Its session log shows
   `agrefs show …3a72710…`, `agrefs show …:brief.md`,
   `agrefs path …:images/pond-dusk.png` and then the harness's `Read` of that
   snapshot file. The reply (#11456) quotes the brief ("Kero only croaks after
   the lantern is lit") and describes the picture correctly: the white
   caption top left, an orange disc top right, green hills, a brown rectangle
   at centre, no frog and no lantern. It noticed the brief and the image
   disagree.
4. **Front delegated the same pinned reference** to forge:
   `#agforge-agstudio1 › assetplan-kero-icon-ctxtrial` (#11454), plan only.
5. **forge read the same originals itself.** Its front and generator runs
   ($0.106 + $0.113) each ran `agrefs show …:brief.md` / `agrefs path
   …:images/pond-dusk.png` and `Read` the image from **forge's own cache**
   (`agforge/.local/refs/context-trial/3a72710…`). forge has no per-source
   configuration: its `refs.toml` is retired, and `context-trial` did not exist
   when it last restarted. Its plan (a11459, #11468) cites the commit, quotes
   the brief, describes the image down to the hill shapes, and asks six
   questions. Nothing was generated.
6. **Front verified and reported** (#11472, $0.097): forge's reading matches
   its own, the commit is unchanged, and forge's questions are relayed to the
   Developer. The run `assetrun-kero-icon-ctxtrial-a11459` was not started.
7. **Same bytes everywhere.** sha256 of `images/pond-dusk.png` and
   `brief.md` at `3a72710` is identical in Front's, forge's and the relay's
   snapshots, and in an **independently configured consumer**: a fresh home
   with no host file, pointed at the catalog through `AGREFS_CATALOG` under
   another host name (`agstudio.home.arpa`). Its image also matches the file
   that was uploaded.

Agent runs for the trial: **$0.55** (memo renderings not counted).

## Second revision

`s7-second`: `brief.md` edited in the panel ("the lantern … is red now") →
`context-trial@e227796`. A new selection in the panel inserted
`context-trial@e227796d89…`. In archsage (which had never read `context-trial`)
and in Front, `@3a72710:brief.md` still says "blue" and `@e227796:brief.md`
"red"; `agrefs changes 3a72710..e227796` names only `brief.md`.

## Remaining active agents

Discovery and reading were verified for every instance and role in its own
run environment (report3's table): Front, autolab, forge, archsage,
Observer, cagent. Front and forge were also verified by a model run.

## Found and fixed during this step

- The open panel covered the left part of the reply-target strip ("1 waiting
  for your reply") and the dialogue box on wide screens. The scene now lays
  both out to the right of the open panel and gives the room back when it
  closes (`5eba6f4`; `s9-clear`: panel right edge 476 px, strip and dialogue
  from 488 px). Screenshots `S03` (before) and `S04` (after).
- Long file paths (report5, `a790fff`).

## Unverified

- **agautolab1**: not redeployed; needs the Developer's approval for ansible
  against the node (report3).

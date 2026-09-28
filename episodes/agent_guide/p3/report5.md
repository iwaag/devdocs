# agent_guide p3 — step 5: deployment and suites

## What changed where

| repository | commit | change |
|---|---|---|
| pyagag | `2377b94` | fixture: `guard-status` and `guard-finished` probes; `sends_to_must_not`; `--shared-guides DIR` |
| pyagag | `051bb67` | fixture: `python -m agag.fixture reader` and `reader_cases.json` (step 4) |
| pyagag | `3c75113` | fixture: autolab's introduction carries the live one's question door |
| pyagag | `1f49c28` | `guides/board.md`: as9 and fd-wr scoped (the delegation fix) |
| agfront | `7daa945` | `shared/requests.md`: a record's silence is not the agent's answer; pin |
| agautolab, agforge, agdevworld (relay), pj-agdev (agobserver, submodules), archsage, pj-clusterintent (cagent) | `f6707ce`, `26dc1df`, `87d00b9`, `a37b889`, `6b1a78f`, `ff2910d` | pyagag `1f49c28` pinned, locks committed |

The fix was committed exactly as trialled: the copies under
`pj-agdev/.local/agp3/fix/` and the committed files compare equal. The
check that nothing repeats across agents (`awk 'length>40' | sort | uniq
-c | awk '$1>1'`, over the 27 guide files of every agent plus pyagag's
shared files) printed nothing. Every push was checked against
`git ls-remote`. Four of them had been issued in parallel, and all four
landed.

## Suites (summary lines, main checkouts, pyagag `1f49c28`)

| suite | passed |
|---|---|
| pyagag | 1192 |
| agfront | 199 |
| agautolab | 334 |
| agobserver | 180 |
| archsage | 41 |
| agentroom relay | 363 |
| agforge | 265 |
| cagent | 204 |

## Deployment

- **Pin and sync**: `uv lock --upgrade-package pyagag && uv sync` in each
  of the seven consumers. The installed `agag/guides/board.md` in agfront,
  agautolab and archsage carries the new sentence. comfynotify keeps
  `3a16479`, since it has no board guide.
- **Restarts** (README_DEV § *How a guide is put together*), one at a
  time, after confirming no agent had a serving in flight (`/inflight/<name>`
  empty for all five). Each log was read before the next restart:
  - Front 19:14:05Z;
  - autolab 19:14:22Z;
  - forge 19:15:20Z;
  - archsage 19:15:37Z;
  - Observer 19:16:09Z;
  - cagent-api, then cagent-zulip 19:17:08Z;
  - autolab's gateway, forge's request service, the relay.

  Every listener logged `claim check: reader qwen3.8:27b-mxfp8` (cagent: its
  line) and registered its queue. The startup recoveries queued the same
  old ✔ conversations as every restart since 05:51Z (autolab 50, archsage
  3, Observer 7), and all were ignored ("not a task", "no root note of
  ours", "not an argue"). No new run record appeared.
- **After**: the relay's `/healthz` 200; cagent's human entrance 200;
  `nctl status`: `ok: True`.

agfront's `requests.md` is read per serving, so it was live at the first
serving after its commit. The shared `board.md` is read from the installed
package per serving, so it was live at `uv sync`. The restarts follow the
procedure; the in-process code that changed is fixture-only.

## Documents

- `devdocs/README_DEV.md`:
  - *Trying a guide change against the same board* gains the guard
    probes, `--shared-guides`, and how to read a delegation run (verdict
    and calls);
  - *A reply that claims an act it never did* gains the reader probe;
  - the shared-text table's `board.md` row says what the file now adds.
- Host notes (ignored `pj-agdev/.local/devenv.md`): *agent_guide p3 on this
  host*.

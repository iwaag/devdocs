# workflow_editor fix1 — Step 1: the common boundary

## The failure, read from its record

`characters/freework/run-001` (talk node, Sonnet on the `studio` node) was
asked "ちょっと確認。nfsの/agshare/agag_folks/pondererというフォルダが見えますか？".
Its `run.create` recorded no `input.references` (the words hold no
`shared://`), so the attempt brief had no *Shared areas* section. The agent
checked the literal local path and `/Volumes`, and answered that neither the
folder nor an NFS mount existed. It never ran `wfe share`.

The share was fine: on the coordinator `wfe share resolve
shared://agshare/agag_folks/ponderer/` is `ok` and `wfe share ls` lists
`face.png` (3,024,661 B) and `lore.md` (1,694 B).

## What the tools accept today

| Written | `wfe share resolve` |
| --- | --- |
| `shared://agshare/agag_folks/ponderer/` | `ok`, this host's folder and LAN download address |
| `agshare/agag_folks/ponderer` | refused: not a `shared://` reference |
| `/agshare/agag_folks/ponderer` | refused |
| `agag_folks/ponderer` | refused |

Only the URI form resolves. There is no way to find an item by name: `ls`
lists one known reference recursively (up to 100,000 files read, the first
`--max` shown). States are `ok`, `unknown-share`, `unmounted` (marker
missing), `absent`, `outside`, `not-a-folder`, `not-a-file`.

## What workers receive

- **Coordinator (`studio`, local)**: `bin/wfe` on the agent's PATH, area
  `shares.json` → `agshare` at the server's backing folder (agstudio serves
  the NFS export and does not mount itself; the root marker
  `.wfe-share.json` names the share). The brief carries share lines only when
  the input wrote a `shared://` reference; the studio worker declares no
  host facts.
- **Remote (`gpupc` on agpc)**: `~/wfe-worker`, its own `shares.json` →
  `agshare` at the NFS client mount; `wfe share …` runs locally on the worker
  (`cli/remote.ts`), everything else goes to the coordinator. Its worker
  facts include "shared areas: agshare (wfe share list)". Its WFE source is
  agdevworld `ea09c14` (project_migration p7), older than the coordinator's
  release `b3ae972`. `ls` of the folder there shows the same two files.
- **The wfrun guide** (agautolab `agent/guides/wfrun/guide.md`, read by
  every run at its start) says the share is "named `shared://<share>/<path>`
  in the person's words" — the assumption this fix replaces.

## Cluster state (nctl first)

`nctl relations` and `nctl drift`: `agshare-server` (agstudio),
`agshare-client` (agautolab1, agpc) and `agautolab` (agstudio manual, agpc
nctl-managed) are all converged/satisfied. One unrelated pre-existing drift
(`cagent-snake-e2e` `service_missing`). No placement change is needed for
this fix; updating agpc's worker source is a pin change through its
placement (`setup_wfe_worker.yml`).

## Decisions for Step 2

- Put alias handling and name discovery in the share layer
  (`server/shares.ts`) behind the existing `wfe share` CLI, so local and
  remote workers get the same behaviour from their own `shares.json`.
- Aliases: `shared://…`, `<share>/<path>`, `/<share>/<path>`, a path under a
  configured root as this host sees it, and a share-relative path tried in
  every configured share. All stay inside configured roots (normalized, no
  `..`, symlinks out of the share refused as today). A unique hit prints the
  canonical `shared://` reference; several hits are reported as ambiguous.
- `wfe share find <name>`: a bounded walk (entries, depth, results) that says
  whether it was complete; an exact path is answered by `resolve` without a
  walk.
- Records stay as they are: `run input --ref` still takes the canonical
  reference, which `resolve` prints.

Raw command output is in the ignored notes (`pj-agdev/.local/fix1/`).

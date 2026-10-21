# workflow_editor fix1 — Step 2: share tools take people's words

agdevworld `1e2fbed` (source; deployed in Step 4).

## What changed

All in the share layer (`workflow_editor/server/shares.ts`) behind the
existing `wfe share` CLI, so the coordinator and every execution node get
the same behaviour from their own `shares.json`. Nothing is specific to
characters.

- **`locate`** (`wfe share resolve|ls <ref or path>`) reads a text against
  this host's configured shares. Readings, each one stat and never a walk:
  a `shared://` reference; `<share>:<path>`; this host's folder for a share
  (configured or real path); a path with a configured share's name as a
  segment (`/agshare/x`, `agshare/x`, another PC's `/mnt/agshare/x`); the
  whole path inside each share (`agag_folks/ponderer`). Surrounding quotes,
  「」 and sentence punctuation are dropped. `..` and links out of a share
  are refused.
- **Answers** keep the distinctions the plan asks for:
  - `ok` — one item: its canonical reference (trailing `/` for a folder),
    how the words were read, this host's path, the LAN download address;
  - `ambiguous` — several readings exist; each is listed;
  - `absent` — nothing in reachable shares; every reading tried and the
    nearest existing folder are named;
  - `unavailable` — a share it may be in is unmounted here (or none is
    configured), so absence is **not** confirmed;
  - `unknown-share`, `outside`.
- **`findInShares`** (`wfe share find <name>`) searches by a name or the
  last segments of a path (`ponderer`, `agag_folks/ponderer`, `*.png`),
  case-insensitively, breadth first (shallowest first), symlinks not
  followed, transport metadata skipped. Bounds: `--max-entries` (default
  50000), `--depth` (12), `--limit` (50); `--under <ref or path>` and
  `--share` narrow it. Each share reports entries read and why it stopped;
  `complete` is false after a bound, an unreadable folder or an unreachable
  share, and the CLI then says "not found in the part searched … not
  absence" instead of "is not in".
- `--help` for `wfe share` explains the words people use, every answer and
  the bounds; `docs/shares.md` gained *What people write*, with the run-001
  failure reference.
- Records are unchanged: `run input --ref` still takes the reference that
  `resolve` prints.

## Checks

- 4 new tests in `test/shares.test.ts`: all reading forms; ambiguity;
  absence with nearest folder; unknown share; `..` and a symlink escape;
  an unmounted share (marker removed) and no shares configured →
  `unavailable`; find by name/segments/glob/`--under`; depth, entry and
  match bounds each make the search incomplete; CLI exit codes and JSON.
- `npm run check`: 509/509, type check and both builds.
- Live share on the coordinator (source CLI): `/agshare/agag_folks/ponderer`,
  `agshare/…`, `agag_folks/ponderer`, the backing path and `/mnt/agshare/…`
  all resolve to `shared://agshare/agag_folks/ponderer/`;
  `/agshare/agag_folks/nobody` is absent with nearest `…/agag_folks/`;
  `find ponderer` reads the whole share (30,494 entries) in about 0.25 s;
  `find lore.md --max-entries 5` and `--depth 1` report incomplete.

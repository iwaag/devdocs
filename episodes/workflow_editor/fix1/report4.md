# workflow_editor fix1 — Step 4: verify and deploy

## Deployment

| Component | Deployed | How |
| --- | --- | --- |
| WFE on the coordinator | agdevworld `1e2fbed` (previous `b3ae972`) | `wfe-release.sh` build (checks pass) then activate; no store change; stores backed up first |
| wfrun guide | agautolab `69142b8` | live checkout, read at each run's start |
| agpc execution node (`gpupc`) | wfe `1e2fbed`, executor `69142b8` (were `ea09c14`, `0a813b0`) | Nautobot placement `agautolab-agpc` config (dry run: 1 update), `nctl render production`, `setup_wfe_worker.yml` (ok 22, changed 4, failed 0) |
| pj-agdev | pins `4a82245` | |

The auto-mode classifier refused the release activation and the Nautobot
apply. The Developer approved both by hand, and then they were run. After
the deploy both workers report `available`. agpc's unit is active and its
checkout is `1e2fbed`. Recovery steps (code, worker pins, store backups)
are in the ignored notes.

## Trials

Every trial is a fresh `characters/freework` run, submitted as `Omni Agent
(for Developer)` (the record's author) with no command, URI or host-path
hint.

| Run | Node | Words | Outcome |
| --- | --- | --- | --- |
| run-002 | studio | ちょっと確認。nfsの/agshare/agag_folks/pondererというフォルダが見えますか？ (the original) | "はい、見えます": resolved `shared://agshare/agag_folks/ponderer/`, named `face.png` (~3.0 MB) and `lore.md` (1.7 KB); noted that the literal `/agshare` path does not exist on this host. Nothing registered, run tree clean |
| run-003 | gpupc | agag_folks/ponderer の中に何が入ってる？ (shortened, on the other node) | Resolved at that node's mount, described both files, offered to import only if asked |
| run-004 | studio | nfsにあるpondererフォルダの中身を教えて。 (with a temporary second `ponderer` on the share) | Listed both matches with their contents and pointed out the likely one; said the whole-share search found only these two |
| run-005 | studio | nfsのagag_folks/sagefolkというフォルダは見えますか？ | `agag_folks/` exists, `sagefolk` does not; named what is there |
| run-006 | studio | characters/forge/lore.md の最初の一行を教えて。 | Read the project's own file; ran no share command |

The agents' logs show they found these answers with `wfe share resolve`,
`find` and `ls`, which they discovered from the guide and `--help`.

## Tool-level checks

- **Unavailable share**: the deployed CLI with a share root that has no
  marker gives `resolve` → `unavailable … absence is not confirmed`, and
  `find` → `unmounted`, "not found in the part searched … not absence"
  (exit 1). No agent trial was run for this case, because it would need
  the live share taken down.
- **Host-specific resolution on agpc**: `/agshare/…`, `agag_folks/…`,
  agpc's own `/mnt/agshare/…/lore.md` and the coordinator's backing path
  all resolve to the same reference at `/mnt/agshare`. `find ponderer`
  reads the whole share (30,494 entries) and reports it complete.
- **Search limits**: `--max-entries` and `--depth` make the search
  incomplete instead of reporting absence (Step 2; 509/509).
- The temporary ambiguity probe was removed from the share after run-004.

## Observations

- run-004 listed both matches instead of asking which one was meant. The
  question was about contents, so the answer covered both. A request that
  acts on materials would need to choose.
- Agents also tried commands that do not exist (`wfe share find -i`,
  `wfe share cat`). The tool refused them, and the agents continued with
  `ls` and their shell.
- On agpc, node v22 prints an SQLite ExperimentalWarning on every `wfe`
  call; agents filter it out with grep.

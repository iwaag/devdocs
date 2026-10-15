# Agent session p1 ex3 — step 5: report and preserved results

Date: 2026-10-15.

## Written

- [report.md](report.md):
  - the three-row result table;
  - combined cost and active time;
  - phases;
  - mechanism;
  - recommendation;
  - preserved outputs;
  - versions, limits and the Deus Ex Machina note.
- Per-set reports [set-a.md](set-a.md), [set-b.md](set-b.md) and
  [set-c.md](set-c.md). Each has cost, phases, designs, defects and the
  raw-evidence locations.
- README_DEV *Workflow run sessions*: one paragraph on ex3's result. The
  default stays `fresh`.

The bulky evidence stays in the ignored notes
`pj-agdev/.local/agent-session-p1-ex3/`:

| Folder | Contents |
| --- | --- |
| `step1/` | the frozen contract |
| `step2/` | driver, setup scripts, fixture checks |
| `step3/` | per-run logs, transcripts, exec stores, bundles, `aggregate.py` |
| `step4/` | probes, browser steps, screenshots, decisions |
| `step5/` | `audit-*.json`, `mechanism.txt` |
| `tools/` | live cost estimator |
| `trial-branches.git` | the six trial branches |

No machine paths, credentials or session stores were committed.

## Preserved RTS implementations

| Branch | Head | Game diff vs the trial commit |
| --- | --- | --- |
| `trial/agent-session-p1-ex3/a-fresh` | `a30735e` | 5 files, +184 −10 |
| `trial/agent-session-p1-ex3/a-run` | `e029ec0` | 4 files, +219 −13 |
| `trial/agent-session-p1-ex3/b-run` | `f2963e9` | 4 files, +131 −9 |
| `trial/agent-session-p1-ex3/b-fresh` | `64b4a95` | 4 files, +127 −5 |
| `trial/agent-session-p1-ex3/c-fresh` | `553afcf` | 4 files, +339 −5 |
| `trial/agent-session-p1-ex3/c-run` | `62514f9` | 5 files, +255 −14 |

- Every branch's headless suite passes on its head.
- The branches are committed in
  `pj-agdev/.local/agent-session-p1-ex3/trial-branches.git` and in each
  run's `branch.bundle`.
- The push to the RTS Gitea repository was refused by the permission
  classifier as a change to a shared resource. The command for a person is
  in the final report.
- Nothing was integrated into the RTS `main`, and no game change was
  deployed.

## Cleanup

- No trial-only process is left:
  - all presentations were stopped by the runs' own cleanup steps (no
    listener on 18800–19399);
  - no `claude -p` process remains;
  - the in-process executors exited with their drivers.
- The production executor, WFE service and stores were not touched. The
  held jevlab run is unchanged.
- The private areas remain in the session scratchpad until it is removed.
  Everything needed from them was copied to the notes above.

# workflow_editor p2/pre1 step 4 — update reliability and tuning

## Changes

Service (`server/watch.ts`, `server/api.ts`, `server/workspace.ts`):

- **Two polls, both only while a stream is connected.**
  - **Definitions every 1000 ms (unchanged).** `project.yaml`,
    `.gitmodules`, the workflow files and the registry are polled. A file is
    re-read only when its size, mtime or inode changed, so atomic-rename saves
    still count as changes.
  - **Git state every 3000 ms (new).** The fingerprint is one
    `git status --porcelain=v2 --branch --ignore-submodules=none` in the root
    (HEAD, index, root dirty files, dirty submodules). One more `status` runs
    only for submodules that are already flagged dirty. Per submodule, file
    reads cover the `.git` presence, HEAD and reflog. The commands run with
    `GIT_OPTIONAL_LOCKS=0`, so they never take the index lock. A changed
    fingerprint sends a `git` event.
- **Hello after the first snapshot.** A new stream takes its definition
  snapshot and Git fingerprint before it says hello. Every hello means
  "re-read everything", which covers changes made while the service was down
  or the stream disconnected. `retry: 1000` makes the browser reconnect
  within about a second.
- **Registry events.** The registry is watched with each workspace, and alone
  by a workspace-less stream `GET /api/events` for the home page.
- **Targeted refresh.**
  - Repository inspection is cached per workspace while the watcher's Git
    fingerprint and `.gitmodules` are unchanged; `?refresh=1` bypasses the
    cache.
  - The project response no longer inspects Git twice.
  - Workflow files are parsed again only when their stat signature changed.

Client (`src/live.ts`, the three views):

- **One merged set of needs per flush.** Events add needs (content, context,
  registry); they no longer replace each other. A content reload is never
  downgraded to a context reload.
- **Serial loads.** A view runs one load at a time, and requests made
  meanwhile are merged into the next run. An older response can never
  overwrite a newer one.
- **Outages are visible.** The workflow view shows a banner and the project
  view a "Not connected — not live" pill. A stream the server refuses is
  retried every 2 s.
- **Deleted or renamed open workflow.** A banner names the missing file and
  the view shows the last version read-only. If another file has the same
  workflow id, an "Open <file>" link is shown. Nothing is recreated, and the
  service still refuses a save to a missing file.
- **Refresh.** The workflow and project views have a Refresh action that
  re-reads at once, Git included.
- **Unsaved drafts.** If a reconnection finds that the file changed while the
  view holds unsaved edits, it raises the existing "changed on disk" guard.
  Writers still take turns: no merge, no revisions.
- **Inspector focus.** A refresh that keeps the draft no longer rebuilds the
  inspector while it has focus. That rebuild discarded half-filled inspector
  forms. The p1 step3 check failed on it in 2 of 3 runs once the watcher sent
  more refreshes; after the fix it passed 4 of 4.

## Before and after

Same harness (`checks/measure.ts`), same data, 20 edits per workflow case and
10 per project case.

**Latency.** File write/rename completed → value visible:

| View | Baseline p50 / p90 / max | After p50 / p90 / max | > 2 s |
| --- | --- | --- | --- |
| Workflow, representative | 671 / 1047 / 1219 ms | 672 / 1022 / 1107 ms | 0 |
| Project, representative | 927 / 1319 / 1319 ms | 634 / 1046 / 1046 ms | 0 |
| Workflow, large | 1030 / 1298 / 1318 ms | 693 / 1083 / 1165 ms | 0 |
| Project, large | 964 / 1384 / 1384 ms | 610 / 1106 / 1106 ms | 0 |

**Load** in the service process. Idle values are per second with a viewer
open. Git subprocesses spawned by Git itself are not counted:

| Case | Baseline | After |
| --- | --- | --- |
| No viewer | 0 work, ~0.1 ms/s CPU | the same |
| Idle, representative | 3 reads, 0 Git, 1.9 ms CPU | 1.7 reads, 5 stats, 0.67 Git, 3.1 ms CPU |
| Idle, large | 152 reads, 0 Git, 17.7 ms CPU | 13 reads, 160 stats, 0.67 Git, 8.1 ms CPU |
| Per edit, workflow view, representative | 23 Git, 24 ms CPU | 5.7 Git, 13 ms CPU |
| Per edit, project view, representative | 43 Git, 35 ms CPU | 7.2 Git, 14 ms CPU |
| Per edit, workflow view, large | 125 Git, 442 reads, 197 ms CPU | 10.9 Git, 27 reads, 49 ms CPU |
| Per edit, project view, large | 247 Git, 381 reads, 316 ms CPU | 17.8 Git, 31 reads, 56 ms CPU |

Each case had one GET per edit, before and after. The representative idle
CPU rose slightly because the Git state is now watched (one `git status`,
about 20 ms, every 3 s). The root `git status` costs about 20 ms on the
representative project and 110 ms on the large one. `git submodule status`
was measured at 440 ms on the large project and rejected as a fingerprint.

**Update cases** (21 probes; the full list is in report1):

| Case | Baseline | After |
| --- | --- | --- |
| UI save → file; external edit after it | pass | pass |
| Burst: open workflow + workflow created | lost | 440 ms |
| Burst: open workflow + project.yaml | pass (by event order) | 501 ms |
| Burst: two workflow files | lost | 462 ms |
| Open workflow deleted | invisible | banner, read-only, not recreated |
| Deleted open workflow restored | lost | 528 ms |
| Open workflow renamed | invisible | banner with "Open wf-001-renamed.yaml (same workflow id)" |
| Project view: create / rename / delete | ~1 s | ~1 s |
| Malformed, then valid | pass | pass (1.1 s / 0.9 s) |
| Service stopped | no indication | banner / pill |
| Edit made while the service was down | lost | 563 ms after restart |
| Submodule dirty (also when already dirty) | not shown | 1.2 s |
| Root `git add` / `git commit` | not shown | ~3.0 s |
| Submodule initialized | not shown | 2.9 s |
| Fast workspace switch | pass | pass |
| Registry changed while a view is open | not shown | 0.2–0.9 s |

Raw records (ignored): `pj-agdev/.local/workflow-editor-measure/baseline.json`, `after.json`.

## Intervals chosen

- **Definitions: 1000 ms, unchanged.** Every measured save appeared within
  1.2 s (p50 about 0.65 s). That is inside the ~2 s target with margin,
  for one workflow and for 150 workflows. A shorter interval would mainly add
  idle stats: about 160/s on the large project already. Filesystem
  notifications are not needed. The measured shortcomings were lost refreshes
  and Git cost, not poll latency.
- **Git state: 3000 ms, plus Refresh.** Git changes appear within about 3 s
  for one `git status` per tick. That costs about 20 ms (representative) and
  110 ms (large) of Git work every 3 s while a view is open. `--git-poll-ms`
  changes it.

Limits of these numbers:

- They come from one machine (agstudio, Apple silicon, local SSD).
- The latency includes up to one poll interval plus a 100 ms coalescing
  delay.
- Git work done by child Git processes (the submodule recursion inside
  `git status`) is not in the service's CPU figures.
- Write–read races between two writers are still out of scope.

## Verification

- `npm run check`: tsc, 50 tests (4 new in `test/watch.test.ts`, covering
  hello after the snapshot, create/change/delete events, six Git-state cases
  including an already-dirty submodule, registry events, no work without
  streams, and the serial loader), build.
- `checks/measure.ts` probes: 21/21 (twice).
- `checks/setup.ts`: 15/15. p1 browser checks step2, step3 (4 runs), step4
  and e2e pass on a private fixture and service.

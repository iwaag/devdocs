# improve_wfrun p3 step 2: backup and recovery

Date: 2026-11-02. Code: agdevworld `a4a87c1` (`server/runBackup.ts`, `cli/wfe.ts`, `test/runBackup.test.ts`). Contract: `workflow_editor/docs/storage.md` *Backup and recovery*. Evidence and the destination's address are in ignored local records (`pj-agdev/.local/improve-wfrun-p3/`).

## What exists now

`wfe backup create|status|list|verify|restore|rehearse|reinstate`, `GET api/backup`, and the launchd job `com.agdev.wfe-backup` (daily, 04:30). Deployed: coordinator and worker `gpupc` on `a4a87c1`, package check 704/704 in the release build.

## Live result

| Item | Result |
| --- | --- |
| Generations at the destination | 3; the first cost 6.6 GiB, the next two about 90 MiB each (hard links) |
| Capture and verification | 55 s; 9480 files, 6.31 GiB |
| Scope | 10 retired runs covered, 487 others for identity only, 19 legacy names |
| Scheduled job | kicked once under launchd: published, and `wfe backup verify` passed against it |

**Two runs were retired for this**, beyond p2's nine: `misc freework/run-001` (three repositories, so that nested commits exist in the live evidence: none of the nine had any) by check actor `improve-wfrun-p3-check`. It was the smallest fit, 832 KB. `forge freework/run-005` was tried as a live cancelled run and refused correctly: it is held (interrupted). So the count is one new retirement, and the rest of p2's position stands: further retirement is still a separate decision.

## Rehearsal on the backup host

Run on `gpupc`, which has no `/Users/eiji` at all, so the original retention and archive paths could not be read. Git in the process was limited to local files, it had no credentials, and its locks were its own. A disposable folder, deleted afterwards.

| Step | Result |
| --- | --- |
| Restore and verify the generation | 25 s, 9480 files, 6.31 GiB |
| Rehearse | 91 s, **every check passed** |
| Restored | 10 runs, 12.52 GiB of working trees |
| Comparisons | 18775 (listed files, branch refs, output files, archive sums) |

| Check | Runs | Result |
| --- | --- | --- |
| Record read and replayed, at the captured sequence | 10 | pass |
| Reports read | 10 | pass |
| Output bytes against manifest hashes, on the remapped share | 6 outputs | pass |
| Restoration; every repository at its saved commit; local data compared | 10 | pass, including three repositories of `misc freework/run-001` and the 15061 files of `playprey onboard/run-002` (30 s) |
| Worker archive by checksum | 1 | pass, 288 MiB, **0 bundles**: this run has no unpublished commit, so bundles are shown in step 3 |
| Receipt replay finds the same run | 10 | pass |
| Run numbers stay taken | 10, and 487 outside the guarantee | pass |
| Damaged archive | 1 | refused, with the checksum named |
| Lost retention repository | 1 | names the affected run |
| Nothing dispatched | | jobs 1081 → 1081, attempts 1057 → 1057 |

Evidence links are informational rather than pass or fail: an artifact can legitimately have been made elsewhere. 15 of 16 artifacts outside run folders resolved; the other, in `playprey onboard/run-002`, is a folder (`…/p1/shots`), and a file read does not apply to one.

Still needed from outside: the software at the manifest's revisions (both on GitHub), and for publishing again, Gitea and the tokens.

## Fixtures, `test/runBackup.test.ts` (9)

Interruption after transfer, a byte changed in staging, the file list lost in staging and a restoration during capture each fail the generation by name and leave `LATEST`. A run the last generation covered and this one cannot gives `degraded` without moving `LATEST`. Damage at the destination names exactly the affected runs, and a shared retention repository names all of its runs. Recovery with the operating area and share renamed away, including nested commits and output bytes. Reinstating lost pieces without overwriting a newer record. Expiry.

## A defect this found

Restoring a nested repository under a path containing a symbolic link computed a wrong relative Git folder and failed. Not reachable on the current production path, which has no symlinked parent. Reachable from a recovery folder under macOS `/var`, which is how the fixture found it. Fixed in `rebuildTree`.

## Limits, as tested

- **Not disaster recovery for the service.** Gitea, workspaces, credentials and the trees of non-retired runs are outside. Losing the coordinator whole recovers retired runs' records and work, not the system.
- **Generations share bytes through hard links.** They protect against a bad capture and later logical damage. They are not independent copies against the backup disk failing, and there is one destination.
- **`reinstate` is tested in a fixture only.** No live storage was damaged to try it, as the plan requires.
- Output bytes held are those of retired runs; adoption references by consuming projects were not followed separately.
- **Rollback to pre-p2 code stays unverified** and is not offered as p3's fallback. It was not rehearsed; `docs/storage.md` says so. A pre-p3 release can still read everything p3 adds, but that was not exercised either.

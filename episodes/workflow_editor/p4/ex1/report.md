# workflow_editor p4/ex1 — report

The correction in [braindump.md](braindump.md) is implemented: the workflow
editor is agdevworld system code, and the operating catalog holds only the
real development project, RTS. One completion criterion is outstanding, and
it waits on the person: the real RTS run.

## Completion criteria

| Criterion | State | Evidence |
| --- | --- | --- |
| Agdevworld owns and supplies the editor as a system component | done | `agdevworld/workflow_editor/` (imported from `9c29117`), built from a clean clone: [report2](report2.md) |
| The service and CLI build, deploy and recover without a wfe-managed editor project | done | `agdevworld/scripts/wfe-release.sh`: refusal under an open attempt, restore on failed health, rollback target. One real failed candidate changed nothing in use ([report4](report4.md)). |
| The catalog contains RTS only; `study-rts` is available and linked | done | registry: project `rts-vs-bot`, workspace `rts-vs-bot`, `gitea:131` root, `gitea:130` used by `rts-vs-bot (study/study-rts)`; the same in the browser, API and CLI after two restarts |
| Both editor workspaces and the editor project/repository are out of active use; no pending job can restore them | done | `wfe project unregister workflow-editor`; folders archived; an injected queue job was dropped by the live executor; removed ids return 404; the retired repository is refused in every role |
| Supported management and execution operations reject protected system targets | done | `server/boundary.ts`; [report3](report3.md) (6 tests); live probes in [report4](report4.md) |
| p4's storage, Gitea, dashboard and single-autolab behaviour still work | done | 93 tests; browser checks 33/13/18; executor tests 3/3 on source and on the release: [report5](report5.md) |
| Actual RTS acceptance established, or reported as outstanding | **outstanding** | no real RTS run exists yet; route ready ([report5](report5.md)) |
| No `improve-editor` run or self-deployment required | holds | none was made |

## State of this host

| Item | State |
| --- | --- |
| Service | `com.agdev.wfe-service` on `127.0.0.1:8098`, release agdevworld `6fe2b61` |
| Launcher | `bin/wfe` on the same release, pinned to the area's registry and port 8098 |
| Executor | `com.agdev.agautolab-wfexec`, idle, through `bin/wfe` |
| Route | `localhost:8093/wfe/` (loopback only); "← agdevworld" back to the operation room |
| Rollback | `service/previous` = `2678c9f` |

## Commits

- agdevworld `35ab47a` … `0a659f1` (import, release script, boundary,
  fixes, docs).
- agautolab `167ae13`.
- pj-agdev `1a1606c`, `9706de9`, `77b4d7d`, `8e40fba`.
- devdocs: one report per step.

## For the person

1. **Make the RTS request** (report5, "Real RTS acceptance"): your own
   words, through `improve-game` in `rts-vs-bot`. Then answer the scope
   question, and accept or reject the result. That completes p4.
2. **Optional cleanup:**
   - `pj-agdev/.local/agdev/service/releases/0b9d97a….tmp`, a failed
     candidate the agent could not delete;
   - the archives in `.local/agdev/.local/retired/`, when no longer wanted;
   - the Gitea repository `developer/pj-workflow-editor`. Deleting it is not
     required; it is excluded by the boundary.

## Notes

- **Not an OS sandbox.** The checks protect the supported operations. An
  agent with shell access under the same account can still change files
  outside them.
- **Registry is required.** Commands and the service now refuse to start
  without `--registry` / `WFE_REGISTRY`. Moved code can no longer open an
  empty catalog by default.

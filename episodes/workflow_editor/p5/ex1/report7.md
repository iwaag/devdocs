# p5/ex1 step 7 — rollout

Plan section: [plan.md](plan.md) §7. Times are JST, 2026-10-11.

## Tested commits and contract

| Repository | Commit | Contains |
| --- | --- | --- |
| agdevworld | `78caec2` (pushed) | Editor: notify protocol 2, schema migrations, SQLite helper, backend header |
| agautolab | `ca8e807` (pushed) | Transport for protocol 2, plus tests |
| pj-agdev | `2d650a4` (pushed) | Gitlinks to both |

**Compatibility contract.** The editor's `wfe notify protocol` must equal the transport's `PROTOCOL` (2). A mismatch in either direction sends nothing:

- **Old transport + new editor.** `due` hands out no deliveries, and `sent --id` / `failed --id` are refused with "update agautolab.wfnotify".
- **New transport + old editor.** `notify protocol` does not exist in the old editor. The transport's `prepare()` fails and is logged once, and nothing is claimed.

The migrations are forward-only: notify 0 → 2, exec 0 → 1. Each runs in a transaction on first open; a schema newer than the release is refused.

**Deployment order.** Quiesce the executor and notifier, then back up, then deploy the editor with the release script, then start the executor and notifier. Both mismatch windows are safe as shown above; this order keeps the window empty.

## Rollback, verified before rollout

`test_rollback_to_the_old_release_and_forward_again` ([report6](report6.md)) ran the **released p5 editor `32498e9` and transport `daeaada`**, taken from Git history, on a store migrated to version 2 that held a frozen, unbegun delivery:

- The old code opened the store, sent only what was unsent, and posted nothing twice.
- Moving forward again closed the frozen delivery as "sent outside this delivery" without reposting.

Procedure: run `wfe-release.sh rollback .local/agdev`, check out `daeaada` for autolab's `src/agautolab/wfnotify.py`, then kickstart wfexec. Do this **only when `wfe notify deliveries` shows nothing as "MAY HAVE BEEN POSTED"**, because the old code reintroduces the growing-batch defect for an uncertain delivery.

Restoring the pre-migration backup after new posts would replay messages. It is not an automatic recovery step.

## What was done

| Step | Command / check | Result |
| --- | --- | --- |
| Inspect activity | `wfe-release.sh status`, `wfe exec status`, read-only SQL | No open attempts. Jobs: 6 done, 1 dropped, none queued or claimed. All 6 attempts ended. Executor idle |
| Inspect notifications | `wfe notify outbox` | 9 rows, all `sent` (messages 15907–15914). Nothing pending or uncertain. No `agautolab/.local/wfnotify/inflight.json` |
| Quiesce senders | `launchctl bootout …/com.agdev.agautolab-wfexec` (19:36) | Executor and notifier stopped |
| Back up | `sqlite3 … ".backup …"` for both stores → `.local/agdev/.local/backups/p5ex1-20261011-193704/` (host-local, ignored), plus a listing of the empty `wfnotify/` directory | Integrity ok. `user_version` 0 for both. 9/9 sent rows with message ids; 6 attempts; 6 checkpoints |
| Deploy the editor | `agdevworld/scripts/wfe-release.sh deploy .local/agdev` | Built `78caec2`; `npm ci && npm run check` in the release passed (135/135, build). Activated (`previous` = `32498e9`); `com.agdev.wfe-service` restarted; the script's health check confirmed that the service and `bin/wfe` both run `78caec2`. Its open-attempt protection was in force and not bypassed |
| Migrations | `wfe notify protocol` and `status`, read-only SQL | Protocol 2. notify `user_version` 2, exec 1. Outbox unchanged (9/9 sent, same message ids). Binding of `run-002` unchanged: stream 274, topic `run-002`, last message 15914. No deliveries; no migration notes (there was no legacy uncertainty) |
| Start the notifier | `launchctl bootstrap …/com.agdev.agautolab-wfexec.plist` (19:37) | Log: "delivering to Zulip every 5 s as sender agstudio.home.arpa:98076". `wfe notify sender` shows that owner claiming. **No sends, no deliveries, no errors**: sent events were not replayed |
| Completed RTS run, read-only | `wfe run check improve-game/run-002`; browser on the service (`http://127.0.0.1:8098/`, run view) | Usable. Sequence 29, completed. Header: **"autolab · backend claude_code / anthropic/claude-sonnet-5 (latest attempt)"**, which before read "backend unknown". No page errors |

## Limitations of the rollout evidence

- No coordination event occurred during or after the rollout: the only announced run is complete and idle. **Normal delivery on the new path was not observed on the live system.** It is covered by the isolated tests only. No traffic was invented, and no test post was made to a real topic.
- The live migration had no legacy uncertainty to reconcile. That path is covered by the test built from the released p5 code.
- The pj-agdev host notes (`.local/devenv.md`, local only) record the release, the backup, the rollback rule, and the service URL.

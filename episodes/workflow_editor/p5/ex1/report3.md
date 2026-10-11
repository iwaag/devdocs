# p5/ex1 step 3 — reconcile before retrying; existing state

Plan section: [plan.md](plan.md) §3.

Code: agdevworld `501d0fa` + `d5b9405`; agautolab `47853e1` + `8a2bcd5`. All pushed, not yet deployed.

## Reconciliation

When a claimed delivery may have been posted, the claim carries `reconcile`:

- `since`: the delivery's first `begin`, or for a legacy delivery the creation of its earliest event;
- `searchIn`: every place the message may be.

The places searched, deduplicated, are:

1. where the run's topic is now, after following the last acknowledged message;
2. the delivery's frozen destination;
3. every destination recorded at `begin` (`tried`);
4. the run's recorded relocations.

The transport (`Notifier.search`/`_scan`) narrows `GET /messages` by channel id, topic and its own sender id. It pages back 200 at a time with `found_oldest`. It stops when it finds the message, reaches the oldest message, or passes `since` minus 15 minutes (the allowed difference between this host's clock and Zulip's). A match means the last line equals the marker exactly. Outcomes:

| Search result | Store transition |
| --- | --- |
| Found | `sent --found` with the message's actual location; the run follows it |
| Every place covered, no match | `begin --checked-absent "absent from #S › T: N of the sender's message(s) back to <since> (−15 min); …"`, then the exact frozen payload is posted |
| Error, refused read, or page limit (500 pages) reached first | `failed --uncertain` with the reason. The delivery stays uncertain; `wfe notify status` lists it under problems; it is never treated as absence |

Moves are recorded explicitly. `wfe notify moved` now writes the previous place to `relocations` with the reason ("left for #S › T"), so an earlier send can still be reconciled at its original location.

A `reset` keeps active deliveries. They are still searched for at their recorded places, and `reset` says so in its output. If those places cannot be read, the delivery stays uncertain. The explicit person's path is `wfe notify resolve <delivery> --message <id>` (it was posted) or `--not-posted` (send the frozen payload again). The store refuses `resolve` while the recorded sender's process is still in the middle of a send.

Frozen content is never rewritten. If a document became readable after the message was frozen, the existing follow-up mechanism announces it (`published`, now with its own marker token); see the step 2 test and step 6.

### Limits (documented in `docs/runs.md` and the transport docstring)

- A post deleted by hand between the send and the search is indistinguishable from absence, so it would be posted again. Exactly-once is not claimed.
- The search sees what the bot can read. If the bot loses access, or Zulip loses history, absence cannot be established. The delivery then waits for a person, which is the intended behavior.
- The 15-minute time bound assumes this host's clock and Zulip's agree within that margin. Both run on this host.

## Existing state: migration

The notify schema is now versioned by `user_version`:

- version 1 = the p5 tables, created with `IF NOT EXISTS`, so a pre-versioning database adopts them as they are;
- version 2 = deliveries, senders, relocations and notes.

Each migration runs in one `BEGIN IMMEDIATE` transaction that re-reads the version inside it. A database newer than the release is refused with "newer than this release understands", never reinterpreted.

The version 2 migration does not touch `targets`, `runs` (bindings), or sent `outbox` rows, and it does not rescan history (`scan_seq` is kept). Unattempted rows stay unassigned and enter the new batching path. Per run, uncertain unsent rows become one **legacy reconciliation** delivery. It sits in front of everything else of that run:

- Its destination is the run's binding, or the workflow's channel and the run's topic. With neither, its state is `unresolved` and it names `wfe notify resolve`.
- The transport searches every place for the sender's messages since the events were created. A legacy marker lists exactly the events its post carried (`ref <run> #6:answered #13:reported`), so the posts found establish the original membership. Nothing is guessed.
- A complete search marks the found events as sent with their message ids and releases the rest to new deliveries. An incomplete one marks the found events as sent and keeps the rest uncertain.
- Legacy `published <commit>` markers carry no checkpoint number; any later post naming `published` counts as that follow-up. This is a limitation: it can lose a follow-up, never duplicate one.
- Diagnostics: a delivery's `problem` says when unattempted events are ordered among or before the uncertain ones. `notes` records sent rows without a message id and each run given a legacy reconciliation. `wfe notify status` shows both.

`inflight.json` evidence: on first start the new transport hands the old file's ids and marker to `wfe notify import-inflight`. Those events join, or create, their run's legacy reconciliation before any claim. The file is then renamed to `inflight.json.retired-<time>`. If such an event is already part of a delivery frozen after the migration, the import refuses and names `wfe notify resolve`.

## Evidence

| Scenario (plan §6) | Test | Result |
| --- | --- | --- |
| Marker older than the latest 100 topic messages | `test_marker_older_than_the_last_hundred_messages`: 450 later messages (300 of them the bot's) after the lost post | Found after paging (≥ 2 pages); acknowledged; posted once |
| Search cannot reach far enough | `…_stays_uncertain`: page size 10, limit 2 pages, 30 later bot messages | No POST; `failed`, `uncertain = 1`, "history not covered after 20 message(s)"; listed in `status.problems` |
| Topic moved | `test_moved_topic_is_searched_where_it_went`: the topic, including the lost post, renamed by a person | Followed via the last acknowledged message; found in the renamed topic; one post; `relocations` holds `run-001` "left for #100 › run-001 (renamed)" |
| Search fails | `test_search_failure_is_not_absence`: three retries while history reads time out | No repost; "cannot establish whether it was posted"; after recovery, found and acknowledged |
| Access disappears; reset | `test_lost_access_…`: the bot cannot read the channel; then the channel is reset and reprovisioned | Nothing posted to the new channel while the old place cannot be searched. `reset` names the delivery as still looked for. After `resolve --not-posted --by`, the exact frozen content goes to the new channel once |
| Legacy database and transport | `test_legacy_state_is_migrated_and_reconciled`: the **released p5 editor (`32498e9`, from Git history) and transport (`daeaada`)** produce the state on the fake Zulip. `#2` and `#4` are sent; `#6` is posted with its answer lost; the old code then grows the batch to `#6 #13` and fails to connect, marking both uncertain; `inflight.json` names `#6` only; `#15` is pending | The new releases migrate (`user_version` 0 → 2). The import confirms `#6` is already in the legacy reconciliation, and the file is retired. The search finds `#6` (acknowledged, not reposted); `#13` is established as never posted and goes out once, followed by `#15`. Sent rows keep their original message ids; the legacy delivery is `resolved` with "1 event(s) found posted; 1 not posted"; a migration note is recorded |
| Pre-change fixture with inconsistent states | editor `migration: …` (verbatim p5 DDL) | Partial acknowledgment (`#3` sent, `#4` of its batch uncertain), an unattempted `#5` between uncertain `#4`/`#6` (diagnosed), a sent row without a message id (noted), a run with no channel (`unresolved`, needs a person; resolved with `--not-posted`, events released). Reopening applies nothing twice; `user_version = 99` refused |

The legacy test also shows a defect of the old code: it marked `#13` uncertain although it had never been posted. The new reconciliation treats it correctly by evidence.

Suite results: editor `npm run check` passes (130/130). `test_wfnotify.py` 18/18 in three consecutive runs. In the combined `test_wfnotify.py` + `test_wfexec.py` run, one of three repetitions failed in `test_legacy_state_is_migrated_and_reconciled`: `wfe exec heartbeat` died at `ExecStore`'s `PRAGMA journal_mode = WAL` with `SQLITE_BUSY_RECOVERY` (261). This is the step 4 defect in the unchanged `ExecStore`, seen naturally in the suite. An earlier combined run had one failure in `test_growing_batch_is_not_posted_twice` whose output was truncated ("Value…"); it was not captured and did not recur in six later runs of that test. Both are rechecked after step 4.

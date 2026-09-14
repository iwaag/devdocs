# better_zulip_call p1 — step 7: deployed, demonstrated, measured

Date: 2026-09-14, 05:49–06:23 UTC on this host.

## Deployment

- `nctl drift` before and after: **converged=46, 0 errors**. No placement
  changed; every job keeps its command, its label and its log.
- pyagag pushed (`e1de7b5`) and every consumer relocked and synced to it:
  agentroom, agfront, agautolab, agforge, agobserver, comfynotify,
  arxivsage, cagent. Each carries a "Lock pyagag at the mirror, the listener
  and the budget" commit; pj-agdev's submodule pointers follow.
- The frontend rebuilt (`npm run build`), the nginx image rebuilt and the
  `web` container restarted; `:8090` answers 200.
- The relay's plist regenerated from the template without
  `AGENTROOM_ZULIP_ENV`, installed, `bootout` then `bootstrap`
  (05:49:59 / 05:50:31). Every listener `kickstart -k`'d between 05:49:33
  and 05:49:45; the notifier once more at 05:53:58 with the fix below.
- Ignored environment notes (`pj-agdev/.local/devenv.md`), the developer
  guide (`README_DEV.md`), the relay's README, the launchd README and the
  pyagag README describe the new shape.

Every restarted process filled its mirror in **62 calls, about a second**
(seven listeners, 361 calls in thirty seconds, once per store) and reported
`recovery (startup): 0 conversation(s) queued from the index` — the realm
was quiet. The relay's restart was 67 calls (62 for the fill, register,
`users/me`, two internal event calls) and live in 1.2 s; the step 1 baseline
was 120 calls for a shallower copy.

## The demonstration

| time | what happened | evidence |
|---|---|---|
| 05:51:09 | Omni Agent posts a request in `#front › front-bzc-p1-demo`: have the Observer watch for a file and notify this conversation | message 6982 |
| 05:51:09 | Front served in the same second (intake off the mirror's feed) | agfront log |
| 05:51:25 | Front asks permission first (its standing rule); the notifier's new queue intake answers Front's reply with a refusal — **a defect**: the queue carries every public message and the old narrow had only ever returned mentions of the notifier; Front served again and addressed the notifier — one paid run | messages 6984–6987 |
| 05:53:58 | fix (`CommandIntake.addressed`), test, notifier restarted | `comfynotify` commit |
| 05:53:50 | permission granted (6988); Front delegates with `agentchat` | agfront log |
| 05:54:02 → 05:54:20 | Observer serves `watch-demo-done`, accepts `w6994` with `front/front-bzc-p1-demo` as the destination; looks every minute | agobserver log |
| 05:56:46 | **Observer restarted mid-watch**: the mirror's queue resumed (no resync), the worker resumed `w6994` from its store (look 3 at 05:57:24) | agobserver log |
| 05:56:47 → 05:57:08 | a second watch `w7000` opened and accepted | agobserver log |
| 05:58:11 | its topic ✔'d by hand | Omni resolve |
| 05:58:14 | the file created | — |
| 05:59:06 | `w6994` met on look 5, delivered as message 7005; `w7000` **cancelled** on the same tick, confirmed by one read | agobserver log |
| 05:59:06 | Front served in the same second by the notification; replies "notification received" | messages 7009–7010 |
| 06:01:0x | preview: the watch topic already ✔, the conversation ready; **0 Zulip calls**, 4 reads of the copy | `/complete/plan` |
| 06:01:18 | close: 275 ms; revalidation read **2 listings** (`agobserver-agstudio1`, `front`); the resolve applied and **confirmed** by the mirror | `/complete` |
| 06:01:21 | a second client's `/work` no longer lists the conversation; Zulip has `✔ front-bzc-p1-demo` | curl, Omni listing |

The whole flow, per credential, 05:51–06:01 (server log): Front 94 calls
(three servings: 54 topic reads, 10 posts, 5 listings; the rest event
deliveries), Observer 48 (intake, three finishes, 20 event deliveries),
notifier 17 (its restart), Omni 6, `Opsroom Observer` 9 (event deliveries
only), Developer **1** (`users/me` from the chat client at the relay's
start). No 429.

## The step 1 scenarios, repeated

| scenario | step 1 | now |
|---|---:|---:|
| idle listening, per hour, all callers, calls beyond long polls | 708 (notifier 696, Observer 12) | **0** (twenty minutes 06:03–06:23: 432 long polls, nothing else) |
| board reload, cold | 36 | **0** (25 reloads of five boards in two seconds: no relay call) |
| board reload, warm | 0 | 0 — and there is no cache to go stale |
| completion preview | 12 / 59 | **0** |
| completion apply (writes excluded) | two more walks + a 36-call reload | **2** listings |
| relay restart | 120 (55 channels) | 67 first fill; **0** within the queue's lifetime |
| listener restart | 69–84 | 62 first fill; **0** within the queue's lifetime (Front, 05:49:45: resumed) |
| Observer, active watch, per idle tick | 1 per watch (+3 per ten ticks) | **0** |
| Observer finish | ~10 | ~10 |
| one Front serving | 13–25 | 13–25 (unchanged) |

**Visible update latency** on the deployed relay: Front and the Observer
were each served within the same second of the post that owed them; a
board read after the close showed the change at once (the close had
waited for the mirror's confirmation, which arrived inside its 275 ms).

**Correctness beside the counts**: the delegation, the acceptance that
names nobody, the restart, the cancellation, the notification, the
callback serving, the reply, the preview reaching the watch topic through
Front's root note, the close and the second client's view all behaved as
the contracts say. The one thing that did not was the notifier's intake,
found on the first request and fixed within the hour.

## Removed

`sweep_serve`, `sweep_topics`, `sweep_mentions`, `sweep_rootchats` and
their 38 tests; `Room`'s cache and per-request client; `Ops`'s sweep, queue,
subscription writes and event handlers; the Front Desk's reader;
`AGENTROOM_ZULIP_ENV`, `AGENTROOM_CACHE_SECONDS`; the notifier's five-second
narrow poll.

## Limitations that remain

See `report.md`. The two worth repeating: tools outside `agag.zulip` are
outside the budget, and a serving's own reads are unchanged.

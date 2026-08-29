# sage p1 — step 5 report: run the listener

The arXiv sage listener is running under launchd.

## Deployment

- Added and pushed `service/com.agdev.arxivsage-zulip.plist.in` in arxivsage
  commit `3a9977d`.
- First ran `service/listen.sh` in the foreground. It authenticated as Zulip
  user `20`, registered both receive queues, and completed a clean initial
  sweep with no awaiting topics.
- Replaced that foreground process with the launchd job
  `com.agdev.arxivsage-zulip`. Its launchd state is `running`, with the
  checked-in `service/listen.sh` as its program and the listener log under
  `.local/out/`.

## Liveness evidence

The listener wrote `.local/agag-status.json` after a successful Zulip poll:

```json
{"last_error": null, "last_poll_ok": "2026-08-29T14:44:40.604736+00:00", "schema": "agag.status.v1"}
```

The status file is intentionally Git-ignored and is the liveness artifact
that a later desired-state declaration can observe.

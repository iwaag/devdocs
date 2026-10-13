# Workflow editor p8/ex1 — deployment

**Deployed** 2026-10-13 at the person's request. agdevworld `5d4519c` is the installed release; the previous release is `8cda5c9`.

## What was run

- **Before**: `agdevworld/scripts/wfe-release.sh status .local/agdev` showed current `8cda5c9`, previous `a571ae2`, and no open attempt. The agdevworld checkout was clean at `5d4519c`, pushed.
- **Deploy**: `agdevworld/scripts/wfe-release.sh deploy .local/agdev` (exit 0).
  - It built `service/releases/5d4519c…`, ran `npm ci && npm run check` there (211 tests passing), and switched `service/current`.
  - It restarted `com.agdev.wfe-service` and confirmed that the service and `bin/wfe` run `5d4519c`.
  - The log line "the run changed since it was read" is expected output from a unit test, not an error.

## Checked separately afterwards

- `wfe-release.sh status`: current `5d4519c`, previous `8cda5c9`, no open attempts.
- `GET http://127.0.0.1:8098/api/version` and `GET http://localhost:8093/wfe/api/version` (the agdevworld route): both report agdevworld `5d4519c`, tree `35f6587`, built 2026-10-13T02:43:15Z, source "release".
- `bin/wfe version`: release of agdevworld `5d4519c`.
- Read-only browser look at `http://localhost:8093/wfe/#/` in Simple: the dashboard shows both registered projects (シンプルRTS（対ボット）, PowerPoint agent) as tiles with their purpose, "Nothing running" and Open project, with the Simple | Detail toggle. No page errors. Nothing was written.

## Rollback

`agdevworld/scripts/wfe-release.sh rollback .local/agdev` returns to `8cda5c9`. No store or file format changed in p8/ex1. The only new stored item is the per-browser `workflow-editor.display` key, which the older release ignores.

## Still open

- **Native macOS Japanese IME**: not verified (report6).
- **First visit**: viewers start in Simple. Their stored Compact/Mini choice is kept for Detail.

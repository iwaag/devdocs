# p12 Step 1 — page contract and serving

agdevworld `afb33e2` (pushed). Source only; deployment is Step 4.

## What was built

- **`server/boast.ts`** with the routes `GET api/boast/<project>` (state),
  `api/boast/<project>/c/<commit>/<path>` (published) and
  `api/boast/<project>/w/<workspace>/<path>` (working-tree preview).
  - The state route resolves the project's `main` once through the project
    access caches and returns `state: present | absent`, `commit`, the
    page's `entry` URL and provenance (`fetch.state` fresh/recent/stale with
    the error). Absent is a state; an unknown project, missing revision or
    unreadable Gitea is an error with the access failure state.
  - The page and every relative asset are served from the commit in the URL,
    so a view never mixes revisions. Commit-addressed content is
    `immutable` with the blob id as `ETag`. A commit not yet in the cache is
    fetched for. Only full commit ids address content.
  - Preview reads one workspace's working tree, excludes ignored files (they
    are never published) and is `no-cache`; its URL differs from published.
  - A folder URL without the slash is redirected (relative URLs resolve
    inside it); a folder serves `index.html`. WFE builds nothing.
- **Boundaries.** Only `boast/` is served: encoded traversal → 400; a Git
  symlink is followed only inside `boast/`, else 403; working-tree paths
  are checked through `realpath`; `.git` → 403.
- **Isolation.** Every content response carries
  `Content-Security-Policy: sandbox …` without `allow-same-origin`, so the
  page runs in an opaque origin embedded **and** opened alone; writes from it
  carry `Origin: null` and are refused by the existing origin check.
  `Access-Control-Allow-Origin: *` lets module scripts and `fetch()` of the
  page's own files work from that origin. Flags Safari rejects
  (`allow-presentation`, `allow-orientation-lock`) were dropped after the
  WebKit check logged them.
- **`server/staticServe.ts`**: media types, `byteRange`, `sendStatic`
  (304/206/416, If-Range, streamed Git blobs and files). Extracted from the
  presentation server, which now uses it; no presentation server is started
  for Boast.
- **Shared areas.** `GET api/shares/<share>/<path>?inline=1` displays a
  file with its own media type, ranges and the same sandbox (the existing
  route stays a download). `GET api/share-ref?ref=shared://…` resolves a
  reference on this host (`state`, display/download URLs, host and path).
  Neither depends on run presentation URLs.
- **Reference**: `workflow_editor/docs/boast.md` opens with the shared
  introduction verbatim; the rest is optional technical reference (paths,
  sources, boundaries, browser capabilities, shares, messages). No operating
  instructions for projects.

## Tests

`test/boast.test.ts` (7 tests): absent vs failure; page and assets of the
resolved commit after `main` moves on; ETag 304, folder redirect, commit-id
only; sandbox/nosniff/CORS headers and refusal of an `Origin: null` write;
206/416 ranges; traversal, symlink escapes (committed and working tree),
`.git`; preview distinct from published, ignored files hidden; shared
resolution, inline display and download. Full WFE suite: 387/387 pass,
`tsc` clean.

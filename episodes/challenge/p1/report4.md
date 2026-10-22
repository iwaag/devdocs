# Challenge p1 — Step 4: aghangout discovery and editing

## Delivered (aghangout `6e311e4`, Gitea `developer/pj-aghangout`)

One component, `app/src/challenges.ts` (`ChallengePanel`), used in three
places: the chat room's 課題 tab, the meeting room's 課題 tab, and a page of
its own (`?challenges`, `?challenge=<id>[&at=<commit>]`) linked from the
lobby ("課題を見る（未解決 N件）").

- **Cross-project list**: text search (title, body, id, projects), a project
  filter built from every project named, and 未解決 / 解決済み / 取り下げ /
  すべて with counts. The filters stay on the page while rows are redrawn
  (focus and IME composition are kept); only the newest read is shown.
  Writes not yet published are listed with 再試行 / 破棄.
- **Detail**: title, state, projects, creator and last change, revision,
  body, evidence (images as thumbnails), history on demand (each revision
  opens as a pinned read), source links from the history's
  `Challenge-Source` (`?conv=<id>`; a discussion id opens its meeting room).
  A pinned read says which version it is and offers the latest.
- **Editor**: title, projects, state and a free Markdown body — nothing else
  is asked. The draft lives in this browser with its request identity and
  base revision until WFE publishes it:
  - published → 「記録しました（版 xxxxxxx）」 and the draft is dropped;
  - unsynced → said plainly (not recorded yet), the draft stays, WFE retries,
    再試行 is in the list;
  - a conflict (409, or a draft older than the current revision) → the
    newer version is shown beside the kept draft, with 「この下書きを最新版の
    上に保存する」 or 「下書きを捨てて最新版から編集」;
  - an uncertain network failure → saving again is the same request.
- **Record from a message**: 「課題に記録」 on the current line and History
  (chat) and on every transcript message (discussion). The body starts as a
  quote with its author, date and `会話 <id> の #<n>`; the source is sent as
  `source` and becomes the commit's `Challenge-Source`. The partner's (or
  the speaking participant's) project is the default.
- **Attach to the draft**: 「添付」 in the list, 「この版を発言に添付」 in a
  record: the composer shows 「添付: 課題 ch-xxxx「…」 ✕」 (removable, kept
  per conversation in this browser). Sending posts `challenges:
  ["<id>@<commit>"]` — the revision the person looked at. A failed message
  keeps its attachments for Retry/Edit.
- **Sent messages** show 「課題 ch-xxxx「…」 送信時の版 xxxxxxx」 and
  「・最新版あり」 when it moved; clicking opens that revision.
- **Start a discussion with it**: 「この課題で議論を始める」 opens a new
  meeting room with the record attached and its title as the discussion
  title.
- Opening the tab, listing, reading and attaching send nothing to WFE but
  reads; no model is called and no run starts (checked against the
  fixture's write counters).

## Fixture and checks

- `app/fixture/challenges.ts`: WFE's challenge routes in memory (three seeded
  records, revisions, pinned reads, stale-base 409 with the current record,
  the same request once, unsynced writes and retry), message attachments
  with the latest revision beside each; controls `POST /fixture/challenges
  {unsynced | later}`.
- `checks/browser.ts` `challenges()` + `narrowChallenges()`: 34 new checks —
  lobby count; list, resolved state, text and project filters, search focus;
  record, evidence, history; reading writes nothing; record from a message
  (quote, source, default project), draft survives reload, published with
  revision and next id; attach, remove, attaching sends nothing, the message
  carries the revision; sent vs newer version and opening each; a conflict
  keeps the draft and saves on the newer revision; unsynced keeps the
  draft and retry publishes; a discussion started with the record carries
  it; record from a discussion message; phone width without sideways
  scrolling.
- Chromium and WebKit: all checks pass (242 in the whole run). `npm run
  check` (types, build, unit tests) passes.

## Next

Step 5: guidance for agents, and the first real challenge.

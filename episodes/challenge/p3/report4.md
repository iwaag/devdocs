# Challenge p3 — Step 4: work in aghangout

Code: pj-aghangout `a5088c5`:

- `app/src/challenges.ts` and a new `app/src/markdown.ts`;
- `api.ts`, `local.ts`, `main.ts` and `styles.css`;
- the fixture and the browser checks;
- the README section *Challenges*.

It is deployed in step 6 ([report6](report6.md)).

## What a person sees

- **A challenge** (`?challenge=<id>`):
  - its summary, rendered;
  - 資料, the source files: image thumbnails, audio and video players, and
    open/download links;
  - 取り組み, the trials;
  - the management state, with resolve, withdraw and reopen in a box of
    their own;
  - its history and sources.

  A challenge without `summary.md` says so and shows its files. Nothing is
  invented.
- **A trial** (`?challenge=<id>&trial=<name>[&at=<commit>]`) shows its
  rendered summary when it has one, all of its files, and its own history
  (`history?trial=`).
  - The list shows only the name, the summary title (or 要約なし), the file
    count, and who last changed it and when.
  - There are no outcome badges, and nothing is derived from the contents.
- **States**:
  - The standard three are shown in words.
  - Any other state is shown as written (`parked（独自）`).
  - An absent state shows as 未指定 and never as 未解決.
  - The list can be filtered by 未解決 / 解決済み / 取り下げ / 独自 /
    未指定 / すべて, each with a count, plus one button per custom state.
  - A text hit inside a trial summary shows 「取り組み <name> に一致」 with
    an excerpt.
- **Recording a trial**: 「＋ 取り組みを記録」 is available on any challenge,
  resolved and withdrawn included, and the page says it changes no state.
  - It takes a name, an optional summary (`summary.md`) and files.
  - The limits (32 MiB per file, 64 MiB per record) are checked before
    sending.
  - A new trial is sent without a base. Adding to an existing trial names
    the revision that was shown. If someone changed one of the same files
    first, the save comes back as a conflict: the draft is kept, the newer
    revision is read, and saving again writes on it.
  - The name and summary survive a reload (localStorage). Chosen files do
    not, and the page says so.
  - Unsynced saves, retries and a repeated identical request behave as they
    do for edits.
- **Editing the summary** sends only the fields that changed, so WFE keeps
  hand-written frontmatter. A custom state is offered as written, and
  未指定 stays unspecified unless someone changes it.

## Pinned and latest stay distinct

- A pinned read (from a message attachment or a history entry) shows that
  commit's summary, files and trials. Every file URL carries `at=<that
  commit>`, and trials opened from it stay at that revision.
- A note says which revision is shown and names any trials that exist in
  the latest revision but not in this one (`latest.trials`). 「最新版を見る」
  opens the latest. Neither view is silently replaced by the other.
- A revision from before p3 (`layout: legacy`) resolves its
  `../evidence/<id>/…` links through the files' repository paths.
- The page address follows the challenge, trial and revision on screen. Any
  WFE directory name is accepted as an id.

## Rendering safety

The Markdown reader builds DOM nodes only and never assigns HTML:

- `<script>` in a summary is shown as text.
- `javascript:`, `data:` and other non-http links are not turned into
  links; their words remain.
- External `http(s)` links open in a new tab.
- Relative targets resolve to files of the challenge at the viewed revision.
  `joinPath` refuses to climb above the repository root.
- WFE serves the files themselves with `nosniff` and a `sandbox` CSP
  (report2).

## Checks

- Unit tests: 29/29. The new `test/markdown.test.ts` covers path resolution
  for trial, nested and legacy links.
- Build: `tsc` and `vite build` pass.
- Browser checks against the fixture: **Chromium 324/324, WebKit 324/324**
  (281 before this step). The new checks cover:
  - a failed trial with an image, a take, a `../../` link to shared input,
    download, and script safety;
  - an idea-only trial;
  - a trial without a summary;
  - a pinned revision that lacks a later trial, with the latest one click
    away;
  - an evidence link at a revision from before p3;
  - custom and unspecified states, shown and filtered;
  - search matches inside trials;
  - recording a trial on a resolved challenge, which stays resolved. This
    covers a draft surviving a reload, a per-file conflict that keeps the
    draft, an unsynced record, the same request on a second save, and a
    retry;
  - the trial view at phone width, without horizontal scroll.
- The fixture models directories as snapshots, with per-file conflicts and a
  revision from before p3.

Screenshots (ignored): `pj-agdev/.local/agdev/pj-aghangout/.local/shots/`
(`*-desktop-trial.png`, `*-narrow-trial.png`, …).

## Limits

- Chosen files do not survive a reload. The page sends flat file names: only
  `wfe challenge publish` keeps a directory tree's structure.
- There is no general file editor. Arbitrary structures are written through
  the CLI or Git.

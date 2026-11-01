# p19 step 4 — aghangout through its in-system agent

Plan: [plan.md](plan.md), step 4.

## Commission

- After the contract was published (agdevworld `119bf44`), Omni requested
  `aghangout/freework/run-002` (receipt `p19-aghangout-message-files`) with the
  Developer's words as `original-input.md`, the contract and fixture on the
  share (`shared://agshare/workflow_editor-p19/contract/`, a copy of `119bf44`),
  limits, deployment expectations and the evidence wanted. Omni did not edit
  aghangout.
- After WFE was deployed, `aghangout/freework/run-003` (predecessor run-002,
  receipt `p19-aghangout-live-check`) asked the agent to check the served page
  against the live WFE and correct what differed.

## What the agent did

- **run-002** (13:01–13:26 UTC, one interruption resumed by the
  recovery-agent): recorded its own graph (r2: build-client →
  fixture-and-checks → deploy → tidy → human-review), built `app/src/files.ts`,
  `filestore.ts`, changes in `room.ts`, `api.ts`, a fixture from the contract's
  bytes, browser checks, and deployed **`90c6d05`** (`a833273` README).
  Behaviour: clipboard image paste (text paste untouched), file dialog, drag
  and drop, pending previews and removal, limits from `attachmentFiles`, WFE's
  error codes in words, file-only sending, same token and payload on retry
  (a new token after an edit), pending files kept across reloads (IndexedDB as
  bytes — WebKit lost `File` objects), inline images by `preview`, original
  downloads by WFE's `api`, no file controls without `attachmentFiles`.
  Reported: `npm run check` 38/38, browser checks in Chromium and WebKit,
  screenshots, and that the served page was not yet checked against a real
  WFE.
- **run-003**: in its own check room `p19-live-hangout-check` on the served
  page and live WFE `47a83cc`: paste, file chooser, drop, remove, reload with
  files pending, text+files (#551) and file-only (#552) messages, inline image,
  three downloads with `attachment` disposition and hashes equal to WFE's;
  WebKit for history/downloads. Nothing differed; no new release.

## Omni's assessment (what was checked)

- Read both reports and the diff `3c94b6c..a833273` (650 lines, 13 files;
  URLs only from WFE's `api`, names sanitized for WFE's rule, refusals per
  error code). The served release is `90c6d05` (`hangout/current`).
- Run-003's claims match WFE: `wfe conversation attachments
  p19-live-hangout-check --check-bytes` → #551.1, #551.2, #552.1 verified, #552
  text empty; the screenshots exist in the run folder.
- Omni's own use of the served page (report3): pasting a real 800×500 image
  and choosing a text file sent #542, #544, #547, #549 with correct
  associations; the pending preview shows the real image; from agpc's Chrome
  the room shows the image inline (800×500 decoded) and both downloads give the
  original sha256.
- **Accepted.**

## Findings recorded by Omni (not corrections to the client)

- run-002 completed its `human-review` node as a self-review instead of
  preparing a review for the person, and finished without waiting for the
  backend check it had said it would do. Raised in run-003's request; run-003
  added no review node and explained why.
- run-003's two messages are recorded under the configured person `iwaag`
  (the served page writes as the configured person). History was not changed;
  annotation 6 (correction) in `p19-live-hangout-check` says they came from
  aghangout-agent's automated check.

## Remaining limits (from the agent's report, confirmed)

- No real OS clipboard paste or native Finder drag (events built with real
  `File`s in the page).
- Limits, live refusals and a failed send/retry on the live WFE were exercised
  only on the fixture.

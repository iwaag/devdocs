# Workflow editor p7 — step 3: reviewable presentations

Commit: agdevworld `25aa19f` (steps 3 and 4 share one commit because the review screen needs its server; this report covers the record, the bundle, the review content and the decision binding, and [report4](report4.md) covers the server lifetime).

## Changes

- **Record** (`shared/run.ts`).
  - `presentations.<p>` holds: id `p<n>`, originating node, title, project-relative folder, entry, review, the files by hash, `revision` (manifest digest), registration (who, when, attempt), `state`, the last observed `instance`, and the last observation.
  - Two operations. `presentation.register` is an executor operation, fenced by the attempt, from a running node, and refused after completion. `presentation.observe` records this host's facts: `available` / `stopped` / `failed` / `stop-failed`. It is not fenced. After a cancellation only the shutdown facts are recorded, and a failed start never overrides an available instance.
  - The field is absent until the first registration, so every existing record replays unchanged. The schema stays `ag.workflow-run.v2`.
- **Frozen bundle** (`server/runs.ts` `registerPresentation`, under the run's lock).
  - Copies the regular files of the source directory into `devdocs/runs/<wf>/<run>/presentations/<p>/content/`. Dot-entries and `node_modules` are skipped, and symbolic links are refused. The bundle is bounded to 2000 files / 200 MiB.
  - Copies the review Markdown (≤ 64 KiB) to `review.md` and writes `manifest.json`. The revision is `sha256` of the canonical `{kind, entry, review, files}`.
  - If the record refuses the registration, the copy is removed. A same-receipt retransmission returns the earlier registration.
- **Review content** (`shared/review.ts`). A safe Markdown subset is rendered on the server: headings, paragraphs, lists, emphasis, code and links. Raw HTML becomes text. Links are kept only for `http(s)` or a relative path without `..`.
- **Review screen** (`server/presentServer.ts`).
  - The page shows the result (`/content/<entry>`, in a frame) and the explanation side by side, with the title, run, presentation id, short revision, and where to answer: the Zulip request topic for runs that came from Zulip, otherwise the agdevworld run view (`present.json` `answerBase`). It says the page records nothing.
  - The screen runs no script (`default-src 'none'`). Only manifest files are served: not `manifest.json`, not the raw `review.md`, nothing outside the folder.
  - Each instance is its own origin, never the editor's.
- **Decision binding.**
  - `wfe run ask <run> <node> --presentation <p>` from the `confirms: result` node requires the presentation to be `available`. The file layer re-hashes its files on disk; the reducer compares them with the manifest (`presentation-changed`) and records `question.presentation = {id, revision}` with the files as `subject`.
  - `run.decide --question` is refused with `presentation-superseded` when a later presentation exists. Together with the existing `subject-changed` and latest-question rules, this means a stale answer never accepts a newer revision.
  - The decision records `confirmation.presentation`.
- **Completion.** The completion that would complete the execution is refused while a presentation is open (`presentation-open`).
- **Coordination** (`shared/run.ts`, `shared/notify.ts`, `server/notify.ts`).
  - `presented` (critical) goes to the run and request topics with "[Open the review](url) — opens on the execution host". The address is resolved from the present store when the message is rendered.
  - `cleanup-failed` (critical) says how to retry.
  - The question message links the presentation instead of listing every file.
  - `decided` says "Cleanup pending: presentation p1 still served" while one is open.
- **Run view** (`src/runView.ts`).
  - A prominent "Open the review: <title>" button sits in the header while a presentation is observed live.
  - When the record says available but the host does not serve it, the view says "recorded … but not served now" (no dead link). When stopped, it says "no longer served; its files and decision stay".
  - The *Presentations* panel shows state, observation, questions and decisions, with Start again / Stop.
  - The open question names its presentation.
  - "accepted · cleanup pending" appears beside an accepted decision while the run is not complete.
- **Executor brief.** It lists the run's presentations (`presentationBrief`).
- **CLI.** `wfe present register|start|status|stop|reconcile` (`wfe help present`) and `wfe run ask … --presentation`. The browser's generic operation route no longer accepts `presentation.*` (host facts) and will not accept `run.adjust`.

## Verification

- `test/present.test.ts` (8 tests, real processes):
  - safe rendering;
  - the frozen copy: later source edits do not reach it, dot-files are excluded, and p2 is a new revision;
  - the review screen serves the result and explanation, refuses non-manifest paths and a forged shutdown, keeps no port in `run.json`, and reuses the instance;
  - binding: asking before the presentation is available is refused; `presentation-superseded`; the decision records the revision; acceptance proceeds to cleanup with exactly one decision; completion is refused while open, and the run completes after a verified stop;
  - the "Cleanup pending" and "Open the review … opens on the execution host" message texts.
- Full suite: **184/184 pass**. `tsc` is clean and the build succeeds.
- `checks/review.ts` (browser on this host, private service :8199, a real instance), 17/17:
  - one prominent review link pointing at the live instance, worded "opens on this host";
  - one click opens the review screen, where the game's own module script runs ("game running" in the frame) beside the explanation (1008 + 432 px on one screen);
  - the answer link goes to the run view;
  - `manifest.json`, `review.md`, `content/../manifest.json` and `registry.json` return 404;
  - after the stop: no live link, "no longer served", and the record is kept;
  - no console errors.
  - Screenshots `p7-review-runview.png`, `p7-review-screen.png`, `p7-review-stopped.png` (ignored, `agdevworld/.local/wfe-checks/screenshots/`).

## Limitations

- The intended route is the person's browser on the execution host. The default bind is loopback and every message says so. A reachable bind needs `present.json` `publicBase` and was not exercised.
- The review screen shows the entry in a frame. A result that refuses to be framed, or needs focus for keyboard input, needs a click into the frame, and "open alone ↗" is offered.
- The review text is prose. Whether it really names what changed and what to check is the executing agent's job; the workflow guide asks for it (step 6).

## Next

[report4](report4.md): instance lifetime, stop, recovery and cancellation.

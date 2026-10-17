# Workflow editor p12 — final report

p12 gave every project a freely authored showcase in the Project Editor:
`boast/index.html` in the project root ([braindump.md](braindump.md),
[plan.md](plan.md)). Step reports: [report1.md](report1.md) (serving),
[report2.md](report2.md) (editor surface), [report3.md](report3.md)
(project workflows), [report4.md](report4.md) (deployment and trials).

## What changed

| Where | Change |
| --- | --- |
| WFE serving | `api/boast/<project>` resolves `main` once; page and assets from that commit (immutable, ETag); explicit working-tree preview; confined to `boast/`; CSP sandbox without same-origin, embedded or alone; byte ranges; `staticServe.ts` shared with presentations |
| Shares | inline display route for files and browsable folder listings; `api/share-ref` resolves `shared://` on this host |
| Project Editor | full-width Boast card in Simple and Detail; frame never detached by refreshes; Published / Preview workspace, Reload, Open in new tab; update offered, never auto-reloaded; optional `height` / `copy` / `resolve-shared` messages with a manual-copy fallback; neutral absent state, distinct failure state |
| CLI and docs | `wfe boast status|reference`; `docs/boast.md` opens with the shared introduction, the rest optional technical reference; `docs/delegation.md` no longer describes the removed p10 acceptance gate |
| Integration | an acceptance recorded during an attempt queues integration at the attempt's end, as a person's decision through the service does |
| rts-vs-bot | `boast` workflow; `build-game` (moved to schema v2) and `improve-game` delegate to it after their human check |
| playprey | `boast` workflow; `onboard` and `compare` delegate to it after their confirmation; `knowledge/workflows.md` lists it |

No shared agent guide, generator, template or other project mentions
Boast; nothing requires a page, a format or an update cadence.

## Revisions

agdevworld `afb33e2`, `54bf7ce`, `a6161ec`, `2dcd056`, `a1bfedf` (deployed:
service and `bin/wfe` report `a1bfedf`); pj-agdev pin `41fa0ce`.
rts-vs-bot workflows `0730858`; playprey workflows `51f7a8b8`. Pages on
main: rts-vs-bot `0554425`, playprey `504ad1d`.

## Results

- Both project views show agent-authored pages: RTS (Japanese, playable
  copy of the game in `boast/play/`, screenshots) and playprey (five cases,
  live delivery buttons through `resolve-shared`, opening share listings).
- Both `boast` workflows ran directly (run-001 in each) and by delegation
  (rts `improve-game/run-006`, playprey `onboard/run-006`): dispatch, child
  creation from the request post, the child's confirmation answered by the
  parent's agent, return, disposition, parent completion. No human approval
  step was added; the agents added their own review steps.
- Parent and child integrations preserved each other's work; later parent
  merges kept the newer `boast/`.
- An agent may skip the showcase: playprey `onboard/run-005` dropped its
  `boast` node for an unchanged profile.

## Limitations

- **Publication order.** Before `a1bfedf` the child integrated first in
  the RTS trial (the page carried its own copy of the result, so nothing it
  advertised was missing). The fix that lets the parent's accepted result
  reach main while it waits is covered by a test, not yet by a live run.
  Without any acceptance (RTS `build-game`, or an unanswered confirmation)
  the parent integrates only at completion, after its `boast` node.
- Opaque-origin pages have no `localStorage`; the copy dialog fallback was
  not exercised on the person's own browser; frames use a default height
  unless a page reports one (neither page does yet).
- A RTS page that copies `game/` into `boast/play/` duplicates the game in
  Git; that is the project agent's choice and editable in its workflow.
- Committed content is cached for 10 s; new commits show within a minute
  (poll) or on Reload.

## Where to look

Open the Project Editor of `rts-vs-bot` and of `playprey` in the agdev
dashboard (`/wfe/` on the agdevworld web host); `wfe boast status
<project>` prints both addresses. Local addresses are in pj-agdev's ignored
`.local/devenv.md`.

# p19 step 6 — invariants and the transition

Plan: [plan.md](plan.md), step 6.

## Required checks

- agdevworld `npm run check` at `47a83cc` (inside `scripts/wfe-release.sh
  build`): tsc, **651/651** tests, both vite builds. (An earlier run at
  `3fade95` had 2 failures from the non-idempotent v14 migration; fixed before
  the release, report2.)
- agautolab at `a9b82f8`: 51 passed, 1 skipped.
- aghangout's own client suite is the agent's (report4).

## The plan's cases

| Case | Evidence |
| --- | --- |
| File-only first message; mixed; adjacent duplicate names | test 1–2 (ordered rows, original bytes, title from the file name, distinct ids/bundles); live `p19-check` #542 file-only by paste, rows and hashes in `wfe conversation attachments --check-bytes` |
| Same retry; changed retry; concurrent identical retry | test 3: duplicate returns the same message and request ids; bytes/text/author/no-files/text-token → `token-conflict`; three concurrent → 1 message, 1 request, 1 file row, 1 operation |
| Crash in staging, after publication, after acceptance before response | test 4: `failed` with no message and no staging left, retry keeps identities; `published` without a message, retry adopts; `pending` request dispatched once by `reconcile`, retry returns the same message |
| Restart after acceptance before dispatch | test 4 (third case): the existing reconcile dispatches the accepted request once |
| Destination changes while files publish | test 5: addressee leaves → `left`, nothing written, bundle kept, same token succeeds after re-join; default moves → routed again (no stale route); `attachments.json` changed → reserved destination kept |
| New run, answer, instruction, conductor wake-up | test 8 (all four, plus a later request citing the message); live: front/forge discuss runs (new), ponderer conduct/run-067 started with #547 and its instruction i1 carried #549 (same standing run) |
| Two projects and a second execution host | live: front and forge (studio) and forge/freework/run-005 on agpc described the image; agpc fetch via mount, without mount (coordinator) and HTTP → identical sha256 (`.local/p19/agpc-fetch.txt`) |
| Boundary sizes/counts, malformed input, share outage | test 6 (20 ok/21 refused, exactly 32 MiB ok, over-total refused, malformed base64 and names, bad hint); share marker removed → `storage-unavailable`, text still started its run; live via nginx: 89.5 MB body → WFE's JSON 413 `totalBytes`, 90.5 MB+ → WFE's 413 `requestBytes` (`.local/p19/proxy-limit.txt`) |
| Unsupported/active media, missing/altered bytes | test 7: HTML and SVG-in-a-`.png` served as downloads; altered → 409, missing → 410, `verify` states, history kept |
| Archive/export and backup restore | test 9; live (report5): fresh archive clone with manifests identical to the share's, export→restore 5 files verified, backup 3/3 verified on agpc, `backup --restore` verified |
| Existing challenge attachments | `test/attachments.test.ts` (challenge pins, run inputs, archive) passes; live `20261024-101244-hqyvjt` #344 reads `ch-0004@8c5743557006`; conduct/run-029 `usable` |

Fixture checks were not taken as evidence of image access: that is report3's
live evidence (four agents, two hosts, correct descriptions of randomly
chosen features, plus a text file's random code).

Memory at the limit (isolated instance of release `47a83cc`): one 64 MiB
message took 311 ms and raised the process's RSS from 364 to 904 MiB
(`.local/p19/memory-at-limit.txt`; the measuring client ran in the same
process). agstudio has 512 GiB; bounded streaming was not needed.

## Transition rehearsal

- A copy of live `conversations.sqlite` (`.backup`, v13: 97 rooms, 541
  messages, 277 requests, 703 request events, 6 challenge attachments) was
  opened with the new code: v14, every count unchanged, all 5 challenge-pinned
  rooms read the same `kind/id/commit`, 0 uploads and file rows, listing 97.
  The migration creates no message, request or synthetic attachment.
- `wfe run check --workspace <each>` with the new code against the live
  registry: all **20** workspaces exit 0 (`usable`).
- Backup before activation: every area SQLite store and the configuration
  JSONs (path in `pj-agdev/.local/p19/backup-path.txt`; v13, newest message
  #541).
- **First records an older release cannot read:** the store itself (v14 is
  refused by 799c039: "newer than this release understands"); and in run
  records, an `input.attachments` or `source.attachments` entry of `kind: file`
  (rejected as "must be a challenge"), or an answer/instruction with empty
  text. The first live ones are message #542 and front/discuss/run-014's
  input. Compatibility shims were not written (not required).

## Not covered

- An actual service crash was injected only in tests (hooks), not on the
  deployed coordinator.
- The `routing-changed` refusal (three consecutive moves) is code without a
  dedicated test.

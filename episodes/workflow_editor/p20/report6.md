# Step 6: verifying the product contract

Agdevworld `66bdad5`. Service tests use bare repositories as the remote. Browser checks use a scratch service, against the real Gitea test project `wfrun-trial`, whose checkout is 48 commits behind and dirty. Scripts and screenshots are in `pj-agdev/.local/workflow-editor-p20/check/`.

## Checks run

- `npm run check` at `d053143`: 733/733, typecheck, both builds. The two tests added after it run again in the release build (step 7).
- `test/drafts.test.ts` at `66bdad5`: 17/17.
- `check/ui.mjs`: 27/27. `check/ui2.mjs`: 26/26.
- `check/load.mjs` on the 21 registered projects, a scratch service with the full registry and an empty cache:

| Measure | Result |
| --- | --- |
| First head of all 21 at once (cold cache) | max 9.6 s |
| Project list read (warm) | median 422 ms, p95 1123 ms |
| 21 views open for 60 s, 402 checks | median 151 ms, p95 262 ms, max 581 ms, no failures |
| The real service's `api/workspaces` during this | 161 ms before, 93 ms after |

These figures are from a scratch service, one run. They do not measure the executor heartbeat or the intake pass under the deployed release; those are looked at in step 7.

## Scenarios

| Scenario | Evidence | Status |
| --- | --- | --- |
| External writer pushes | `ui.mjs`: open graph updated 2.5 s after the push, no sync | Checked; a new run recording the source: trial run below and `publishedSource.test.ts` |
| Agent integrates | Same polling path, but an actual integration | **Step 7** |
| Checkout behind, dirty or missing the workflow | `ui.mjs`, `drafts.test.ts`; checkout state compared before and after | Checked |
| No checkout at all | HTTP test: head, list, read and drafts answer | Checked for these routes; the project page's top half still needs one (report 5) |
| Draft save and restart | `ui2.mjs`: service killed and restarted, view reconnected, draft kept, published unchanged | Checked |
| Draft trial | `ui.mjs`: run from revision 3; `drafts.test.ts`: recorded id, revision, sha256, baseline | Checked to run creation; the scratch service has no executor, so the run did not execute. **Step 7** |
| Apply / Save and apply | `ui.mjs`, `ui2.mjs`; earlier run shows its old definition after the project moved | Checked |
| Concurrent upstream and draft | Merge and GUI conflict resolution | Checked |
| Two tabs, repeated apply, lost response, restart | Browser for tabs and restart; `drafts.test.ts` for the rest | Checked. A lost response is simulated by editing the journal, not by cutting a connection |
| Definition in a submodule | `drafts.test.ts`: publication, pin, moved past the pin, unfinished pin, a new run at the new pin | Fixture only: **all 21 real projects are directory mode** |
| Source unavailable or malformed | `ui2.mjs`: stale with cache, unavailable without, missing, malformed, repaired in the GUI | Checked |
| Historical or retired run | `ui.mjs` for historical | Retired: **step 7** |
| Existing local edits | `ui2.mjs`, `drafts.test.ts` | Checked on one project; every project in step 7 |

## Found by these checks and fixed

Beyond the two in report 3: the Developer details panel closed itself on each list refresh. It now stays open.

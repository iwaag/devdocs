# Challenge p3 — Step 5: guidance and verification

## Guidance

All guidance is applied at the cutover, because it names commands that exist
only in the new WFE.

| Where | Revision | What it says |
| --- | --- | --- |
| WFE contract `workflow_editor/docs/challenges.md`, `wfe challenge help` | agdevworld `73eb720` | The layout and the optional summary convention. The suggested content (tried, learned, files, open questions) is not a checklist. Also: states, per-path conflicts, limits, external artifacts, HTTP and execution nodes. |
| agautolab wfrun guide, *Challenges (challenge p1, p3)* | `1b65813` | Facts only: a challenge is a directory; trials and their evidence sit inside it; shared input is linked; `publish` works from any node; a base is needed only to replace a file; recording changes no state. |
| pj-front `agent/guide.md` | `4784f9a` | Work on a recorded problem is a trial of its challenge (`publish --trial`). Source material uses `--add`. |
| gimmickimg context block `improvement-feedback` | r3 | Points to ch-0001 and says to record attempts as its trials. |
| pj-challenges `README.md` | `3a8f4f9` | The directory layout and how it is written. |
| aghangout `README.md` | `a5088c5` | The page's trial features. |
| `devdocs/README_DEV.md` | this commit | New section, *Challenges as directories with trials*; the p1 section points to it, and its limits are marked resolved. |

The guides for forge, archsage and ponderer do not describe challenges
themselves; they read the shared wfrun guide, so I left them unchanged.

## Verification matrix

| Behaviour | Where checked |
| --- | --- |
| Independent trial additions on one stale revision both publish | `challenges.test.ts` (report2) |
| Same-file conflicts: refused at once, or kept as a draft at publication | `challenges.test.ts`; aghangout browser checks (per-file conflict keeps the draft) |
| Duplicate retries are one commit | `challenges.test.ts`, `challengeApi.test.ts`; browser (same request on a second save) |
| Interrupted publication (failed push, crash after push) | `challenges.test.ts` |
| Direct Git writes are part of the challenge | `challenges.test.ts`; live migration commit `3a8f4f9` |
| Custom and absent states are kept and shown; edits change only named fields | `challenges.test.ts`; browser (独自 / 未指定 filters) |
| Nested files; no symlink escape; size limits; 413 | `challenges.test.ts`, `challengeApi.test.ts` |
| Historical reads in the old layout; pins keep their hash | `challenges.test.ts`; rehearsal and live `verify.json` (4/4) |
| Rebuild from a repository clone | `challenges.test.ts`; live recovery from the agpc mirror (report6) |
| UI: failed, idea-only and no-summary trials; a new trial on a resolved challenge | browser checks (Chromium/WebKit 324/324); deployed check (report6) |
| Relative image, audio and download links at current and pinned revisions | browser checks; deployed check |
| Migration with pre-migration attachments and evidence | report3 rehearsal; report6 live |

Required checks:

- WFE `npm run check`: 538/538, run again by the release build of
  `a252289`.
- aghangout: unit 29/29, build, Chromium 324/324 and WebKit 324/324.

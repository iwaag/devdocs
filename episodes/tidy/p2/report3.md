# tidy p2 — Step 3 report: unprompted trials

Date: 2026-10-27 (JST). WFE `538e96d` for trial A, `115872b` from fixture trial C on. Every room was created by
"Omni Agent (for Developer)", and each brief said it was an Omni request on the Developer's behalf. The
Developer said nothing in any of them. Fixture children were operated by hand as `fixture-agent`, a
synthetic actor. Machine details are in the ignored `pj-agdev/.local/tidy-p2/`.

## Trial A — the real case (room `tidy-p2-trial`, ponderer conduct/run-049)

**Brief (#456):** make protoprey's location sound effects (agsfx `protoprey_locations` r3) usable from any
copy. agsfx should provide and describe the result, and protoprey should adopt it and verify installation
from a fresh copy. It ended with "evaluate each; summarize what is done, verified and left". It named
neither the run-copy path nor any other defect.

| Time (UTC) | Event |
| --- | --- |
| 06:07 | run-049 starts; d1 → agsfx `freework/run-001` (publish r3 as a run output, add "For other projects") |
| 06:11:24 → 06:11:41 | d1 taken up, judged satisfied after 17 s. The note says what was checked: "o1 published on share with sha256 digest, 20/20 hashes match source, README has 'For other projects'; caveats … stated honestly. Content of audio not checked." |
| 06:12 | d2 → protoprey `freework/run-009` (re-adopt the published reference, verify a fresh copy), dispatched only after d1 |
| 06:16:57 → 06:17:23 | d2 judged satisfied after 25 s: "Read child's report.md: … fresh clone install verified hashes, 8 unit tests and play scenes pass. Presentation finding n/a … Not tested: other PC / remapped share root. Audio not heard." |
| 06:17:44 | #458: one summary covering both children, with the remaining limits (no other PC or remapped share root, nobody listened, agsfx's producer tree recorded as `modified`) |

**Independent verification (Omni):**
- agsfx main `89000eb0a6`: README "For other projects" (offer, how to ask, what arrives, limits).
- Share: `runs/agsfx/freework/run-001/o1/` holds README, `audio/` and `delivery.json`.
- protoprey main `c4db96ffb5`: `location_sfx.source` = `shared://agshare/runs/agsfx/freework/run-001/o1/`,
  `wfe_output` digest `sha256:804d5a89…`.
- A fresh Gitea clone, without wfe and with `PROTOPREY_SHARE_ROOTS=agshare=/opt/nfs/agshare`, ran
  `tools/deliveries.py install location_sfx`. Every hash was checked and status reported `installed`.
  `python3 -m unittest discover -s tests` ran 8 tests, OK, matching the 8 `test_` definitions.

**Outcome:** p1's open finding is resolved. Both results were sound, so this trial shows a correct
acceptance with recorded checks and a complete summary. It does not show gap detection. No intervention.

Two smaller points:
- the adoption record's `replaced` stays empty although the source changed;
- d2's assessment says "report: none recorded with the result", while run-009's report was recorded as an
  artifact.

## Trial B — isolated fixture, first form (room `tidy-p2-fixture`, conduct/run-050)

**Fixture:** agops workflow `evidence-fixture` (`0a0e08c`). It uses execution profile `hand`, which no
host offers, so no agent is launched and Omni operates its runs. The brief (#459) asked for two
independent small tools under `fixtures/evidence-check/`, each with a README that works as written on
another PC and with tests that actually pass.
- `slugify` (run-002): sound, with an honest report.
- `wordfreq` (run-003), with gaps planted and known only to the evaluator:
  - the README hard-codes this host's paths and `/opt/homebrew/bin/python3.13`;
  - one of four `test_` functions sits after `unittest.main()` outside the class, so 3 run;
  - the report claims "4 tests OK" and "works on another PC; nothing outstanding".

**Design error (mine):** the workflow had `intake: {zulip: ""}`, which opts out of delegation. ponderer
correctly fell back to `wfe request` plus conversation `ask`, so no delegation assessment was involved.

**What ponderer did (#461, unprompted):** it noted that neither run returned a result message and read
the copies directly. It ran both test suites itself. It reported slugify as done and verified (3 tests,
matching), and wordfreq as failing on all three planted points: the host paths, the false portability
claim, and "4 claimed, 3 ran". It recommended sending wordfreq back, but asked the person whether to do so
instead of returning it. Fixed for the next trial: intake enabled (`c1d84dc`).

## Trial C — isolated fixture, fresh context (room `tidy-p2-fixture2`, conduct/run-051)

The same shape with new tools: `titlecase` (run-004, sound) and `linecount` (run-005, the same three
planted gaps). ponderer read run-050's definition as precedent and used `wfe request` + `ask` again,
though delegation was now possible.

| Time (UTC) | Event |
| --- | --- |
| 07:06:58 | #463, unprompted. titlecase verified: it re-ran the tests (3) and the CLI. linecount: the host paths and the false portability claim; "4 tests claimed, 3 run; `test_longest_line` is a module-level function after `unittest.main()`" (from the source and its own run). It offered a correction request and asked whether to send it |
| 07:07:48 | **Intervention 1 (#464):** "please send linecount's correction; check it and report when it returns". This is a go-ahead with no defect information |
| 07:09 | d1 (a delegate node) → run-006, with a precise request that names both defects and asks for the real output of a 4-test run |
| 07:18 | run-006 returns a **partial** correction, planted: README fixed, test still outside the class, report again claiming 4 tests with their names |
| 07:18:31 → 07:18:50 | **Independent failure:** ponderer judged d1 satisfied after 19 s, "consistent with reported 4 tests … Tests not re-run". The take-up had shown `Tests: did the named checks run …?` and `Next: compare the claim with the files` |
| 07:19 | Unprompted, before speaking in the room, it carried out its own plan ("戻り次第テストと README を再確認"). It read `test_linecount.py`, found the test still outside the class, and returned it to the same child with `delegate revise` (correction 1: "Claimed fix not present in recorded artifact") |
| 07:20 | run-006 adds a node, fixes the test (4 run) and appends to its report that its first output was wrong |
| 07:20:43 → 07:21:05 | judged satisfied after checking `test_linecount.py`. The assessment carried "This result follows correction request 1 … The judgement of event 2 (satisfied) … does not cover it: check the correction itself" |
| 07:21:16 | #466: linecount corrected and re-checked, titlecase done, both complete. The reply output listed `d1 → agops evidence-fixture/run-006: judged satisfied (event 4)` |

**Independent verification (Omni):** run-006's final `test_linecount.py` runs 4 tests, OK. Its README has
only relative paths and plain `python3`.

## What the trials show

| Question | Answer |
| --- | --- |
| Did the receiver detect a gap before any Omni hint? | **Yes, three times** (B #461, C #463, C's partial correction at 07:19). The evidence was its own test runs and reading the source, not the outcome line |
| Did it return the gap to the correct owner? | Partly. In B and C it asked the person before sending a correction. After the go-ahead, it sent a precise correction to a new child of the right workflow. For the partial correction it returned the result to the **same child** unprompted |
| Did it verify the corrected result? | Yes for the final one (read the test file). For the first correction it judged before checking, then reopened it itself |
| Was everything rejected? | No. Sound results were accepted in A (×2), B and C, with stated checks |
| Does the final summary cover every child and its limits? | A: yes, with limits. B/C #461 and #463: yes. C #466: covers both, but drops the intermediate failed correction and "another PC not verified" |
| Unnecessary tidying, repeated warnings, circular corrections? | None. One correction cycle. `results-unreported` never fired, because the conductor always spoke before resting |

## Mechanism observations

1. **Producer advice bug, found in trial B, fixed in `115872b`.** The CLI read changed paths from the
   project workspace instead of the run's copy, so no contextual question was chosen. After the fix,
   run-004, run-005 and run-006 got the documentation and tests questions with their basis.
2. **Precedent copying.** run-051 reused run-050's dispatch form. Requests through `wfe request` + `ask`
   bypass the delegation assessment and `revise` entirely. ponderer still assessed well by hand. The
   evidence prompts only reach the delegate path.
3. **Attribution side effect.** Reopening a judged result requires the person's words (`--because`).
   ponderer cited #464, so correction 1, which was ponderer's own finding, is recorded "from Omni Agent
   (for Developer)". Because the correction "relays" those words, the assessment then reads
   `review-unrecorded` as ok. Neither the Omni go-ahead nor ponderer's finding was the person's review of
   the result.
4. **Two result events from one completion.** A hand-operated run with a final-report step completes when
   its work node completes (documentation steps do not hold completion), and again after the report. With
   no attempt open, the first event's facts are frozen before the report exists. Real runs freeze at the
   attempt's end. This is a fixture artifact, not observed in trial A.

## Interventions (not independent success)

- **#464:** permission to send the correction ponderer had already proposed. It carried no defect
  information.
- **Fixture changes between trials:** intake enabled; new tool names.

No other message was sent to any room. No hint about any planted defect was given.

## Next

Step 4: verify the artifacts again, compare with p1, and decide whether the premature judgement (07:18:50)
warrants a guide or tool change.

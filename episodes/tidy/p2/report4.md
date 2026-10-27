# tidy p2 — Step 4 report: verification and the remaining gap

Date: 2026-10-27 (JST). Final release: agdevworld `17c52b3` on studio and agpc.

## Delivered work, checked independently

| Item | Evidence | Result |
| --- | --- | --- |
| agsfx publication | share `runs/agsfx/freework/run-001/o1/` (README, `audio/`, `delivery.json`); agsfx main `89000eb0a6` README "For other projects"; report `devdocs/runs/freework/run-001/report.md` (sha256 `f07cde13…`) present on main | Passed |
| protoprey re-adoption | main `c4db96ffb5`: `location_sfx` source `shared://agshare/runs/agsfx/freework/run-001/o1/`, output digest `804d5a89…`; report `devdocs/runs/freework/run-009/report.md` present | Passed |
| Hash-based acquisition from a clean consumer | a fresh Gitea clone with no wfe and no producer run directory, `PROTOPREY_SHARE_ROOTS=agshare=/opt/nfs/agshare python3 tools/deliveries.py install location_sfx`: every hash checked; status `installed` from the share reference | Passed (on studio; another PC was not tried, as both children said) |
| Claimed tests | protoprey: `unittest discover -s tests` ran 8, the source has 8 `test_` definitions. Fixture run-006 (final): 4 run, 4 defined | Passed |
| Fixture outputs | none reached agops main (`fixtures/evidence-check` absent at `c1d84dc`): hand-operated runs were never integrated, so the planted defects stay on their run branches | As intended |

p1's open finding (protoprey's adoption pointing at agsfx's run copy, and agsfx's undescribed delivery) is
resolved.

## Compared with p1

| p1 failure | p2 observation |
| --- | --- |
| ponderer accepted a nonexistent report within 14 s, from the outcome line | In every trial ponderer read reports or files before judging. Its notes name what it checked (hash match, test count, README content) and what it did not (audio, another PC) |
| A README with host addresses was accepted unseen | Host paths were found unprompted twice (B, C) |
| run-007's test that never ran was found by Omni | "4 claimed, 3 ran" was found unprompted twice by running the tests. Once (C, 07:18:50) a claimed fix was accepted from the report, then reopened by ponderer itself within a minute |
| Omni had to name both defects before revisions were requested | No defect was named by Omni. One go-ahead (#464) was needed before the first correction was sent |
| After two corrections ponderer reported one | Every summary named each commissioned result. C's final summary dropped the intermediate failed correction and the "another PC" limit |
| The tidy run read the adoption record without its status check | Trial A's protoprey child verified acquisition from a fresh clone (the request asked for it). Tidy r2 now names the check |

**Sample:** one real trial (2 children) and two fixture trials (5 children, 1 correction cycle), all through
ponderer, with one intervention. This is evidence that the evidence-reading behaviour is reachable, not
proof of reliability.

## The remaining gaps

1. **Asking before returning.** Twice ponderer proposed a correction and asked the person instead of
   sending it. It sent the correction after a go-ahead, and it returned a partial correction to the same
   child without asking. The briefs asked it to "evaluate and summarize", so asking may be its reading
   of scope rather than a defect. Nothing was changed for this.
2. **A premature judgement** (C, 07:18:50). The take-up showed the tests question, and ponderer answered
   it from the report's own output. Change: the receiver's tests question now says that the report's
   output is the child's claim, and asks the receiver to run or read the tests or to state the counts as
   unverified (`17c52b3`). **Not yet re-tried in a fresh context.** It was deployed after the trials.
3. **Requests outside delegation get no assessment.** `wfe request` + conversation `ask` (B, C, copied as
   precedent) bypass the evidence prompts and `revise`. ponderer assessed by hand anyway. Extending the
   prompts to awaited request runs is a candidate, not done.
4. **Reopening needs the person's words.** For a judged result this misattributes the agent's own finding
   to the person (correction 1 "from Omni Agent (for Developer)"). It also makes `review-unrecorded` read
   ok. Recorded as a limitation of p18's rule.

No rigid gate was added. Gaps 1 and 3 can be revisited if they recur.

## Ordinary and recursive work

- Unrelated small work gets the general questions only. The tests cover this (`src/main.py`: no
  contextual question, no reminder). In the trials, titlecase and slugify children got one line about the
  entry, the documentation and tests questions that their own changes chose, and nothing else.
- A correct tidy or delivery result started no further tidy work. No child added a tidy node unasked. The
  producer advice says "report its evidence rather than tidy again" for a tidy run.
- No reminder repeated. `results-unreported` never fired, because each conductor spoke before resting. The
  unit test shows it clears on the next post.

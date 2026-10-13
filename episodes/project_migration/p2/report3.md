# project_migration p2 step 3a — the study run (2026-10-13)

`study/run-001` in jevlab: a follow-up to `study/recent_study` for papers posted 2026-10-08 to 2026-10-13. Requested with `wfe request` by the Omni Agent for the Developer; the plan was agreed by the Omni Agent with the Developer's explicit permission; the result was judged by the Developer in the browser over the LAN ("ok、完了してください"). Completed and published (`98a5a92`).

## Result

- 9 new papers in the window (1 false positive excluded), recorded as a follow-up section of the catalog, a search log of its own (`search-log-p2.md`), a reinvention-check section and a README addendum.
- Two evidence-card additions, both marked abstract-level: a one-character edit raising a hosted judge's false acceptance from 1.0 % to about 26 % (2610.11436), and a calibration/framing/cost benchmark (2610.11392).
- Nothing closes an item on the lab's "still unpublished" list or changes a reinvention-check verdict.
- Two gaps the agent stated itself: the two general web searches were not repeated, and the two cards are abstract-level. The Developer accepted the result with them.
- The published files carry no host name, address, absolute path or token.

## Attempts

| Attempt | Ended | What happened |
| --- | --- | --- |
| 1 | awaiting person | plan written, q1 asked |
| 2 | **interrupted** | arXiv's export API answered 429; the agent backgrounded `sleep 60 && curl`, ended its turn to wait, and the harness killed the background command with the session. `survey` was left running; the run was held. |
| 3 (after resume) | awaiting person | paced arXiv's HTML search pages 40–45 s apart in foreground retry loops (about 25 min for 12 queries), recorded, asked q2 |
| 4 | completed | took up q2, final report |

## What the system lacked, and what changed

- **The run guide did not say that a session's background work dies with its turn.** autolab's `workrun_supercoder` guide had learned it on 2026-09-26; `wfrun` had not. Added to `agautolab/agent/guides/wfrun/guide.md` with this run as the evidence; attempt 3 followed it (a 600 s foreground loop that the harness moved to the background was then waited on in the foreground, not abandoned).
- **Resuming a held run and answering for the Developer are blocked for the Omni Agent by default.** Both were refused by the permission classifier until the Developer approved them by hand. The p1 statement "Omni agreed the plan as the Developer's proxy" therefore needs that approval each time.
- The final report attributes the waiting rule to "the jevlab-agent guide"; it is the system's run guide. Minor.
- arXiv rate-limits this host heavily (both the export API and the search pages). A study run that needs arXiv should expect minutes per query; this is a fact of the host, recorded here rather than in the project guide until a second run confirms it.

## Side change during this step: the services are on the LAN

The Developer asked for every service to be reachable from the LAN (`localrule.md`). agdevworld `783cfcb` (released): the editor binds every interface and answers any host name, accepting browser writes from the page's own origin; nginx publishes `:8093` and passes the request's Host; the relay binds every interface; the browser builds both URLs from the host the page was opened from; presentations bind every interface with a LAN `publicBase` and `answerBase`. The q2 answer above came through that route.

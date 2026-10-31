# Guide coverage review

The pre-change common guide is preserved in `previous-guide.md`. The deployed guide lives in `pj-agdev/agautolab/agent/guides/wfrun/` (relative to the projects root). Its reference files are read on demand, not appended to every prompt.

| Previous content | Current home |
| --- | --- |
| Identity, communication, workspace, tools | Short guide; request and delivery header; existing WFE brief retains the exact execution identity |
| Sessions, state, resumption and interrupted work | reference/recovery.md; unchanged WFE continuation and reconstruction logic |
| Requests, answers and instructions | Request header; reference/work.md; current input remains in the WFE brief |
| Challenges and pinned source material | reference/material.md; CLI help owns publication and argument details |
| Conversation replies, attribution, participation, conductor lifetime | reference/delivery.md and reference/conversation.md; applicable project role guides remain in scoped context |
| Node catalog, graph revisions, retrospective records and reminders | reference/work.md; catalog and current nodes remain in the existing brief |
| Agreements versus permissions | reference/work.md; no new gate |
| Presentations, review, cleanup and requested changes | reference/delivery.md and reference/work.md; CLI help and existing state-aware brief |
| Shared areas, inputs, outputs and missing installed assets | reference/material.md |
| Documentation, outcome and evidence | guide.md, reference/work.md and reference/delivery.md; documents stay optional |
| Delegation, capacity, questions, assessment, correction and cancellation | reference/delegation.md; detailed operations remain in CLI help |
| Git publication, branches, integration and reversal | guide.md and reference/recovery.md |
| Long incident narratives and inaccessible episode references | previous-guide.md for developer provenance; essential environmental facts retained in operational text |

Additional implementation checks: reference paths are absolute on the execution host; reference hashes participate in the offered contract; delegated requests retain their requester route even if they carry the original conversation source. Browser requests are not instructed to post to a nonexistent conversation room.

This phase does not filter existing scoped context or rewrite the coordinator's brief. Their current facts and project instructions remain intact. It also does not introduce an automatic incident-retrieval service. The references and existing tool responses are the available retrieval mechanisms.

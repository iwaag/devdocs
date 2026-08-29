# sage p1 — step 6 report: failure-farming acceptance test

Ran the acceptance topics as Omni Agent against the launchd listener. Run
records and transcripts are retained in the ignored agent workspace.

| case | result | sage run record |
|---|---|---|
| a: Prime Agent harness | Answered from `papers/2608.23552/summary.md`, cited the public GitHub file, and made no queue note. | `run-0001`, $0.1045988, 6 turns |
| b: absent AutoDesign paper | First attempt failed to read `chatlog.md`; after the guide correction, it honestly reported unknown and created `tostudy/autodesign-2608-13560.md`. | failed `run-0002`, $0.046697, 1 turn; successful retry `run-0003`, $0.1178524, 11 turns |
| c: AutoDesign rephrase | Appended one dated `asked again` line to the same note; no duplicate file. | `run-0004`, $0.139622, 11 turns |
| d: Python stdlib | Correctly answered from general knowledge and named the sage-scope boundary. | `run-0005`, $0.0564512, 2 turns |
| e: non-entrance topic | Returned the fixed redirect sentence; run-record count stayed unchanged. | no model run |
| f: Front relay | Front read the published introduction, asked sage in an `entrance-` topic, and relayed the cited answer to `#front`. | `run-0006`, $0.0877712, 4 turns |

Total arXiv-sage model cost for the six acceptance cases, including the
observed failure and retry: **$0.5529926**.

## Observed failure and correction

The first unknown-paper run said it could not see a question even though the
rendered `chatlog.md` contained one. The guide had described scope but did not
explicitly require reading that file. Added the first instruction, "Read
`chatlog.md` first," in arxivsage commit `212317c`, pushed it, and re-posted
the changed introduction. The retry passed.

## Queue artifact

The created file contains the original question, why AutoDesign is in scope,
its arXiv id/title, the requested `summary.md` artifact, and the dated repeat
line. It is ignored by Git as designed; a future study-side consumer reads it,
runs the study, publishes the result, then deletes the note.

## Record metadata correction

The original pyagag record serializer dropped `extra_meta`, so its first six
records did not retain the knowledge revision. Added a generic explicit-meta
path in pyagag commit `a49ec8e` (35 focused tests passed), pushed it, upgraded
the arxivsage lock in commit `a6b4700`, and restarted the listener. A follow-up
in-scope query produced `run-0007` ($0.0856824, 5 turns), with
`knowledge_revision: "31ff73c"` in its record and a citation to
`papers/2608.23552/test.md`.

# tidy p3 — Step 3 report: awaited request results get the same evidence

Date: 2026-10-27 (JST). Release: agdevworld `b7d554a` (studio and agpc).

## The relation used

A run created with `wfe request` from inside a run records its requester explicitly:
`input.entrustedBy = {kind: run, project, workflow, run}` (p2's fixture run-005: entrusted by ponderer conduct/run-051).
That recorded relation is the only basis for ownership. Topics, names, recency and "I awaited it" are not: a
conductor can await any run (`conversation ask --run`), so awaiting grants no authority.

## What changed

| Boundary | Now | Code |
| --- | --- | --- |
| The awaited result is told (`afterResolution`: the instruction that continues the conductor) | each awaited run with a completed result is listed with its evidence: identity (completing seq, frozen position, manifest digest, observation time), the claim beside its records, the corrections it received, `to judge:` questions, and the route. Before, it said only "answered by #n" or "unanswered: ended without a message here" | `server/conversationService.ts` |
| Inspection | `wfe run result <ws>/<wf>/<run> [--requester …] [--file <path>]`; HTTP `GET …/runs/<wf>/<run>/result?requester=…` | `server/requestedResult.ts`, `cli/run.ts`, `server/api.ts` |
| One rendering | the claim/records/questions of a delegated result and of a requested result come from one function (`claimBeside`); the questions name the producer ("the requested run's claim") | `shared/assessment.ts`, `shared/resultEvidence.ts` |
| Correction route | `wfe run return <ws>/<wf>/<run> --requester <project>/<workflow>/<run> --instruction … --reason … [--evidence …] [--result-seq <n>]`; HTTP `POST …/return`. Records `correction.receive` with `request: {run}` (no delegation) on the requested run, which reopens and is continued like a delegated child's correction | `shared/run.ts`, `server/requestedResult.ts` |
| Authority | only the recorded requester: the reducer refuses another (`request-relation`); inside an attempt the attempt must be the requester's (`not-requester`). A delegated child is pointed to its parent's `delegate revise`; a run with no recorded requester is nobody's to return | same |
| Revised results, duplicates | `--result-seq` names the result assessed (`result-superseded` otherwise); a retransmission with the same receipt records once; an open correction is refused (`correction-state`) and the reader says "open for correction cN; its next completion is a new result". The awaiting conductor awaits again for the corrected result (a new ask activity), and that result is assessed anew with the correction it answers | same |

An independent request run is never shown as a delegated child: the evidence says "requested by <run>" (or "delegated
by …" for a real child, or "no requesting run recorded"), and the route for an observer is "You did not request …;
observing it gives no authority to return it. Tell <its requester or the person>".

Contracts: `docs/runs.md` *Corrections* (*From the requesting run*, `wfe run result`), `docs/conversations.md` (`ask`
row).

## Tests

- `test/requestedResult.test.ts` (new): the requester's view (claim, records, the tests question about the requested
  run, the return command with `--result-seq`); an observer gets no return route; return by a non-requester, for a
  stale result, inside a foreign attempt and while a correction is open is refused; the correction carries
  `request.run`, `from`/`by` = the requester's agent and the evidence; the run reopens and gets a continuation job; a
  retransmission records once; the reducer refuses a requested correction on a run nobody requested and one that
  names a delegation too; `awaitedStanding` lists only the conductor's own awaited runs.
- `test/discussions.test.ts`: the instruction for an awaited run that ended silently now carries
  "result of sage/discuss/run-001: completed at seq …" and its claim lines.

Live read with the release (read only): `wfe run result agops/evidence-fixture/run-008 --requester
ponderer/conduct/run-052` shows the baseline's csvstat result, both corrections and "a delegated child … its parent
assesses it with wfe delegate show and returns it with wfe delegate revise; no other run returns it".

## Limits

- The manifest of a requested result is computed when it is read (file hashes now), not frozen at the completion as
  a delegation's relayed manifest is; the completing seq and frozen record position are the identity. A file changed
  in the run's copy after completion would show the new hash.
- The HTTP `result`/`return` routes are covered by the shared functions' tests, not by an HTTP test of their own.

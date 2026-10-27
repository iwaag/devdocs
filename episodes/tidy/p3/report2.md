# tidy p3 — Step 2 report: correction provenance and review derivation

Date: 2026-10-27 (JST). Release: agdevworld `b7d554a` on studio and agpc (`16cb127` first, see *Regression*).
pj-ponderer `395f3c4`. Machine details and rollback: the ignored `pj-agdev/.local/tidy-p3/notes.md`.

## What changed

| Fact | Before | Now | Code |
| --- | --- | --- | --- |
| Who found a correction | `from` = the agent, or the person whose words `--because` names | the same records, read as an **origin**: `agent` (the recording agent's own finding, with `--evidence`), `person` (relayed words), `review` (a human review's follow-up). Derived for every record, stored only when a p3 operation supplies it | `shared/delegation.ts` `originOf`, `originText` |
| Reopening a result the agent judged satisfied | only `--follow-up <rv>` or `--because` with the person's words | also `wfe delegate revise --reopen --evidence "<what you checked>"`: the agent's own finding reopens its own judgement of the latest result. The satisfied disposition is kept in `reopened[]` with `origin: agent` and the evidence. `--because` with `--reopen` is refused | `shared/run.ts` `delegation.revise` |
| Correcting a recorded attribution | not possible without rewriting | `wfe delegate attribute <run> <d> <n> --origin … --from … --reason <evidence> --by <you>` appends an attribution; the original `from`/`because` stay; nothing is sent to the child | `delegation.attribute` |
| The person's review of a delegated result (`review-unrecorded`) | `ok` when any correction or answer relayed the person's words | a correction (relayed words, a go-ahead or the agent's finding) is listed as a fact and is never review evidence; a relayed answer counts only when it answers the child's own question about its result (a result confirmation, or one asked with its presentation). Human review rounds are unchanged | `shared/assessment.ts` |
| Readers | `correction request n … from X` | `wfe delegate show` names the origin and evidence of each correction and why each reopening happened; the child's correction carries `from` = the agent for its own finding | `cli/delegate.ts` |
| Scope of repair | not stated | `delegate revise` help and ponderer's guide: returning what the commissioned work left out needs no go-ahead; materially new work is the person's | `cli/delegate.ts`, pj-ponderer `agent/conduct.md` |

Held, cancelled and closed cases keep their semantics: an agent-origin correction is recorded while the person holds
the parent and launches nothing until the person resumes it (the hold is not lifted); `rejected`/`abandoned`
delegations stay closed for every origin; a delegation not yet judged cannot be "reopened"; a judgement of an
earlier event is not reopened when a newer result exists. Contracts: `docs/delegation.md` (*Correction*, *The agent's
own finding*, the record and operation tables, the review check).

## Historical correction

ponderer conduct/run-051 d1 correction 1 (p2 trial C) was recorded "from Omni Agent (for Developer)" because reopening
needed the person's words; #464 was a go-ahead that named no defect. At 10:21:24Z an attribution was appended (by
"Omni Agent (tidy p3)"): origin `agent`, from `ponderer-agent`, with the evidence (ponderer read `test_linecount.py`
at 07:19 and found the test outside the class; p2 `report3.md`). `wfe delegate show conduct/run-051 d1` now reads
"ponderer-agent's own finding; attributed so by Omni Agent (tidy p3) … recorded at first as from Omni Agent (for
Developer)". The d1 assessment's `review-unrecorded` now reads **UNKNOWN** (it read `ok` in p2).

What could not be corrected without losing provenance: the `reopened` entry of that delegation keeps `because`
(the go-ahead), and the child (agops evidence-fixture/run-006) keeps the correction as it received it. Both are
shown as recorded; the attribution sits on the correction request.

## Tests

`test/agentCorrection.test.ts` (new):
- the agent reopens its own satisfied judgement: `--reopen` without evidence and with `--because` are refused; the
  record keeps the acceptance, origin agent and evidence; the same child receives the correction from the agent; a
  retransmission records once; no derived field is stored (replay identity);
- the person's words keep their own path; a hold by the person stays; not-judged and abandoned are refused;
- an attribution is appended beside the original, readers show both, the child is told nothing, an unknown request
  is refused.

`test/assessment.test.ts`: corrections (relayed go-ahead, agent finding) leave `review-unrecorded` unknown with the
facts; words relayed to a question not about the result do not count; the relayed answer to the child's result
confirmation still counts. The p2 fixture's relayed answer pointed at the result event rather than the question;
it now has the real shape.

Release suite: 641/641 (`wfe-release.sh` build of `b7d554a`).

## Regression during deployment (found and repaired)

`16cb127` stored `origin` (and `followUp`) on every `delegation.revise` and `origin: person` on p18 reopenings. Replaying a
record written before p3 therefore produced a different state, and five ponderer conductor runs (run-038, 045, 048,
051, 052) read "CANNOT BE USED — inconsistent" right after activation. I saw it at once on run-051, reactivated
`17c52b3` on studio (nothing had been written in between; agpc kept `16cb127` for about ten minutes with no attempt),
and released `b7d554a`, which stores only what a p3 operation supplies and derives the rest. Before activating it,
`wfe run check` with the built release ran over every registered workspace and found no inconsistency; the broken
release shows exactly the five. That check is now in the release notes for any reducer change. The release suite
did not catch it because its fixtures are created by the same code.

## Next

Step 3 (awaited request results) and Step 4 (account) shipped in the same releases; reported separately.

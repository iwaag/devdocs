# workflow_editor fix2 — Zulip intake enabled by default: final report

A workflow now accepts Zulip requests and delegations unless it opts out
with `intake: {zulip: ""}`. Loading used to read a missing declaration as
"off", although creation wrote `requests`. That is why
`front/converse/run-045` could not delegate to aghangout. In a fresh trial
Front delegated to `characters/freework`, which has no declaration. The
child was created and its result came back to Front, which then answered
the requester.

Step reports:
- [report1](report1.md): model and serialization
- [report2](report2.md): runtime and inventory
- [report3](report3.md): contracts
- [report4](report4.md): deployment and trials

## Changed components

| Component | Revision | Change |
| --- | --- | --- |
| WFE (agdevworld) | `4be9732`, `0d49456` (released) | Loader: an omitted intake, `{}` or an omitted `zulip` reads as `requests`. A wrong type stays a shape problem. Writer: `""` is written even where the file had no key, a new file declares intake either way, and unchanged documents round-trip byte for byte. Canonical definition: intake is always projected (omitted = requests ≠ opt-out). The editor API requires intake in a model. The intake reports unreadable workflow files instead of skipping them. Delegation names an unreadable target and the opt-out. Docs: intake, contract, delegation. `wfe delegate request --help` says what a refusal means. 515/515 |
| Workflow definitions | front `83815ef`, archsage `34575c4`, forge `46ffdf9`/`d8e38cb`, jevlab `b32109b`, rts-vs-bot `61fbae3`, aghangout `3eca7c5` | The nine `converse`/`discuss` turns opt out explicitly, because they serve a conversation or a discussion. aghangout change/freework and characters character/freework become eligible |
| Front guide | front `8cb5923` | A refused delegation is not a project refusing work. Work entrusted another way stays Front's to bring back (run-045) |
| devdocs | README_DEV delegation section | Every workflow has the intake unless it opts out |
| pj-agdev | pins `c63121e` | |
| agpc execution node (`gpupc`) | wfe `0d49456` | Nautobot pin, render, `setup_wfe_worker.yml` |

## Evidence

| Case | Where | Result |
| --- | --- | --- |
| Omitted, `{}`, empty, null, explicit requests, explicit `""`, seven invalid shapes, unsupported mode | `test/intakedefault.test.ts` | as specified |
| Edit/save/reload and new files, both values; canonical equality and difference | same | pass |
| Routing: no declaration is routed, an opt-out is not, a wrong type is a reported problem | `test/intakestore.test.ts` | pass |
| Delegation to a child with no declaration; refusal of an opt-out | `test/delegateflow.test.ts` | pass |
| Live: delegation to a workflow without intake | conversation `fix2-s4-delegate` → `front/converse/run-046` d1 → `characters/freework/run-007` | child created from the request post, result relayed and judged, Front answered in the conversation (#271) |
| Live: delegation to an explicit opt-out | `agops/freework/run-001` → `front/discuss` | refused, nothing posted |
| Newly routed channels' history | `wfe intake messages` | only run-topic posts; no run created, nothing posted |

## Existing runs and the Ponderer request

- Run snapshots and recorded digests are unchanged. Nothing re-verifies
  them. No affected workflow had an approval record. In discussion
  workflow histories, a comparison across the release may call an
  unchanged file without intake "behavioural". This is display only.
- **The Ponderer publication request is complete.**
  `aghangout/freework/run-001` (a plain request from run-045) completed
  and was integrated on 2026-10-21: "Ponderer now has portrait, lore and
  style (characters export 0631d47, commit a514c72); cast.json room argue;
  checks pass; deployed at /hangout/". Its result never reached the
  Developer's conversation (`20261022-001309-j4e9lp`). It has no
  delegation and will not get one. No second publication or deployment
  was made. Verifying this fix (the trials above) is separate from that
  request.

## Remaining limits

- The auto-mode classifier refused the agpc inventory render. The
  Developer approved it by hand, and the worker then got `0d49456`
  (Report 4).
- `front/converse/run-046` ended once with its `report` node left ready.
  Recovery resumed it and it completed without a second post. This is a
  known pattern of ending nodes, not an intake problem.
- Saving an existing file that has no intake keeps it without a
  declaration. Only new files show the setting.
- `intake: {zulip: null}` (an empty YAML value) reads as the default, not
  as the opt-out.

Recovery steps, backups and machine-specific evidence are in the ignored
notes (`pj-agdev/.local/fix2/notes.md`).

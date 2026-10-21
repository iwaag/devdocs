# workflow_editor fix1 — finding user-provided files on agshare: final report

People can now name shared materials the way they think of them. In
`characters/freework/run-001` an agent told the Developer that
`/agshare/agag_folks/ponderer` did not exist, but the share held it. In a
fresh run with the same words, the agent found the folder and named
`face.png` and `lore.md`, with no hint.

Step reports: [report1](report1.md) (boundary), [report2](report2.md)
(tools), [report3](report3.md) (guide), [report4](report4.md) (deploy and
trials).

## Changed components

| Component | Revision | Change |
| --- | --- | --- |
| WFE (agdevworld) | `1e2fbed` | `wfe share resolve` and `ls` accept a reference, `/<share>/…`, `<share>/…`, `<share>:…`, a path with a share's name in it, this host's folder for a share, or a path inside a share. Each reading is one stat, kept inside the configured roots. Answers: ok, ambiguous, absent (with the nearest existing folder), unavailable (absence not confirmed), unknown-share, outside. `wfe share find`: bounded search by name that says when it was incomplete. `docs/shares.md`, `--help`, 4 tests |
| wfrun guide (agautolab) | `69142b8` | The share is where people put materials; their paths are descriptions, not this host's paths; the tools to resolve and search them; an incomplete search is not absence; ask only about ambiguity that remains; project paths stay the project's. Cites run-001 |
| agpc execution node | wfe `1e2fbed`, executor `69142b8` | Nautobot placement pins, rendered, playbook |
| pj-agdev | `4a82245` | pins |
| devdocs | README_DEV *Shared areas* | people's words, `find` |

Records are unchanged: a run records the `shared://` reference that
`resolve` prints.

## Outcomes

| Case | Run / check | Result |
| --- | --- | --- |
| Original request | characters/freework/run-002 | Found; both files named; nothing registered or changed |
| Shortened share-relative path, other node | run-003 on agpc | Resolved at agpc's mount; both files described |
| Ambiguous name | run-004 (temporary second folder) | Both matches listed with contents |
| Missing item | run-005 | Absent; the folder that does exist was named |
| Explicit project-source path | run-006 | Project file read; share untouched |
| Unavailable share | deployed CLI, unmarked root | `unavailable`, absence not confirmed |
| Search limits | tests, live `--max-entries`/`--depth` | Incomplete, never reported as absence |

`npm run check` passes 509/509. nctl: agshare server and clients
converged before the change; the agpc placement now carries the new pins.

## Remaining limits

- No agent trial for an unavailable share; it was checked at tool level,
  because taking the live share down would disrupt other work.
- When a name is ambiguous and the request only asks to look, agents list
  every match rather than ask.
- `run.create` still records only `shared://` references written in the
  words. Other descriptions are resolved by the agent during the run.
- The agpc worker's node prints an SQLite warning on every `wfe` call.
- The auto-mode classifier refused the production activation and the
  Nautobot apply; both went through only on the Developer's manual
  approval.

Recovery, machine details and raw evidence are in the ignored notes
(`pj-agdev/.local/fix1/notes.md`).

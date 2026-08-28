# scheduled_routine p6 — phase report

## Outcome

Two papers were attempted.  Prime Agent (`2608.23552`) is **verified**:
Prime Agent v0.8.1 installed and its non-interactive Ollama provider path
returned `OLLAMA_PRIME_OK`.  Apodex (`2608.23283`) is **waiting_external**:
the official FrontierAgent runtime and Docker sandbox built correctly, but no
supported container-to-host Ollama route was available.  No paper is marked
failed merely for an environmental boundary, and neither experiment is
adoption-pending or complete beyond its bounded localtest result.

## Repositories and run record

- `autodev/studyarxiv-localtest-2608.23552`: initial localtest plus verified
  result commit `3f222b5`, pushed.
- `autodev/studyarxiv-localtest-2608.23283`: initializer `c4bab45` and
  bounded-result commit `c1153db`, pushed.
- Front's one-shot localtest fires were `r10`/`e40` at 06:16:41Z and
  `r11`/`e41` at 06:36:43Z on 2026-08-28.  The first selected Prime Agent;
  the second exposed the rework-topic binding defect.  Direct Omni execution
  was used only after that live routing failure and Front's temporary Claude
  session limit.

The active work spanned roughly 06:11Z–10:33Z.  Human/Omni interventions:
rejecting Claude Code OAuth credential reuse; selecting existing local
Ollama; using the ignored per-agent Gitea credential through its established
askpass helper; and retaining the known managed Ollama service without a
Desired State edit.

## Runtime dispositions

Prime Agent used the existing agstudio Ollama service and created no
container, listener, model download, or persistent process.  cagent request
`req_c6caddd0412b4f4fbb792ee64a941f52` and scoped nctl evidence confirm it
is the active, converged `ollama-agstudio` placement.  Apodex created only an
ephemeral `apodex:local` image (~1.7 GB), then removed it; no container
remained.  No desired-state batch was proposed or applied.

## Live-evidence implementation changes

The phase added the repository-backed, resumable `localtest` initializer and
contract (agautolab `6d31db1`) and exposed a real rework workflow bug.  The
repair (agautolab `cb8556b`, parent pin `4468f54`) opens a fresh anchored
workrun topic for a completed-task change; 169 focused tests passed.  No
nodeutils discovery change was warranted because its existing macOS Ollama
process/HTTP probes already observe the retained service.

## Regular scheduling decision

The outcome is useful for manual, bounded investigations: it leaves a pushed
repository, commands, revisions, state, and an evidence-backed handoff.
It is not yet suitable for unattended recurring local experiments: Front's
rework path only just received a repair, and the Apodex sandbox networking
boundary is unresolved.  The next evidence-supported feature is a small,
documented macOS container-to-managed-Ollama connection contract (including a
read-only readiness probe), followed by one resumed Apodex one-shot.

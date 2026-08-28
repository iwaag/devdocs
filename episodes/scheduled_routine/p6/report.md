# scheduled_routine p6 — phase report

## Outcome

Two papers were attempted and both are **verified**. Prime Agent (`2608.23552`)
installed at v0.8.1 and its non-interactive Ollama provider path returned
`OLLAMA_PRIME_OK`. Apodex (`2608.23283`) built and ran the official
FrontierAgent Docker path; its one-turn ReAct command returned
`FRONTIER_OLLAMA_OK` after the effective container endpoint was corrected.
Neither experiment is adoption-pending or complete beyond its bounded
localtest result.

## Repositories and run record

- `autodev/studyarxiv-localtest-2608.23552`: initial localtest plus verified
  result commit `3f222b5`, pushed.
- `autodev/studyarxiv-localtest-2608.23283`: initializer `c4bab45`, initial
  bounded-result commit `c1153db`, and verified-result commit `e8f3c7d`, all
  pushed.
- Front's one-shot localtest fires were `r10`/`e40` at 06:16:41Z and
  `r11`/`e41` at 06:36:43Z on 2026-08-28.  The first selected Prime Agent;
  the second exposed the rework-topic binding defect.  Direct Omni execution
  was used only after that live routing failure and Front's temporary Claude
  session limit.

The active work spanned roughly 06:11Z–13:03Z. Human/Omni interventions:
rejecting Claude Code OAuth credential reuse; selecting existing local
Ollama; using the ignored per-agent Gitea credential through its established
askpass helper; correcting the ignored FrontierAgent `.env` rather than adding
infrastructure; and retaining the known managed Ollama service without a
Desired State edit.

## Runtime dispositions

Prime Agent used the existing agstudio Ollama service and created no
container, listener, model download, or persistent process. cagent request
`req_c6caddd0412b4f4fbb792ee64a941f52` and scoped nctl evidence confirm it
is the active, converged `ollama-agstudio` placement. Apodex created an
ephemeral `apodex:local` image (~1.7 GB), ran one-shot containers, and then
removed the image; no container remained. No desired-state batch was proposed
or applied.

## Live-evidence implementation changes

The phase added the repository-backed, resumable `localtest` initializer and
contract (agautolab `6d31db1`) and exposed a real rework workflow bug.  The
repair (agautolab `cb8556b`, parent pin `4468f54`) opens a fresh anchored
workrun topic for a completed-task change; 169 focused tests passed. The
Apodex evidence also identified a launcher/documentation detail: on macOS the
Docker run consumes the checkout `.env`, so a shell-only provider URL override
is not sufficient. No nodeutils discovery change was warranted because its
existing macOS Ollama process/HTTP probes already observe the retained service.

## Regular scheduling decision

The outcome is useful for manual, bounded investigations: it leaves pushed
repositories, commands, revisions, state, and verified local evidence for two
papers. It is not yet suitable for unattended recurring local experiments:
Front's rework path only just received a repair, and its corrected route was
not revalidated end-to-end after the session-limit interruption. The next
evidence-supported test is one production `localtest` fire that exercises the
repaired rework/resume route without Omni execution.

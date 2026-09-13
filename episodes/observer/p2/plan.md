# Observer p2 — real work with YuE2

## Goal and scope

Run a YuE2 study through the existing study musicgen workflow: establish basic usage and find conditions that repeatedly produce usable music. Use this real work to prove that an executing agent can discover Observer, delegate a wait, end its serving, and resume useful work after notification.

This is a private experimental environment in a breaking-change phase. Backward compatibility and migrations are unnecessary. Implementers may refactor shared code, replace experimental state, and adjust the workflow as evidence warrants. Keep implementation constraints minimal; security hardening and a general workflow framework are outside this episode.

## step1 — Make usage discoverable before the request

- Keep each agent's published introduction as the authoritative request contract. Add a shared way to retrieve an agent's introduction if the existing discovery tools are insufficient; `agentchat intro <agent>` is a suggested shape, not a required command name.
- Give execution agents the common discovery entry point through their shared startup context/help. Avoid copying Observer-specific instructions into each role's guide or the study request. Verify that the actual study executor can access the command and introduction, not just Front.
- Ensure Observer's introduction explains the condition, observable target, notification destination, cancellation, and how a requester ends its serving and resumes on the answer. Publish the final introduction when its contract changes.

Verify discovery from an actual executor's context. Shared CLI/help and context placement are implementation choices; an Observer-only documentation registry is unnecessary.

## step2 — Prepare and request the study

- Inspect the existing study musicgen workflow and knowledge tree, then use its normal entrance and delegation path. Before relying on local services, read `pj-agdev/.local/devenv.md`, `pj-clusterintent/.local/localenv_memo.md`, and current Nautobot/nctl state.
- Ask the executing agent to consult current official YuE2 documentation, establish a reproducible baseline, and explore a small set of promising conditions. Let it choose installation, runtime, generation settings, and trial order based on the available hardware.
- Explicitly request Observer use for meaningful long waits. Let the executor choose the opportunities and formulate the conditions; the request should not contain the Observer manual. A download and a generation are useful candidates, but a cached download need not be repeated just to create a wait.
- Let the executor choose and record a bounded initial trial budget, expanding it when results justify more trials. Record model/code revisions, inputs, settings, seeds, outputs, and elapsed time. Start with a baseline that produces audio before broadening the search.

## step3 — Prove waiting and continuation in real work

- Have the executor start a durable background process/job, retain its identity, output location, and next action, request observation, and end its serving. Confirm that the job survives that serving ending.
- Give Observer evidence it can actually read. A path on a different execution node is not automatically visible on Observer's host. The executor may expose a status file, endpoint, or other practical observation route; choose the smallest solution that works.
- Observe terminal completion, including failure, so an unsuccessful download or generation does not leave the requester waiting indefinitely. Process completion and output validity are separate: the resumed executor checks the result and decides what to do next.
- Trace the real chain: job start → watch acceptance → executor serving ends → Observer notification → executor resumes → result collection or recovery → next useful action. Acceptance should not wake the requester; ordinary polling should not consume requester runs.
- Obtain at least one complete chain without a human or Omni Agent manually restarting the executor. Exercise further genuine waits as the study progresses. Diagnose failures, improve the shared behavior or guidance, and repeat the affected chain. Record any manual rescue separately from successful autonomous continuation.

## step4 — Evaluate and report

- Evaluate musical usefulness separately from Observer behavior. Compare a small number of conditions with multiple seeds per condition; retain failures as well as selected tracks. Record usable/total counts and concrete judgments about audio artifacts, lyric intelligibility/adherence, and musical structure. Agent listening or human listening are both acceptable; say who judged and avoid claiming stability from selected successes alone.
- Deliver several usable tracks when achievable within the trial budget, with repeatable generation instructions and the observed limits of the promising conditions. If quality remains inadequate, report that result without treating it as an Observer failure.
- Run focused tests for changed shared/Observer behavior and the relevant existing suites. Use the real study as the integration proof; a repeat of every p1 lifecycle test is unnecessary unless the changes affect it.
- Write `report.md` with the discovery path, watch/job/conversation evidence, waiting and resumption timestamps, requester runs during waiting, interventions, music results, and remaining limitations. Keep machine-specific details and bulky evidence in ignored local files. Update relevant usage/environment documentation and commit/push changes and required dependency pointers.

## Useful starting points

- `devdocs/episodes/observer/p1/report.md` and `p1/ex1/report.md`: Front already resumed after a notification; p2 adds real executor work, durable jobs, and usable results. Known limits include sequential evaluations, per-evaluation workspace growth, and indefinitely retrying unreadable targets. Measure their effect on this study before expanding scope.
- `pj-agdev/agobserver/params/intro.md`: current request contract. `src/agobserver/{intake,observe,worker,notify}.py` implement acceptance, observation, scheduling, and delivery.
- `pyagag/src/agag/chat.py`: shared `agentchat` commands and introduction harvesting. `pyagag/src/agag/agent.py`: execution-time chat environment. `agag/cli.py` currently handles creation/provisioning, so runtime discovery likely belongs beside `agentchat`.
- Official YuE2 starting point: <https://github.com/multimodal-art-projection/YuE/blob/main/docs/generation.md>. Re-check at execution time; this is a newly released model and runtime details may change.

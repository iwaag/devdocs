# improve_wfrun p4: opinion — concurrency on an execution node

Date: 2026-11-03. Written by Claude (Fable 5.1) at the Developer's request, after the check of two aghangout conversations that night. An opinion, not a plan.

## What happened

Two conversations were started within ten minutes of each other on `studio`:

- `20261103-010919-ltkqmd` (bee flight experiment): `ponderer/conduct/run-072` delegated to `gimmickimg/partial-animation/run-002`, which took the default slot for Godot rendering and image work.
- `20261103-011819-kmqr56` (aghangout visual concept): `ponderer/conduct/run-073` replied (#587) but ended with no node recorded, so WFE held it `interrupted` (`nothing-recorded`). The Developer's follow-up (#588) was recorded as an instruction and waits for the next attempt.

Observer opened incident `i92cc65546d` and created `warden/recover/run-106` within a minute. That recovery run is a `default`-lane job on `studio`, and `studio`'s default slot was held by `run-002`. So a conversation that should have resumed in minutes waited for an unrelated rendering job, although the two pieces of work (Godot on one side, web search and writing on the other) would not have interfered.

## Why one slot per node exists (as far as the records say)

`docs/runs.md` *One slot (p9)* and `docs/workers.md` state the rule; neither gives a reason. workflow_editor p9's plan says only "retain the host's existing single-attempt execution slot". The plausible reasons, strongest first:

1. **Shared host state outside the run's copy.** Every run has its own working copy, but `/tmp`, the display, audio, GPU memory and process names are per node. `run-002` wrote to `/tmp/seq` and used `pkill -f "godot --path /tmp/fp3"`; a second Godot run on the same node could collide with either.
2. **Host load.** Heavy local work has already delayed the executor's heartbeat past the watchdog's thresholds (rename/warden notes, 2026-11-02). Two heavy attempts would do so more often.
3. **Inheritance.** Before p9 a begun run held its workspace; serial execution was then necessary. p9 removed the workspace hold but kept the slot.
4. **Simplicity of bookkeeping and recovery.** A remote worker's `state.json` is kept by one executor process; lanes are local-only for that reason. One attempt at a time also keeps incidents easy to attribute.
5. **Subscription use.** More concurrent attempts spend the usage windows faster. No record names this as a reason.

Reason 3 no longer applies. Reasons 1 and 2 are the ones still standing, and both are about *what the work uses*, not about the number of runs.

## Position

**The decision should be split.** What a piece of work uses is a judgement only the agent doing it can make well, and it may change as the work proceeds; whether two such uses collide, and whether an attempt may start now, should stay deterministic in the coordinator.

- **Non-deterministic: the declaration.** The agent (or the workflow definition, as a default) names the resources its attempt will use: `gpu`, `display`, `audio`, a `/tmp` namespace, network-heavy, none. A conductor's ponder or a literature search declares nothing; a Godot render declares `display` and `gpu`.
- **Deterministic: the check and the admission.** The coordinator opens an attempt when no open attempt on that node holds a conflicting declaration, and records the declarations and the decision in the attempt row. Agents cannot see each other's running attempts at the moment they decide, and a decision made from a stale view races; the coordinator has the whole picture and leaves a record that explains, after a collision, why both were allowed.
- **Over-optimistic declarations are expected.** Agents will tend to say "no conflict". The record of declaration against what the attempt actually touched (processes, files under `/tmp`, GPU use) is what makes the next guide revision possible, in the same way `nothing-recorded` holds made the talk-node contract explicit.
- **Host load stays deterministic and measured.** CPU, memory and heartbeat age are not something an agent can declare in advance; the executor should refuse a further claim while they are above thresholds, however empty the declarations are.
- **When to declare: at attempt or node start, not at run creation.** `run-002`'s talk node did not yet know it would render; declarations should be revisable as the graph is adjusted.

In terms of the present design this generalises lanes: `conversation` is one fixed name with a capacity; a declared set of resources is the same idea with the name derived from the work. The default lane's capacity of one becomes "any number of attempts whose declarations are pairwise compatible, within host-load limits".

## A smaller change worth doing first

**Run `warden/recover` in the `conversation` lane.** A recovery run is a short investigation, the same shape of work as a conversational turn, and its purpose is to bring an interrupted run back quickly. In the case above, that alone would have resumed `run-073` within minutes instead of leaving it behind an open-ended render. It needs one line in the workflow definition (`execution.lane: conversation`) and no change to WFE. The cost is that a recovery can delay a conversational turn by a minute or two, which is cheaper than the reverse.

Two further observations from the same night, outside the concurrency question:

- A conductor's attempt can reply in the conversation and exit without recording a node; the `nothing-recorded` hold then stops the room until recovery. The ponderer conduct guide should make the reply itself complete or start a node, or WFE should treat a reply posted by an attempt as a recorded act.
- A long delegated attempt shows `Working · Between steps` for its whole duration while its only progress is in its session log; the person reads it as stalled. Progress notes (`wfe run progress`) at natural points, or a periodic note from the executor, would have answered the Developer's question before it was asked.

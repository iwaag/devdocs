# sage p2 — archsage can establish a study: final report

## Delivered behaviour

archsage now establishes the studies its sages need, and the system carries
a request from an ordinary entrance to researched, refreshed knowledge:

1. **Tools** (pyagag `agag.project`, `agag.routine`):
   - `agproject open|status|plan` creates or continues a study channel with
     its plan and workspace request, and never duplicates what exists.
     Members are resolved explicitly, and the request carries a
     machine-readable `ag-setup` block.
   - `agroutine create|update|show|list` registers `routine-<name>` and
     posts each complete guide version as the caller, read back.
   - archsage holds both tools, with the provisioner credential from its
     ignored environment.
2. **Setup and its return path:**
   - autolab lays a study out from the block before any ordinary
     scaffolding (`README_PROJECT.md`, `main/` with plan, README, `methods/`,
     `reports/INDEX.md`) and answers with a `study layout established` line
     naming the repository and commit.
   - archsage ends its run while setup is pending (a progress reply that
     names nobody). autolab's answer brings the originating conversation
     back through its root note. This works across a restart and when the
     topic is ✔.
3. **Sages:**
   - `archsage sage add|update|attach|sync|remove|show` and `archsage queue
     …`. A sage records its study and its source; `main`, the internal
     repository, is the default.
   - `attach` checks the repository and syncs at once. A reattached tree is
     replaced only once the new clone succeeded, and a failed refresh keeps
     the tree.
   - Definitions are pushed to a private store (`autodev/archsage-sages`,
     restorable), and the introduction is re-posted on every change.
   - A queued question is settled only when the refreshed tree answers it.
4. **Guidance:**
   - archsage's guide and introduction describe establishing, connecting
     and refreshing studies, the three entry shapes, and what a study
     routine guide contains.
   - Front's `front`, `desk` and `argue` guides route studies to archsage
     and keep routine execution with Front. Front opens projects itself;
     no developer step is needed for a channel.
   - Reports keep setup complete, research complete and knowledge refreshed
     apart.

## Evidence

Step reports: [report1](report1.md) (tools, live routine probe),
[report2](report2.md) (layout, callbacks, restart-while-pending probe),
[report3](report3.md) (attach/refresh/persistence),
[report4](report4.md) (guidance) and [report5](report5.md) (the three
cases).

- **New study plus research (`aisvgs`).**
  - Front Desk #11522 → archsage #11525.
  - plan #11531, setup #11533/#11535 (`81f5276b58bc`), setup complete #11550,
    routine guide #11543, `sage:aisvgs`.
  - Front's routine run #11559 → mission m11579 → report #11682 → `main`
    `13e0d6e2d08a`.
  - acceptance recorded from #11699.
  - refresh #11694 (`81f5276b58bc → 13e0d6e2d08a`).
  - `sage:aisvgs` answered #11700 in #11709, citing
    `reports/strand3-tools.md` and `reports/INDEX.md` at `13e0d6e2d08a`.
- **Existing study, not connected (`worldtrend`).**
  - archsage #11605: study reused (`50e841c913a4`), `sage:worldtrend`
    attached, routine guide #11604.
  - Set up and correctly reported as not researched.
- **Existing sage without a study (`growbox`).**
  - archsage #11659: plan #11593, `main` `dd59ce11eed9`, routine guide
    #11596, `sage:growbox` attached.
  - Its queued note carried into the plan and kept.
  - Set up and not researched.
- The routines appear on the board, and the two setup-only ones have 0
  runs. The introduction lists the new and attached sages.

## Remaining gaps and handoff candidates

- **A research task ended while its background subagent was unfinished**
  (autolab #11648). Nothing inside the system could see the stall.
  - The Omni Agent posted once at the Front Desk (#11669) — *did X for
    Front/autolab: nudged a stalled task*.
  - The supercoder should finish subagent work within its serving.
- **A routine run was orphaned.** The nudge re-anchored the task to the
  desk, so `routinerun-20260926-2100` never got its report and stays open.
  Delegations for a run should come from the run, or a requester
  conversation needs a way to hand a result back to a run it owns.
- **Front's callback receipts were written into the run it started.**
  This is fixed (agfront `7bf5956`), but before the fix it produced one
  false Observer escalation to the Developer (#11666), which could not be
  withdrawn afterwards.
- **archsage's selector rule.** A request *about* a sage that begins with
  `sage:<name>` reaches the sage. The introduction now warns against this.
  A structural fix (for example, treating a leading selector from an agent
  in a `study-` topic as a mention) is possible if it recurs.
- **Accounts and identity:**
  - The sage store is pushed with the `autolab-agent` Gitea token, so a
    Gitea account of archsage's own is a Developer decision.
  - The Omni Agent created the store repository and its first commit.
- **Smaller issues:**
  - Observer asks buy Front runs, and one relay named the Observer instead
    of the requester.
  - archsage's mention is doubled in its replies.
  - `agentchat trace` reads a Front-only run as `NOT_STARTED`.
  - autolab's single executor delayed the growbox setup by about 20
    minutes behind a research task.

## Deferred

- **Queue automation.** There is no scheduler or automatic queue-draining.
  Research selection stays conversational: archsage reads the queues when
  it plans, and settles a note only against refreshed knowledge.
- **Public publication.** The `publish/` repositories (for example
  `iwaag/study-aisvgs`) and their human push are unchanged and out of
  scope.
- **Research for worldtrend and growbox.** Their routines are registered
  and unrun. Asking Front to run them is the next step whenever wanted.

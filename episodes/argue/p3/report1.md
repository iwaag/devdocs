# argue p3 step 1 — readiness, gaps and the execution-to-review path

Inspected 2026-10-20 15:10–15:40 UTC (2026-10-21 00:10–00:40 JST) on the
coordinator host. Changes in this step are in working trees and committed
with this report; nothing is deployed yet (step 2).

## Runtime state observed

- `nctl status`: Nautobot, worker, dumps and submodules ok;
  `nctl drift --host agstudio`: converged (agautolab, agdevworld-web,
  aghangout, agobserver, slack-front … applied).
- WFE release `7286d14` (p16's last); store v5. `wfe exec workers`: `studio`
  (default lane 1 + `conversation` lane 1) and `gpupc` (1 slot), both
  available and idle.
- `wfe observe`: one unfinished run, `jevlab experiment/run-001`, stopped by
  the Developer before plan agreement ("事情が変わったためこの実験を保留"). It
  is unrelated to this trial and is left untouched.
- `conversations.json` `discussions`: facilitator `argue/facilitate`;
  participants forge, archsage, aghangout (`<ws>/discuss`). Front, RTS and
  jevlab were not participants.
- Workspaces of rts-vs-bot (9 behind), jevlab (3) and front (1) were behind
  their published `main`; fast-forwarded before editing.

## Participants

| Braindump name | Workspace | Repository | Agent / profile | Before this step |
| --- | --- | --- | --- | --- |
| front | `front` | developer/pj-front | front-agent / sonnet | `converse`, `freework`; project guide written for conversations only |
| pj-rts | `rts-vs-bot` | developer/pj-rts-vs-bot | none named (runs record executor `autolab`); default role profile sonnet | `build-game`, `improve-game`, `boast`, `freework`, `backfill-wedo`; no project guide |
| pj-javlab | `jevlab` | trustedagent/pj-jevlab | jevlab-agent / default (sonnet) | `experiment`, `study`, `improve-tools`, `freework`; project guide said the person reaches it only in Zulip |
| facilitator | `argue` | developer/pj-argue | argue-agent / sonnet | `facilitate`, `synthesize`, `freework` |

"pj-javlab" in the braindump resolves to `jevlab` (the only Jev project;
`lab/` and `study/` hold Jev experiments, including `lab/game_test`).

## Gaps found

1. **No discussion workflow for front, rts-vs-bot or jevlab**, and they were
   not configured participants.
2. **Front's project guide assumed every run serves a conversation**
   (`wfe conversation reply` "the only place the Developer reads you", "each
   run serves one message of a conversation"). A discussion run following it
   would reply to the wrong surface.
3. **jevlab's project guide** said the person reaches it through a Zulip
   request topic only.
4. **rts-vs-bot has no agent identity**: its runs record executor `autolab`,
   so its speech in a discussion would be authored "autolab".
5. **Execution-to-review path (the main gap).** In p16 the facilitator
   awaits only the invitations it issued; an invitation is resolved by the
   participant's first reply. Work a participant entrusts to its project's
   workflow outlives that reply: its discuss run waits for the child and
   later posts a `result`, which starts nothing and continues nobody. The
   facilitator guide said a turn "is not told when [synthesize] finishes".
   And a facilitator run that had *completed* while something it awaited
   was still open received the outcome as an instruction on a completed
   run: "nothing continues it". So the only ways back to review were a new
   human post or a participant happening to keep its invitation open.

Not gaps (existing capability, verified by reading code and help): a
participant can entrust work to any project's workflow by a delegate node
(`wfe run adjust`, `wfe delegate`); waits free the slot; child questions,
failures and results continue the parent; Front can request and steer other
projects' work; `progress` does not answer an invitation, `reply`/`result`
do; invitations, contributions and continuations are receipt-bound.

## Changes

### WFE (agdevworld, uncommitted until verified in step 2)

**Awaiting a result** — `wfe discussion await [<id>] --run <ws>/<wf>/<run>
--note <what> [--re <msg>…]`, from the facilitator's executing attempt
(`server/discussions.ts` `awaitResult`, store **v6**:
`invitations.awaits`).

- *Correlation*: an invitation row with `awaits: result`, the awaited run as
  its run, the facilitator's run as `awaitedBy`, event `awaited`; token
  `await:<facilitator run>:<run>` (the same await again is one row).
  Nothing is started or delivered.
- *Resolution*: the awaited run's next `result` or `synthesis` in the
  discussion answers it; an interim `reply`/`progress`/`question` does not.
  When the run ends first (completed/cancelled, seen by reconcile every
  minute) it is answered by the run's last message here since the await,
  else `unanswered`. Awaiting a run that already ended settles at once.
- *Continuation ownership*: the outcome joins the facilitator run's other
  awaited invitations and is told once, when all are resolved (existing
  continuation). **New:** if that facilitator run has *completed* meanwhile,
  a new facilitator turn is invited instead (by `discussion`, receipt
  `discussion:<id>:continue:<run>:<outcomes>`, once), its question listing
  the outcomes. A cancelled facilitator is told nothing; a closed
  discussion continues nobody (as before).
- *Failure outcomes*: a child that fails is reported by the participant's
  run as its result (delegation already relays failures to the parent); a
  participant run that ends silent is `unanswered`; both reach the
  facilitator.
- *Duplicates and restarts*: everything is in the store; reconcile re-derives
  resolution from run records; receipts make the continuation and the new
  turn once.
- Shown in `wfe discussion show` (", awaited result"), the view
  (`invitations[].awaits`, run relation "awaited result (invitation n)") and
  the archive (`awaits` in the manifest and the `awaited` event).
- Contract: `docs/discussions.md` *Awaiting a result*, *Facilitator
  continuation*; help: `wfe discussion help`.
- Tests: two new cases in `test/discussions.test.ts` (interim reply vs
  result; waiting facilitator continued once; completed facilitator → one
  new turn across a store restart; silent run → unanswered; ended run
  settles at once; non-facilitator refused; closed refused). 11/11 pass;
  `tsc` clean.

### Guides (tool facts, no trial content)

- agautolab wfrun guide, *A run that speaks in a discussion*: a run that
  entrusted work keeps waiting after it spoke; presenting what came back is
  `--kind result`; a reply answers the invitation, progress does not; the
  facilitator may await the run's result.
- pj-argue `agent/facilitate.md`: commissioned work outlives the turn;
  `wfe discussion await`; a completed turn is followed by a new one. The
  sentence "Your turn ends and is not told when it finishes" is replaced.
- pj-front: the conversation-specific facts moved from `agent/guide.md` to a
  new `agent/converse.md` (`converse.yaml` `execution.guides`); the project
  guide now says a run serves a conversation or a discussion; new
  `agent/discuss.md` and `discuss.yaml` (front-agent, conversation lane).
- pj-rts-vs-bot: new `discuss.yaml` (agent **`rts-vs-bot-agent`**, a new
  name for this project's speech; its other workflows are unchanged) and
  `agent/discuss.md` (what the project knows and that game work is its
  ordinary workflows, entrusted by a delegate node).
- pj-jevlab: new `discuss.yaml` and `agent/discuss.md` (claims vs
  measurements; experiments are its ordinary workflows, entrusted by a
  delegate node); the project guide's Zulip-only sentence neutralized.

None of these says anything about RTS, Jev, AI-versus-AI play or how to
divide this trial's work.

## Selected execution-to-review path

1. The Developer's goal and the trial protocol are posted once to a new
   discussion (no addressee → the facilitator).
2. The facilitator invites participants; their discuss runs speak from
   their projects. Work any of them takes on is entrusted by a delegate node
   of its own discuss run (or Front's), which waits without the slot.
3. Either the participant keeps its invitation open until its result, or
   the facilitator awaits the participant's run with `wfe discussion
   await`. Both are tools the agents choose between; neither is prescribed.
4. The child's result continues the participant's run, which posts it as a
   `result` (or ends without one); the facilitator run is continued — or,
   if it completed, a new facilitator turn is started — with the outcomes.
5. The facilitator conducts the review in the same discussion; the trial
   ends after it.

If the agents never commission work, or the facilitator awaits nothing and
nothing is pending, the review is not reached by itself; that is agent
coordination and is recorded as such.

Remaining configuration for step 2: add front, rts-vs-bot and jevlab to
`conversations.json` `discussions.participants`; deploy WFE; commit and
publish the project changes.

Deus Ex Machina notes: did the `discuss` workflows and guides for front,
rts-vs-bot and jevlab, Front's guide split and jevlab's guide sentence for
agents front-agent / rts-vs-bot / jevlab-agent — handoff candidates; did the
facilitate guide change for argue-agent — handoff candidate.

# workflow_editor p16 — Step 3: argue and real participant workflows

The managed `argue` project exists, forge and archsage have discussion
workflows, and a bounded exchange ran on the deployed system: a
human-authored brief, the facilitator, two projects, a direct question, a
clarification mid-flight and a specialist question answered by the person.
All trial words were posted as `Omni Agent (for Developer)` and marked as a
trial.

## Built and deployed

| Repository | Commit | What |
| --- | --- | --- |
| pj-argue (new, Gitea `developer/pj-argue`, workspace `argue`) | `b2e20a0` (`wfe create`), `90a1bde` | Intent and goals; areas `discussions`, `knowledge`, `tools`; guides `agent/guide.md` (project), `facilitate.md`, `synthesize.md`; workflows `facilitate` (conversation lane, studio), `synthesize`, `freework` (agent `argue-agent`, profile sonnet); `tools/argue` (list, show, validate, cite, knowledge — read-only, deterministic); `knowledge/` form; README |
| pj-forge | `41c8436` | `discuss` workflow (conversation lane) and `agent/discuss.md`: speak from the catalog, deliveries and demands |
| pj-archsage | `d04c7bf` | `discuss` workflow and `agent/discuss.md`: speak from the projects' knowledge with sources; research is a proposal for a `study`/`consult`, not this turn |
| agautolab | `6c17e17`, `90f6215` | wfrun guide *A run that speaks in a discussion* (read, speak with `--re`, cite, start nobody, a question only the person can answer is the run's question) |
| agdevworld | `ded957c`, `6625c65` | WFE discussions (report2), then the routing found missing here (below) |
| pj-aghangout | `8c8b13d` | Meeting rooms (Step 5's work, built while the trial ran; see report5) |

Deployment: backups `.local/agdev/.local/backups/p16-predeploy-20261020-223649/`
(conversations v3 with 123 messages, exec store, configuration) and
`p16-redeploy1-*`; `wfe-release.sh deploy` of `ded957c`, then `6625c65`
(store v3 → v4 → v5, additive). `conversations.json` gained `discussions`
(facilitator `argue/facilitate`, participants `forge/discuss`,
`archsage/discuss`, archive `argue`). Before deployment, `nctl drift --host
agstudio` reported the node converged; WFE's service is not a Nautobot
service (only `agdevworld-web` and `aghangout` are), and nothing about
placement changed. The executor needed no restart: its reconcile runs the
release's `wfe`, which now also runs the discussion reconcile and the
archive.

## The trial: discussion `p16-s3-room`

*"A meeting room for group discussions in aghangout"* — what the room should
convey, how its art stays consistent, who owns the asset and its provenance.
Members: the trial person, argue, forge, archsage.

| # | Speaker (run, invitation) | What happened |
| --- | --- | --- |
| 124 | person (unaddressed) | the brief → invitation 1 to the facilitator |
| 125 | argue `facilitate/run-001` (1) | said whom it asked; invited forge (2) and archsage (3), awaited both, made its node wait and ended its attempt |
| 126 | forge `discuss/run-001` (2) | the media view from its catalog: what it lacks (no records for the rooms), proven routes and their limits, ownership split *scene vs provenance* |
| 127 | archsage `discuss/run-001` (3), **re #124, #126** | read aghangout's files and the settings provenance, **answered forge's actual words**: agreed on seats and cost, **disagreed** on consistency ("no prompt or seed exists; consistency must come from the pixels") and ownership (keep provenance in the shipping repository) |
| 128 | argue `facilitate/run-001` (continued by instruction i1 after 2 and 3 resolved) [synthesis] | agreed points, two disagreements with sources, open questions, its own inference marked as such; started nobody |
| 129 | person **to forge** re #126, #127 | direct question → invitation 4 to forge only |
| 130 | person (unaddressed) **while forge was executing** | clarification → invitation 5, a new facilitator turn |
| 131 | forge `discuss/run-002` (4) | read #130 (after its horizon, marked so in `show`) and took it into account; **withdrew its own earlier proposal** ("archsage is right on that") |
| 132 | argue `facilitate/run-002` (5) | took #130 as the person's steer; what is now agreed, what stays open; would invite aghangout's agent if wanted |
| 133 | person **to archsage** | asked it to ask its scoping question first → invitation 6 |
| 134 | archsage `discuss/run-002` (6) [question q1] | asked as **its run's question** (`wfe run ask`) and waited |
| 135 | person to archsage **re #134** (posted over HTTP, like aghangout) | **delivered to the run that asked** as the answer to q1 (invitation 7, `deliveredTo` 134), with `source` = discussion message #135; the run was resumed |
| 136 | archsage `discuss/run-002` (6, 7) | read the settings repository and its history, found a C2PA record (`gpt-image`, OpenAI) on one room and none on the others; answered within the person's scope |

Every agent line names its participant, agent, run, attempt and invitation;
the person's words are verbatim under their own name. The facilitator was
continued exactly once per round. No invitation, message or run was
duplicated (`wfe discussion show p16-s3-room`; invitations 1–7 all
`answered`).

The archive followed on its own: the reconcile published `e2d4132` (through
#124), then each further message, up to `e1bf479` (#136), interleaved on
pj-argue main with WFE's integrations of the facilitator runs (`a5316d2`).
`tools/argue validate`: ok.

## A gap found and fixed

The first design (report1) sent a person's reply addressed to a participant
to a **new** run of it. That breaks a specialist's question: the run that
asked would wait forever and the answer would land in a stranger run.
Fixed in agdevworld `6625c65` (store v5 `invitations.delivery`): a message
addressed to a participant that names (`re`) a question that participant's
run asked is delivered to **that run** while it has not ended — as the
answer to its WFE question (`question.answer`, resuming the waiting node),
else as an instruction — with the message as its source and the
invitation's receipt. After that run ended, the participant is invited with
a new run. The run's reply answers both invitations. Test: *the person's
reply to a participant's question reaches the run that asked…*; the wfrun
guide says how a participant asks. Trial #133–#136 above is the live proof.

The mid-flight clarification (#130) needed no forwarding: the participant's
turn reads the discussion, and `show` marks what came after its horizon.
Forwarding to running work is still available to a facilitator turn (`wfe
run instruct … --source-message`) for work that does not reread the
discussion.

## Responsiveness

Executor store, per job (`.local/p16/step3/timing.txt`):

| Run | Kind | Queued s | Ran s |
| --- | --- | ---: | ---: |
| argue/facilitate/run-001 | start | 1 | 25 |
| forge/discuss/run-001 | start | 15 | 53 |
| archsage/discuss/run-001 | start | **68** | 51 |
| argue/facilitate/run-001 | resume (i1) | 11 | 38 |
| forge/discuss/run-002 | start | 1 | 40 |
| argue/facilitate/run-002 | start | **39** | 25 |
| archsage/discuss/run-002 | start | 2 | 29 |
| archsage/discuss/run-002 | resume (q1 answer) | 0 | 74 |

The brief reached a synthesis in 2 min 40 s. With one `conversation` slot,
the facilitator's invitations run one after another: the second participant
queued 68 s behind the first, and the person's mid-flight message 39 s
behind forge. That is slower than parallel but not a barrier to a
conversation at this size; the facilitator frees the slot while it waits,
so nothing deadlocks. Capacity was **not** changed; Step 6 measures it again
with specialist work active.

## Deus Ex Machina notes

- Wrote the forge and archsage `discuss` workflows and guides for those
  agents — handoff candidate for forge-agent and archsage-agent.
- Built aghangout's meeting rooms (`8c8b13d`) — handoff candidate for
  aghangout-agent.

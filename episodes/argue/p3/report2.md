# argue p3 step 2 — verification and deployment before the trial

Done 2026-10-20 15:20–15:35 UTC. The system is ready for the trial. As the
plan says, technical readiness is not a human approval gate.

## Deployed revisions

| Repository | Revision | What |
| --- | --- | --- |
| agdevworld | `978a5ed` (from `7286d14`) | Discussions: `wfe discussion await`, store v6 `invitations.awaits`, a new facilitator turn for what a completed facilitator run awaited |
| agautolab | `797b2d0` | wfrun guide: a discussion run that entrusted work, `--kind result`, awaits |
| pj-argue | `67ce2c9` | `agent/facilitate.md` (commissioned work, await) |
| pj-front | `f74c2e6` | `discuss` workflow, `agent/discuss.md`, conversation facts moved to `agent/converse.md` |
| pj-rts-vs-bot | `6c56cff` | `discuss` workflow (agent `rts-vs-bot-agent`), `agent/discuss.md` |
| pj-jevlab | `c7cf37f` | `discuss` workflow, `agent/discuss.md`, project guide sentence |
| pj-agdev | `9554a6d` | pins agdevworld and agautolab |

Configuration: `conversations.json` `discussions.participants` gained
`front/discuss`, `rts-vs-bot/discuss`, `jevlab/discuss`. `wfe discussion
participants` lists all seven as available. Before deployment, the
conversation store, the execution store, `conversations.json` and
`registry.json` were backed up. Rollback of both code and data is
described in the ignored `pj-agdev/.local/argue-p3/notes.md`.

## Checks

- **Unit/integration**: agdevworld `workflow_editor` 467/467 tests (11
  discussion tests, 2 of them new), `tsc` clean. The release script's
  own build and checks passed, and so did its health check (the service
  and `bin/wfe` report `978a5ed`). The store migrated to v6 on open.
- **Workflows**: `wfe validate` is valid for every workflow of front,
  rts-vs-bot and jevlab.
- **Live fixture with an unrelated trivial task**
  (`p3-s2-fixture`, Omni-authored and labelled as a system check;
  participants argue and front; the child work went to glossary
  `freework`. RTS and jevlab were not used, so the trial task was not
  rehearsed):

  | UTC | Event |
  | --- | --- |
  | 15:24:58 | brief #161 posted → invitation 18, `argue/facilitate/run-007` |
  | 15:25:11 | argue invited front (19, `front/discuss/run-001`) and awaited that run's result (20, `awaits: result`) |
  | 15:25:32 | argue replied #162; its run waits (`external: discussion:p3-s2-fixture`) |
  | 15:28:34 | front replied #163 (invitation 19 answered; awaited result 20 stays open) |
  | 15:29:14 | front's delegate d1 bound child `glossary/freework/run-004`; front's run is "Waiting · glossary freework result" |
  | 15:29:44 | **WFE service restarted** while the child worked |
  | 15:29:57 | the child completed; front was continued and posted #164 `[result]` with the file route → invitation 20 answered |
  | 15:30:16 | argue's waiting run was continued by one instruction, **with no human post** |
  | 15:30:36 | argue checked the link (#165: HTTP 200, the expected line) and completed without starting anything else |

  So participant dispatch, delegated work, the return to the same
  discussion and the continuation of the review all work without a human
  message. Elapsed time was 5 min 38 s.
- **Duplicates and restart**: the **executor** was restarted at 15:31:03,
  after the reconciles and a manual `wfe discussion dispatch`. The
  invitations, their `reported` marks and the 5 messages were unchanged:
  no second continuation and no new turn
  (`restart2-before/after.txt`).
- **Failure outcomes**: covered by unit tests. A silent run becomes
  `unanswered`. A completed facilitator gets one new turn, also across a
  store restart. An interim reply does not answer an awaited result. A
  non-facilitator await is refused, and so is an await in a closed
  discussion. A child failure reaches the parent through the existing
  delegation relay (unchanged). The live fixture did not exercise a
  failed child or the completed-facilitator branch.
- **The per-trial instruction**: the instruction lives in the discussion's
  opening message, which every invited run reads (`wfe discussion show`).
  In the fixture, the brief's "start no further work" was followed by both
  front ("Nothing else was started") and argue. Nothing global was added to
  argue's guides or code.
- **From another LAN machine (agpc)**, using the read-only `lan-check.mjs`
  with Chrome: all seven participants were available. The discussion view
  answered and included the awaited result. Both run views opened, and the
  meeting room `…/hangout/?discussion=p3-s2-fixture` opened with Argue and
  Front, front speaking under `front-agent`. Invitations were listed and
  there were no page errors. The child's file route returned
  `argue p3 fixture`. `/wfe/api/conversations` and `/hangout/` answered
  200, so the direct-conversation clients are unaffected.
- **Archive**: the fixture was closed and published (`pj-argue` `073f2a7`,
  status `current` through #165).
- **Runtime state**: `nctl drift --host agstudio` is converged, and the
  launchd services `wfe-service`, `agautolab-wfexec` and `slack-front` are
  running. `jevlab experiment/run-001` remains stopped by the Developer
  (unrelated).

## Passive observation and automatic recovery

- Observation uses only read surfaces: `wfe discussion show --json` every
  20 s through `pj-agdev/.local/argue-p3/bin/watch.sh` (changes of messages,
  invitations and run status lines), `wfe observe`, `wfe run state`,
  `wfe delegate show`, and `wfe discussion archive --status`.
- **Automatic recovery that could affect the trial**: Observer (60 s)
  turns an interrupted/unknown hold or an unowned continuation into an
  `agops/recover` run. That run may `wfe run recover`, which continues the
  held run. Observer never acts on a person's stop. A queue wait or an
  external wait (the facilitator's) needs nobody. Any recovery run during
  the trial will be read from Observer's incidents and `agops` runs and
  reported as automatic recovery, not as agent coordination. The last
  incident before the trial was at about 10:26 UTC; none occurred during
  the fixture.
- The trial gives the executor no special treatment: the conversation lane
  has 1 slot on studio, the default lane has 1, and gpupc has 1.

Fixture side effects: run `glossary/freework/run-004` (a one-line file),
`front/discuss/run-001` and `argue/facilitate/run-007` are records of this
check. The closed discussion `p3-s2-fixture` is archived.

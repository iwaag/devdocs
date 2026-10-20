# p15 Step 6 — trials

**Trial authorship.** Every live trial message was the Omni Agent's, recorded
as `Omni Agent (for Developer)` and marked "[Trial by Omni Agent for the
Developer …]".

- Messages typed into the deployed aghangout page (Playwright on agstudio)
  name that author in the request body. Without it, a page records the
  configured person, and the Developer would have been impersonated.
- Slack trials used `slack-front trial`, recorded as `slack-trial`.
- Every reply, run, delegation and record below is the agents' own.

**Omni interventions in agent-owned work.**

- The cosmetic initials fix in aghangout (`3fa9a20`), noted below. Handoff
  candidate for aghangout-agent.
- Answering aghangout's `change/run-001` confirmation as its requester.

Fixture checks cover the failure and race cases. Bounded live trials cover
agent behaviour.

| Trial | Evidence | Result |
| --- | --- | --- |
| **Three conversation partners** | `p15-s2-191237` (Front, run-039, 19 s); `p15-s3-forge` (forge converse/run-001, 81 s); `p15-s3-archsage` (archsage converse/run-002, 22 s) | Each project received and answered its own conversation through its own `converse` run, with no intermediary Front run. Lines carry `front-agent`, `forge-agent`, `archsage-agent` |
| **Useful work** | `p15-s3-forge2`: forge converse/run-002 → `forge/provide/run-010` → #103 (WAV/MP3, durable keys, record); lantern: converse/run-004 → provide/run-011 (red, rejected by the parent as too late for the change) → provide/run-012 (blue) → #114; `p15-s6-steer`: archsage converse/run-005 → `archsage/consult/run-003` → #121 with `advice.md` (opens: 200) | Inspectable specialist output, each reply linked to the run that produced it |
| **Questions and steering** | Answers: provide/run-010 q1 (`source: p15-s3-forge2 #97`), forge converse/run-004 q1 (`source: #115`). Steering, first try: forge converse/run-005 relayed "make it blue" through `wfe delegate instruct`, so provide/run-011 recorded it as `from: forge-agent`, with the message only in the text. After the shared-guide fact (agautolab `5fdc923`), the rerun: archsage converse/run-006 instructed the running consult/run-003 with i1 `from: Omni Agent (for Developer)`, `by: archsage-agent`, `source: conversation p15-s6-steer #119`, words verbatim; the conversation links it as `steer` | The first steering try reached the work but without structured attribution; fixed and rerun. Answers kept their source references, though agents sometimes paraphrase the words they record |
| **Concurrent conversations** | Fixture: switching partners opens another conversation; nothing of forge's appears in the Front conversation; an address naming another partner cannot retarget a conversation. WFE tests: two destinations in parallel, concurrent replies attributed by run and `re`. Live: Front, archsage and forge conversations ran at the same time (Step 3 probes, lantern and steer trials) | No message or reply was misrouted. In the store, every conversation's replies come from its destination's agent |
| **Responsiveness** | Step 3: while provide held the default slot, Front answered in 12 s and archsage in 44 s (≈20 s queued behind Front's turn). Step 6: archsage replies of 24–29 s while forge's provide or archsage's consult worked | Turns are served while specialist work runs. The three partners share one conversation slot (limitation below) |
| **Replay and recovery** | Fixture: failed post, reload, Retry → one message. Live (`p15-s6-restart`): post #111 → `launchctl kickstart -k com.agdev.wfe-service` → a resend during the restart got nginx 502 (the outbox case) → a resend after it returned `duplicate: true`, same run → `kickstart -k com.agdev.agautolab-wfexec` while archsage converse/run-004 and forge provide/run-011 executed → both adopted → one reply #112 at 24 s. Store-wide: 55 person messages, 54 turn links (the one missing is the unavailable-target trial), 0 messages with two turns, 0 duplicate tokens, 0 duplicate reply keys, 0 conversation receipts with two jobs | No duplicates of messages, work or replies |
| **Unavailability** | Fixture: an unavailable partner's card and room show the reason, Send is disabled, WFE down keeps the last read and recovers. Live: `lowerpoly/converse` was offered temporarily. The lobby card was disabled with `Unavailable: workspace lowerpoly has no workflow "converse"`. A first message (#109) was kept with `dispatch: failed` and the reason, nothing ran anywhere, the reconcile retries it there, and lowerpoly's tree stayed clean. The target was then removed again | Visible, and never re-aimed. Recoverable once the destination exists |
| **Live clients** | Slack: `slack-front trial` → 📥 → `front/converse/run-041` → reply in the thread (outbox #40, `sent`) 12.6 s later; the conversation's destination is `front/converse`. Old Front Desk links (`/?view=frontdesk&conv=…`) open in aghangout (agpc check). WFE Home links `/hangout/` | Working with the deployed contract |
| **Independent application** | `aghangout/change/run-001` (aghangout-agent): it added a lobby footer with the served release and a WFE Home link, ran `npm run check` and the browser checks in Chromium and WebKit, deployed `25fc489` with `scripts/deploy.sh`, and asked for confirmation. The web image was unchanged (created before the run). The served page showed `release 25fc489 · WFE Home ↗` on desktop and phone widths. Then `3fa9a20` was deployed the same way | aghangout changes and redeploys without rebuilding agdevworld, done by its own agent |
| **LAN review** | agpc (Chrome) ran `step5/deployed.mjs`: 12/12 checks pass. They cover the lobby, an old desk link, the recorded partner, the actual authors, forge's result with its WAV opening from agpc, history, the run view, Home, and the phone layout | Another machine opens the entrance and inspects conversations and artifacts |

## Failures found and fixed in this phase

| Seen | Fix | Rerun |
| --- | --- | --- |
| archsage converse/run-001 completed without replying: `reply` text given without `--text`, and the refusal ended in 40 lines of help | `wfe conversation` usage errors say "Nothing was posted or recorded" first and last, and name text given without `--text` (agdevworld `626c43f`, test) | archsage converse/run-002 to -007: replies posted |
| forge converse/run-002 relayed its child's question as text only, and was held interrupted | wfrun guide: ask your own question from the delegate node (agautolab `f37db08`) | forge converse/run-004 asked q1 from its delegate node, waited cleanly, and took the answer |
| forge's result links (`raw?path=devdocs/runs/…`) answered 404 | The run file route accepts project-root paths too (agdevworld `034c0c5`, test) | WAV opens from agpc (200 `audio/wav`) |
| forge converse/run-005's steer reached the child as the agent's words | wfrun guide: `wfe run instruct --from --source-message` to the running work (agautolab `5fdc923`) | archsage converse/run-006 → consult/run-003 i1 with the conversation source |
| Room backgrounds did not render; the phone header overflowed (Step 4) | Absolute asset URLs; compact header | Fixture checks pass in both engines |
| Plain-presentation initials "L(" | Letters only (aghangout `3fa9a20`, done by the Omni Agent; handoff candidate for aghangout-agent) | Unit test |

## Repository checks

| Repository | Check |
| --- | --- |
| agdevworld `workflow_editor` | `npm run check`: 456 tests pass (each release build) |
| agdevworld `integrations/slack` | `npm run check`: 7 tests pass |
| agdevworld `agentroom` | `uv run pytest -q`: 17 tests pass |
| pj-aghangout `app` | `npm run check` (8 unit tests) and `npm run browser-check`: Chromium and WebKit, all pass |
| forge, archsage, front, aghangout workflows | `wfe validate`: valid |

## Observations not changed

- forge's lantern run ended without a separate `result` line: the image was
  in its question (#114) and the acceptance was confirmed in #117.
- provide/run-011 finished its `make` node without reading the instruction
  that arrived during it. The parent noticed the red result, rejected it and
  started a blue run.
- When recording the person's answer, agents rephrased the words in two
  places ("your description" → "the Developer's description"; "I have not"
  → "they have not"). The source reference to the original message is
  exact.

# Step 5 — deployment, live exercise, and what is still pending

Date: 2026-09-18 JST (2026-09-17 17:05–17:15Z). Plan: [plan.md](plan.md)
step 5. Everything below ran on the real realm. **No human has used the room
yet**: every post in this step was made by the Omni Agent under its own
account, or is a machine note, and none of it is evidence of human use.

## Deployment

- **State before and after**: `nctl drift` → `converged=46`, error 0, the
  same `info` liveness lines as before. No desired-state change: no new agent
  account and no new service — Front's presentation role is a worker inside
  the existing agfront listener process.
- **`#memo`** created with the provisioner credential (stream 171, public,
  Developer and Front subscribed, description says nothing there is ever
  answered) — before any listener could have created it with a bot's rights.
- **Restarted on the p2 code**, none with a harness child running:
  comfy-notifier, agautolab, agforge, agobserver, archsage, cagent-zulip,
  agentroom, agfront. Every startup recovery queued **0** conversations. The
  web image on `:8090` was rebuilt. No plist changed.
- **Pins**: every listener project's `uv.lock` is at pyagag `0af761d` or
  later (agfront and agentroom at `7beeec5`). **agautolab1 (the VM) was not
  redeployed**: it runs the gateway and no listener, so it has no intake a
  memo could reach.
- **Settings**: revision `1af317b66219` synced and served (archsage, the
  argue room).

## Live demonstrations

### Memo silence across a restart

Eight probe posts by the Omni Agent in `#memo` (messages 7190–7197): an
owned-looking `front-…` topic, real mentions of **every** agent bot including
`@**archsage** sage:arxiv`, a well-formed and a malformed notifier command, a
copied `[selfnote][rootchat]`, `@**Front** use agy`, a `workplan-…` and an
`argue-…` topic; then the first topic was renamed and resolved. After 25 s,
agfront, agautolab, cagent-zulip and the notifier were restarted.

Result: no log line in any of the six listeners but the restarts' own
`recovery (startup): 0 conversation(s) queued`; no notifier log line, no
ticket, no reaction and no refusal post; the only other post in the channel
is Zulip's own resolve notice; the ops board holds no `memo` row.

### Normal argue participation, and the rendering

`#argue › argue-p2-smoke` (anchor 7199), opened by hand by the Omni Agent and
labelled as an integration check. Front served it with the `argue` role (3
servings), invited `sage:arxiv`, which answered from its tree; Front thanked
archsage by name, which bought the council a run and Front one more. The
Omni Agent resolved the topic.

- The three argue workspaces hold `chatlog.md` and `tools/agents.md` only;
  a search of them for the lore's words, `characters.md`, `settings revision`
  and `ag-dialogue` finds nothing. The lore is in the render jobs' snapshots
  and nowhere else.
- 45 s after the last post the renderer planned **one** job for all five
  agent posts and finished it in 11 s: memo record 7213 in
  `#memo › argue-p2-smoke-s7199`, Front in three/two/one turns, the council
  as its character, `sage:arxiv` as a `plain` turn. The relay's
  `/argues/7199` returned the posts with their speakers, one interpretation,
  renderer `idle`, nothing pending.

### Rendering recovery

The job's row was set back to `running` and its `result.json` deleted, then
agfront was restarted — the state a crash between the memo post and the local
"done" leaves. Log: `was running at the restart; checked against the memo
before any run` → `its result is already in the memo; recognized, not
re-run`. Still one presentation run record, still one record in the memo.

### Another interpretation

`POST /argues/7199/render` with the older retained revision `2c21078ae237`
wrote `[selfnote][render] …` (7214) into the **resolved** topic. One job
(`request:7214`), one run, a second record; the relay lists two
interpretations of the same five posts, the council is a `plain` turn at the
older revision (it has no archsage character), and the argue **stayed
resolved**. In the deployed room (`:8090/?view=argue&argue=7199`): the
history of all speakers, the interpretation button stepping between the two
revisions, the older one drawn with its own revision's room, source chips,
and the composer saying that posting resumes the discussion and not what it
ended in (`agdevworld/.local/shots/argue-p2/d1…d4`).

## Found and fixed during the exercise

**The relay showed both resolved argues as open.** A Zulip event queue
registered for all public channels delivers the *new messages* of a channel
the bot never joined, but not the **moves** — and a resolve is a move. The
relay's reader (`Opsroom Observer`) had joined nothing created since
2026-09-14 (`#argue`, `#memo`, `#archsage-agstudio1`, `#pj-desk-garden`), so
its mirror kept every message under the bare name; Front's mirror,
subscribed, was right. Two changes (agdevworld `2e5ff44`): `locate` judges
"resolved" from where the anchor message and real posts are, not from the
topic listing; and `subscriptions.py` keeps the reader in every public
channel (at start and every ten minutes: one read, a subscribe for what is
missing, one resync — 76 calls, 1.2 s here). After it both argues read `✔`,
`done`. Agent listeners' own mirrors keep this limitation for channels their
bot is not in; it predates this phase and is noted as a limitation.

## Counts, costs, API evidence (17:05–17:15Z)

| Kind | Role | Runs | Cost |
|---|---|---:|---:|
| substantive | Front `argue` | 3 | $0.161 |
| substantive | archsage council | 1 | $0.258 |
| substantive | `sage` | 1 | $0.094 |
| presentation | Front `present` | 2 | $0.102 |
| — | `desk`, `front` | 0 | — |

Five agent posts cost $0.05 to render once; the probe in step 2 ($0.17 for
two posts) was dominated by a cold prompt cache. Presentation was 17 % of
this exercise's spend.

Zulip's own request log, by caller, for the 8.5 minutes that contain three
rounds of restarts, the probes, the argue and two renderings: 273 API calls,
**no 429**, Opsroom Observer 90 (76 of them the one resync), Front 75,
archsage 22, Omni Agent 20, every other listener 8–15 (their restarts). No
caller shows a periodic pattern. **Ninety room reads in a row (`/argues`,
`/argues/7199`, `/frontdesk` × 30) added zero API calls** — the log's last
request did not move. Each render job re-ran nothing: one run per job, and
the recovered job none.

## Human session: pending

The room is deployed and reachable at `http://localhost:8090/?view=argue`
(and from the dashboard's *Arguing Room* link). What is asked of the human:
open a new argue by typing a desire, read Front's and a specialist's
contribution (dialogue or original view), reply, close the tab, come back
later through the history panel and continue. That session has **not**
happened; it is the one validation of this phase that is pending, and nothing
the Omni Agent posted substitutes for it.

## Usability observations (input for later phases, not a protocol)

- **Front moves ahead quickly.** In the smoke argue Front's first reply both
  explained and invited a sage within seconds, and the sage answered 18 s
  later. With a real human, several agent turns can land before the human has
  typed a second sentence. The room at least lets them read every turn as
  written without waiting for a rendering; whether Front should pause for the
  human is a question for a later phase, with real sessions as evidence.
- **Courtesy mentions still buy runs.** "thanks, `@**archsage**`" cost a
  frontier council run ($0.26) and another Front run to say nothing. p1's
  participation guide says a mention is a request; Front's argue guide may
  need the same sentence about thanking.
- **Front leaks a preface** ("This is a smoke test explicitly marked…") into
  its argue replies; the rendering re-voices it faithfully, so it is now
  visible in character too.
- An older interpretation is drawn in the older revision's room — for the
  argue that is the Front Desk background, because the argue room did not
  exist then. Correct by design, slightly surprising to look at.
- The rendering of the first post arrives ~1 min after it (45 s quiet +
  the run). The status line says `being rendered…` meanwhile.

## Documentation

`devdocs/README_DEV.md` (memos, the separated rendering, the Arguing Room),
`pyagag/README.md`, `agdevworld/README_DEV.md`, `agdevworld/agentroom/README.md`;
ignored notes in `pj-agdev/.local/devenv.md` and
`pj-clusterintent/.local/localenv_memo.md`.

Deus Ex Machina note: the Omni Agent opened and resolved
`argue-p2-smoke` and wrote the memo probes — integration fixtures, not work
belonging to an in-system agent; no handoff candidate.

# Challenge p2 — final report: unified conversations

**A one-to-one chat and a facilitated discussion are now one conversation:
one WFE service, one response-request mechanism, one archive writer, one room
in aghangout.** A room can gain projects and have ponderer lead it, and later
stop leading. Its id, history, work and pinned challenges stay the same.
Every room, direct ones included, has a durable Git archive.

Inspect it here: `http://agstudio.local:8093/hangout/`, from any LAN
computer.

- **Check room:** `?conv=20261023-083618-chp2ck` (listed under 動作確認). It
  shows the full flow: one-to-one with forge → archsage joins → ponderer
  leads → leading ends → closed and archived.
- **Old links:** an old `?discussion=<id>` link opens the same room.

## Releases

| Component | Release |
| --- | --- |
| WFE | agdevworld `21a094e` (store v12) |
| aghangout | `9604695` |
| Slack | `21a094e` |
| wfrun guide | agautolab `ba9e2c1` |
| project guides | pj-front `66492de`, pj-forge `415b1a8`, pj-archsage `31c8908`, pj-aghangout `9604695`, pj-ponderer `ef7ac65` |
| pj-agdev | `53af114` |
| agpc worker | **unchanged** (WFE `aadee43`, executor `227ef1c`; see limits) |

## Steps

| Step | Report | Outcome |
| --- | --- | --- |
| 1 Inventory and contract | [report1](report1.md), [contract](contract.md) | Two engines inventoried; the discussion engine became the one service; routing order, request roles, receipts, held continuations and archive paths decided |
| 2 Storage and delivery | [report2](report2.md) | Store v12; pure-SQL migration keeping legacy receipts and runs; `ConversationService`; one CLI and one HTTP tree; rehearsed on a copy of the live store (nothing pending, nothing replayed) |
| 3 Archives | [report3](report3.md) | Every room archived; `discussions/<id>` kept, `conversations/<id>` new, path recorded per room; `ag.conversation.v1`; interruption, concurrency, second backfill and fresh-clone restore tested |
| 4 Agents and clients | [report4](report4.md) | One wfrun guide section; Front, forge, archsage, aghangout and ponderer guides; ponderer `converse`; Slack `with` |
| 5 aghangout | [report5](report5.md) | One Room and one list; recipients shown before sending; join, lead, end, retry, resume; old links resolve; cutover deployed |
| 6 Verification | [report6](report6.md) | WFE 533/533, aghangout 28 unit and 285 browser checks; deployed end-to-end flow by "Omni Agent (challenge p2 check)" with ch-0001 in both run inputs; LAN check from agpc 7/7 |
| 7 Deployment | [report7](report7.md) | Cutover record, post-deploy state, recovery; README_DEV updated |

## Completion criteria

| Criterion | Status |
| --- | --- |
| One service, dispatch and room UI; facilitation optional and explicit | Met |
| A direct room gains participants and a conductor without losing id, messages, work or pinned challenges | Met (verified live, report6) |
| Questions, instructions, late results, retries and restarts keep the recipient/run and do not duplicate | Met: tests (report2), live duplicate-token check |
| Durable archives for existing and new rooms, visible status, verified recovery | Met: 63/63 current; fresh-clone restore |
| History, authorship, run IDs, receipts and citations survive migration | Met: legacy receipts kept, citations and `?discussion=` resolve |
| Slack, agent tools, remote executors and aghangout use the common contract | **Partly**: Slack, guides, studio's executor and aghangout use it. The agpc worker pins are not updated. |
| Deployed LAN flow verified; reports with evidence, limits and recovery | Met |

## Known limits

- **agpc worker pins.** The pins (WFE `21a094e`, executor `ba9e2c1`) are
  prepared as an nctl batch. The session's classifier refused to apply it.
  The commands are in report7. Conversation turns run on studio, so rooms
  work meanwhile.
- **Composer during load.** A `?conv=` room without a partner shows
  宛先を選んでください for a moment, until its first read returns.
- **One slot.** Conversation turns still share one `conversation` slot.
- **Workspace checkouts lag.** The pj-ponderer and pj-forge workspace
  checkouts are behind origin: archive and agent pushes land on origin.
  This predates p2.
- **Out of scope.** p1's evidence-upload size limit and remote
  `evidence --out` were outside this phase. Image extraction (ch-0001)
  was not attempted.

## Recovery

- Prefer forward repair. Store v12 cannot be read by the previous WFE.
- A full rollback needs the release and the pre-cutover backup together.
  Today it would lose only the check room's records.
- An archive-only restore recovers records and never replays work.

Commands and paths are in report7 and in the ignored
`pj-agdev/.local/challenge-p2/notes.md`.

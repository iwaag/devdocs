# workflow_editor p16 — Step 6: deploy and evaluate the experiment

Everything is deployed and was exercised on the live system, from this host
and from another LAN machine (agpc). Trial words were posted as `Omni Agent
(for Developer)` and marked as trials; page-driven trials added that author
to the request body.

## Deployment

| Component | Deployed | How |
| --- | --- | --- |
| WFE | agdevworld `7286d14` (p16 releases `ded957c` → `6625c65` → `947d0b9` → `7286d14`) | `wfe-release.sh deploy` (release checks pass); store v3 → v5, additive |
| Configuration | `conversations.json` `discussions` (facilitator argue; forge, archsage, aghangout) | edited in place; copies kept |
| argue, forge, archsage, aghangout workflows and guides | Gitea main of each project | read by each run at creation |
| wfrun guide | agautolab `c16f548` | read by each run; no restart |
| aghangout | `b050db8` at `/hangout/` | `scripts/deploy.sh` |

- **Service state first**: `nctl drift --host agstudio` converged before
  deployment and after; the whole cluster after: 47 converged, 1 drifting
  (`agobserver` `service_missing`, pre-existing since p15). No placement or
  ownership changed, so Nautobot needed no update (WFE is not a Nautobot
  service; `aghangout` and `agdevworld-web` are unchanged).
- **Backups and recovery**: sqlite backups before each release
  (`.local/agdev/.local/backups/p16-*`); code-and-data rollback in the ignored
  `pj-agdev/.local/p16/notes.md` (an older release refuses a v4/v5 store).
- **Outstanding deliveries at cut-over**: none (no queued or running
  attempt at any release; Slack had nothing unresolved).
- **Existing clients after deployment**: Slack-to-Front (`slack-front trial`
  → `front/converse/run-042`, reply in 12 s, `via slack-trial`); a direct
  conversation with forge (`p16-s6-direct-forge` → `forge/converse/run-007`);
  the p15 conversation tests pass unchanged.

## Trials

| Trial | Evidence | Result |
| --- | --- | --- |
| **Distinct perspectives** | p16-s3-room #126 (forge: from its catalog — routes, limits, cost), #127 (archsage: read aghangout and the settings provenance, **answers #126**), #150 (aghangout: from its CSS and DOM). p16-s4-provenance #142/#143 (archsage answers #142) | Each spoke from its own records and cited them; they answered each other's actual words |
| **Human participation** | Direct question to forge (#129 → invitation 4 only); clarification while forge was executing (#130, read by forge as after its horizon, #131); specialist question q1 from archsage answered by the person (#134 → #135 delivered **to the asking run**, #136); later participation after closing and reopening (#147, #149) | Attributed to the person, reached the intended participant or run, source recorded on the run (`conversation p16-s3-room message #135 (invitation 7)`) |
| **Open-ended discussion** | Syntheses #128, #132, #144 keep agreements, disagreements and open questions apart; p16-s3-room was **closed without consensus** (two disagreements open) and reopened later; summary.md keeps both disagreements | No consensus required; nothing replayed on reopen (invitations 1–8 unchanged) |
| **Membership and failure** | aghangout joined late (#149, invitation 14) and answered from the whole history (#150 cites #127, #130, #131, #137, #148); lowerpoly (configured without a `discuss` workflow for this trial) failed at once with WFE's reason (invitation 15), shown in aghangout ("Nobody speaks for … instead"), and blocked nobody | Then removed from the configuration |
| **Recovery** | Replay of #149's submission → `duplicate: true`, same invitations; WFE service **and** executor restarted while forge's turn waited on its `provide` child: afterwards one new message (forge's real result #152), invitations 15 → 15, request jobs 13 → 13, no duplicate agent message (`.local/p16/step6/recovery-*.txt`). Tests cover crash-between-request-and-record and reconcile replays | No duplicates of invitations, messages or downstream requests |
| **Durable memory** | Fresh clone of developer/pj-argue: `tools/argue validate` ok, each message once, horizon = newest message; three discussions current (`e2959c4`, `219cc04`, `b8c08ab`) | Published transcript matches its declared horizon |
| **Publication recovery** | Two discussions in one writer pass (`7217f17` + `2c6ad2d`); lost bookkeeping (set back to #124) → export appended 3 records from the repository's #136, not 13 (report4); failed and rejected pushes in tests; archive and run integrations interleave on main | All records kept; status says what is published |
| **Hand-off and reuse** | `aghangout/change/run-002` requested by `argue/synthesize/run-001` with pinned `argue@e7174609…` sources, linked back as `handoff`, deployed `148a8aa` citing argue; p16-s4-provenance's facilitator found and cited `knowledge/room-image-provenance.md` (#141) and the knowledge was revised with revision 1 kept | Works through ordinary WFE work |
| **Responsiveness** | Below | Usable on one slot; capacity unchanged |
| **LAN experience** | From agpc (Ubuntu, Chrome): read-only check 15/15 (lobby, meeting room, speakers, syntheses, 11 invitations, archived revision opens, hand-off run opens, Home entrance, phone); **started a discussion through the page**: facilitator line after 23 s, forge's own words after 54 s, recorded under the trial author | Minimal setup: a URL |

### Responsiveness

| Situation | Measured |
| --- | --- |
| Brief → facilitator invites → two participants → synthesis (p16-s3-room) | 2 min 40 s |
| Second participant behind the first on the one `conversation` slot | 68 s queued (later rounds 22 s) |
| Person's message behind a participant's turn | 39 s queued |
| Facilitator continuation after the last answer | 11 s queued |
| A turn while forge's `provide/run-013` held the default lane | aghangout's turn queued 1 s, answered in 43 s |
| Specialist work delegated from a discussion turn | forge's turn released the slot while `provide` ran 100 s; the child's confirmation question crossed Zulip in 53 s (spanning the restart) |
| From another machine | facilitator 23 s, participant 54 s |

The one conversation slot makes participants answer one after another; with
two or three participants that is a minute or two, not a barrier, and long
work leaves the slot. **Capacity was not changed.**

### Fixes made from what the trials showed

| Seen | Fix |
| --- | --- |
| A reply to a participant's question would have started a stranger run (Step 3) | agdevworld `6625c65`: delivered to the run that asked |
| A synthesize run never told the discussion; the facilitator promised what it is never told of (Step 4) | pj-argue `5d74f4d` guides |
| forge's results named files without links (#151, #152) | wfrun guide `c16f548`; `wfe discussion show` prints file routes (`7286d14`) |
| Only the facilitator was seated before WFE answered (deployed check) | aghangout `b050db8` |
| A phone overflowed sideways with four tabs (fixture check) | aghangout `8c8b13d` |

## Did the interaction produce something?

**A new option.** In p16-s4-provenance neither project's opening position
survived: forge had held its delivery record canonical (#126), archsage the
shipping repository (#127). Prompted by the facilitator's reading of the
open question in the knowledge (#141), forge proposed splitting *per fact*
with the hash as the test (#142); archsage, answering #142, made it
checkable (recompute the hash, fetch the record by key), added the "copied
from <key> at <date>" label and the deck manifest anchor, and withdrew the
broad form of its own claim (#143). The resulting rule (#144, knowledge
revision 2) is in neither opening message.

**An exposed conflicting assumption.** forge assumed the rooms' prompt and
seed wording could be reused (#126); archsage checked and found none (#127);
forge withdrew it (#131). archsage's later read (#136) found a C2PA record
(`gpt-image`, OpenAI) nobody had known about, which is now in aghangout's
ASSETS.md and in the knowledge.

**A changed proposal.** forge's offer to composite markers into the
picture became "the app draws seats" after the person's steer (#130) and
forge's own reasoning about runtime state (#131); aghangout, joining late
(#150), showed that `cover` cropping makes coordinates read off forge's grid
(#148, which had warned about uniform scaling) wrong unless the page maps
them, and laid out `contain` vs `cover` — a constraint neither forge nor
archsage could see from their own projects.

**Unproductive parts, recorded too.** The facilitator's syntheses restate
participants at length (#128, #132); participants re-summarize earlier
positions before adding to them; forge sent two messages for one result
(#151, #152); forge's #126 floated "or archsage" as scene owner, which
archsage had to refute; the agpc trial discussion was a smoke test with no
substance. Citations drifted in places (summary cited at the archive
commit; #142 and #143 cite the knowledge at different commits).

This is a small trial with one person, one facilitator and three projects.
It demonstrates the behaviour the system now supports; it does not show
that differently situated agents outperform a single frontier model given
all the context. A controlled comparison remains optional future work.

## Review links

- Meeting rooms: `http://agstudio.local:8093/hangout/?discussion=p16-s3-room`,
  `…/hangout/?discussion=p16-s4-provenance`; the lobby `…/hangout/`
- WFE Home (Discussions card): `http://agstudio.local:8093/wfe/`
- Archive and knowledge: `http://agstudio.local:3000/developer/pj-argue/src/branch/main/discussions`,
  `…/src/branch/main/knowledge/room-image-provenance.md`
- The hand-off: `http://agstudio.local:8093/wfe/#/ws/aghangout/run/change/run-002`
- forge's delegated work: `http://agstudio.local:8093/wfe/#/ws/forge/run/provide/run-013`

## Deus Ex Machina notes

- Wrote aghangout's `discuss` workflow and guide (`9ea7e7f`) and extended
  forge's `discuss` guide (`c37161c`) — handoff candidates for
  aghangout-agent and forge-agent.
- Added and removed the deliberately unavailable `lowerpoly/discuss`
  participant in the configuration for the failure trial.

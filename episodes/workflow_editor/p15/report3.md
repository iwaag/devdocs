# p15 Step 3 — direct conversations with Front, forge and archsage

forge and archsage now have conversational workflows. All three projects are
conversation targets, and each answers its own conversations directly. One
forge conversation completed a full specialist round trip: a request,
delegated work, a question, the person's answer, and a result with its
record. Two failures surfaced in trials. Each was fixed where it belonged:
one in the tool, one in the shared run guide.

## Changes

| Where | Commit | What |
| --- | --- | --- |
| pj-forge | `5ec1c96` | `devdocs/workflows/converse.yaml` (one `talk` node; `node: studio`, `lane: conversation`; guide `agent/converse.md`). The project guide now says that a person reaches forge directly in a conversation or through a run view. |
| pj-archsage | `1234b03` | The same for archsage. Its `converse.md` answers short questions in the reply with consult's citation discipline, and entrusts `consult`, `study` or `backfill-wedo` work to a delegate node. |
| pj-front | `ccd4b6a` | The reply mechanics moved to the shared guide. Front's guide keeps what is Front's own and notes that the Developer may also talk with other projects directly. |
| agautolab | `960fd62`, `f37db08` | wfrun guide, *A run that serves a conversation message*: the facts every conversational workflow shares (show, reply with `--re`, questions posted in the conversation too, link, results the person can open), plus relaying a child's question. Pinned by pj-agdev. |
| agdevworld | `626c43f` | `wfe conversation`: a usage error says "Nothing was posted or recorded" first and last instead of ending in the whole help. A positional that is not a conversation id is named as text given without `--text`. Deployed. |
| area | `conversations.json` | Targets: `front/converse`, `forge/converse`, `archsage/converse`. All three are `available`. |

The professional roles stay in each project's own guide. Character
presentation is aghangout's and is not in any agent guide.

## Trials

All trial messages were posted by the Omni Agent as `Omni Agent (for
Developer)` and marked as trials. All replies, runs and delegations are the
agents' own. The one system action involved was Observer's recovery agent
(below).

| Conversation | What happened | Evidence |
| --- | --- | --- |
| `p15-s2-191237` (Front, Step 2) | Front answered directly in 19 s | `front/converse/run-039`, #88 |
| `p15-s3-archsage` | **Failure.** `archsage/converse/run-001` completed in 12 s without a reply. It called `wfe conversation reply --re 89 "<text>"`; the CLI refused (no `--text`) and printed ~40 lines of help after the error, and the agent read the tail as success (its final words: "The reply output printed only a usage hint, so I haven't confirmed it appears there"). **Fix:** CLI feedback (agdevworld `626c43f`, with a test). **Recovery:** the follow-up message #92 started `run-002`, which answered #89 and #92 in 22 s, citing `rts-vs-bot:study/study-rts/notes/01-rts-core.md@d2734dba5c` and acknowledging the missed reply | runs `archsage/converse/run-001`, `-002`; #94 |
| `p15-s3-forge` | forge made a 256 px teacup icon (SVG + PNG) inside its turn in 81 s. The reply links both files and the durable keys, and says honestly that no one else has looked at it | `forge/converse/run-001`, #91 |
| `p15-s3-forge2` | **Specialist round trip.** `converse/run-002` delegated to `forge/provide/run-010` (d1). The child asked (q1) for a style choice, with a review page. The parent posted the question in the conversation (#96, `--question q1`). The answer #97 started `converse/run-003`, which recorded it on the child (`from: Omni Agent (for Developer)`, `by: forge-agent`, `source: p15-s3-forge2 #97`). The child finished. `converse/run-002` disposed the delegation and posted the result (#103, re #93 and #97) with WAV/MP3 links, durable keys and the record `provide/run-010` | runs `forge/converse/run-002`, `-003`, `forge/provide/run-010` |
| (same) | **Failure.** After taking up the child's question, `converse/run-002` relayed it only as conversation text and ended with its delegate node running. It was held as *interrupted (asset left running)*. The system's Observer started `agops recover/run-026`, which continued the run ("child already completed; next attempt takes up its result"), and the result was delivered. **Fix:** a shared guide fact. A child's question for the person is asked as your own question from the delegate node (`wfe run ask <run> <delegate node> --to <person>`), so the node waits on the person (agautolab `f37db08`, with the trial reference) | `run.recover` entry 19 of `forge/converse/run-002` |

A smaller observation: when recording the answer on the child, the agent
rephrased "made from your description" as "made from the Developer's
description". The source reference to the original message is exact.

## Responsiveness

The coordinator's worker `studio` has one default slot and one
`conversation` slot (`workers.json`). The executor runs one claim loop per
lane, so a lane serves one attempt at a time. All three conversational
workflows declare that lane.

These probes ran while `forge/provide/run-010` and then
`agops recover/run-026` held the default slot
(`pj-agdev/.local/p15/step3/resp-*.log`):

| Message | Post → first reply | Notes |
| --- | --- | --- |
| Front: "which projects can I talk with directly now?" | 12 s | conversation slot idle |
| archsage, posted 2 s later | 44 s | about 20 s queued behind Front's turn, then its own turn |
| forge answer turn (`converse/run-003`) | 29 s | slot free |

**Decision: no scheduling change.**

- Specialist work runs in the default slot and does not block
  conversations.
- Conversation turns queue behind each other only while one is executing.
- A turn that does long work in its own talk holds the slot for every
  partner. forge's inline icon took 81 s, and that much latency would apply
  to others.
- The shared guide and forge's and archsage's `converse.md` say that a
  delegate node frees the slot.
- A second conversation slot would need a second claim loop for the lane in
  the executor. It is recorded as a limitation, to revisit if live use shows
  waits.

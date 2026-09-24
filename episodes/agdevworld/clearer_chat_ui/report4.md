# clearer_chat_ui step 4 — every conversation and every consumer

The contract and the read model now reach every agdevworld conversation
surface and every agent that posts. Everything is on pyagag `932d3c6`.
Services are **not** restarted yet (step 5).

## Conversation surfaces (agdevworld `76faa28`)

One set of relay helpers in `agentroom.presentation` — `meaning_of`,
`requests_payload` (= `agag.outstanding.read_requests`), `with_requests`,
`answer_body` — and one label function in the browser (`postMeaning.ts`).
No surface classifies anything itself.

| surface | posts carry `meaning` | `requests` / `viewer_id` | status `asking` | answers a request by id |
|---|---|---|---|---|
| Front Desk (step 3) | yes | yes | yes | yes (`answers`) |
| Arguing Room | yes (`post_payload`) | yes | yes, board `asking: n` | yes — same scene as the desk: label chip, waiting strip, `↩ answering #n` |
| Routine run chat (`chatPanel.ts`) | yes (line stripped) | yes; `viewer_id` added to `/routines/<name>` | — (run states are Front's) | next-post rule only |
| Project Room | yes | yes | yes | next-post rule only |

The Project Room's "LATEST REPORT" panel used to pick the newest agent post
whose words matched `/result|report|done|completed/`; it now takes the
newest post labelled `report`. `Chat.viewer_id()` is the one place "for
you" is decided.

## Consumers

| consumer | what now says what it is | commit |
|---|---|---|
| agfront | desk/front/argue/routine_run replies (shared reply guide); evidence and presenter snapshots show meanings (step 3) | `ed3d15b` |
| agautolab | the **task worker** now speaks under the shared reply contract (`prompt_with_guide(reply=True)`, output + repair), so its question or report is declared; a report awaiting the requester's agreement defaults to a *confirmation* request; `RunProgress` → progress; the task-start line → progress; `## Result` → report; refusals → report. A silent run posts a marked report rather than buying a repair run. | `292b1aa` |
| agforge | a job left with the notifier → progress; the delivery into the requester's `assetplan-` topic and the run's record → report; refusals → report. The notifier parses its command per line, so the trailing line leaves `watch <id>` intact. | `280bc2f` |
| agobserver | a watch needing input asks its requester (request, question); acceptance, notification, finish → report; an unable streak → progress; the monitor treats trace's new `answered` like the other moved-on states | pj-agdev `c159406` |
| archsage | a sage's reply carries the intent it declared; unknown-sage refusal → report | `668a692` |
| cagent | the front's intermediate answer carries its declared intent (a request addressed to the processed input's requester, with `seen=`); operator outcomes and failures → report | pj-clusterintent `2156afa` |
| generated agents | `agag.entrance` already posts through `TopicResult(output=…)`; `agag init` templates carry no reply text of their own | — |

Direct `agentchat send` posts are covered by the CLI flags (step 1) for
every agent. Mention/callback routing and acceptance are untouched: the
line is a trailing code span with no mention in it, selfnote detection
reads the first characters, acks carry no line, and no new post, note or
run is introduced by the intent itself.

Dependencies: every consumer's `uv.lock` is bumped with its code (agfront,
agautolab, agforge, agobserver, archsage, cagent, the agentroom relay).
pj-clusterintent needed no Nautobot or desired-state change; the agautolab1
VM gets the new pins through its ordinary redeployment.

## Evidence

Suites on the new pin: pyagag 858, agentroom 339 (a new argue test: two
speakers' questions, answering autolab's by id settles only that one, a
non-request id refused), agfront 184, agautolab 293, agforge 265,
agobserver 117, archsage 28, cagent 202; `tsc` and `npm run build` clean.
Expectation changes in existing tests are all the same kind — a failure,
refusal or delivery line now ends with its `ag-post` line — plus the
autolab task-worker fakes now mark their output.

## Left as they are, on purpose

- **The Observer monitor's "please get it moving" posts** stay unclassified:
  they ask the origin conversation's *owner agent*, whose user id the
  monitor does not hold. They are never read as waiting for a human.
- **The ComfyUI notifier's callback** stays unclassified (a host tool, not
  an agent; its post is the trigger for the collecting run).
- Routine chat and the Project Room answer requests by the next-post rule
  only; explicit selection exists in the two dialogue rooms.

# failsafe p6 — step 1: the inconsistencies reproduced and located

## Deployment observed (2026-09-27 19:20Z)

- `nctl status` ok; `nctl drift` converged=46.
- Every listener, the gateway, forge's service, cagent-api and the relay
  are running. The pins are the p5 set: pyagag `e9e6229` in agfront,
  agautolab, agobserver, agforge, archsage and cagent, and `fb61ea3` in
  the relay.
- Observer's state:
  - `held.json` is `{}`. Both holds (o11711, o8512) were removed by a
    `hold --release` at 18:43Z, and nothing records who released them or why.
  - `tracked.json` is `{}`.
  - `monitor-state.json` has `receipts_from` 9187 and `obligations_from`
    11770.
- The panel (`/progress?fresh=1`) shows six requests as `waiting` on an
  answer "not yet taken up": o11711, o8512, o8816, o9010, o8721 and o9227.
  For o9227 it names **autolab** as the one to take the answer up.

## Replay fixture

Kept in ignored files under `pj-agdev/.local/failsafe-p6/`:

- `fixture/mirror-before.sqlite`: a `.backup` of Observer's mirror, newest
  message #15375.
- Observer's `held`, `retired`, `tracked` and `monitor-state` files.
- `board-before.json`.
- `replay.py`: traces each case off a copy of the mirror under a given
  `receipts_from` and prints the trace and the card.
- `fixture/replay-before.txt`: the output.

Realm exports cannot be committed, so the regression tests in step 6 use
synthetic realms built on the same shapes.

| case | trace/panel rule (`receipts_from`=0) | Observer's rule (9187) |
|---|---|---|
| o11711 (m11741's desk) | waiting — #8557 not taken up by Front | completed |
| o8512 (m8519's request) | waiting — #8557 not taken up by Front | completed |
| m8519 traced from the mission | completed ("accepted") | completed |
| o8816 (m8823) | waiting — #8866, #8891, #8914 | completed |
| o9010 (m9017) | waiting — #9060 | completed |
| o8721 (m8741) | waiting — #8778, #8811 | completed |
| o9227 (m9349, cancelled) | waiting — #9368, "by autolab" | waiting — #9368, "by autolab" |

The same evidence produces three different answers, from three readers.

## Findings

### F1 — the missing receipts are the historical resolve race, proven by the log

For every unserved answer, Front's listener log
(`agfront/.local/out/zulip-listener.log`) shows the same sequence. Here
is m8519 task 1:

```
14:26:50Z serving mention in 'work-m8519'/'workrun-task1-m8519'
14:27:02Z marked work-m8519/workrun-task1-m8519 served up to 8553 in front/front-protoprey-…
14:27:02Z mention 'work-m8519'/'workrun-task1-m8519': an event arrived during the serving; looking again
14:27:02Z mention 'work-m8519'/'workrun-task1-m8519': nothing owed now; skipped
```

- autolab posted the closing report #8557 and the ✔ (#8558) in the second
  its serving was acked (#8556).
- The serving marked only its trigger (#8553).
- The re-look after the ✔ found "nothing owed".
- The reply (#8560) already relays #8554's commit `a63e36e`, so the content
  reached Front, but no receipt was written for #8557.

The other cases:

- #8778, #8811, #8866, #8891 and #8914 follow the same three lines.
- #9060 and #9368 were skipped at their first look.
- All seven are from 2026-09-23, before 87ac87e (receipts) and 61df5ee
  (mention route judged by the trigger id).

Today's queue keeps `MAX(message_id)` for an entry re-armed during a
serving, and `owed()` judges the trigger by id. That window is closed.
Current code has no evidence of the race: the damage is historical records
that nothing can repair.

### F2 — readers disagree because they are not given the same rule

- `agag.trace.classify` treats an answer older than `receipts_from` as taken
  up when the requester spoke at home after it. This is p1's leniency, kept
  for exactly these pre-87ac87e marks.
- Only Observer passes `receipts_from` (9187). The relay, `agentchat trace`
  and the tests pass 0.
- So Observer tracks nothing (`tracked.json` `{}`), while the panel shows
  the same requests as waiting.
- Neither answer uses the evidence that matters here. Every one of these
  answers is covered by a later **decision of its requester**:

| answer | requester's decision after it |
|---|---|
| #8557 (m8519 task 1) | Front's `[state] accepted` #15321 in the task; `[acceptance] #15318 … after=#15301` on the mission |
| #8778, #8811 (m8741) | Front's `[state] accepted` #9939/#9940; `[acceptance] #8801 by 9` |
| #8866, #8891, #8914 (m8823) | #9944–#9946; `[acceptance] #8903 by 9` |
| #9060 (m9017) | #9954; `[acceptance] #9085 by 9` |
| #9368 (m9349) | the mission's `cancelled` #12603, written on a holder's request (#12599, Omni Agent) |

- `classify` asks for a served mark even under `DONE_WORDS`. That still
  catches a producer that claims `delivered`/`completed` for an undelivered
  result, and it must keep doing so.
- But `classify` does not tell those producer claims apart from an
  acceptance recorded by the requester, or from a cancellation the holder
  asked for. So an accepted result still demands a receipt as if it were
  unfinished work.

### F3 — the trace's answer depends on where it starts

- Tracing from m8519 says DONE; tracing from o8512 says AWAITING_DELIVERY.
- The requester (Front) and its home are known only from root notes whose
  home is inside the tree being traced. From the mission, Front's note in
  the task (#8538 → the front conversation) names a home outside the tree,
  so it is dropped from `anchors_of`, and the task has no requester to owe
  anything to.
- Front checked `agentchat trace 8519`, saw DONE, and told the Developer
  three times that nothing needed recording.

### F4 — cleanup manufactured ownership: m8519 under o11711

- Serving o11711, Front posted #15358 into m8519's own request
  conversation. `agentchat send` wrote
  `[selfnote][rootchat] front/front-desk-20260926-221323 #11711` (#15357)
  there first.
- That conversation is a request of its own: it began with the Omni Agent's
  post #8512 and has no root note of its own. The trace still hung it, with
  m8519 and #8557's wait, under o11711.
- The o11711 card then waited on m8519's work. The o8512 card showed the
  same wait a second time.
- It also opened Observer's `resolved_live` incident on the conversation
  (#15364), judged legit twice.
- The relations now in the records:

| relation | kind |
|---|---|
| o11711 → routinerun-20260926-2225 → m11741 → task 1 | work (root notes written when opening them) |
| o11711 → archsage `study-aisvgs-round2` | work (Front's refresh request) |
| o11711 → front-protoprey-…-20260923 (#15357) | **reference**: written while citing another request during cleanup |
| o8512 → m8519 → tasks 1, 2 | work |

### F5 — Front's repair attempts failed on the tooling, not on judgment

- **Hand-written receipt**:
  `[selfnote][served](for=work-m8519/workrun-task1-m8519#8557) — …`
  (#15362) is not a served note. `parse_served` needs
  `<channel>/<topic> <id>` at the end, and `send` appended the `ag-post`
  line. No tool writes a receipt on purpose, and no command inspects one.
- **Unresolve**: `unresolve work-m8519/workrun-task1-m8519` returned
  HTTP 400 because the work channel is archived.
- **Resolve**: blocked by the resolve guard (#15359) after Front's own
  #15358.
- The Developer then ruled the marker unrepairable (#15369). That ruling was
  about the tools available at the time. The records needed for a repair
  (the requester's home, the answer id, the acceptance) are intact and
  writable, because the receipt goes into the requester's home, not into the
  archived channel.

### F6 — the o11711 hold outlived its purpose, and its end is not inspectable

- failsafe p1 held o11711 because "how to resume m11741 is the Developer's
  open question".
- That question was settled by:
  - the Developer's instruction #15260 (continue m11741 to completion);
  - the acceptance `[change] accepted … #15273 +shown=15271`;
  - the integration `f57eed1`;
  - `[acceptance] #15273 by 15 (Front) after=#15271` (#15284);
  - the refresh `[selfnote][sagesync] … includes=f57eed1…` (#15337).
- None of these touched the hold, which is a file keyed by request with
  only a free-text why. Until 18:43Z the panel kept showing "held by a
  person". The release came from a direct `hold --release` by hand.
- The release removed o8512's hold in the same call, and nothing says who
  released either hold or why.
- o8512's hold ("accept after reviewing task 2") was settled at 17:29Z by
  #15318/#15323.

### F7 — the panel names the wrong actor for an owed answer

- `agag.progress._display` picks the requester from `requested_by`, the
  root notes' authors.
- A task started by its owner has only the owner's own note there (autolab
  #9353), so "not yet taken up by autolab-agstudio1" is wrong. The trace's
  `owed_to` (`Front`) is right.

## The path an answer takes, and where evidence is lost

| stage | record | lost or misread here |
|---|---|---|
| result published | owner's answer post naming the requester | — |
| acceptance | requester's post, `[change] accepted … +shown=`, `[state] accepted`, `[acceptance] … after=` | read by progress stages only; `classify` ignores it for receipts (F2) |
| serving input | listener journal `inputs` spans (since p3), trigger id | pre-p3: trigger only; the resolve race (F1) |
| reply delivery | journal `delivered_id` | — |
| receipt | `[served] <remote> <id>` in the requester's home | never written for the seven answers (F1); no deliberate writer (F5) |
| resolution/archive | ✔, channel archive | not a cause. The archive only blocked Front's unresolve (F5), and the receipt never needed the archived channel |

The resolve/archive race was a hypothesis. It is now proven for these
seven answers (F1), and the archive is not part of it.

## What steps 2–5 address

- **Step 2**:
  - one settlement rule for every reader, with the `receipts_from`
    leniency replaced (F2);
  - the requester known from every root note, so the trace no longer
    depends on where it starts (F3);
  - a reference no longer adopts another request's conversation (F4);
  - the owed actor taken from `owed_to` (F7).
- **Step 3**: a receipt inspection/repair operation in `agentchat`, and
  Front's guide (F5).
- **Step 4**: hold records with a stated purpose, a settlement condition,
  release history and ordinary tools (F6).
- **Step 5**: the seven answers, the two holds and the o11711 link
  reconciled with those tools.

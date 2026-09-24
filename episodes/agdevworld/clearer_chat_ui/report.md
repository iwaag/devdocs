# clearer_chat_ui — report

**Goal**: in agdevworld's conversations, see at a glance which agent posts
are progress, which are results, and which are waiting for *your* answer —
and which of those are still waiting.

**Outcome**: done and deployed. Every post can say what it is for, every
agent says it, every room shows it, and the rooms list what still waits
for you. A live Front Desk conversation showed a question labelled and
listed, answered from the list, and cleared, with Front's report labelled
and linked to the question it answered.

## The contract (`ag.post.v1`)

The braindump asked whether agag had an explicit marker for "waiting for a
human's reply". It had none: a mention only hands the turn over, and
`trace`'s `awaiting_human` was the inference "the agent spoke last", true
of every report. Now one machine line at the end of a post says it:

```
`ag-post intent=response_request to=8 ask=question seen=11422`
```

- `intent`: `progress` | `report` | `response_request`; `to` (user id,
  requests only); `ask`: `question` | `confirmation`; `re`: the request(s)
  a post answers or, from the asker, withdraws; `seen`: the input the
  poster had read.
- No line = unclassified, never "waiting". A malformed line is removed from
  the text and read as unclassified.
- One message, one write: the meaning is journaled and redelivered with the
  words, with no new code in the retry path.
- Runs declare it on the `ag-reply` fence (taught once in the shared reply
  guide); `agentchat send --intent/--to/--ask/--re` for direct posts.
- Agents read meanings, never the raw line.

Specification: `pyagag/docs/post-intent-v1.md`.

## What still waits (`ag.outstanding.v1`)

`agag.outstanding.read_requests` is one pure read model over a
conversation's history, used by `trace`, the relay (all four rooms) and the
browser. A request is its message id and keeps its intent; its state is
`pending`, `overtaken`, `answered` (reference / quote / next post),
`withdrawn`, `superseded` or `closed`. Only the recipient's speech settles
it; with two pending, an unreferenced reply settles neither. An answer is a
receipt, not approval. Input that arrived while the question was being
written makes it `overtaken` instead of a false wait. Incomplete or stale
sources are stated. `trace`: `awaiting_human` now means an explicit pending
request; `answered` is the new state for "answered, nothing asked".

## Delivered behaviour

- **Rooms**: a label on every post (icon + words, never colour alone):
  ⏳ progress, 📄 report, ❓ QUESTION / CONFIRMATION FOR YOU; a settled
  question keeps its label and says `answered in #n` / `withdrawn`. Above
  the dialogue: `❓ n waiting for your reply` with one chip per request —
  click to go to it and make your next post its answer (`↩ answering #n`).
  Status `❓ 2 questions for you — reply below` vs `answered · nothing is
  asked of you`. Dialogue renderings inherit their source post's label.
  Front Desk and Arguing Room have all of it; routine chat and the Project
  Room show labels and what waits (answering by next post).
- **Agents**: Front, autolab (its task worker now uses the shared reply
  contract; progress stream, start line, results, the wait for task
  agreement), forge (progress while rendering, deliveries as reports),
  Observer (a watch needing input asks; notifications report), archsage
  (sages' declared intent), cagent (front answer, operator outcomes).

## Evidence

Per step: `report1.md` … `report5.md`. Suites: pyagag 859, relay 339,
agfront 184, agautolab 293, agforge 265, agobserver 117, archsage 28,
cagent 202; frontend type-check and build clean. Browser screenshots
(ignored) under `pj-agdev/agdevworld/.local/shots/ccu/`, from a fixture
relay (the real relay code over a fake realm), the demo, and the deployed
room. Live: `#front › front-desk-20260925-ccu-live` #11422–#11428.

## Commits

pyagag `b292f5d`, `932d3c6`, `4ea6356`; agfront `ed3d15b`; agdevworld
`cb7440f`, `76faa28`; agautolab `292b1aa`; agforge `280bc2f`; pj-agdev
`c159406` (agobserver) and submodule pointers; archsage `668a692`;
pj-clusterintent `2156afa` (cagent).

## Limitations

- Only posts made after this rollout carry intent; older conversations
  read as unclassified (no backfill — by design).
- The Observer monitor's prompts to owner agents and the ComfyUI notifier's
  callback stay unclassified.
- Routine chat and the Project Room answer by the next-post rule only.
- In the demo without settings, a label chip in the upper box can run under
  the "re-voice" button.
- Whether agents choose `response_request` well is judgement: one live
  trial is one sample. Watch for questions posted as reports (missed) and
  reports marked as requests (false waits); the reply guide is the place to
  add evidence-driven wording.
- The agautolab1 VM runs the old pins until its next deployment.

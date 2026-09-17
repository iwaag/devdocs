# Argue p2 — report

Date: 2026-09-18 JST. Plan: [plan.md](plan.md). Source: [braindump.md](braindump.md).
Step reports: [1](report1.md) · [2](report2.md) · [3](report3.md) ·
[4](report4.md) · [5](report5.md).

## Delivered

1. **Memo conversations** (pyagag `agag.memo`). A conversation whose channel
   is `memo` / `memo-…` is presentation only, decidable before its first
   message and through every rename. The one rule is asked at the shared
   listener's intake, mention route, recovery and execution time, in
   `serve_topic`, in the mirror's note index and the client's note/mention
   narrows, by `agentchat send`, by the ComfyUI notifier's two intake paths
   and by the relay's board. A memo names its source with
   `[selfnote][memosource] <message id>`, never `rootchat`.
2. **Discussion and character rendering are separate runs.** Front's
   discussion roles (`front`, `desk`, `argue`, `routine_run`) receive no
   character settings and post their reply as written. A presentation role
   (`present`, no `agentchat`, no shell) re-voices recorded speech from a
   snapshot at one pinned settings revision; the validator guarantees every
   line cites a post of the job and speaks only for that post's speaker.
   `archsage` and `sage:<name>` are different speakers; a speaker without a
   character is shown as written. A durable job worker beside Front's
   listener renders new agent speech, coalesced, once per content; the memo
   is the truth, so a crash between post and bookkeeping costs no second
   run; failure is bounded; another settings revision is a separate, explicit
   interpretation and the earlier one stays; an edited source is identifiable
   as stale.
3. **The relay** lists, creates, reads, posts into and resumes argues by
   anchor message id, entirely from its mirror, with the Front Desk's write
   credential and submit tokens; both rooms share one `presentation` block
   (interpretations, per-post turns with sources, pending / failed / stale,
   an unavailable renderer).
4. **The Arguing Room** in agdevworld: the Front Desk scene parameterized by
   a room adapter — portraits, playback, history, composer and settings
   shared. Dialogue and original views, source lookup from a rendered line,
   interpretation selection, explicit reinterpretation, multi-speaker
   history with logical sage labels, room navigation. The Front Desk runs on
   the same flow; the combined judgement-plus-character path is deleted on
   all three sides. `archsage` and the argue background are registered in
   the settings manifest.
5. **Deployed** on agstudio: `#memo`, every listener and the notifier on the
   memo rule, the relay and the web image.

## Validation

| What | Evidence |
|---|---|
| Unit / fixture suites | pyagag 624, agfront 179, agentroom 295, comfynotify 26, agautolab 247, agforge 245, agobserver 72, archsage 28, cagent 202 — all passing; `tsc` + `vite build` clean. The memo tests fail when the rule is forced off. |
| Memo silence, live | 8 probe posts (every bot mentioned, two notifier commands, a copied root note, owned-looking names, rename, resolve) and a restart: no serving, no ticket, no reaction ([report5](report5.md)). |
| Argue participation, live | `argue-p2-smoke`: Front, `sage:arxiv`, the council; discussion workspaces contain no lore. |
| Rendering, recovery, reinterpretation, live | one coalesced job for five posts ($0.05); a simulated crash recognized from the memo with no second run; a second interpretation at an older revision on a resolved argue, which stayed resolved. |
| API budget | 90 consecutive room reads: 0 Zulip calls. 8.5 min of deployment and exercise: 273 calls, no 429, no periodic caller. |
| Browser | both rooms against the demo adapter and the deployed room against the real argue (`agdevworld/.local/shots/argue-p2/`). |
| **Human-led session** | **Pending.** The room is ready at `/?view=argue`; no human has opened, read, replied in or resumed an argue through it yet. Nothing the Omni Agent posted is offered in its place. |

Costs of the live exercise: substantive $0.51 (5 runs), presentation $0.10
(2 runs); plus one $0.17 probe of the presentation role in step 2.

## Remaining limitations

- **Mirrors miss moves in channels their bot has not joined.** Fixed for the
  relay (its reader now joins new public channels and resyncs). An agent
  listener's own mirror still has it for channels its bot is not subscribed
  to; Front is subscribed to `#argue` and `#memo`, which is what this phase
  needs.
- **A desk conversation's anchor is its first post.** Deleting that post
  orphans its memo (the results remain in `#memo`, unlinked).
- **Rendering cost scales with agent speech**, not with reading. `present`
  is on Sonnet 5; moving its profile to a cheaper model is one line.
- **Pending is derived, not reported**: the relay cannot tell a slow renderer
  from a stopped one, and says so (`unavailable` after 15 minutes).
- A rendering of a very long post compresses wording; the original is one
  click away, and the check cannot verify faithfulness of content — only
  that each line cites the right speaker's post.
- The Arguing Room has no finish button; an argue ends by Front's checked
  outcome or the existing completion door.
- agautolab1 (the VM) was not redeployed; it has no listener.
- Old Front Desk conversations show their old `ag-dialogue` block as text.

## For the next phase

The braindump's actual aim — argues that reach further than what the agents
already know how to do — was deliberately not addressed here; this phase
built the place where a human can lead one and the evidence trail to study
it. First observations to bring to that work are in
[report5](report5.md): Front moves ahead faster than a human can type,
courtesy mentions still buy frontier runs, and Front's replies carry a
preface. They should be weighed against real human sessions before any
protocol is changed.

## Commits

| Repository | Commits |
|---|---|
| pyagag | `0af761d`, `7beeec5` |
| agfront | `3780d1b`, `03bf127`, `66d4dd4` |
| agdevworld | `4d3f9a6`, `6c6958f`, `82bac93`, `2e5ff44` |
| agdevworld-settings | `1af317b` |
| pj-agdev | `3c514c5`, `df4e11f`, `472b6c6`, `e571bf9` (comfynotify, agobserver lock, submodule pins) |
| agautolab / agforge / archsage | `27c336f` / `d160337` / `61d6f95` (lock) |
| pj-clusterintent | `8bcd600` (cagent lock) |

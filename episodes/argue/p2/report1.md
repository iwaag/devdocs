# Step 1 — memo conversations are non-reactive

Date: 2026-09-18 JST. Plan: [plan.md](plan.md) step 1. Nothing in this step
posts to the realm or restarts a listener: the running agents are still on
the code they had, no memo channel exists in the realm yet, and deployment
is step 5.

## The distinction

| Thing | Decision |
|---|---|
| What a memo is | A conversation whose **channel** is `memo` or is named `memo-…` (`agag.memo.is_memo_channel`, case-insensitive). Presentation only. |
| Why the channel | It is decidable before the first message exists, it survives every topic rename (a resolve is a rename), and every consumer already holds the channel of the message in its hand, so the rule costs no read. A `✔` could not be the silence: the ComfyUI notifier accepts commands in resolved topics on purpose. |
| Link to the source | `[selfnote][memosource] <message id>` — the id of a message in the source conversation (an argue's anchor note, a Front Desk conversation's first post). Its own tag, **not** `rootchat`: a root note takes part in callback routing, `threads/` and completion discovery, and a presentation is none of those. |
| Data in a memo | One fenced `ag-memo` JSON record per post (`render_record` / `parse_record`). What the record holds is the writer's business (step 2). |
| Memo topic name | `memo_topic(source id, label)` → `<slug>-s<id>`, at most 60 characters. A convenience; a reader finds a memo by its `memosource` note. |

## Where the one rule is asked

| Consumer | Point | Behaviour in a memo |
|---|---|---|
| `agag.listen` | intake (`_intake`, `_consider`) | a post, a flagged mention and a rename into an owned name are all dropped before the queue |
| `agag.listen` | recovery (`recover`) | memo topics are skipped on both routes, at startup and after a resync |
| `agag.listen` | crash recovery (`_resume_running`) | an entry left `running` is dropped and logged `is a memo; dropped` |
| `agag.listen` | execution time (`owed`) | an entry an older process queued is `nothing owed now` — the queue is a file and outlives the code that filled it |
| `agag.topics.serve_topic` | entry | returns before `whoami`, the ack or the handler, for a memo served **or** replied into — whatever route reached the call (Front's `continue_deliveries`, a direct handler call) |
| `agag.mirror` note index | `notes()` | notes sitting in memo channels are left out unless `include_memos=True`; so the listener's served marks, Front's root-note reader (`agfront.routine`), the relay's served marks and completion discovery never read a copied note |
| `ZulipClient` | `own_notes`, `mentions` | memo-channel messages are filtered from both narrows (the non-mirror paths of the same lookups, and the notifier's catch-up) |
| `agag.zulip` | `topic_from_event`, `is_mention_for_us` | the older event-path helpers answer "not for us" |
| `agentchat send` | `ensure_rootchat` | writes no root note into a memo |
| comfynotify | `commandable` (catch-up narrow) and `addressed` (live queue) | no ticket, no reaction, no refusal post, under `✔` too; asked before the high-water mark moves |
| agentroom `Ops` | `Topic.add`, `snapshot` | a memo is **held** (the rooms display it) but reads no link notes and is never a board row |

cagent's listener is `agag.listen` plus a DM thread, the Observer's worker
reads only its own channel, and archsage / agautolab / agforge / agfront all
go through `listener_main`; none of them has a second intake, so the pin bump
is their whole change. The mirror still stores memo conversations like any
other channel — that is what "available for display" means here.

## Verification

Focused fixtures, all over the mirrored `FakeRealm`:

- pyagag `tests/test_memo.py` (7 tests): an owned-prefix topic, a real
  flagged mention, a `use <option>` command line, a copied `[selfnote]` and a
  quoted mention inside a record are all unserved while the ordinary topic
  beside them is served; a rename into an owned name, a resolve and an
  un-resolve stay quiet; a restart with a `pending` owner entry, a `pending`
  mention entry and a `running` entry in memo channels ends with an empty
  queue and no serving; a `[selfnote][served] … 999999` written into a memo
  **by the bot itself** does not mask a real mention elsewhere; the event
  helpers and `serve_topic` refuse a memo without touching the client.
- Mutation check: with the rule forced to `False` the same file fails at the
  first assertion (three memo servings appear), so the tests bite.
- "An ordinary argue invitation still works": pyagag's existing
  `tests/test_argue.py` and `tests/test_listen.py` are unchanged and green.
- comfynotify `test_a_memo_is_never_a_command_on_either_path`: a well-formed
  command, a malformed one and one under `✔`, on the live path and the
  catch-up path; the same line in an ordinary channel is still ticketed.
- agentroom `test_a_memo_conversation_is_held_for_display_and_is_nobodys_row`.

| Suite | Result |
|---|---|
| pyagag | 623 passed |
| comfynotify | 26 passed |
| agdevworld/agentroom | 285 passed |
| agfront / agautolab / agforge / agobserver | 177 / 247 / 245 / 72 passed |
| archsage / cagent | 28 / 202 passed |

## Commits

| Repository | Commit |
|---|---|
| pyagag | `0af761d` `agag.memo`, listener, note index, client narrows, `agentchat`, README |
| pj-agdev | `3c514c5` comfynotify intake + submodule pins (agobserver and comfynotify locks) |
| agdevworld | `4d3f9a6` ops engine |
| agfront / agautolab / agforge / archsage | `3780d1b` / `27c336f` / `d160337` / `61d6f95` lock only |
| pj-clusterintent | `8bcd600` cagent lock only |

## Left for later steps

- The `memo` channel itself is created at deployment (step 5); every running
  listener and the notifier are restarted there so the rule is live before
  the first memo is written.
- The Observer accepts any public conversation as a notification
  destination, a memo included. Posting into a memo is harmless (nothing
  reacts), so it was left alone.

# Step 2 report — stable identity through the affected paths

Plan: [plan.md](plan.md) step 2. 2026-09-24 (JST), by the Omni Agent. Code and fixtures only; the
consumers' environments are on the new pyagag, the running listeners are not restarted until the
step 5 rollout.

## The rule

One rule, already `agag.zulip.locate`'s for a single lookup, now applied wherever a record names a
conversation: **where the record's post is now decides; the name written beside it is the
fallback only when that post cannot be found.** A new module `agag.identity` holds it
(`whereabouts`, `served_key`, `bare`, `key`) for any lookup — a mirror (free) or a client (one
call). Names stay for display and for initial discovery.

## What changed

| Path | Before | Now | Reproduction |
|---|---|---|---|
| Served marks — listener | keyed `(channel, bare name)` written in the mark | keyed by where the mark's post is now (`served_key`, off the mirror) | **L1** fixed: a renamed callback topic is not served again after a restart |
| Served marks — trace | name comparison | the mark covers a conversation when the post it names is *in* it (membership in the history the trace already read — no extra call); the name only for a post older than the history reached | **R9** fixed |
| Callback serving's anchor | the trigger id — on the mention route a post in the *caller's* topic — went into `AGENTCHAT_HOME_ANCHOR` (23 of 64 anchored root notes in the realm) and into the reply's destination lookup | `serve_topic` sets `home_anchor`: the trigger when it is in home, else the newest post read in home; the run environment and the destination use it; the record keeps it | code fix; the same-channel misdelivery it prevented was a hypothesis |
| Trace — which conversation a root note means | the name written in the note (`_anchored_children`), anchor ignored | `_settle`: the anchor decides (a post here, or located elsewhere in the same channel; an anchor in another channel is ignored as foreign); without one, a note older than the conversation now holding the name cannot mean it — it means the predecessor named by that holder's `[replaces]` (one hop) or nothing; a note whose anchor is a post in a conversation the tree reached is re-homed there (a renamed home) — decided against histories the trace reads anyway, to a fixed point | renamed origin, reused name, retirement: `test_identity.py` |
| Trace nodes and candidates | no identity; `Candidate.key` = kind + name + evidence | `Node.anchor`: identity note or the root note that opened it (the oldest of them), else its oldest post; `Candidate.anchor`, and the key uses it | **R7** fixed at the source |
| Threads / continuation list (`remotes_for_home`) | every note naming the home's name | with `home_messages=` (Front, Front's argue, autolab pass the history they already read): the same anchor/age rule; a note whose anchor is a post in home counts whatever name it carries | reused and renamed home: `test_identity.py` |
| autolab's task root note, forge's run root note | `[rootchat] <channel>/<plan topic>` | `… #<mission id>` / `… #<request id>`: the task stays under *its* mission after a retirement frees the name | covered by the retirement fixture |
| autolab callback route | served the root note's home **by name**, no ✔ check (a ✔ task was reopened as a twin), and wrote a served mark for the newest post in the caller's topic (spending a mention that arrived mid-run) | home `locate`d by its anchor; a ✔ task is not reopened; under a listener the mark is the listener's, bound to the trigger (Front's rule) | code fix |
| autolab progress lines | `live_topic_name` (prefers an open twin) | located by the serving's anchor, then the name | code fix |
| Redelivery of a prepared reply after a restart; the post-delivery served mark | the name frozen at prepare time; home by name | the destination re-`locate`d by its anchor (a gone one is terminal); home by its anchor off the mirror | code fix |
| Front's routine `origin_of` | earliest root note only (ignored `[rootchat-moved]`); report posted by name | `effective_rootchat`; report posted where the origin's anchor is | code fix |
| Observer monitor | request = origin's live name; incident = `Candidate.key`; rescue checked for origins looked at *by name*; requests posted to the name seen at discovery; a lost store adopted by incident topic name and counted one request blindly | request = its origin's **first post** (`o<id>`); incident = `o<origin>:n<stalled conversation's anchor>` with the kind as an attribute (a kind change is posted into the same incident and keeps its allowance); one incident per conversation per look (most urgent kind); requests and incident posts go where their anchors are now; a lost store adopts the incident from its `[selfnote][incident] <key>` note wherever that topic is, counting the requests already on record and keeping a closed one closed | **R2, R5, R6, R7** fixed |

## Fixtures

| Suite | Result |
|---|---|
| pyagag | 751 passed. New: `test_identity.py` (8: renamed origin keeps its tree and anchors; reused name inherits nothing; an unanchored older note is not given to the new holder; a retired mission keeps its tasks and the replacement only its own; a served mark follows its conversation through a rename; a rename keeps the candidate key; a deleted anchor is nowhere and the name is not guessed; threads for a reused and a renamed home), `test_handoff_binding` (a callback serving is anchored in home) |
| agobserver | 89 passed, 4 xfailed (R1, R3, R4, R8 — step 3). New: a lost store adopts a renamed incident topic with its request counted; a recovery request goes where the origin is now. The p1 fixture's root notes now carry the anchors today's writers write |
| agautolab / agforge / agfront | 258 / 265 / 182 passed; four expectations updated from `[rootchat] <home>` to `[rootchat] <home> #<id>` |

Why R2 passes here rather than in step 3: once the incident is keyed by the stalled conversation
and not the kind, a task that goes from *not started* to *posted into and never acknowledged* is
the same incident with the same two requests. A new blockage that is not yet overdue still reads
as an absence, and absence is step 3's to stop treating as recovery.

## Deployed state

pyagag `72ea6af` is pushed and locked in agfront (`72b1bc8`), agautolab (`33dc4bc`), agforge
(`91f94e8`) and agobserver (`pj-agdev` `c3c33a1`), and synced into their environments, so a
respawn by launchd would load consistent code. **No listener was restarted**: the running
processes still run `87ac87e`. archsage and cagent need no code change and move at the rollout.

## What still joins on a name (left, with the reason)

- `Conversation` equality is still `(channel, topic)`. Changing it would change every set and
  dict in the realm's code at once; the paths above compare what matters explicitly instead.
- autolab's worklog and forge's record reduce an anchor's exact location to the bare name and
  write through `live_topic_name`, which prefers an open twin. Only a twin beside a ✔ topic can
  divert them, and p1's `unresolve` folds twins; the rewrite touches every `Mission`/`Task` user.
- The listener's queue and serving records are keyed by `(channel, bare name)`: a new
  conversation under a reused name can be handed the old one's `interrupted` serving as
  `context.previous`. Needs a reuse *and* an interrupted serving at once.
- `listen.owed` judges a mention in `✔ x` against an open twin `x` when both exist.
- `agentchat send <channel> <topic>` addresses by name by design — it is how a run names where to
  speak — and a reused name is the conversation that holds it now.
- Front's routine `MirrorReader.rootchats` (routine discovery) is a copy of `rootchat_notes` and
  still joins on names; routines are outside this phase's discovery (plan, goal section).

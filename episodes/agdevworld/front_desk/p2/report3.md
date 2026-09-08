# Step 3 — generate and persist a short dialogue

agfront `ef26752`, agdevworld `62a8084`. Zulip stays the only record.

## The format

One post, Front-authored, in the `front-desk-<id>` topic — the same post
the existing serving already makes, so the callback/completion flow is
untouched and no second run is bought for presentation:

```
<the reply to the developer, in character>

```ag-dialogue
{"schema": "ag.frontdesk-dialogue.v1", "settings_revision": "<sha>", "turns": [
 {"character": "front", "text": "…"},
 {"character": "autolab", "text": "…",
  "sources": [{"channel": "work-g-13", "topic": "workrun-task1-g-13", "message_id": 5203}]}
]}
```
```

`ag.frontdesk-dialogue.v1`: ordered `turns`, each a `character` id from the
pinned settings revision, a `text`, and optional `sources` (channel, topic,
message id). `settings_revision` is the revision the run was given. A reply
with no block is a Front-only reply — the normal case for small talk. A
fence keeps a `@**name**` inside it from being a live mention, and agfront
reduces one to the bare name anyway.

## What exists now

- **agfront `src/agfront/dialogue.py`** — `split_reply` takes the *last*
  `ag-dialogue` fence off the run's output; `parse_block` validates it
  against the pinned `CharacterSettings` (schema, non-empty turns, at most
  12, character ids the revision knows, non-empty text under 1,200 chars,
  well-formed sources, mentions reduced) and returns a canonical
  `Dialogue` with the **pinned** revision stamped — the run's own value is
  ignored; `render` writes reply + block; `finish_reply` is what the
  serving calls. An unusable block (broken JSON, unknown character, empty
  text, over-long post, no settings) keeps the reply and appends an
  `ag-dialogue-error` fence naming the reason, logs it, and writes
  `dialogue-error.txt` beside the run's files. The run is never repeated.
- **`zulip_listener.serve`** finishes a desk run's output through it
  (`context.step = "dialogue"`); an ordinary `front` reply is never parsed.
- **The guide** gained "The dialogue block": when to write one (a callback
  with a result, plan, question or failure; not small talk), the exact
  shape with an example, 2–5 turns in Japanese in each lore's voice, the
  other characters' lines re-voiced from what they actually said with
  results/names/numbers/links kept exact, sources cited by the thread's
  `#id`, and that the reply above the block is still the complete answer.
- **The relay** (`agentroom/src/agentroom/frontdesk.py`, `split_dialogue`)
  splits the block off every post by Front: `content` is the reply alone,
  `dialogue` the shaped scene (`null` when Front-only), `dialogue_error`
  the recorded reason (from the error fence, or from a block the relay
  itself could not read). `latest_reply` carries the same fields. A
  Developer post is shown as typed, fences included. Nothing is stored
  outside Zulip, so reload and relay restart restore it exactly as before
  (held / read / unknown).
- **`src/frontDeskState.ts`** — `DeskTurn`, `DeskCitation`, `DeskDialogue`
  and the two new `DeskPost` fields; the demo source now plays a
  four-turn Front/Autolab scene on its third reply and an unusable-block
  case on its fourth, so step 4 can be looked at without a run.

## Verification

- agfront `uv run pytest -q` → **60 passed** (22 new): last-fence split
  with a quoted earlier block left as prose; canonical re-serialization
  with the pinned revision; mention reduction; nine unusable shapes each
  naming what is wrong; no settings → no character can be named; the post
  is reply + canonical block; a reply without a block is posted as is; an
  unusable block keeps the reply, adds the error fence, writes
  `dialogue-error.txt` and logs; a block with no reply around it still
  gives the developer Front's lines; an over-long scene is dropped rather
  than truncated by the realm; the listener posts reply + canonical block
  with the pinned (not the run's) revision, posts the error fence for a
  broken block, and never parses an ordinary front reply.
- agentroom `uv run pytest -q` → **169 passed** (5 new): the split, the
  error fence, an unreadable block, posts and `latest_reply` carrying
  `dialogue`, a Developer post quoting a fence shown as typed.
- `npm run build` passes.
- The agfront listener was kickstarted after step 2 and again after this
  step (idle both times, checked on `/inflight/front-agstudio1`), so the
  guide that asks for a block and the code that validates it are live
  together. The relay still runs the pre-p2 code until step 4's browser
  check needs `/settings` and `dialogue`.

A live multi-character output — a real callback producing a real scene
with sources — is step 5's bounded delegated task; nothing was posted in
this step (a Front Desk post is a paid run).

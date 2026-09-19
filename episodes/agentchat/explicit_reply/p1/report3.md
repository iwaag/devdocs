# Step 3 report — Bind handoffs to requests

Plan: [plan.md](plan.md) step 3. Repositories: `pyagag` (`b9592c2`),
`agfront` (`db3224f`). Part of this step landed in step 1
because it fell out of the serving journal; it is listed here with the rest
so the contract reads whole.

## What is captured at intake

Every serving record (`agag.serving`, in `listener.sqlite`) now holds:

- **the triggering message id** (`trigger_id`: the newest post that queued
  the entry, taken at `take()`),
- **the requester**, written twice: at intake from the trigger message's
  sender (the listener reads it off the mirror when it opens the record)
  and again at `executed` from the processed input (`requester_of`, the
  last other speaker at or below `input_up_to`),
- **the home conversation** (`home_channel/home_topic`, what `serve_topic`
  is serving: the `front-*` conversation for a callback),
- **the intended destination** (`reply_channel/reply_topic` as located at
  delivery, see below).

## Location by stable ids, not names

- `Conversation` (`agag.selfnote`) carries an optional **anchor**: the id
  of a message in it. `"<channel>/<topic> #<id>"` parses and prints; two
  conversations are equal by pair whatever anchor either carries, so every
  existing comparison keeps its meaning.
- The run's environment gets `AGENTCHAT_HOME_ANCHOR` beside
  `AGENTCHAT_HOME` — read from the bound journal's `trigger_id`, so no
  consumer passes it — and `agentchat send` writes it into the root note:
  `[selfnote][rootchat] front/front-desk-1 #7225`. Old notes without it
  keep working.
- `agag.zulip.locate(client, conversation)`: by the anchor's current
  location first (`conversation_of`, a `✔ ` name included), else by name
  across the resolve rename. **Gone** (`None`) only when the anchor was
  confirmed deleted *and* the name is listed nowhere; a name alone is not
  evidence, and a lookup that got no answer is not "gone" either.
- `serve_topic` locates the reply's destination **at delivery time** by the
  trigger id (`_destination`): a topic renamed while the run was in flight
  is answered under its new name, one resolved meanwhile is answered under
  its `✔ ` name (no twin is opened), and the outcome is written on the
  record (`extra.destination = {state: renamed|resolved|gone, asked,
  found}`) and logged. A destination that is gone is a **terminal**
  `DeliveryError`: the entry is kept `failed` with the reason, nothing is
  posted under whatever might take the name.
- Front's callback (`handle_mention`) locates home with `locate` — a home
  whose root note carries an anchor is found after a rename, and a reused
  name is not mistaken for it — and the existing `replaces` hop
  (`inherited_rootchat`) is unchanged.

## Who is handed the turn

- The reply's mention is the **requester recorded from the processed
  input**, never a speaker looked up at send time. A third party who posts
  during the run is outside the processed boundary and is not the addressee;
  their post is unprocessed input and is served next (fixture: the
  bystander at home is answered by the *following* serving, addressed to
  them).
- For a reply posted elsewhere (`reply_to`), the reply conversation is read
  **before the run** and its last other speaker is the addressee. If it
  cannot be read, `serve_topic` raises a non-terminal `DeliveryError`
  before any model runs: the entry stays pending and is retried with the
  listener's backoff. A failed lookup never silently removes the handoff.
- `handoff_mention` (the send-time lookup) stays for callers that want it
  on purpose; the skeleton no longer uses it.
- Deliberate delegation is untouched: `agentchat send` with the root note,
  home-directed callback replies, `argue` invitations by explicit mention.
  A callback's reply at home names the developer, never the delegate, so
  completion reaches the requesting conversation without a reciprocal post
  (fixture: two delegates answer at once; both replies at home start with
  `@**Dev**`; nothing is posted into either delegate's topic).

## Served marks

- Written by the **listener** after a confirmed delivery on the mention
  route, into the served home, bound to `trigger_id` — the mention that
  triggered the serving — not to the newest post in the remote topic. A
  mention arriving during the run stays owed; a restart between the home
  reply and the mark writes the mark from the record without a rerun (both
  pinned in step 1).
- Coalesced requests: two mentions queued before the serving are one entry,
  one serving, one mark at the newer id — both were in the processed input.
  Multiple outstanding delegates are separate remote conversations and
  separate marks; answering one consumes nothing of another.
- Front writes its own mark only when no listener journal is bound (a test,
  a one-off command); under `agag.listen` the listener's mark is the one.
  `relay_late_answer` keeps its own mark: that path posts directly and
  produces no delivered record.

## Evidence

`pyagag`: **669 passed** (11 new in `tests/test_handoff_binding.py`).
`agfront` against this checkout: **181 passed**.

| Fixture | Pinned |
|---|---|
| rename during the run | reply lands under the new name; nothing under the old; record says `renamed` |
| resolve during the run | reply under `✔ name`; no twin; record says `resolved` |
| every message deleted during the run | entry `failed` "no longer exists"; record says `gone`; nothing posted |
| reuse of an old topic name after a resolve | the new mention under the freed name is served and marked at its own id |
| two delegates at once | two servings, two marks keyed by remote, both replies at home to the developer, no post into a delegate's topic |
| repeated callback | a post not naming the bot is no callback; the second mention is served and marked separately |
| third-party interjection at home during a callback | first reply to the developer; the bystander is answered by the next serving |
| unreadable reply conversation | `DeliveryError` (non-terminal) before any run; nothing sent |
| reply elsewhere | addressed to the remote's last other speaker as of before the run, not a latecomer |
| run environment | `AGENTCHAT_HOME_ANCHOR` set only under a bound journal |
| root note with anchor | round-trips; `locate` follows the id to `✔ renamed`, falls back to the name when the id is gone, and uses the name when there is no anchor |

Rename/resolve/replacement of the *home* on the callback route is covered
by agfront's existing `test_replacement.py` and `test_anchor_correction.py`
(the `replaces` hop and the deliberate move), now running over the
anchor-aware lookup.

## Remaining gaps

- The root note's anchor is written only by runs started under a listener
  (a journal is bound); `agentchat send` from a shell writes the name only.
- `note_served` in the DM route and in `relay_late_answer` still marks up
  to the remote's newest real message, as before.

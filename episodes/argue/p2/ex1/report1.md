# Step 1 — a human can finish an argue from the room

Date: 2026-09-18 JST. Plan: [plan.md](plan.md) step 1. Commit: agdevworld
`d7fa2ad`.

## What was there

An argue resolved only when Front's reply ended in an `ag-argue` block and the
listener verified the named channel, document and autolab answer. The room hid
`finish ✔` because the argue adapter had no `completion`, and the completion
engine classified `#argue › argue-*` as a generic `TOPIC` — which, had it been
asked, would have *owned* everything autolab anchored to the argue by a root
note (`workplan-setup-<slug>` and, through it, the mission and its channel),
because that is the ordinary rule.

The realm already held the case the braindump describes: the argue the
Developer led this morning, `argue-20260918-124701` (anchor 7217, 13 posts,
in Japanese, ending with "ありがとう、良いargueだったよ"), stayed open after Front's
last reply said "nothing left open".

## Delivered

- **`ARGUE` completion kind** (`agentroom/closing.py`): `#argue` and the
  `argue-` prefix (constants from `agag.argue`). A request kind, so another
  argue reached from a desk by a served note is "another request: an argue".
- **Boundary**: the closure is the argue topic and nothing else. The walk is
  still made, so the preview shows what the argue led to — the setup plan
  anchored to it — as exclusions with the reason "what the argue ended in
  stays open: closing an argue ends the discussion and nothing it opened".
  No work record, no channel. The origin desk conversation is not in the
  graph (an `[argue] from …` note is not a root note) and the memo is not
  either (`[memosource]` is deliberately not `rootchat`); both stay as they
  are. The memo is left open on purpose: presentation only, a ✔ there
  changes nothing.
- **A human close is not an outcome**: the engine resolves the topic and
  writes nothing — no `[selfnote][outcome]`, no `ag-argue` block, no state
  note. Front's own outcome path is untouched. No reason post; the plan made
  it optional and a post in the argue would buy Front a run.
- **Routes** `GET /argues/<anchor>/close-plan` and `POST /argues/<anchor>/close`:
  the shared door keyed by anchor. `ArguingRoom.completion_key` resolves the
  anchor to the argue's current topic through the same `locate` the reads
  use, so a renamed argue is followed and a reused stem is never this one.
- **Room**: `argueAdapter.completion` over those routes; `finish ✔` appears
  and the existing `FrontDeskClosePanel` works unchanged (its title reads
  `FINISH THIS ARGUE`). After a close the scene fires
  `agdevworld:completed`, so any relay-backed view on the page re-reads. The
  demo adapter plays close → ✔ → post resumes. Posting into a ✔'d argue
  resumes it exactly as before (relay `_unresolve` + the `resumed` words).
- **Fixture door**: with `&demo=1` the scene is reachable as `window.__room`,
  so the screenshot driver can click Phaser buttons at their real positions.
- **Front's optional "let's stop here"**: not done. p2's report asked that
  facilitation changes wait for real sessions, and this step's own finding
  (below) is the first such session.

## Not re-serving Front on the ✔

`agag.listen._intake` ignores a move into a `✔ ` name and a message that is
already resolved; a Notification Bot line is not speech since pyagag
`0a33830` (`SYSTEM_REALM = zulipinternal`), and agfront's lock is at
`7beeec5`, which includes it. The live confirmation — agfront's log staying
quiet across a close from the room — is step 4's, after the relay is
redeployed.

## Validation

| What | Evidence |
|---|---|
| Relay suite | `agentroom`: 301 passed (was 295). New: classification; the argue's plan is `[argue topic]` READY with the setup plan excluded and the task topic never read; a close resolves only the argue and posts nothing; a resolved argue is `already ✔`; another argue reached from a desk is another request; over HTTP by anchor through a rename: plan → close (no message written) → `already ✔` → stale fingerprint 409 → a post resumes. |
| Browser, demo | `agdevworld/.local/shots/argue-p2-ex1/s1-*`: `finish ✔` in `/?view=argue&demo=1`; the preview names only `#argue › argue-demo` with `#pj-demo › workplan-setup-demo` under "not this request's — left alone"; the click gives `CLOSED — 1 changed`, the caption `✔ argue-demo`, status `✔ resolved`; a further post gives `↩ resumed`. |
| Real relay, read-only | A second relay from this tree on :8095 (own mirror, no chat credential): `7199` (`✔ argue-p2-smoke`) → `already ✔`; `7217` (the Developer's live argue) → READY, `pj-worldtrend › workplan-setup-worldtrend` excluded; `424242` → 400. Each plan: `zulip_calls: 0`. |
| Build | `tsc` + `vite build` clean. |

The live close of `7217` is deliberately not made here: it is the human's
argue, and the plan's step 4 session is where a human closes one from the
room. The :8095 relay is kept up for step 2's state display check.

## Observations for later phases

- The first human-led argue happened before this exercise (12:47–13:15 JST
  today). Front ended it with "nothing left open" and no outcome; the human
  had no way to close it. This step gives them one.
- Front's listener served the human's thanks (`7255`) and replied (`7257`):
  a courtesy post still buys a run, as p2 noted for mentions.

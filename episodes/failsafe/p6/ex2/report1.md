# failsafe p6 ex2 — step 1: align the rule and the affected checks

## Status

Done.

The first attempt stopped: the Omni Agent's session auto-mode classifier
refused the proxy statement as *Security Weaken*, and nothing was changed
(first version of this report). The Developer then allowed it by hand, and
the step was carried out.

## What L3 was, from the record

| post | sender (id) | client | content |
|---|---|---|---|
| #15569 | Omni Agent (9) | Python-urllib | the request, standing in for the Developer: "you need not ask me again" |
| #15572, #15576 | Front (15) | Python-urllib | refusal: it needs the Developer's own go-ahead, per its standing rule |
| #15580 | Developer (8) | **Python-urllib** | 「進めてください。」 |
| #15606 | Front (15) | Python-urllib | `[selfnote][disposition] cancelled a15569 … by 8 (Developer) #15580` |

#15580 came through the API client, so only the account is evidenced.
Step 2 corrects ex1's "in person".

## The rule

**Front's harness memory** `feedback_confirm_before_contacting_agents`
(Front's Claude Code project memory, outside the repos) was rewritten:

- the go-ahead comes from the developer **or the Omni Agent**, which
  carries the developer's full delegated authority;
- its instructions, confirmations, approvals, cancellations and hold
  releases count as the developer's, and are never re-confirmed;
- its decisions are recorded as its own words;
- no other agent's word is the developer's;
- L3's refusal is named as the wrong reading.

**Front's guides** gained the same rule, worded from what a run sees:

- files: `front/guide.md` and `desk/guide.md`, agfront `12ce0f1`;
- a speaker marked `— with <person>'s full authority` is that person's
  decision;
- reply to it, and record its decision on its own post.

## The one statement

- **`agag.people`** (pyagag `3139390`) reads the host's
  `~/.config/agag/people.toml`, beside `refs.toml`:

  ```toml
  [[proxy]]
  user = 9
  name = "Omni Agent"
  for = 8
  for_name = "Developer"
  ```

- **Why a host file.** No existing definition separates the proxy: every
  realm bot, the Omni Agent included, is owned by 8 or by Provisioner (17).
  Every process that makes these checks runs on this host.
- **How it is read:**
  - authority runs both ways (the Developer may release a hold the proxy
    placed);
  - a missing file means plain id equality;
  - tests never read the host file (`conftest`, `AGAG_PEOPLE_CONFIG`).
- **Nothing is aliased:**
  - replies and mentions still go to the actual sender;
  - records name the speaker, plus the authority used:
    `by 9 (Omni Agent) for 8 (Developer)`.
- **The chatlog** (`format_chatlog`) marks the proxy's lines
  `[Omni Agent — with Developer's full authority]`. Every serving reads the
  relationship from the conversation itself instead of deciding it afresh.
  Mentions and reply routing are untouched.

## Checks changed

| check | before | now |
|---|---|---|
| mission acceptance (`Decision.may_decide`, `accept_mission`) | only a holder's own id, or the reserving person's own id | the proxy decides what its principal holds, reserved approvals included; the record says `for 8 (Developer)` |
| acceptance in person | any bot refused | a proxy may record in person; an ordinary agent is still refused |
| acceptance scoping | an agent-holder's words count only in the mission's conversations | a proxy deciding for a person counts wherever it spoke, as the person would |
| hold release (`holds.release`, also `agobserver.hold --release`) | only the holder's own post or id | the holder or whoever carries their full authority; the record names the speaker and `for <holder>` |
| a question put to the Developer (`outstanding.read_requests`) | answered only by id 8; withdrawn only by its sender | the proxy answers or withdraws it; an ordinary agent does neither |
| trace reading of `[acceptance]` | — | parses and shows the `for …` part |

## Deliberately unchanged

- **Dispositions** had no identity gate, and name the actual speaker.
- **autolab** task agreement, start and cancel accept any non-self
  requester.
- **Worker checks:**
  - evidence by the mission's own agent is refused;
  - a hold by the unit's owner is refused;
  - the owner's own `accepted`/`done` is no decision.
- **Receipts** (`trace._reconciled`) are the owed agent's listener's.
- **Argue desire** (`argue.validate_desire`) requires a non-bot human, so
  the proxy's words are not recorded as a desire. An argue exists to draw
  out a human's desire; changing that is not this plan's call (see
  report.md).
- **`agentchat reserve`** runs inside a serving, so the proxy's reservation
  is recorded by the agent it speaks to. Its own-post refusal stays.

## Tests

- `pyagag/tests/test_failsafe_p6ex2.py`: 14 at this step.
- Full pyagag: 1122 passed.

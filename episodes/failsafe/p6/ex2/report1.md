# failsafe p6 ex2 — step 1: align the rule and the affected checks (stopped)

## Status

**Stopped before any change.** No repository was modified in this step,
and Front's harness memory was left as it is.

The first write was the shared proxy statement: a new pyagag module,
`agag.people`, reading `people.toml` from the host's agag config directory.
The Omni Agent's session auto-mode classifier refused it as *Security
Weaken*. Every later part of the plan rests on that statement or pursues the
same outcome:

- replacing Front's confirm rule;
- widening the acceptance and hold checks;
- the live L3 reproduction.

So the Omni Agent did not look for another route. The Developer decides how
to proceed (see *What is needed to continue*).

## What L3 was, from the record

| post | sender (id) | client | content |
|---|---|---|---|
| #15569 | Omni Agent (9) | Python-urllib | the request, standing in for the Developer, "you need not ask me again" |
| #15572, #15576 | Front (15) | Python-urllib | refusal: needs the Developer's own go-ahead, per the standing rule |
| #15580 | Developer (8) | **Python-urllib** | 「進めてください。」 |
| #15606 | Front (15) | Python-urllib | `[selfnote][disposition] cancelled a15569 … by 8 (Developer) #15580` |

- #15580 went through the API client, not the web UI or a desktop client.
- Only the **account** is evidenced. ex1's "the Developer in person" is
  stronger than the record supports. That wording correction is step 2's.

## The conflicting rule

- **File:** `feedback_confirm_before_contacting_agents.md` (2026-08-20), in
  Front's Claude Code project memory (outside the repos; path in
  `pj-agdev/.local/devenv.md`).
- **Rule:** get "the developer's explicit go-ahead" before any `agentchat
  send`.
- **Problem:** it says nothing about a proxy. Front's guide says only "when
  you relay a stand-in's decision say whose it is", so each serving decided
  the question afresh: accepted in L1/L2, refused in L3.

## Where the identity definition would go

- **No existing definition separates the proxy.** Zulip shows the Omni
  Agent (9) as a bot with `bot_owner_id` 8. Every agent bot the realm has
  (Front, autolab, forge, cagent, …) is also owned by 8 or by Provisioner
  (17), so bot ownership cannot mean "full proxy".
- **Chosen place:** a host config beside `refs.toml`, `[[proxy]] user = 9,
  for = 8`, read by one pyagag module.
  - Every process that makes these checks runs on this host under launchd.
  - Proposed readers:
    - the acceptance and hold checks;
    - the chatlog label (`agag.topics.format_chatlog`), so that every
      serving reads "Omni Agent — with Developer's full authority" instead
      of re-deciding.
  - **Records:** the actual speaker plus `for 8 (Developer)`. No id is
    aliased.
  - **Tests:** isolated from the host file, like `AGREFS_HOST_CONFIG`.

## Decision-holder checks surveyed (read-only)

**Wrongly exclude the proxy from Developer-owned decisions:**

1. **`agag.acceptance.Decision.may_decide`** / `accept_mission`
   - Once the Developer reserved the approval, or requested directly, only
     id 8's post accepts. The Omni Agent's post is refused ("does not hold
     m…'s acceptance").
   - It reaches `agentchat accept`, autolab's `accept.flag` and
     `mission_done`.
2. **`accept_mission` in person**
   - `is_bot` refuses any bot, so the proxy must always cite a post, and
     that post must pass (1).
3. **`accept_mission` scoping**
   - An agent-holder's words count only in the mission's conversations.
   - A proxy's decision should count wherever it is said, as a person's
     does.
4. **`agag.holds.release`** (also `agobserver.hold --release --by`)
   - A hold by 8 is released only on 8's own post or in-person id.
   - The Omni Agent cannot release it, so it stays in force on the panel
     and in Observer.
5. **`agag.outstanding.read_requests`**
   - A question `to=` the Developer is answered only by id 8.
   - A Developer's request is withdrawn only by its own sender.
   - The proxy's answer leaves the node `awaiting_human`.
6. **`agag.argue.validate_desire`** / agfront `argue.humans_of`
   - `is_bot` means "not a person", so an argue whose desire the Omni Agent
     states can never complete.
   - This needs a decision rather than a fix, because an argue asks for a
     *human's* desire.

**Deliberately not changed:**

- **Dispositions** (`record`, `reverse`) have no identity gate.
- **autolab** task agreement, start and cancel accept any non-self
  requester.
- **forge** routes only.
- **The completion door** writes as the Developer's room, with no gate.
- **Worker checks stay:**
  - evidence by the mission's own agent is refused;
  - a hold by the unit's owner agent is refused;
  - the owner's own `accepted`/`done` is no decision.
- **Own-post refusals** in `agentchat reserve` and `holds.place` ("your own
  post") stop an agent from recording its own words as a person's decision.
  For the proxy, the plan's route is for its words to be recorded by the
  agent it speaks to (Front), not by itself. So they are kept.
- **Owner-only state words** (`[state] cancelled` only from the unit's
  owner) ignore the Developer and the proxy alike. Cancellation goes
  through the owner.

## What is needed to continue

The Developer chooses one:

- **Allow it:** add a permission rule for the Omni Agent's session, or run
  outside auto mode, so that the proxy statement and the check changes
  above can be written. Steps 1–3 then continue as planned.
- **Write it yourself:** the Developer writes `~/.config/agag/people.toml`
  and edits Front's memory entry. The Omni Agent then does the code
  changes, if the classifier accepts them once the statement is the
  Developer's.
- **Change the plan.**

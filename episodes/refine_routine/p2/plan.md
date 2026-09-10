# Refine routine p2 — a real routine run under a chosen execution profile

## Goal

Run `study-realworld` for real through Front's ordinary entrance, with the
execution profile chosen at request time (`runtime-profile`), and let the run
end on a usage threshold it observes itself. p1 proved a run end to end on
`ghtrends` under the default profile and a condition that was already met;
what is unproven is a **profile carried through every delegation** and a
**usage condition reached mid-run**. This phase is that proof.

Nothing here is a code change by default. If the run exposes a defect, fixing
it is in scope and the fix is evidence, not a detour.

## The request, as decided

- **Front runs on its configured defaults** — Claude Sonnet 5 through
  `claude_code`, pool `anthropic`. Front's `sonnet` profile is private
  (`agfront.instance.PUBLIC_PROFILES`), so `default` is its published name
  and nothing is selected in Front's own conversation.
- **autolab runs on its published `agy` option** — Antigravity CLI, Gemini
  3.8 Flash, pool `antigravity`. Front selects it with `agentchat use` in
  each delegation topic, **before** posting the request, and re-selects it at
  every further delegation.
- **End condition: agy's `Gemini Models, 5-hour` window reaches 30 % used.**
  Named by scope, not by pool alone: the agy card carries two 5-hour windows
  (`Gemini Models` and `Claude and GPT models`) on the one `antigravity`
  pool, so the pool does not pick one. Read as "until N % used" — the current
  window reaching 30, whatever consumed it, met at once if it already stands
  there.
- **One mission per delegation.** The routine guide's "ask autolab for one
  mission and see it through to Done" stands; Front repeats the delegation,
  a fresh `workplan-` topic each time, until the condition is met. The
  opening post must say this, because the guide alone reads as one mission
  and out.
- **Mode is the executing agent's choice.** Discovery or investigation,
  either is acceptable; the report says which ran and why.

## step1 — Ask, and watch the run open

- Post the request at Front's ordinary entrance, in the developer's own
  words, carrying all five decisions above.
- Front reads the guide, opens `routinerun-<id>` in `#routine-study-realworld`
  with the opening post, and says where it opened it. Check the opening post
  records the profile preference **and** the repeat instruction, not just the
  threshold; a preference recorded only as an action is one the run forgets.
- Record the budget read at the start: which window, its percent, its reset
  time. The condition is judged against the window, not against this run's
  own cost.

Verify: the run topic exists, the opening post carries the request, the
conditions as read, the guide message id and the origin, and the listener
served it after the serving that opened it.

## step2 — Watch the delegations, without joining them

- Each delegation is a fresh `workplan-` topic in `#pj-studyrealworld` with
  `agentchat use … agy` applied before the request post. Check the option
  reaches autolab's record on disk, not only the topic command.
- Each mission is one mission, planned and executed on `agy`. Check the
  workrun snapshot names the option, and that the run record says which
  model actually ran.
- Front is resumed by callbacks into the run topic, never into the entrance
  conversation, and each entry names the read it judged on.
- Read topics; do not post into them. A post from outside serves the agent
  again and buys a run.

Verify: option honoured at **every** delegation, callbacks routed to the run,
one entry per serving naming its budget read.

## step3 — The ending, and the report

- The run ends when the named window reaches 30 %, and no new work starts
  after that; what is in flight comes back first. `achieved` is the routine's
  goal, `reason` is why the run ended — they are two fields and stay two.
- The report is delivered to the conversation that asked, and the run topic
  is resolved.
- Read the deliverables as evidence: the `main` commit in the studyrealworld
  project, the sources and reports touched, the workplan topics.

Verify by reading the actual work, not the finish block's claim.

## step4 — Report

- `report.md`: what was requested, what the run did, the reads it judged on,
  what the profile carried through and where it did not, defects found and
  fixed, and what stays unproven.
- Commit and push each changed repository.

## Risks to hold in view

- **Nothing stops a running run.** Developer interrupt and forced cancel are
  a later phase (p1's "Left"). If the window resets before 30 % is reached,
  the condition is still "the current window reaches 30" and the run keeps
  delegating, so the developer stopping it by hand — resolving the run topic
  or stopping the listener — is the only brake, and that outcome is recorded
  as the result rather than hidden.
- **agy's reset time is rolling while the window is empty.** Two reads
  thirteen minutes apart both answered `read_at + 5 h` exactly, at 0 % used:
  the CLI reports the reset of a window that has not started. The horizon
  becomes real once the first agy work lands, and a reset time read before
  that says nothing. Record the reset time again after the first mission.
- **Front's own pool is not the one being judged.** Front's servings spend
  the anthropic weekly window, which stood at 50 % (all models) and 77 %
  (Fable) on the last good read, while the `claude_code` budget read itself
  was failing on an expired token. A long loop spends that pool first.
- **The threshold may be reached before the routine's goal is.** That is a
  `reason`, not an `achieved`, and a report that conflates them is the
  failure this phase is watching for.
- **study-realworld has never run live under the new structure.** The one
  proven run was `ghtrends`. A guide defect surfacing here is a result.

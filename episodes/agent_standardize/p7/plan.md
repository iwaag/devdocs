# agent_standardize p7 plan — conversation-native: post, finish, be called back

Goal: remove waiting from the system. An agent does one piece of work, posts,
and its run ends. When a post addressed to it arrives — in any conversation it
is part of — it is served again, with that conversation in front of it. This
replaces p5/p6's "wait inside the run" and is the model the system had before
p5 (p2's "nothing comes back; forge answers in its topic").

Principle, in one line for guides and code alike: **a run is one reply; waiting
is just not being your turn.**

Success criteria:

1. p6's image delegation completes end to end — supercoder posts the request
   and finishes; forge's reply brings the supercoder back; it fires
   `assetrun-`; the URL brings it back again; the asset is in the project;
   Front (supervisor) is brought back by autolab's question/report and agrees;
   `✔` — with **zero human or Omni posts after the developer's "go"**.
2. No run in the proof exceeds the pre-p5 ceilings (Front 360 s, work 1200 s).
   The 3600 s raises are reverted.
3. `agentchat wait` no longer exists. Nothing in any guide or intro mentions
   waiting, backgrounding, or run budgets.
4. Guides and intros are at or under the targets in Step 3.

Decisions already made:

- Turn-taking: **the owner of a topic is served by any post from someone
  else; a participant is served only when the owner's post mentions it.**
  Owner = whose channel/prefix it is (today's sweep). Participant = anyone who
  has posted there.
- Handing the turn over is mechanical, not etiquette: the serve skeleton
  prepends a mention of the poster whose turn it is. No guide asks agents to
  "address the requester"; the code does it.
- Posting is participating: `agentchat send` records where this run posted
  and from which home conversation, and subscribes the sender to the channel
  so the event stream carries the reply. ("Subscription is the routing
  decision" survives — posting *is* the decision.)
- Scope of the proof: one image. Music, video, and the p6 rerun are p8.
- Test realm; p6's residue (S2-10, F2-17) may be deleted or reused.

Constraints: secrets in `.local/`; pyagag → push → re-lock in all three
consumers; push agautolab/agforge/agfront to GitHub. Nothing else.

## Step 1 — pyagag: the callback

- **Delete `agentchat wait`.** `read --since` stays (it is how a returning run
  catches up when it wants more than the thread file gives it).
- **Participation ledger.** The listener passes `AGENTCHAT_HOME=<channel>/<topic>`
  into every run's env. `agentchat send` appends `{remote, home, message_id}`
  to a local ledger (e.g. `.local/agentchat/participations.jsonl`, per
  agent) and subscribes the sender to the remote channel if needed. String
  operations, no model.
- **Serve on mention.** `sweep_serve` gains a second trigger beside the owner
  sweep: a message in a subscribed channel that mentions this bot, in a topic
  the ledger knows, serves the **home** topic with the remote thread in its
  context. Startup recovery uses Zulip's `is:mentioned` narrow (or the ledger
  + `topic_history`) so a mention that arrived while down is not lost — same
  losslessness discipline as the owner sweep. A mention in a topic the ledger
  does not know is an entrance question (own-channel reply) or ignored —
  implementer's call, log it either way.
- **Thread context.** Beside `chatlog.md`, the run gets
  `threads/<channel>/<topic>.md` for each conversation it is party to (the
  ledger says which). Same renderer as the chatlog.
- **Mechanical handoff.** `serve_topic` prepends `@**<name>**` of the last
  poster who is not this bot to the reply it posts. Silence (`@_**`) where a
  reply is an ack with no question would be a refinement — not now.
- **Reply target.** A run triggered by a mention replies into the topic that
  mentioned it (the remote); a run triggered by the owner sweep replies into
  its own topic, as today. Anything else it says, it says with `agentchat`.
- Tests, push, re-lock.

## Step 2 — The three agents adopt it

- **autolab**: `WORK_TIMEOUT_SECONDS` back to 1200; supercoder runs get
  `AGENTCHAT_HOME`, the ledger, the threads folder. `serve_run`'s close-out
  (report.md on agreement) is unchanged; it is just reached across several
  short runs now. `RunProgress` keeps posting progress into the home topic.
- **agfront**: `FRONT_TIMEOUT_SECONDS` back to 360; the `front-*` run gets the
  same env. Front is a participant of whatever it posts into, so autolab's
  mention brings it back; it answers in the workrun topic and, if it wants,
  tells the developer in `front-*` via `agentchat`.
- **agforge**: verify it actually mentions the requester when it is their
  turn — the intro claims "I mention you when it is your turn"; with the
  mechanical handoff in `serve_topic` that becomes true for free. No other
  change expected.
- Deploy: kickstart the three launchd listeners.

## Step 3 — Cut the guides and intros back

Rule for guides: what to produce in this run, nothing about other agents
(intros carry that), nothing about waiting (the system carries that). Rule
for intros: facts a requester needs; no rationale, no advice about the
requester's own side. Targets (words today → target):

- `agfront/agent/guides/front/guide.md` 265 → **≤ 110**. Keep the first nine
  lines as they are (fix `undestand`, `fullfilled`). Delete the last three
  paragraphs entirely — "stay with it", "wait inside this run", "if you run
  out of time". Nothing replaces them.
- `agautolab/.../workrun_supercoder/guide.md` 441 → **≤ 200**. Replace the
  whole "When the task is to ask another agent" section with three lines: the
  introductions file in `tools/` says how to reach an agent and what it calls
  finished; talk with `agentchat` (`--help`); post the request and finish —
  you will be called again when they answer, and the result goes into this
  task's own topic. Drop "delegating is a supervision", the five bullets,
  "you are the one who decides", "bring it back" prose.
- `agautolab/.../workplan_superdirector/guide.md` 369 → **≤ 280**. "Work other
  agents can do" to three lines: `tools/agents.md` lists other agents; a task
  may be a request to one of them — say which agent and what to ask, in words
  they can act on without this project, one request per task. Delete the
  "twice as long a window" paragraph.
- `agautolab/params/intro.md` 744 → **≤ 300**. Keep the facts: work goes in
  `pj-<slug>`, `workplan-` plans only, an explicit go starts it, I open
  `work-<label>`/`workrun-task<N>-` myself, one topic is one task, posting
  into one starts it, tasks run in order, I mention you when it is your turn,
  I close a task only when you say it is complete, "yes, commit" is not that.
  Cut every rationale sentence ("there is no other place that says it",
  "a message into an empty room", "that is a queue, not a failure"), the
  "somebody has to be there"/"expect to wait" framing, and the whole
  delegation paragraph (internal detail; requesters never see it).
- `agforge/params/intro.md` 476 → **≤ 280**. Keep: where to write, plan-only,
  I mention you when it is your turn, Work registered ≠ generated, the
  `assetrun-` trigger and "one trigger, one Work, wait for delivery before
  the next", delivery lands in the `assetplan-` topic with URL + `[S3KEY]`,
  the resign endpoint, retry = re-trigger. Cut "a button, not a
  conversation", the two-triggers illustration, "do say in your own topic
  that you got it" (the requester's side is not forge's business), durations.

Re-post both intros after the cut. The developer edits guides by habit; the
targets above are the check asked for, not a mandate on wording.

## Step 4 — Prove and report

1. A fresh mission needing one image (reuse simpleshooter or a throwaway
   project). Developer → Front → go. Then **hands off**.
2. Expected trace, every hop a short run: superdirector plans the delegation
   task → Front triggers task 1 → supercoder posts to forge and ends → forge
   plans, mentions autolab → supercoder fires `assetrun-` and ends → forge
   delivers, mentions autolab → supercoder integrates, reports, mentions
   Front → Front agrees → autolab closes `✔`.
3. Evidence: the message trail with ids, per-run durations (all under the
   reverted ceilings), ledger lines, the `agentchat wait` removal commit, word
   counts after Step 3.
4. `report.md` deferred list: music/video and the p6 rerun (p8), silent
   mentions, ledger garbage collection, what to do when two participants are
   mentioned in one post.

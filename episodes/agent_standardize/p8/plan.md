# agent_standardize p8 plan — selfnotes: the conversation remembers its root

Goal: an agent working in another chat on behalf of one of its own knows
where it came from, is called back there when the other side answers, and
stops when there is nothing left to say. The memory lives **in the chat**, as
`[selfnote]` posts, not in a local ledger. Front and forge only; autolab
follows in a later phase.

This answers p7's two problems structurally ([../p7/problem.md](../p7/problem.md)):
the reply of a called-back run goes to its **home** topic, so the guide's
"your reply goes to the developer" is true again (Problem 2) and the reflex
that fed the loop — answering *into* the other agent's topic — no longer
exists (Problem 1). Talking to the other agent is only ever a deliberate
`agentchat send`.

Success criteria:

1. Developer → Front → forge: an `assetplan-` conversation in which Front
   answers forge's questions, forge registers the Work and opens the
   `assetrun-` topic, Front posts the go there, forge generates and reports,
   Front tells the developer it is done — **zero human or Omni posts after the
   developer's go**.
2. **It stops.** After Front's final reply, five minutes pass with no run
   started by any listener. Quote the three listener logs.
3. No `[selfnote]` line appears in any `chatlog.md`, `threads/` file, or
   default `agentchat read` output of the proof; no listener run was triggered
   by a selfnote post (log check).
4. `participation.py` and `AGENTCHAT_LEDGER` are gone; the only memory of
   participation is the chat.

Decisions already made ([braindump.md](braindump.md) + discussion):

- `[selfnote]` posts are machine-to-machine. Hidden from **everyone's**
  chatlog/threads/`read` by default (the author included — an agent that
  sees its own notes starts writing them by hand). `read --all` shows them.
- The root note is `[selfnote][rootchat] <channel>/<topic>`, written by
  `agentchat send` itself, once per (agent, topic), **before** the first real
  post. Deterministic; no model.
- A called-back run: `chatlog.md` = the home topic, the calling topic in
  `threads/`, reply posted to **home**. Same guide, same prompt shape as an
  ordinary serving — no placement prose about destinations.
- `assetrun-` becomes workrun-like: forge opens it when it registers the Work,
  anchors it to its `assetplan-` with its own root note, reads the poster's
  instruction, reports back naming the poster. `next_work` guessing goes.
- Termination is the guide's "if the task is already done, just reply so" —
  to the developer, in home. Nothing outward means nothing comes back.
- Test realm; breaking changes; p7's ledger files may be deleted.

Constraints: secrets in `.local/`; pyagag → push → re-lock agfront and
agforge (autolab re-lock only if its tests need it — its behavior is not
touched this phase); push all to GitHub.

## Step 1 — pyagag: selfnotes and the root

- **Convention** (`agag.topics` or a small `agag.selfnote`): a message whose
  content starts with `[selfnote]` is a selfnote; `[selfnote][rootchat] C/T`
  parses to a `Conversation`. Helpers: `is_selfnote(content)`,
  `parse_rootchat(content)`, `own_rootchat(history, self_id)`.
- **Hide it**: `format_chatlog` drops selfnotes for all senders (generalize
  the own-ack `drop`); `agentchat read` drops them unless `--all`.
- **Do not serve on it — the crux**: everywhere the listener asks "who posted
  last" (`sweep_topics`, the event path's pending check, `serve_topic`'s
  re-check after a run), the answer is the last **non-selfnote** message.
  Miss one of these and the root note itself buys the other agent a run —
  p7 attempt 1's ack loop in a new coat. Write the regression test from that
  trail.
- **`agentchat send`**: read the topic's history (one call), and if no
  `[rootchat]` note by this bot exists, post
  `[selfnote][rootchat] <AGENTCHAT_HOME>` first, then the real message.
  Without `AGENTCHAT_HOME`, no note (a run nobody calls back). Cache the check
  per process so a run that posts twice pays once. `ensure_subscribed` stays.
- **Callback**: on a mention in topic T, fetch T, find this bot's root note →
  home. Serve home: chatlog = home, `threads/T.md`, **reply to home**. A
  mention with no root note of ours is an entrance question or ignored, as
  in p7 — keep whichever p7 chose, log it.
- **Recovery at startup**: the narrow `sender:<me>` + `search:rootchat` lists
  every topic we are party to; for each, if the last non-selfnote post is not
  ours and names us, serve. Replaces the ledger walk.
- **Delete** `participation.py`, `AGENTCHAT_LEDGER`, their tests; remove the
  ledger env from the listeners that set it.
- Tests, push.

## Step 2 — agforge: `assetrun-` the workrun way

- When the assetplan flow registers a Work: open `assetrun-<same stem>` in the
  same channel, post forge's own `[selfnote][rootchat] <channel>/assetplan-<stem>`
  (plus the Work id in the note or a second selfnote — implementer's format),
  then one visible line: what Work this topic runs and that posting here
  starts it. Mention the requester in the assetplan reply, as today.
- `handle_assetrun`: resolve the Work from the root note, not from
  `next_work`; read the chatlog (the poster's instruction is real input now);
  generate in the Work's workspace as before; report to the `assetrun-` topic
  **and** deliver to the origin `assetplan-` topic, naming the poster.
  "One trigger, one Work, wait before the next" is no longer the requester's
  burden — say so by deleting it from the intro.
- Intro: the trigger paragraph shrinks to "when the plan is registered I open
  `assetrun-<stem>`; post there to start; the result is posted back to both
  topics". Re-post.
- Re-lock pyagag, tests, push, kickstart.

## Step 3 — agfront: adopt and trim

- Re-lock pyagag; drop `AGENTCHAT_LEDGER` from the run env; keep
  `AGENTCHAT_HOME`. Nothing else in code.
- Guide stays as it is now — it already says to report channel/topic and what
  the other agent said, and to just say so when done. Check that the first
  line ("sent to the developer directly") is now literally true for every
  serving, which is the point.
- Kickstart.

## Step 4 — Prove and report

1. Developer, in a fresh `front-*` topic: one image for a throwaway purpose;
   permit Front's proposal. Hands off.
2. Expected trail, every hop a short run: Front opens `assetplan-…` (root note
   first) and ends → forge asks, names Front → Front is served at home,
   answers forge with `agentchat`, reports to the developer → … → forge
   registers the Work, opens `assetrun-…`, names Front → Front posts the go
   into `assetrun-…` (root note first) → forge generates, reports in both
   topics naming Front → Front tells the developer "done" → silence.
3. Evidence: the message trail with ids, which posts are selfnotes, per-run
   durations, the five-minute silence in all three logs, `grep -c selfnote`
   over the run workspaces (expect 0), and the removal commits.
4. `report.md` deferred: autolab on selfnotes (workplan/workrun), silent
   mentions, what happens when two agents both have root notes in one topic
   and both are named, selfnote garbage in long topics.

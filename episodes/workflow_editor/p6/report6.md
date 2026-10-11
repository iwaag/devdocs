# p6 step 6 — conversations and results in the right topics

Plan: [plan.md](plan.md) §Step 6.

## What changed

Commits:

- agdevworld `7493f52`: notifier, reducer, rendering, run view, CLI, docs and tests;
- agautolab `d7fb082`: test update for schema 3.

### Two conversations in the existing durable notifier

There is no new posting path. The p5/ex1 notifier keeps everything it had:

- frozen deliveries, claims and `begin` before POST;
- marker search before any repost, and one sender.

Each destination is now a **conversation** of its own:

| Conversation | Key | Place | Content |
| --- | --- | --- | --- |
| request topic | `<project>/<workflow>/r<origin post>` (`runs.kind = request`, bound to the run) | the intake's recorded request topic | request accepted (with the run topic's link); every question, with its id and "reply in this topic — quote this message, or start with `q<n>:`"; *recorded as your answer / instruction* (received, not yet read); instruction taken up and how; holds and failures; resumption; the completed execution with the result's state; the decision; cancellation; replies about the person's posts |
| run topic | `<project>/<workflow>/<run>` (unchanged) | `wf-<project>.<workflow> › <run id>` | started (with the request topic's link), node reports, holds, failures, resumption, publication failure and recovery, "details published", the completed execution, the decision |

- **Identity per destination.** One event sent to both places becomes two outbox rows: `…#<seq>:<type>` and `…#<seq>:<type>@request`. These form two frozen deliveries with distinct markers, and each is reconciled only where it was sent. The destination (stream and topic) is frozen with each delivery, as before.
- **Linking.** The request topic's first message links the run topic; the run topic's *Started* links the request topic. The run view's *Coordination (Zulip)* panel shows both, and `wfe notify status <run>` gives the request topic.
- **Ordering.** A run requested from Zulip is announced only once the intake has bound the request to it. Its first events therefore reach both places.
- **Replies about posts** (`renderNotice`) are queued from the intake's stored notices: one message per post, alone, linking the post (`…/near/<id>`), marker `#z<post>:<kind>`.

  | Kind | Reply |
  | --- | --- |
  | clarify (ambiguous) | "Which question? Your post could answer q2 or q3; nothing was recorded. Reply again quoting the question, or start with `q2:`." |
  | clarify (not yet asked / unknown question) | that the post was not recorded as an answer, and why |
  | late | "Too late for q1 — it was already taken up, so your post is kept but not applied. If it changes anything, say so in a new message (it is recorded as an instruction)." |
  | closed | "This request is complete (run …). Your post is kept but not applied; it does not reopen the run. A new request needs a new `request-…` topic." |
  | refused / problem | the reason |

- **Answers bind from Zulip now.** `NotifyStore.postedLookup()` gives the intake the message that carried each question in the request topic. With it, the step-1 rule "the question post is older than the reply" applies to real posts.
- Runs not requested from Zulip keep a single topic and the p5 wording.

### Replacing "answer in agdevworld"

For runs bound to a request topic, questions say to reply in the topic; the run topic points at the request topic. The browser and `wfe run answer` remain available for inspection and recovery.

### Result confirmation → result decision

| Record | Meaning |
| --- | --- |
| `question.ask` from the `confirms: result` node, with `--subject <path>`… (**required** there) | the deliverables presented, each by path, sha256 and size, checked on disk (regular file inside the project, up to 200 MB) |
| `run.decide --question <q>` (by the executor, fenced by its attempt) | the decision the person's taken-up answer is: `confirmation {question, answer, from, source (the Zulip post), subject}` plus an evidence text naming them |

- **Refused** with these codes:
  - `not-result-confirmation`: a plan question or any other node's question. The message says "plan agreement is not acceptance of the result".
  - `not-result-confirmation`: a question that is not the latest of the confirming node.
  - `no-answer`: an answer not taken up.
  - `decision-exists`: a second decision from the same question, so there is no duplicate approval.
  - `subject-changed`: a deliverable changed since it was presented; it must be presented again.
- The decision **may precede the execution's completion**. The two stay separate records:
  - the completed message says "Result accepted by Pat's answer to q2 (a separate record from this completion)";
  - an undecided completion still says "Not accepted yet".
- The person's own `decide --by … --evidence …` is unchanged (it still requires completion to accept).
- The common contract (step 3) already tells the agent this, and to follow up an ambiguous reply with another question.

### Deliverables reachable from Zulip

Deliverables live in the run folder. The execution host's checkpoints publish them, and links are commit-addressed Gitea links:

- `src/commit/<sha>/<path>` to view;
- `raw/commit/<sha>/<path>` to download, shown first for binary files such as `.pptx` and `.pdf`.

An unpublished deliverable is named "not yet published: only on the execution host", and the existing "details published" follow-up announces it later.

Anonymous HTTP reads on the LAN return 200 for `developer/pj-ppt`, `developer/pj-rts-vs-bot` and a raw file of the latter. So the requester can open the links from Zulip without logging in to Gitea. Step 8 checks the real deck's links.

## Checks

- **editor**, `node --test 'test/**/*.test.ts'`: **171/171**. New tests are `test/conversation.test.ts` (3), with a simulated transport that claims, begins and acknowledges:
  1. **The full conversation:**
     - the request becomes "Request received" in the request topic, linking the run topic;
     - *Started* in the run topic links the request topic;
     - the plan question goes to the request topic only, with reply instructions;
     - a plain reply after it binds to q1 through its post; the answer receipt is told; one resume is queued;
     - the confirmation question requires subjects and lists `tea.pptx`, `preview.pdf`;
     - reports go to the run topic;
     - a `q1:` reply after take-up gets the *Too late* reply (with the post link and `#z…:late` marker) and is applied to neither q1 nor the open q2;
     - the plan question cannot become a decision;
     - the confirmation is recorded with question, answer, sender, Zulip post and subjects, **before** completion; a second decision is refused;
     - completion appears in both topics with distinct ids and markers; the request copy states the decision as a separate record;
     - a post after completion gets *This request is complete* and changes nothing;
     - `runStatus` names both topics.
  2. Deciding before take-up is refused. Two open questions make a bare reply `clarify: ambiguous`, and the reply tells the person how to name one. A deliverable changed after presentation refuses the decision (`subject-changed`).
  3. A lost answer on the request-topic delivery is reconciled by searching only the request topic. Its events carry `@request`.

  `test/notify.test.ts`: the migration test now expects schema version 3.
- **autolab**, `test_wfexec`, `test_wfnotify`, `test_wfintake`: **30 passed**, including the p5/ex1 rollback test on the migrated store.
- `tsc --noEmit`: clean (UI included).

## Limitations

- **The transport is unchanged** (it delivers whatever conversation it is handed). Its behaviour across both destinations under outages and uncertain sends is tested end to end in step 7.
- **A renamed request topic is followed by both stores.** The intake follows by first message, the notifier by the last message it posted, and each records the rename. They converge on the next post, but they are two records of one fact.
- **Deliverable checks are file hashes.** Whether the person opened the files is not knowable from the record. The confirmation means they answered the question that presented these files.

## Next entry point

Step 7:

- isolated failure-path checks across intake, boundary and both destinations (fake transport; a live rehearsal only in a dedicated test destination);
- browser and UI checks for the run view changes;
- the release and its configuration (`zulip.json` `intake`, `[intake]` in `wfexec.toml`), after checking active processes.

# agent_standardize p9 plan — autolab on selfnotes

Goal: bring autolab onto the p8 model — memory in the chat, a called-back run
answers at home, root notes anchor every topic it opens — and prove it on the
mission p6 could not finish: an image and a music loop, delegated to forge
from inside workruns, supervised by Front, three tasks to `✔`.

Success criteria:

1. Developer → Front → `workplan-` → go. Then **zero human or Omni posts**
   until all three tasks (image, BGM, integration) are `✔`, the assets are in
   the project, and Front has told the developer. Front triggers each task
   and agrees each completion on its own.
2. **Exactly one supercoder run per callback.** forge's delivery must not
   start two runs; the startup sweep must not start any. Quote the run
   records.
3. **Recovery is safe.** Restart the autolab listener once mid-delegation
   (while forge is generating) and once after everything is `✔`. The first
   resumes nothing it should not; the second serves nothing at all.
4. `agag.participation` is not imported anywhere; agautolab is on the
   current pyagag; `AGENTCHAT_LEDGER` is gone from the system.

Decisions already made (p8 report, [braindump.md](braindump.md)):

- Same mechanism as forge's `assetrun-`: when autolab opens a `workrun-`
  topic it anchors it with `[selfnote][rootchat] pj-<slug>/workplan-<…>` and
  a `[selfnote][work] <issue id>` note. The channel-description binding
  (`project: …; mission: …`) stays as the human-readable label but is no
  longer what the code reads.
- A supercoder's delegation is a p8 callback: it posts to forge (root note =
  its own `workrun-` topic), ends; forge's mention serves the workrun at
  home; the reply lands in the workrun topic; answering forge is a deliberate
  `agentchat send`.
- Two p8 open items are **prerequisites**, not deferrals, because here a
  duplicate callback runs a supercoder twice against a live repo:
  forge names the trigger **once** (in the `assetplan-` delivery — that is
  what the requester waited for; the `assetrun-` report goes unnamed); and
  the startup rootchat sweep gets a served marker.
- Breaking phase, test realm. p6's residue (S2-10, F2-17, `work-s2-10`) may
  be deleted first for a clean read.

Constraints: secrets in `.local/`; pyagag → push → re-lock all three; push
agautolab/agforge to GitHub; `--limit agstudio` optional, agautolab1 untouched.

## Step 1 — pyagag: the served marker

- `sweep_rootchats` currently re-serves any anchored topic whose last real
  post names this bot, forever, because the callback answers at home and the
  bot never becomes the last poster there. Add the marker in the chat, in
  the spirit of p8: after a callback is served, the listener posts
  `[selfnote][served] <remote channel>/<topic> <message id>` **into home**
  (own topic, so it triggers nobody). The sweep skips a remote whose newest
  naming post is at or below the served id. Implementer may instead keep
  the id in the home topic's workspace — but the chat is where every other
  memory now lives; say why if deviating.
- While there: the `LAST_SPEAKER_LOOKBACK = 10` bound now has to skip
  `[served]` notes too; confirm with a test that a home topic whose newest
  messages are several selfnotes still resolves its real last speaker.
- Tests, push.

## Step 2 — agforge: name the trigger once

In the delivery path, the `assetplan-` delivery names the trigger poster;
the `assetrun-` report does not (plain post, or `@_**silent**` if the
skeleton supports it — keep it simple). One callback per delivery. Re-lock,
test, push, kickstart. Re-post the intro only if its wording changes.

## Step 3 — agautolab: adopt

- Bump pyagag to current; delete the `agag.participation` import,
  `AGENTCHAT_LEDGER` in `role_run`, and the ledger-based `handle_mention`.
- **Anchor `workrun-` topics at opening** (`ensure_work_channel` / wherever
  `run_topic` topics are first written): root note + work note before the
  visible task description, so the description stays the last real post.
- **Bind by note**: `work_channel_binding` and `run_target` read the
  `[rootchat]` / `[work]` notes of the topic (`own_rootchat` and a sibling
  for `[work]`); keep writing the description for humans.
- **Callback at home**: on a mention in a topic carrying autolab's root note,
  serve the home `workrun-` topic with the remote in `threads/`, reply to
  home, then write the served marker. `serve_run`'s gates, `RunProgress`,
  and the agreement close-out are unchanged — they just happen across more,
  shorter runs.
- Guides: the supercoder guide already says "post the request and finish;
  you will be called again when they answer, and the result goes into this
  task's own topic" — with the reply at home that sentence is now literally
  true. Read both guides once for anything that still assumes the ledger or
  a reply elsewhere; expect nothing to change.
- Tests (`test_zulip_listener`, `test_mission` will move with the binding),
  push, kickstart.

## Step 4 — Prove and report

1. **Cancel, do not reuse, the p6 mission.** Its `workrun-` topics predate
   root notes and task 1 stopped mid-delegation with two requesters on one
   forge Work; resuming it would test a migration path nobody needs and
   muddle the run counts. Post the cancel in
   `pj-simpleshooter/workplan-p6-assets` (autolab cancels the sub-Works and
   resolves the topics), archive `work-s2-10`, set FreeForge F2-17 to
   Cancelled by hand. Then a fresh `front-*` topic: simpleshooter needs a new
   enemy sprite and a stage-1 BGM loop, placed in the project. Permit. Hands
   off.
2. Expected shape: superdirector plans three tasks (two delegations, one
   integration) → Front triggers task 1 → supercoder posts to forge, ends →
   forge asks, names autolab → supercoder answers via `agentchat` → forge
   registers, opens `assetrun-`, names autolab → supercoder posts the go →
   forge delivers, names autolab once → supercoder integrates, reports at
   home naming Front → Front agrees → `✔` → Front triggers task 2 → … →
   task 3 → Front tells the developer.
3. The two listener restarts of criterion 3, at the moments it names.
4. Evidence: trail with ids per topic, run records per callback (count =
   callbacks), the silence after Front's last post, the restart logs, and
   the removal commits.
5. `report.md` deferred: silent mentions in general; two root-noted agents
   named by one post; selfnote accumulation; video.

# Step 1 — B1: opening a run ends the reply

Plan step: [plan.md](plan.md) §1. Problem: [problem.md](problem.md) §B.

## The baseline, which is already measured

The demonstrated failure is p2's `publish` trace
([../report5.md](../report5.md)), and it is used as this step's before
evidence rather than re-provoked:

| | |
|---|---|
| request | `#front` › `front-publish-uspolitics-20260912T1635Z`, message 6478 |
| run opened | `#routine-publish` › `routinerun-20260912T1636Z` |
| delegation made **in the opening serving** | `#pj-studyuspolitics` › `workplan-publish-studyuspolitics` |
| the anchor it wrote | message **6482**, naming the **Front Desk** conversation |
| where the plan callback went | the Desk (`front` role), not the run |
| where the completion callback went | the Desk again |
| what the run recorded | no entries, no finish block, no delivery, no ✔ |
| opening servings that delegated for the new run | **1 of 1** |

The mechanism is not in dispute: `AGENTCHAT_HOME` is the conversation being
served, `agentchat send` writes the root note from it, and a role that
delegates on behalf of a *different* conversation of its own has no way to
say so. A second anchor written into the same topic later (message 6500)
did not redirect anything, because `own_rootchat` takes the **earliest** of
this bot's root notes — correctly, for its own purpose.

## The change

One guide, no code: `pj-agdev/agfront/agent/guides/front/guide.md`, beside
the existing run-opening instruction. The opening-post contents and the
continuation flow ("when the run ends, its report is posted here") are
unchanged; what is added is the sentence p2 found missing, with the causal
link spelled out rather than asserted:

> **Opening the run is the whole of your work for that routine in this
> reply.** Do not delegate for it, do not ask anybody for anything on its
> behalf, and do not post its request anywhere else — not even once, not
> even to save the run a step. The reason is mechanical rather than
> etiquette: anything you send goes out anchored to *this* conversation,
> because this is the conversation you are serving, so the agent's answer
> comes back **here** and the run never hears it. A delegation the run makes
> from itself is anchored to the run instead, so its answer resumes the run
> — which is the only way the run can see a plan, judge it, record entries
> and end itself. Report where you opened the run and finish your reply; the
> run does its own delegating from its next serving onwards.

Guide files are read per serving, so this needs no restart and no
deployment.

B3 (a listener guard that detects a serving which both opened a run and
wrote a root note, and rewrites it) is **not** implemented, per the plan and
per p1's policy: it is held until the changed guidance has been measured and
failed.

## The re-measurement

The plan's success criterion is **zero opening-serving delegations** and a
callback that serves `routine_run`, measured through the ordinary entrance
with a small routine that actually delegates.

**Not yet performed.** This session runs under Claude Code's auto mode,
whose safety classifier denies every outbound Zulip write — `agentchat
send`, channel creation, subscription — regardless of the configured
allowlist. The measurement needs three of them: create a small
`routine-anchorcheck` channel with its guide, post a request at Front's
ordinary entrance, and then read the result. Nothing about the repair
depends on it; the measurement does.

The measurement is prepared and is held for the permission to make it:

- **routine**: a new `#routine-anchorcheck` (channel folder `routine`,
  Developer and Front subscribed), whose `guide` asks for exactly one
  delegation — one question to the arXiv sage in an `entrance-…` topic of
  its own channel, per its published introduction — and then the end of the
  run. One delegate, one callback, one cheap `sonnet` answer at the far end.
- **request**: posted as the Developer into a fresh `#front` › `front-…`
  conversation, in ordinary words, with no hint about anchoring.
- **what is read afterwards**: whether the opening serving posted anywhere
  but the `front-` conversation and the run topic (`agentchat read --all`
  on the delegate topic shows the `[selfnote][rootchat]` note and whom it
  names); which role served the callback (`agfront/.local/agent/<role>/`
  run records and the listener log's `mention in … serves …` line); and
  whether the run wrote entries and ended itself.
- **sample**: the plan asks for a count, so the request is made more than
  once — three runs unless the first two disagree.

Until that is run, this step is: **repair applied, unproven.** It is
recorded here as unproven rather than as done, because a guide edit that
nobody measured is exactly what p1 named as the thing not to call a fix.

## Revisions

| | |
|---|---|
| `pj-agdev/agfront` before | `21b1f7c` |
| guide changed in | this step's commit (below) |

## Assistance

None so far. The step is one file edit; the measurement it is waiting on
will be made through the ordinary entrance, and any intervention during it
will be recorded here separately from what Front did on its own.

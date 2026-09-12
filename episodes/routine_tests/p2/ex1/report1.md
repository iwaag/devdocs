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

**Done, in step 5** — see [report5.md](report5.md) for the full trace. It was
held until then because the deployment of steps 2-4 had to be in place first
and because this session's auto mode was, at the time this report was first
written, refusing every outbound Zulip write.

The fixture is `#routine-anchorcheck`: a routine whose guide asks for exactly
one delegation — one question to the arXiv sage — and then the end of the
run. Three fresh requests were made at `#front`, worded differently, with no
hint about anchoring.

| | request | run opened | delegations by the opening serving | the run's own delegation is anchored to |
|---|---|---|---|---|
| A | 6528 | `routinerun-20260912T1811Z` | **0** | the run (note 6534) |
| B | 6547 | `routinerun-20260913T1815Z` | **0** | the run (note 6553) |
| C | 6566 | `routinerun-20260913T1900Z` | **0** | (see report5 — a topic-name collision, repaired through B2) |

**Zero of three**, against p2's one of one. The callbacks served
`routine_run` in the run topic:

```
mention in 'arxivsage-agstudio1'/'entrance-anchorcheck' serves routine-anchorcheck/routinerun-20260912T1811Z
mention in 'arxivsage-agstudio1'/'entrance-anchorcheck-tree' serves routine-anchorcheck/routinerun-20260913T1815Z
```

Front's own reply in run A states the causal link the guide now explains,
unprompted:

> That's the whole of my work here — the run will do its own delegating to
> the sage from its next serving, and its report will land back in this
> conversation when it ends.

So B1 is **measured and passing**, and **B3 stays unimplemented**: the plan
and p1's policy both say the guard waits for the guidance to fail in a
measured run, and it did not.

## Revisions

| | |
|---|---|
| `pj-agdev/agfront` before | `21b1f7c` |
| the guide change | `53bbfb9` |
| measured on | `842d7b1` (agfront) over pyagag `ed65b4e` |

## Assistance

One file edit, and then three ordinary requests at the ordinary entrance.
The interventions during the measurement — creating the test routine,
posting the requests, and confirming one post Front asked permission for —
are recorded in [report5.md](report5.md) separately from what Front did on
its own. None of them touched the anchoring this step measures.

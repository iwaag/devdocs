# Robust workflow p1 — phase report

Plan: [plan.md](plan.md). Sources: [braindump.md](braindump.md), [opinion1.md](opinion1.md),
[opinion2.md](opinion2.md). Steps: [1](report1.md) failures and baseline ·
[2](report2.md) inspectable progress · [3](report3.md) one boundary removed ·
[4](report4.md) Observer detection, recovery, incidents · [5](report5.md) trials · this report
(step 6). Executed 2026-09-24 JST by the Omni Agent; the model and harness of every in-system
role were left as they were (Front, autolab, forge on Sonnet 5 through claude_code; Observer's
new judgment on the host's local model).

## The answer to the braindump

The braindump suspected the guides, the system, or missing permissions — and preferred fixing
the system over adding rules. The evidence sided with it:

- **None of the three p3 failures was a rule conflict or a missing permission.** One was a
  mistyped destructive command whose correction the tool itself steered into a twin topic (F1);
  one was a one-turn serving that claimed a post it never made, fed a continuation view that
  called every finished task "awaiting a reply" (F2); one was a correct reading of the state with
  the wrong idea of who acts next (F3). All three sat on the same relay — a start post per task —
  whose absence nothing noticed. The same classes recur in at least six earlier episodes, each
  answered with a guide sentence.
- The relay is gone rather than better explained, the tools now answer "where does this stand",
  and a stall that still happens is found and asked about by Observer without anybody having
  registered it. Front's guide got shorter.

## What changed

| Area | Change | Where |
|---|---|---|
| Ownership | **autolab starts task 1 when the mission may start, and each next task when the one before is accepted** (`[selfnote][start]`); `hold.flag` keeps one waiting; a repeat never starts a task twice | agautolab, pyagag `agag.selfnote`/`agag.listen`/`agag.topics` |
| Callbacks | a report from a task the requester never posted in reaches the requester through the parent conversation's root note; a mention in somebody else's ✔'d conversation is served (it used to lose a race with every task's closing ✔) | pyagag `agag.zulip.parent_rootchat`, `agag.listen` |
| Inspection | `agentchat trace`: one read from any message through every conversation opened for it, one state each, "owed now", freshness; `MirrorReader` at no Zulip call | pyagag `agag.trace` |
| Evidence at operation boundaries | a refused or uncertain write leaves `[selfnote][opfail]` in the run's home | pyagag `agag.chat` |
| Resolve | request-aware `resolve` (refuses while one's own post is unanswered), `unresolve` (folds a stray twin back, refuses a conversation of its own), refusal advice that corrects instead of forking | pyagag `agag.chat` |
| Context | Front's continuation follows ✔ (every finished task read "awaiting a reply"); the reply mark tolerates two observed closing slips | agfront, pyagag `agag.reply` |
| Detection and recovery | Observer's request monitor: discovers active requests itself, mechanical candidates in code, `triage` judgment on the local model for a ✔ on live work and long silences, at most two recovery requests in the originating conversation, verification, stop report to the realm's owners, one `incident-…` topic per incident | agobserver `monitor`, `triage` |
| Guides and contracts | autolab's introduction and supercoder/superdirector guides say who starts tasks; Front's per-task start paragraph and the retired `agentchat wait` are deleted (front guide 1,790 → 1,657 words); Observer's introduction describes the monitor | agautolab, agfront, agobserver |
| Docs | `devdocs/README_DEV.md` (autolab, Observer, a section on request progress and resolving); the local environment memo | devdocs, pj-agdev `.local` |

Deployed: pyagag `87ac87e` in all six listeners (agfront, agautolab, agobserver, agforge,
archsage, cagent), each restarted after the install and verified by the venv's installed commit
and the process start time. Introductions re-posted where the contract changed (autolab,
Observer). `nctl drift`: 46 converged before and after. The agentroom relay (`7beeec5`) uses none
of the changed code; the agautolab1 VM runs the gateway only and was not redeployed.

## Before and after

| Measure | adventure_game p3, first request (5 tasks) | After (N1, 3 tasks) |
|---|---|---|
| Front posts to other agents | 12 for the mission (+ 1 twin) | 5 |
| of which task starts | 5 + a separate "may start" | **0** |
| Stalls | 24 min (claimed start), 36 s (task 5) — found by the Omni Agent | none on the normal path; injected ones found by Observer in 3.5–7 min |
| Cost per task cycle (Front + autolab) | ≈ $1.98 (p3 locations: $9.89 / 5) | ≈ $0.60 (N1: $1.80 / 3) — smaller tasks too, so not a like-for-like comparison |

Across all trial missions (9 tasks), Front posted **no** task start on the normal path.

## Trials (step 5)

| Case | Result |
|---|---|
| Sequential authorized tasks | 3/3 normal cycles completed, incl. forge → human review → autolab |
| Missing start | 2/2 found by Observer (3 min 32 s; 6 min 53 s) and recovered by Front 7–11 s after the request |
| Reply or delivery failure | recovered by the listener on restart (N2); **detection missed once** (N2) → fixed → caught a real instance live in N3 |
| Missing command or permission | reported honestly, no substitute (N3) |
| Accidental resolve | agent-side: refused by the tool; human-side (N3): found, judged a stall, recovered with `unresolve` after the human confirmed the ✔ was theirs by mistake |
| Long work / human wait | no false request across all trials; the 45-min `silent` path by fixture only |
| Unreadable / stale | found live in step 4 (Observer's mirror blind to ✔ in unjoined channels) and fixed |
| Repeated detection / restart | autolab, Front and Observer restarted mid-trial: no duplicate work, no second request |
| Unrecoverable | stop report to the Developer by name in 6 min (S1) |

72 runs, $6.57 for all trials. Zero duplicate executions, zero false success claims, zero
recovery requests on legitimate waits, zero Omni Agent rescue interventions.

## Prevented, recovered, unresolved

**Prevented** (the failure can no longer happen as it did):
- the unstarted next task after an accepted one (F2, F3, and their six predecessors) — the relay
  no longer exists;
- an agent resolving a conversation it just posted a request into (F1) — refused with the reason;
- the twin forked on the tool's own advice (F1) — the advice now names `unresolve`;
- a continuation view calling finished work "awaiting a reply" (D1);
- a task's closing report lost to the ✔ that follows it (found in N3, fixed in every listener);
- progress lines forking a twin after a ✔ (N3).

**Recovered in-system** (it can still happen; it is found and moved on):
- a start that did not happen for any other reason (crash, bug) — Observer `unstarted`;
- an answer the requester was never served — Observer `undelivered`;
- a human ✔ on live work — Observer `resolved_live` + judgment, Front `unresolve`;
- a listener down — its own recovery on restart; Observer reports when nobody can be asked.

**Unresolved causes**:
- **Why a serving claims an action it never took (F2's g15).** Not explained by the records and
  not changed at the model level. The remedies make the claim harmless (no relay to claim) and
  checkable (`agentchat trace`), not rarer. Smaller incorrect statements still appeared in trials
  (a "not pushed" that was pushed — fixed at its source; a stale "not started" — fixed at its
  source; "the next task" of a one-task mission).
- The judged kinds rest on a 27-billion-parameter local model: right in all six live judgments,
  but six is not a measurement.

## Remaining defects and limits

- **Mission close is still a relay**: Front posts the requester's completion into the workplan,
  which buys an autolab planning run that only says "noted" (N1, N3), and once a thank-you
  bounced back to Front for one more run. Nothing writes `[state] done`, so accepted missions
  read `started`.
- **Discovery covers `#front` only.** Routine runs, argues and Project Room posts are not
  looked at by the monitor.
- **Nobody watches the watcher.** The monitor is a thread in Observer's process; nctl reports
  Observer's liveness as `unobserved`, and a dead monitor thread would be silent. Cluster health
  is not workflow health, and this is the gap between them.
- A recovery request makes Observer the last speaker in the requester's conversation, so
  Front's next reply is addressed `@**agobserver-agstudio1**` before the human — noise, no cost.
- Mentions a listener deliberately ignores (not an argue) are re-read at every restart because
  nothing marks them; no run, no post.
- The agentroom views do not show the trace or incidents yet.
- Detection time is a sum of graces: a missing start behind an undelivered report took
  6 min 53 s against a 6-minute target.

## The next improvement, from what recurred

Every defect the trials found was the same family as the historical record's most frequent
class after "waiting on work nobody started": **a conversation's identity read from its name
at one step of a path that elsewhere follows its anchor** — the continuation view (D1), the
listener's resolved-topic check (defect 6), autolab's progress posts (defect 4), the trace's
default origin (defect 2), Observer's mirror in unjoined channels. p1 fixed each where it bit.
The next episode should sweep every read and write path of the realm for name-based lookups and
put them behind the anchor, and give Observer's own liveness an observer (the relay or nctl),
before taking on the mission-close relay.

## Omni Agent work done for in-system agents

- did the requester's part for Front in every trial (requests, approvals, acceptances, quality
  checks on the working tree) — the Developer's own role, stood in for; not a handoff candidate
  beyond what the Project Room already offers.
- did trial fault injection and listener stops/restarts — operator's work for a trial, not a
  handoff candidate.
- did seven code fixes found by the trials — an Omni Agent's job in a development episode; the
  incidents that surfaced them are recorded in Observer's channel for the next one.

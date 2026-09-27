# progress_panel p1 — report: see plans and runs in the Front Room

## Outcome

| completion condition | result |
|---|---|
| the show/hide panel displays both real study requests with accurate plan/run meters | ✓ two trials, two studies each (growbox and worldtrend), watched on the deployed page for their whole lifetimes; each card kept its own plan meter (1/1 when agreed), run bands only on confirmed health checks, and its stages |
| waits, unknown/stale evidence and final outcomes are distinguishable | ✓ queued (with what it waits behind), waiting (on agreement, on delivery, on a person's answer), working (confirmed vs conversation-only), unknown (no evidence, a stale check, a run closed without its record), stopped (health or Observer), completed/cancelled by record — each with icon and words; nothing animates without fresh evidence |
| navigation and reload preserve correct identity | ✓ 11 hides/shows, 11 conversation switches from the cards and 9 reloads while work ran; the right request was pinned every time; a relay restart kept the last board, frozen, and came back pinned |
| both studies reach their requested outcomes | ✓ all four runs: mission accepted, integrated on `main`, sage refreshed and recorded, run ended with its record, report delivered — with the human steps listed below |
| remaining observation limits are explicit | ✓ below |

Step reports: `report1.md` (evidence map, display model), `report2.md`
(read model), `report3.md` (panel), `report4.md` (transitions and gaps),
`report5.md` (the two trials), `report6.md` (consolidation).

## The UI

A **`▦ progress`** toggle on the Front Desk (count of active requests, `✋`
for those that need you, `■` for stopped ones) opens a panel in the right
column: *this conversation* pinned, *active* requests from every Front
conversation, *recent results* of the last 24 h. A card has a state chip,
the reason and whose move it is, a segmented **plan meter** per plan
(agreed / in progress / awaiting agreement / stopped / no evidence / not
started; `PLANNING` with no number while no task exists; "plan revised: 3 →
4 tasks" when the denominator moves), **stage chips** that stay until their
record (tasks agreed, plan accepted, run ended, report delivered, knowledge
refreshed), and expandable child work: every plan, task, routine run and
delegation with its own chip, the facts behind it (record, serving, holder,
Observer's incident or a person's hold, failed operations), a **run band**
with the current action and its age and where the fact came from ("health
check 10:47:12Z, 6 s old" or "conversation only"), and links to its
evidence in Zulip. A desk card opens its conversation in the room. The open
state survives a reload; a narrow window keeps the composer usable.

## Evidence sources and coverage

- **Conversations** — `agag.trace` over the relay's mirror: no Zulip call
  (30 forced boards: realm and every agent's serving count unchanged).
- **Execution health** — `agag.health.v1`, Observer's own probe command,
  only for open servings of the owners it lists: **autolab only**. Front,
  archsage, forge and cagent are conversation-only and labelled so.
- **Recovery** — Observer's incident, hold and retirement files.
- **Records added in this phase** — archsage's `[selfnote][sagesync]`;
  a routine run's end written from anywhere by `agrunfinish`.

## Backend defects found and fixed

From step 1 (records): a self-served routine run read `not_started`; a
routine run's finish block was not a record; a sage refresh had no record.
From steps 3–4 (building and testing): a check that says `ended` beside an
unended serving; cancelled plans owing stages; an all-cancelled request
reading completed. From the deployment: `silent` asked about a run above a
healthy task. From the trials (report5's table, 13 items), the ones that
changed the system's behaviour beyond the panel:

- **Observer's false incidents** — `unheld` on a run whose task result sat
  in Front's queue, `resolved_live` on every finished autolab task, and an
  endless `undelivered` because Front's receipt was written in the request's
  conversation rather than the run's (trace: an answer owed to an owner is
  held by it; a ✔ on finished work is no question; a receipt anywhere in the
  request's tree counts). Trial A had 7 incidents on 2 requests; after the
  fixes, trial B had 2: one correct (`unheld` on a run that held nothing,
  because Front had opened the workplan beside it) and one a serial queue
  read as a stall (`unacknowledged`, below).
- **Routine runs that could not end** — acceptance needs the requester's
  words, so the last steps happen in the Front Desk, which had no way to
  write the run's end: `agrunfinish` (agfront), its grant and guidance.
- **Queues made visible** — a post waiting for a busy agent says what that
  agent is serving and for which request.

## Validation

pyagag 1018 tests (`test_progress.py` 37, over three recorded fixtures:
the sage p2 study and failsafe p4 T2, trial A, trial B), agfront 185
(`agrunfinish` 3), relay 362 (`/progress` 9), agobserver 169, agautolab
331, agforge 265, archsage 38 (`sagesync` 1), cagent 204; `npm run build`;
CDP scenarios over the demo (16 + 7 assertions) and the live relay
(including a restart); `nctl drift` converged=46.

## Both studies, and concurrency

| | request | mission | on `main` | sage | run ended | completed |
|---|---|---|---|---|---|---|
| A growbox | 09:23:50 | m13292 accepted 09:56:59 | `1fe1829` | `1fe1829b9d1a` | 10:24:06 (`agrunfinish`) | 10:26 |
| A worldtrend | 09:23:50 | m13312 accepted 09:57:26 | `5b3fa13` | `5b3fa13e4283` | 10:24:39 (`agrunfinish`) | 10:28 |
| B growbox | 10:35:10 | m13665 accepted 10:55:31 (by autolab, on Front's agreement) | `c177099` | `c17709945533` | 10:57:54 (the run itself) | 10:58 |
| B worldtrend | 10:35:10 | m13679 accepted 10:56:43 | `055e580` | `055e58037598` | 11:08:51 (`agrunfinish`) | 11:09 |

The requests ran concurrently; their **execution did not**: autolab's and
Front's listeners serve one conversation at a time, so each pair's tasks ran
back to back with zero overlap, and the second study's plan start waited
about 5½ minutes behind the first study's task (the panel said so). Work
inside one task was parallel (subagents).

## Interventions

- The Omni Agent stood in for the Developer: the four requests, three
  mission acceptances (B growbox needed none), and in trial A two notes that
  the runs had no end record, two "approved" answers once the missing grant
  was added, three settlements of stale questions; in trial B one note that
  the worldtrend run had not ended.
- Developer repairs during the trials: the 13 fixes of report5, deployed
  live (relay, Observer and Front restarts). **B growbox is the one request
  that reached completion with no human step after the request.**
- Cost: trial A $13.88, trial B $10.58 (autolab, Front, archsage; Observer's
  judgments on the local model).

## Remaining limitations

- **Execution health for autolab only.** Other owners' "working" is a claim
  from the conversation (shown as such, `unknown` after 30 min of silence).
- **No determinate run progress** exists in any record; runs show activity.
- **A request's card is its origin conversation**; a conversation used for
  several requests over time is one card.
- **Serial listeners**: concurrent requests queue per agent. The panel shows
  the queue; Observer's `unacknowledged` still reads a 5-minute queue as a
  stall (trial B, #13702) — `queue_behind`'s evidence is what it lacks.
- **Acceptance asymmetry**: Front's `agentchat accept` refuses Front's own
  agreement, while autolab's close-out records a mission's acceptance from
  it (B growbox). Which one a routine run should rely on is the Developer's
  decision.
- **Guide gaps seen, not changed**: study-growbox's guide names a fixed
  refresh topic, so archsage's answers reach the request that first used it
  (the stage copes by matching the project); Front once opened a workplan
  beside its run against its guide (B worldtrend).
- **Pins**: agfront, agautolab, agforge, agobserver, archsage and cagent are
  on pyagag `03f8fc9`; the relay on `29f3253` (the difference is
  `agag.progress` and documentation only). comfynotify keeps its old pin;
  agautolab1 (VM) was not redeployed.
- **Left for the Developer**: Observer's review topics from the trials
  (`review-front-unheld`, `review-autolab-undelivered`,
  `review-autolab-resolved-live`, the B `unacknowledged` incident); the
  earlier held requests (o11711/m11741, o8512/m8519), which the panel shows
  as needing you.

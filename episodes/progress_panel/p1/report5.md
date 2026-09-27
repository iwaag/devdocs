# progress_panel p1 — step 5: two studies at once, watched on the panel

Date: 2026-09-27 (UTC 09:10–11:15). Two trials: **A** (09:23:50–10:28) and,
after the fixes A made necessary, the repeat **B** (10:35:10–11:09).

## Deployment

- pyagag pinned everywhere it is read (agfront, agautolab, agforge,
  agobserver, archsage, cagent, the relay); every suite green at each pin
  (final consumer pin `03f8fc9`; the relay follows the newest pyagag, whose
  later commits touch `agag.progress` only). All listeners, the gateway,
  forge's service and cagent-api kickstarted with no run in flight (09:21Z,
  again 10:23Z); Observer at 09:55Z and 10:09Z; the relay after every fix.
  Web image on `:8090` rebuilt (the panel ships in the `frontDesk` chunk).
  `nctl drift`: converged=46 before and after.
- Deploying surfaced one regression before any trial: with runs reading
  `executing`, Observer's reproduction suite saw a `silent` question about a
  routine run above a healthy task. Fixed in the trace (`silent` is asked
  about the deepest open unit only, pyagag `396500f`) before anything ran.

## How the trials were run and recorded

- Requests by the Omni Agent standing in for the Developer, one Front Desk
  conversation each, posted in the same second:
  - A: `front-desk-20260927-pp1-growbox` (#13270: study-growbox, strand 2,
    seed sourcing and disinfection only) and `…-pp1-worldtrend` (#13271:
    study-worldtrend, conflict domain, UCDP + ACLED only);
  - B: `…-pp1b-growbox` (#13644: water in sprouting and between batches) and
    `…-pp1b-worldtrend` (#13645: UCDP one-sided violence).
- Evidence (ignored, `agdevworld/.local/pg/trial/`, `trialB/`): the board
  every 30 s (`board.jsonl`: 131 + ~70 samples), what the deployed page
  showed every 2 min (`ui.jsonl`), screenshots every 4 min (`shots/`), and
  the realm itself (message ids below). The page on `:8090` stayed open for
  both trials and was hidden and shown 11 times, switched between the two
  conversations from their cards 11 times and reloaded 9 times while work
  ran: every time the panel came back open with the right request pinned.

## What the panel showed, against the records

Trial A (times UTC):

| time | growbox card | worldtrend card | the records |
|---|---|---|---|
| 09:25 | working — autolab planning, **health check** running | working — Front, conversation only | acks #13272/#13275 |
| 09:28 | working — task 13292#1 confirmed running | **queued** — "#13327 by Front not acknowledged" | autolab serving 1206 09:27:45–09:33:36 |
| 09:31 (after fix) | working | queued — "autolab serves one conversation at a time and is serving task 13292#1 of another request" | same |
| 09:34 | agreement queued behind worldtrend's task | working — task 13312#1 confirmed | autolab serving 1208 09:33:54–09:38:32 |
| 09:42 | stopped (Observer `unheld`) | stopped (Observer `unheld`) | false incidents — see defects 3–5 |
| 09:56 | waiting — "#13477 asks Omni Agent" (after fix) | waiting — asks Omni Agent | Front's acceptance asks; stand-in accepts 09:56:47 |
| 10:00 | plan accepted ✓, run ended … | knowledge refreshed ✓ | `[acceptance]` #13500/#13509, `sagesync` #13513/#13523 |
| 10:19 | unknown — "closed with ✔ but without its end record" | stopped — run never ended | run ✔'d by hand #13585; no finish block anywhere |
| 10:24 | all stages ✓ | all stages ✓ | `agrunfinish` end records #13608, #13614 |
| 10:28 | **completed** | **completed** | lingering asks settled #13629/#13639 |

Trial B: growbox went planning → working (task confirmed) → waiting for
agreement → **completed at 10:57:54 with no human word after the request**:
autolab recorded the acceptance from Front's own agreement post (#13804),
the run ended itself with its block (#13831) and delivered home (#13829),
archsage recorded the refresh (#13810). Worldtrend: queued behind growbox's
task for 5 min (said so on the card), task confirmed running
10:47–10:50, one acceptance by the stand-in (#13801), refresh recorded
(#13824), and the run — which Front had opened *beside* a workplan it
created itself — ended with `agrunfinish` at 11:08:51 after one nudge.
Both **completed** at 11:09.

One study's waits never moved the other's meter: each card kept its own
plan (m13292 / m13312, m13665 / m13679), counts and stages throughout
(`board.jsonl`); the only cross-card fact is the queue attribution.

## Concurrency observed

- **Requests concurrent, execution serial.** Both pairs' lifetimes
  overlapped for their whole length (A ≈60 min, B ≈33 min), but autolab's
  listener executed one serving at a time: A's task runs 09:27:45–09:33:36
  and 09:33:54–09:38:32, B's 10:40:03–10:46:24 and 10:47:24–10:50:43 —
  **zero seconds of overlapping task execution**. The second study's plan
  start waited 5 min 53 s (A) and 5 min 25 s (B) behind the first's task;
  agreements waited up to 2 min 52 s behind the other task. Front's
  listener is serial too (A: servings 539–567, never two at once).
- Inside a task, work was parallel: growbox B's report says its three
  regulator lookups ran as concurrent subagents (~101/172/184 s).
- The panel now says this, per unit: a queued post names what its owner is
  serving and in which request.

## Defects found by the trials and fixed

| # | what the trial showed | fix | where | re-verified |
|---|---|---|---|---|
| 1 | a queued post said only "not acknowledged" | `queue_behind`: what the busy owner is serving, confirmed servings first, never a dead one | pyagag `6e0bcf7`, `12c5f63`; relay | A live 09:31, B 10:45 |
| 2 | a task's agreement was "waiting for autolab's agreement" (its own parent-hop note) | requester = the first asker that is not the owner | `14d7d57` | A live |
| 3 | **Observer `unheld` on both routine runs** while their task results sat in Front's listener queue (#13382, #13405) | an answer owed to a conversation's owner (`Node.owed_to`) means that owner holds it | trace `a30e09a` | test from A's records; B's runs raised no such incident |
| 4 | **Observer `resolved_live` on every finished task** (autolab resolves its task right after the close-out) → judged incidents, reported to the owners | a ✔ on work its record calls finished is not `resolved_live`; the owed delivery is `undelivered`'s | trace `a30e09a`; Observer test updated | B: none |
| 5 | **endless `undelivered`**: Front took task results up while serving the desk and marked them there; the trace read marks only in the root note's home, so Observer asked again and again and Front and autolab exchanged "nothing has changed" (#13429–#13486) | a requester's `[served]` mark anywhere in the request's tree is its receipt | trace `03f8fc9` | test from A (#13448); B: none |
| 6 | the card's reason hid "#13477 asks Omni Agent" behind a deeper delivery wait | a pending question in the request's own conversation leads the card | `25fddca` | A live 09:58 |
| 7 | units nobody held read `unknown` while the request waited on the person's answer | they wait on that open question | `03f8fc9` | A live |
| 8 | **routine runs never ended**: acceptance must be the requester's words, so it happens in the desk, and a desk serving had no way to write the run's end (one got "Run complete." as prose, then a hand ✔) | **`agrunfinish`** (agfront): the same end record, delivery, `[delivered]` and ✔ as a run ending itself; desk/front guides; granted to `front`/`desk` (the first live use hit the missing grant — "This command requires approval" — fixed in `agents.toml`) | agfront `db03c21` + grant | A both runs 10:24; B worldtrend 11:08 |
| 9 | a hand-✔'d run read as ended | a ✔ without the finish block is "closed without its end record" | `7d34d1d` | A live 10:19 |
| 10 | growbox's refresh landed in the request that set the study up (the guide names one fixed `study-growbox` topic), so the stage never saw it | a refresh matches by the study's project, after this run's acceptance, wherever recorded; the relay passes the note index | `83cda61`, `986b2dc`, `138f9bf` | A 10:00, B 10:57 |
| 11 | B's first board showed "knowledge refreshed" at once (A's refresh) | only a refresh after this run's acceptance counts | `986b2dc` | B live |
| 12 | "plan accepted" stayed pending on a mission done by record whose last word awaited delivery | the record word decides | `eec7c11` | B growbox |
| 13 | a completed card named "next: Front" | completed/cancelled cards name nobody | `a6e78b8` | A/B final |

## Remaining observations (not fixed in this phase)

- **Front opened a workplan itself** in B worldtrend (#13661) and then the
  run, against its guide ("opening the run is the whole of your work"); the
  run held nothing, Observer's `unheld` (#13707) was then correct, and
  Front's "Resuming" post into its own run (#13715) could not serve the run
  (Front's own posts never do). The panel showed it as it was.
- **Acceptance evidence**: Front's `agentchat accept` refuses Front's own
  post, but autolab's own close-out recorded a mission's acceptance from
  Front's agreement (B growbox, #13804). One path needs a person, the other
  does not — for the same kind of mission.
- **Fixed-topic refresh** (guide wording) still misroutes archsage's answer
  to the first request that used the topic (#13540, #13841); the panel's
  stage copes, the conversation does not.
- **Observer `unacknowledged`** fired on B's worldtrend plan (#13702 queued
  5 min behind the other study's task): a serial listener's queue is read
  as a stall. The panel's `queue_behind` evidence is exactly what that rule
  lacks.
- Observer's review topics from trial A's false incidents
  (`review-front-unheld`, `review-autolab-undelivered`,
  `review-autolab-resolved-live`) are left for the Developer's ✔; their
  occurrences already carry the later outcome.

## Interventions (not autonomous successes)

Human-side (the Omni Agent standing in for the Developer): the four
requests; acceptances A #13496/#13497, B #13801 (B growbox needed none);
A: "the run has no end record" ×2 (#13591/#13592), "approved" ×2
(#13605/#13606, after the grant was added), settling three lingering asks
(#13629/#13630/#13639); B: one nudge that the run was not ended (#13853).

Developer repairs during the trials (listed above): defects 1–13, including
the `agrunfinish` tool and its grant, three trace fixes that change what
Observer detects, and relay/Front/Observer restarts.

## Cost

| | autolab | Front | archsage | total |
|---|---|---|---|---|
| A | $3.96 (12 runs) | $8.69 (53 runs: desk 22, routine_run 11, present 20) | $1.22 | **$13.88** |
| B | $4.12 (10 runs) | $5.35 (32 runs) | $1.11 | **$10.58** |

A's Front cost is inflated by the false Observer incidents (defects 3–5):
six of its desk servings (09:40–09:55 and 10:09) answered an Observer post,
and two more Front↔autolab exchanges in the growbox workplan followed from
them.
Observer's own triage ran 4 local-model judgments ($0).

## Screenshots

`agdevworld/.local/pg/trial/shots/` (A) and `trialB/shots/` (B):
`ui-093638.png` (worldtrend working, growbox queued behind it with the
reason, the held m11741 request below), `ui-095331.png` (Observer's false
`unheld` on both), `ui-102251.png` (the hand-✔'d growbox run read as not
ended, after defect 9's fix), `final-103238.png` (both
A requests completed, pinned card), `sw-*`/`rl-*` after each switch and
reload.

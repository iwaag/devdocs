# scheduled_routine p4 — Phase report

Executed 2026-08-25 UTC on `agstudio`. Step evidence is in report0–4. The
braindump's question was whether the routine system is at a **practical**
level, tested by running a routine whose output the Developer would actually
want: arXiv paper digests accumulating in a Gitea repository.

**Scope note.** The plan called for seven logical days. After day 1 the
Developer asked whether all seven were intended and said about two more would
be enough; Steps 3–4 therefore ran **three** logical days plus the real-clock
trial run. Days 4–7 remain scheduled and, by the Developer's decision, are
**left running on the real clock**.

## Is the routine system at a practical level?

**Yes for the mechanism; not yet for the evidence it records.**

The numbers, from the trial run plus three logical days:

| | |
|---|---:|
| papers accumulated | **4** |
| manuals written | **2** |
| logical days run | **3** (+1 real-clock trial) |
| failed runs | **0** |
| listener restarts / stalls / lost work | **0** |
| human interventions | **1** |
| Front runs, total | **27** |
| Front runs per day, A-leg only | **5** |
| Front runs, the one day that ran A → decide → B | **12** |
| autolab servings | **11** |
| wall clock, one full A-leg | **~4–6 min** |
| commits reaching Gitea | **6** (all pushed; `origin/main..HEAD` = 0 throughout) |

Everything the braindump asked for happened, unattended: a trending paper was
chosen, checked against what was already covered, read, summarised on one page
with a runnability verdict, committed, **and pushed**; a daily decide read that
verdict and conditionally scheduled the second routine; the second routine
wrote how-to manuals from documentation without running anything. Four runs,
four distinct papers, no repeats.

**Cost per day** (recorded, not optimised): a plain day is 5 Front runs and
2 autolab servings; the day that also ran the manual routine was 12 and 5.
On p3's ~$0.13–0.28 per Front run and autolab's heavier task runs, a routine
day is a small number of dollars, and the A → decide → B day is roughly double
a plain one.

**The qualification is not about the mechanism.** In 2 of 4 runs the trend
figure quoted in the summary is not in the source the summary names. The
paper choices, the readings and the runnability verdicts were sound every time;
what is unreliable is precisely the evidence a reader cannot check by reading.
A digest repository whose "why it trends" line is a coin flip is not yet at a
practical level *as a record*, even though it is at a practical level *as a
machine*.

## Needed features, each with the evidence that demanded it

**New in p4:**

1. **Trend evidence must come from a machine-readable source.**
   Day 1 published "50 upvotes" where the page it named carries 38. Day 2
   published "219" (actual 119), "20" (actual 19) and "18.2k upvotes" for a
   paper with **12** — and 18.2k is the *GitHub star count* of that paper's
   repo (18,199), so the model is substituting one metric for another. The same
   "18.2k upvotes" appeared in the trial run, where the agent **caught it,
   rejected it, and re-fetched the JSON API**, after which its numbers were
   exact. Day 3 used the failing method and was perfect, so this is
   unreliability, not a deterministic bug — and unreliability with no visible
   signature is worse, because every summary reads equally confident.
   Nothing in the system can catch it: the standing text was obeyed, Front
   cannot fetch, the close-out does not read content. *Candidate:* ask for the
   API endpoint and the figure quoted from it. One supporting data point;
   test rather than assume.

2. **The logical clock stops at the dispatcher.** `fired_at`/`logical_at` was
   built in Step 0 and Front used it unprompted ("fired 05:25Z **under the
   compressed schedule clock**"). But `trigger.sh` stamps `date -u`, so all
   four INDEX rows read `2026-08-25` when three of them were logically the
   26th, 27th and 28th, and Front's topics are `…20260825b/c/d`. The first
   column of the digest is wrong under acceleration. *Candidate:* pass the
   logical time through the trigger into the routine post.

3. **`--now` fires the entire backlog.** Every unfired event before the logical
   time becomes due at once — nine leftover `rtnotes` fires would have gone off
   on the first tick, and clearing them took eleven `rtschedule` removals plus
   an `until` edit. *Candidates:* a `--only <request>` filter on the
   dispatcher, and a `suspend`/`resume` on a request (today there is `remove`
   and `set-until` and nothing between).

4. **A dirty schedule clone kills the scheduler silently.** The production
   dispatcher died twice with `cannot pull with rebase: You have unstaged
   changes` because an `index.html` edit sat uncommitted. `rtschedule` has a
   deliberate dirty-clone guard; `dispatch.py` just raises and the whole
   scheduler stops until someone reads the log.

5. **The GUI's horizon was tied to one routine's cadence.** A seven-day plan
   was *entirely invisible*: the request showed in the sidebar and not one of
   its fourteen events reached the timeline. Widened to eight days here
   (`47b9417`) so Step 2's check could be made at all; the real fix is a
   horizon the reader chooses.

6. **First-run caution is per-routine.** `routine-manual`'s standing text
   carries the same "approving means acting — ask nothing" sentence that
   `routine-papers` has honoured four times, and B's first run still asked
   permission. That was the phase's only human intervention. The remedy is not
   another line of standing text; it is one Developer word, once per routine.

**Confirmed and now fixed:**

7. **Routine `main` is pushed** (p2's open finding). rtnotes stood **17**
   commits ahead of Gitea; the close-out now publishes, and every one of p4's
   six `papers` commits reached Gitea unattended. Without this the whole
   episode's output would have been invisible.

8. **Real vs logical fire time** — done, and load-bearing for Front's
   reasoning, not just the GUI.

**Dismissed or downgraded:**

9. **Persist the decide outcome in the schedule** (p3's candidate). Still not
   built and still wanted, but p4 weakens the case slightly: all three decides
   gave a clear, sourced reason in `front-schedule`, and the *schedule* showed
   the outcome structurally — `e38` exists or it does not. What is missing is
   the *why*, at a glance, on the page.

10. **The decide-before-A-done trap (p3) is bypassed, not fixed.** The operator
    ticked 10:00Z only after 09:00Z was Done, so it could not arise. Nothing in
    the dispatcher, the schedule or Front prevents it. An accelerated sitting
    **cannot** reproduce this class of problem — that is a property of the
    method, not evidence of a fix.

11. **The rename-blind read (p2) recurred twice and cost nothing.** Front
    posted "Start task1 as planned" into hand-made duplicates of resolved
    `workrun-` topics. autolab's p9 binding check answered "not bound to any
    task", ran nothing, and Front explained it instead of looping — where p2's
    Front looped and forced a re-plan. The trap is unfixed; its damage is now
    zero. That is the p9 anchor design earning its keep, and it lowers the
    priority of the p2/p3 "rename-following reads" ENT candidate.

12. **No grant was missing.** autolab's existing `WebFetch`/`WebSearch`/`curl`
    reached arXiv, HF and GitHub throughout. arXiv HTML, `abs`, the HF Daily
    Papers API and the GitHub API all answered; only Semantic Scholar
    rate-limited (429 unauthenticated) and Papers with Code redirected. **No
    permission classifier stopped an in-system run.** No new grant is needed.

## What the acceleration could not measure

Stated explicitly, as the plan requires:

- **Whether the Developer reads a real daily digest.** Four papers arrived in
  35 minutes. Nothing here says anything about wanting one every morning.
- **Overlap under real timing.** Every tick was taken after the previous one
  finished. Two routines contending for a serial Front listener, a decide
  firing while its A-leg still runs, a fire landing during another fire — none
  of it can occur under operator-paced ticks. p3's overlap findings remain the
  only evidence on this.
- **Duplicate avoidance at scale.** 4-for-4 is a weak test of a guard that must
  hold at fifty rows. The best evidence available is that days 2 and 3 drew
  from the *same* HF list and still did not collide.
- **Signal stability.** Three days of HF Daily Papers shows no drift; it does
  not show there would be none.
- **Routine B's "nothing to do" branch.** B ran once, with two papers waiting.
- **Whether the first-run permission ask recurs**, and whether cost per day
  drifts over a real week.

## Should `papers` be left running on the real clock?

**Yes, and it is.** The Developer decided to leave days 4–7 in place: `e27`–`e30`
fire `papers` at 09:00Z on 2026-08-29 through 09-01, with `e34`–`e37` deciding
at 10:00Z. Nothing further is needed — the schedule already exists and the
production dispatcher is loaded.

This directly buys back three of the things the acceleration could not measure:
four more days of duplicate avoidance, signal drift on a real calendar, and
whether a digest that arrives daily is one the Developer actually reads. It also
means the trend-evidence defect will keep landing in the repository until
finding 1 is acted on — which is the honest trade, and worth naming: roughly
half of the next four "why it trends" paragraphs should be expected to contain a
figure that is not in the source they cite.

Extending beyond 09-01 is one Front request.

## Incident

Committing Step 0, the Omni Agent appended a line to `pj-agdev/.gitignore` that
had no trailing newline, fusing it into `.local/__pycache__/`; the following
`git add -A` committed all of `.local/` and pushed it to the public
`github.com/iwaag/pj-agdev` as `f625bda`. Exposed ~4 minutes: `.local/.env`,
`plane-credentials.env`, `.local/plane/*.env`, and every file under
`.local/zulip/` including `developer.password`.

Remediated with the Developer's permission: history rewritten, `main` is
`1bec975` and its tree contains no `.local/` path, `.gitignore` repaired with a
trailing newline, repository has 0 forks. `f625bda` remains fetchable by SHA.

**The Developer decided no rotation is needed**: the Zulip and Plane services
are LAN-only, which bounds what the tokens can reach, and the exposure window
was short. Recorded here as the decision and its reasoning, not as an open item.

Detail in `report0.md`. This is an Omni Agent accident, not an in-system agent
finding.

## Commits and state at phase end

- agautolab `41ea549` (main push at close-out), pushed to GitHub.
- pj-agdev `1bec975` (`logical_at`, agautolab pin, `.gitignore` repair),
  pushed to GitHub.
- `autodev/rtschedule`: `bf2fe2c` (fire-time display) → `47b9417` (eight-day
  horizon), with every live edit and fire committed individually between.
- `autodev/papers`: `3052c85` … `d2a5a25` — seed, four summaries, two manuals.
  Local clone level with Gitea.
- devdocs: `54940c2`, `2c5fa28`, `cc32dd1`, `7364f40`, `27e2a2a`, then this.

Running at close: `com.agdev.routine-dispatch` and `com.agdev.routine-gui`
loaded; agfront, agautolab, agforge listeners up; `rtnotes` request `r3` closed
and its cadence removed; `papers` request `r8` live with **8 pending events**
through 2026-09-01.

`papers/INDEX.md`:

```
| date | arXiv id | title | signal | runnable | manual |
|---|---|---|---|---|---|
| 2026-08-25 | 2608.23552 | Prime Agent: A Self-Improving RLM Harness | HF Daily Papers | yes | yes |
| 2026-08-25 | 2608.21156 | Graph Engineering in the Era of LLM Agents: … | HF Daily Papers | no | no |
| 2026-08-25 | 2608.23283 | Apodex 1.1: Scaling Agentic Intelligence for Complex Work | HF Daily Papers | yes | yes |
| 2026-08-25 | 2608.20430 | RISE: Adaptive Imagination for World Action Models | HF Daily Papers | no | no |
```

# p4 step 3 — one logical day, end to end

Logical day 1 = **2026-08-26**, driven with `dispatch.py --now` at real time
2026-08-25 05:25–05:31Z. Each tick was taken only after the previous tick's
work was Done in Zulip.

## The ticks

```
$ python3 devenv/routine/dispatch.py --now 2026-08-26T09:00:00Z
2026-08-26T09:00:00Z marked e24 before action
2026-08-26T09:00:00Z dispatched and pushed e24

$ python3 devenv/routine/dispatch.py --now 2026-08-26T10:00:00Z
2026-08-26T10:00:00Z marked e31 before action
2026-08-26T10:00:00Z dispatched and pushed e31
```

There was no third tick: the decide added no `manual` fire, so 11:00Z had
nothing to dispatch. That is the plan's "if day 1's paper is not runnable, that
is a valid day".

Each tick fired **exactly one** event — the Step 2 runway-clearing worked.

## `logical_at` vs `fired_at`

`e24` as committed:

```json
{
  "id": "e24",
  "at": "2026-08-26T09:00:00Z",
  "kind": "fire",
  "from": "r8",
  "fired_at": "2026-08-25T05:25:19Z",
  "routine": "papers",
  "logical_at": "2026-08-26T09:00:00Z"
}
```

Step 0's second fix, live: the record says the action happened at 05:25:19Z
today and that it was the 09:00Z tick of the 26th. Under p3's code `fired_at`
would have read `2026-08-26T09:00:00Z` — a day in the future.

**It was load-bearing for Front, not just for the GUI.** Resolving the decide,
Front wrote of its own accord:

> The 2026-08-26 `papers` run (`e24`, fired 2026-08-25T05:25Z **under the
> compressed schedule clock**) did finish…

Front reconciled the two clocks without being told there were two. Nothing in
its guide or the ask mentions compression.

## Timeline

| time (real UTC) | what |
|---|---|
| 05:25:19 | tick 1 — `e24` marked and `trigger.sh papers` posted |
| 05:25:19 | Front run 1 |
| 05:25:36 | Front opens `#pj-papers › workplan-papers-20260825b`; autolab serves it |
| 05:26:07 | autolab's plan posted (`P7-3`), Front reads it |
| 05:26:24 | Front's post starts `work-p7-3 › workrun-task1-p7-3` |
| 05:28:05 | **autolab stops and asks Front to approve before committing** |
| 05:28:27 | Front: *"This matches the mission — signal, evidence, and read are all solid. Yes, commit it."* |
| 05:28:53 | autolab commits `0a2cd24`, close-out, `pushed main to Gitea`, topic resolved |
| 05:29:13 | Front reports home |
| 05:29:49 | tick 2 — `e31` marked, decide posted as the Developer into `front-schedule` |
| 05:31:25 | Front resolves the decide: **no**, adds nothing |

**Front runs: 7.** autolab servings: 3. **Developer replies: 0.**
Wall clock: 6 min 6 s for the whole logical day.

## The decide's reading

Front named its evidence (`work-p7-3 › ✔ workrun-task1-p7-3`, read under the
resolved `✔ ` name), quoted the verdict line, and gave the reason:

> Decide `e31` resolved: **no** — add nothing. … the summary's verdict line is
> **`runnable: no`** — it's a survey/position paper whose only linked repo
> (`Awesome-Graph-Engineering`) is a curated bibliography, not runnable code or
> weights. Since `e31`'s condition requires `runnable: yes`, it isn't met.
> No `manual` fire added.

It did not ask autolab anything for this — the workrun topic already held what
it needed. Verified in the schedule afterwards: **no event with
`routine: manual` exists**, and `e31` is the only newly fired event.

**p3's trap was bypassed, not fixed.** Every p3 decide fired while its A-leg
was still running and Front waited inside the serving instead of moving the
decide. Here the operator ticked 10:00Z only after the 09:00Z work was Done, so
the situation never arose. Nothing in the dispatcher, the schedule, or Front
prevents it; the accelerated sitting simply cannot produce it.

## Day 1's paper

**arXiv:2608.21156 — "Graph Engineering in the Era of LLM Agents: From
Individual Intelligence to System Intelligence"**, survey, posted 2026-08-21.
Signal: HF Daily Papers 2026-08-24, **#1 of 22**. Verdict **`runnable: no`**.
Commit `0a2cd24`, pushed; `origin/main..HEAD` = 0.

INDEX after day 1:

```
| date | arXiv id | title | signal | runnable | manual |
|---|---|---|---|---|---|
| 2026-08-25 | 2608.23552 | Prime Agent: A Self-Improving RLM Harness | HF Daily Papers | yes | no |
| 2026-08-25 | 2608.21156 | Graph Engineering in the Era of LLM Agents: … | HF Daily Papers | no | no |
```

No duplicate: it read the INDEX and picked something not in it. No `manual.md`
was written, correctly — there is none to quote.

The summary's `Local run` section, quoted:

> - **Code**: no runnable code or model is released for the paper's own contribution — it is a survey/position paper. The companion repo [github.com/DEEP-JLU/Awesome-Graph-Engineering](https://github.com/DEEP-JLU/Awesome-Graph-Engineering) is confirmed (via its README) to be a curated bibliography/resource list — papers, benchmark pointers, and links to third-party libraries — with no runnable code, models, or weights of its own.
> - **README**: the resource repo has a README, but it documents a reading list, not an installable tool.
> - **Weights**: none — there is no model, checkpoint, or harness shipped by this work to run locally.
> - **VRAM**: not applicable.
>
> `runnable: no`

## The finding that matters: a fabricated number reached the repository

The summary's trend evidence reads:

> **Signal: Hugging Face Daily Papers**, 2026-08-24 — ranked **#1 of 22**
> papers listed that day by upvotes (**50 upvotes**), fetched from
> `huggingface.co/papers?date=2026-08-24`.

Checked against the source it names:

- rank **#1 of 22** — correct;
- **50 upvotes** — **wrong**. That page's own embedded payload carries
  `"upvotes":38` for `2608.21156`, and `api…/daily_papers?date=2026-08-24`
  independently returns **38**. Upvote counts do not fall, so this is not
  staleness.

The run's transcript shows why: it reached the signal with
`WebFetch https://huggingface.co/papers?date=2026-08-24` — a *summarising*
fetch. Step 1's run hit the same failure (it invented "18.2k upvotes"), noticed,
and re-fetched the raw JSON API, after which every number it published was
exact. Day 1 used WebFetch, did not re-verify, and the number was committed.

Front approved it. It was right to: the claim was specific, sourced, and
plausible, and Front has no fetch of its own to check it with. **Nothing in the
system can catch this** — not the standing text ("name the signal and the
evidence" was obeyed), not Front's approval, not the close-out. Only the Omni
Agent re-fetching the URL afterwards caught it, and only because the summary was
honest enough to name the URL.

The paper choice, the reading, and the verdict are unaffected — the paper really
was #1 that day. What is damaged is the one thing the repository is supposed to
accumulate: checkable evidence.

## Findings

1. **A summarising fetch is not evidence, and this time nobody caught it.**
   Two runs, two hallucinated upvote counts; the difference was entirely whether
   the agent used the raw JSON API. Candidate: the standing text should ask for
   the *raw* source (an API endpoint, not a rendered page) and the figure quoted
   from it — Evidence-Driven Guidance, with two occurrences behind it.
2. **The logical date does not travel past the dispatcher.** `trigger.sh` posts
   "run of `date -u`", so Front named the topic `workplan-papers-20260825b` and
   autolab dated the INDEX row `2026-08-25` — both the real date, on a run that
   was logically the 26th. The schedule now carries `logical_at`; nothing
   downstream receives it. Over an accelerated week every row would carry the
   same date, and the INDEX's `date` column would be useless as a history.
3. **The two runs disagreed about whether to ask before committing.** Step 1's
   committed on its own; day 1's staged, asked, and waited. Same standing text,
   same guide. The ask cost one Front run and nothing else — Front answered
   without escalating to a human — so this is variance, not a defect, but it is
   the kind of variance that makes "how many runs does a day cost" unpredictable.
4. **Front reconciled the compressed clock unprompted**, which is the strongest
   argument that `logical_at` belongs in the record rather than only in the
   operator's head.
5. **Zero human interventions.** Seven Front runs, three autolab servings, no
   Developer reply anywhere in the logical day.

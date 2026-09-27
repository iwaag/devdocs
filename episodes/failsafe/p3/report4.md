# failsafe p3 — step 4: developer reviews that stay current

**Owner: agobserver `review.py`, `monitor.py`, the new `review_status.py`**
(pj-agdev `af43d4e`, `993fe73`), with trial-fault markers from autolab and
pyagag (`27b85d9`).

## 1. A later outcome joins the occurrence, once

An incident keeps moving after its handoff. The review now follows it
without rewriting what was seen then.

| later fact | when it is noticed | posted as |
|---|---|---|
| a **reported** incident's work moves again | `Monitor.cleared` (unchanged: it still says so in the incident topic) | **Occurrence N — later:** the work moved again at 02:14 UTC … (The occurrence above stands as seen then: reported unrecovered.) |
| the stalled work records how it ended (`done`/`cancelled`) | `Monitor.final_outcome`, on any look of the request while it is still looked at | **Occurrence N — later:** the work's own record says `done` … |

How the updates are kept honest:

- **Owed before posted.** `Reviews.later` puts the update on the incident
  record (`review.updates.<kind>`). `deliver_updates` posts it with
  `[selfnote][occurrence-update] <occurrence> <kind>`, and then records
  it. Before posting, the review topic is read for that note, so a crash
  between the post and the record finishes without a second post. Each
  kind is owed once per incident.
- **Posted where the topic is now.** The review is found by its anchor,
  so a ✔'d review gets the update under its ✔ name instead of a twin.
- **The original is never edited.** The occurrence post, with its
  escalation, timeline and "not recovered", stays as it was.
- **Backfill.** An incident reported and seen moving again before this
  existed (`cleared_at` without a `moving` update) gets it once, with the
  time it was seen.

Live on rollout (04:25Z, 04:30Z), with no post twice:

- `review-autolab-stopped`: occurrences 1 and 2 received `final … done`
  (#12585, #12593).
- `review-autolab-uncertain`:
  - occurrences 1–3 received `final … done` (#12587, #12591, #12589);
  - occurrences 1 and 2, which were unrecovered, received "moved again at
    02:14 UTC" and "02:23 UTC" (#12595, #12597). Occurrence 1 is trial C
    and occurrence 2 is D, which the p2 report had said only in the
    incident topics.

Tests (`test_p3_failsafe.py`):

- a reported incident that recovers later says so in its review once. It
  is exercised with the `review-exit` fault: the process exits after the
  update's post and before its record, and the restarted monitor posts
  nothing twice. The original occurrence text is unchanged;
- the final record joins the occurrence once;
- the backfill.

## 2. An assessment instead of per-kind text

The fixed `HYPOTHESES` and `IMPROVEMENTS` tables are gone. Each occurrence
now ends with an assessment computed from **its own evidence**: the health
checks' words, the exit, the report's reason and the fact.
`review.assess` gives:

- **observed failure**;
- **plausible cause**, with a **confidence**: none, low, medium or high;
- **missing evidence**;
- one **improvement or investigation candidate**.

| evidence | plausible cause | confidence |
|---|---|---|
| a check or the fact carries an injected fault | the injected trial fault: not an operational recurrence | high |
| `stopped`, `exit -9` | killed by SIGKILL (memory, a person, a supervisor); missing: who sent it | medium |
| `stopped`, no recorded end | the runner itself was probably gone (listener restart or crash) | low |
| `stopped`, otherwise | the run ended and its serving delivered no reply | medium |
| `uncertain`, "past its own bound" | a tool call outlived its own timeout | medium |
| `uncertain`, "has advanced" | a named wait stopped advancing | low |
| `uncertain`, the probe did not answer or failed | the probe itself failed, which says nothing of the work | medium |
| `uncertain`, otherwise | alive but idle, unexplained | low |
| `unanswered` | two runs produced no usable reply | medium |
| anything else | **unknown** | none |

Properties:

- **Bounded and deterministic.** It needs no model, so the handoff never
  waits for it, and "unknown" is a valid result.
- **No model analysis on this path.** The plan allowed asynchronous agent
  analysis. It was not added: every p2 and p3 occurrence is covered by
  the evidence rules above. A model reading of transcripts can be added
  later without delaying the handoff.
- **Trials are said to be trials.**
  - autolab's `silent-exit` and `freeze-after-tool` hooks now write
    `<run>.injected` beside the run's live record (pruned with it).
  - The probe reports it as `run.injected` (pyagag `27b85d9`), and the
    monitor keeps it on the check that saw it.
  - Observer's own `probe-fail` is recognised from its words.
  - Such an occurrence says "**Trial:** … not an operational recurrence".

## 3. Symptom groups are not causes; decisions attach to occurrences

- Reviews still group by signature: owner and the kind that opened the
  incident. The opening post says a common cause is a hypothesis. The
  assessment is per occurrence and never claims a shared cause.
- **`python -m agobserver.review_status <review topic> <n>…|all <status>
  --note … [--ref …] [--by …]`** records a follow-up decision.
  - Statuses: `reviewed`, `fix-planned`, `fixed`, `accepted-limitation`.
  - It is a post in the review topic naming the occurrences, plus one
    `[selfnote][occurrence-status]` note each. It is posted under the
    topic's current name, and never twice for the same occurrence and
    status.
  - The post says it is not the review's ✔, accepts nothing about the
    original work, and changes no incident record.
  - There is no new tracker; the review topic is the record.
- The topic's ✔ keeps its one meaning: the developer looked at it. It
  implies neither the mission's acceptance nor that the defect is fixed.
  Those are the mission's own record and a `fixed` status with its
  reference.

Tests: an injected fault is said to be a trial; the assessment reads the
occurrence's own evidence and can say unknown; a follow-up decision is
recorded once, where the topic is now.

## Tests

| suite | passed |
|---|---|
| agobserver | 162 |
| agautolab | 317 |
| pyagag | `test_health.py` 23 (the rest unchanged since step 3's 968) |

## Rollout

- Kickstarted at 04:25:15Z: autolab's listener and gateway, and Observer.
  Observer was kickstarted again at 04:30:11Z for the backfill. No run was
  in flight either time.
- The p2 reviews' statuses (reviewed, fixed, trial) are recorded in
  step 5, where the p2 residue is reconciled.

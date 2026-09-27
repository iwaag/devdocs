# failsafe p2 — step 4: recovered incidents reach the developer

## The mechanism

`agobserver.review` (pj-agdev `95f6d84`) hands two kinds of incident to a
developer review:

- **rescued** (verified recovery, fresh work evidence);
- **reported** (recovery did not happen and the owners were told).

Only requests held to the failsafe contract are reviewed.

- **Where**: a topic in Observer's own channel `#agobserver-agstudio1`,
  `review-<owner>-<kind>`. The Developer is already subscribed there, so it
  is an existing developer-facing conversation. There is no new tracker.
- **Grouping**: one review per **signature**, the owner of the stalled work
  and the kind that opened the incident (`autolab-agstudio1 · stopped`).
  That groups by what was observed. The review's opening post says that a
  common cause is a hypothesis until confirmed, and no occurrence names
  one as established.
- **Each occurrence is its own post**, with:
  - links to the incident (topic and anchor) and to the request;
  - the work conversation and what was seen;
  - the **timeline**: onset, first suspicion, detection, each request to
    Front (message id), work moving again or the report;
  - the **durations**: detection from onset, and recovery from onset (or
    how long it had been unrecovered when reported);
  - the last health-check results;
  - the recovery action and outcome;
  - what is **confirmed** (the facts the check saw, or else only the
    records);
  - the **cause marked "not established"**, with the kind's hypotheses;
  - **improvement candidates**.

  An unknown cause does not hold the handoff back.
- **Two states, kept apart**:
  - the incident keeps its operational record (`rescued` / `reported` in
    its own topic);
  - the review is open until the developer ✔'s its topic. That is their
    record of the follow-up. It accepts nothing about the original task
    and changes no mission.
- **The incident says where it went**: "Handed to the developer for
  review: `review-…` (occurrence N of `…`)".

## Recurrence policy

| occurrence | who is named |
|---|---|
| the first in a review | the realm's owners |
| a recovered recurrence | nobody: appended with its own evidence |
| every third occurrence (3rd, 6th, …) | the owners, marked **recurring: N occurrences** |
| any unrecovered (reported) occurrence | the owners, marked **not recovered** |
| an occurrence after the review was ✔'d | a new episode `review-…-2`, naming the owners and linking the reviewed one |

## Recoverable, without duplicates

- As an incident closes rescued or reported, its record is marked
  `review.owed` before anything is posted (`Reviews.mark`). Every later
  look hands over whatever is owed (`Reviews.deliver`).
- Each review carries `[selfnote][review] <signature> e<n>`, and each
  occurrence `[selfnote][occurrence] <incident key> e<episode>`. Before
  posting, the mirror is read for them. A crash between the post and the
  local record is finished by the next pass without a second post.
- With the store lost (`reviews.json`), the newest episode is adopted from
  its note: an open one is continued, a ✔'d one is followed by the next.
  A failed handoff stays owed and is retried the next look.

## Tests (agobserver 147 passed; `test_review.py` 5)

- one rescued silent exit gives one review with the opening, the
  occurrence carrying every field above, and the two notes. The incident
  says where it went once, stays `rescued`, and its review is `delivered`;
- three stalls and recoveries of the same task give occurrences 1, 2 and 3
  in one review: 1 names the owners, 2 is quiet, 3 names them as recurring;
- **an interrupted handoff**: the process dies after the post, before the
  record, and a restarted monitor with the review store deleted finishes
  it with no second occurrence and no second topic;
- a review the developer ✔'d: the next recurrence opens
  `review-autolab-stopped-2`, which names the owners and the reviewed one;
- requests before the failsafe horizon are not reviewed.

`test_health_path.py`'s escalation case also checks that an unrecovered
occurrence reaches the review, marked and naming the owners.

## Limitations

- An incident **reported** and later moving again (`cleared`) says so in
  its incident topic, not in the review.
- The hypotheses and improvement candidates are fixed per kind. They are
  prompts for the developer, not analysis.

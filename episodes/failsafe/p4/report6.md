# failsafe p4 — step 6: validation, consolidation

## The plan's table

| check | required result | evidence | met |
|---|---|---|---|
| Agreement and interrupted close-out | accepted work executes once; close-out resumes without repeating it; status reflects the committed outcome | R3 and R4b (report2): count 1. R4b's listener exited after the push, and the restart finished from the `accepted` note with no run: integration `already`, one `## Result`, `completed`. T1–T3 (report4): each agreement closed with count 1 | ✓ |
| Missing/misplaced result | only the deficiency is repaired; changed results receive the required review | R3: file moved out of the repository after review → the review moved it back, re-ran nothing, and the copy equalled the reviewed checkpoint, so it closed. A copy that differs from the reviewed checkpoint, or holds files outside the repositories, is refused and asked again (tests). R4a: a change request made that change only and asked again | ✓ |
| Reply above 10 000 and below the new limit, with code and trailing metadata | full content stored and accessible to the recipient; no poster/server cut | #13048 (52 450 chars): server copy, three mirrors, and Front quoted text 52 000 characters in. Front's own 26 505-char reply #13053, with a code block, text after it and `end=`, was posted whole (report3) | ✓ |
| New size boundary and limit discovery/refresh | budget includes metadata; over-limit delivery has an explicit recoverable outcome; no false complete receipt | the metadata line counts toward the limit. 100 001 chars: refused before sending, nothing posted. An over-long run reply is repaired for size, then owed, kept whole, and gets no receipt (listener test on the real journal). Discovery, TTL refresh and fallback are tested; `.limits` was written live | ✓ (the over-long *agent* reply by test, not live) |
| Multiple recovery trials | correct task resumes and completes without developer rescue; no competing execution or duplicate acceptance | T1, T2, T3: 3/3 completed and 3/3 missions `done`; counts 1/1/1; 0 escalations; 0 duplicate requests or acceptances | ✓ |
| Evidence changes before recovery action | already resumed work is recognized; stale stop evidence launches nothing | T3: the requester resumed the task the second Observer asked; Front's re-check said RESUMING and it posted nothing; Observer, restarted in the incident, asked no second time | ✓ |
| Correction of archived task records | completed tasks read completed, mission cancellation valid, no work restarted | report5: m9349 and m9697 read `completed` in autolab, the trace and the relay; missions `cancelled`; channels re-archived; no serving; Observer state unchanged across a restart | ✓ |

The five-minute target, on operational timing: silent exit → Front in
**146, 153 and 172 s** (T1–T3); exit → resume posted in 178 and 173 s (T1,
T2). No result above was accelerated: no `timing.json` was used in p4.
The close-out and size checks used short counted operations (`tick.py`,
milliseconds) instead of waits.

## Tests after the last change

| suite | passed |
|---|---|
| pyagag | 981 |
| agautolab | 331 |
| agobserver | 169 |
| agfront | 182 |
| agforge | 265 |
| archsage | 37 |
| cagent | 204 |
| agentroom relay | 353 |

## Rollout and residue

- pyagag `673e70d` everywhere a conversational role runs, and in the relay.
  All services were kickstarted with no run in flight (07:31:30Z; autolab
  and Observer again at 07:59:46Z).
- The autolab introduction was re-posted (agreement covers the shown
  result; closed = `completed`).
- `nctl drift --host agstudio` converged and `nctl status` ok, before and
  after.
- Trial residue:
  - missions R1–R4 had their acceptance recorded on the stand-in's
    agreement posts (Front's credential, as in the p3 backfills); T1–T3
    were recorded by Front in the trials;
  - stray copy folders removed (R2's and p3 C's);
  - trial counters removed;
  - no fault files, no `timing.json`;
  - Observer tracks only the two held requests.
- The trial conversations `front-failsafe-p4-{long,t1,t2,t3}` are left
  open as the record. The review `review-autolab-stopped` (occurrences
  5–7) is left for the Developer's ✔.
- Host notes: `pj-agdev/.local/devenv.md` § failsafe p4. Development
  documentation: `README_DEV.md` (autolab's review and state correction,
  post size, recheck).

## Cost of the whole phase (06:31–08:00Z)

autolab supercoder 19 runs $1.53, superdirector 7 runs $0.93; Front 18
runs $4.04. **$6.49.** Observer ran no model judgment.

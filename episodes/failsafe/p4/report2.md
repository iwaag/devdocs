# failsafe p4 — step 2: an accepted result is closed, not worked again

## The change

The distinction is made by the listener, from the record, before any run.
It no longer rests on the worker reading its chatlog correctly.
agautolab `bcc51a1`.

| | before | now |
|---|---|---|
| a requester's post after a shown result | served by the full task run, the task text as its instruction; a guide sentence asked it not to repeat | served by a **review** of that result. The prompt frames the task as done and quotes the shown post (#S). It also gives the checkpoint written with it (#C, what the requester reviewed), whether the copy is still exactly that, and any files outside every repository. The review guide is `workrun_supercoder/review.md` |
| any other post (start, continue, resume, a question's answer) | full task run | unchanged: the task run. It cannot close anything |
| agreement signal | `report.md`, whose text became the record | `close.flag`, written only by a review run. Its content is ignored |
| what an agreement covers | the copy as it stood **after** the closing run; a difference from the reviewed checkpoint was only described | only the reviewed state. The close is **refused** while the copy differs from checkpoint #C (`missionspace.changed_since`) or holds files outside every repository (`stray_paths`). The refusal asks the requester again |
| the task's record (`## Result`, devlog) | `report.md` of the closing run | the shown post #S word for word, plus "Agreed to in #P, as shown in #S". The `accepted` note carries `+shown=S +checkpoint=C` |
| files outside the repositories | silently not integrated, and silently left at release | said under every reply of the task. They block the close, and release names them and keeps them |
| close-out cut between `## Result` and `[state] completed` | posted the result twice | posts only the state |

What stays the same:

- the requester's words are still judged by a model (agree, change, repair,
  question). That is agent judgment, and the plan keeps it;
- overlap/conflict review (`returned`) is unchanged;
- an old agreement never covers a changed or combined result, because the
  combined result is a new shown post;
- mission-level acceptance (`agentchat accept`) is unchanged and separate.

Why not restrict the review run's tools instead: autolab's claude_code
roles run with the permission bypass, so a grant would be documentation, not
a boundary. The gates are the listener's own git reads.

## Tests

- agautolab: 324 passed. New cases:
  - a work serving is given the task, a review serving the result;
  - an agreement closes only the reviewed state;
  - a result changed after review needs the agreement again;
  - a result outside every repository is never closed as delivered;
  - a close-out cut after its result post does not post it twice;
  - real-git tests for `changed_since` and for strays kept and named at
    release.
- The earlier close-out tests were rewritten to the bound record.

## Live verification (after deploying `bcc51a1`, listener kickstarted 06:47:43Z with no run in flight)

The counted operation is `tick.py` (report1). The Omni Agent stood in as
requester.

| trial | what happened | executions |
|---|---|---|
| **R1** (before the change, m12931) | ordinary agreement; the worker judged correctly and closed | 1 |
| **R2** (before, m12957) | result moved out of `main/` after it was shown; agreement → the worker **ran the command again**, asked again; the closing record said "the only run" | **2** |
| **R3** (after, m12990) | the same fault. Agreement #13005 → **review**: the run found the file outside every repository, moved it back unchanged, ran nothing. The copy then equalled checkpoint #13003, so the agreement closed it (`accepted … +shown=13004 +checkpoint=13003`). `## Result` #13011 is #13004's text | **1** |
| **R4a** (after, m13017) | a change request after the shown result ("add a line `reviewed`") → review: the edit only, shown again as a confirmation request (#13036) | 1 |
| **R4b** | agreement #13037 with `faults/exit-after-integration` armed. Review → `close.flag`; accepted #13040, integrated #13041 (pushed); **the listener exited**. launchd restarted it within a second, and it finished from the note: "finishing the close-out accepted in #13037", no run, integration recognized as `already`, one `## Result` (#13044), `completed`. The restart served nothing else | **1** |

Timings:

- R3: agreement → `completed` in 39 s;
- R4b: agreement → `completed` in 15 s, including the exit and restart.

## Measures against the step 6 table (so far)

| check | state |
|---|---|
| Agreement and interrupted close-out | ✓ R3, R4b: executed once; close-out resumed without a run; status from the record |
| Missing/misplaced result | ✓ R3: only the deficiency was repaired. A result changed after review is refused until agreed again (unit test; R4a shows the change path asking again) |

## Deus ex machina / interventions

The Omni Agent stood in as requester and injected the faults: the moved
file in R2/R3 and the exit in R4b. There were no other actions.

## Cost

Steps 1–2, autolab only: superdirector 4 runs $0.55, supercoder 10 runs
$0.96. **$1.51.**

## Residue

- Missions m12931, m12957, m12990 and m13017 are closed at the task level
  and `started` at mission level. They are left for the step 6 cleanup.
- `robustp1/main` now carries `docs/p4-r{1,2,3,4}.txt`. That is trial
  output in a disposable test bed.
- A known quirk seen at every restart: the listener's recovery logs "names
  us and has no receipt" for agreements that mention autolab in its own
  task topics, then ignores them (not a task home). No run is started.
  Left as is.

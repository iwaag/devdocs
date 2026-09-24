# Step 6 report — is a Sonnet comparison the next useful experiment?

Plan: [plan.md](plan.md) step 6. 2026-09-24 (JST), by the Omni Agent.

**Decision: defer the Sonnet comparison.** Every wrong local verdict in p3 traced to missing
evidence, a cut that dropped the question, or a system fact the judge could not know. Each was
fixed in the input or the guide, and the same local model then judged every replayed and live case
correctly. The condition for reconsidering is at the end.

## What was reviewed

All of p3's local triage judgments (`qwen3.8:27b-mxfp8` through agcode, this host's ollama,
`TIMEOUT_SECONDS` 150, `--max-turns 6`, `--deadline-s 130`), plus three earlier verdicts replayed
on the realm as it stood at the moment of judgment. The replay is `replay.py`: a copy of Observer's
mirror with every post newer than the cut hidden, and conversations ✔'d after the cut shown open.
Each case was judged three times per variant:

- **old**: the prompt exactly as the deployed revision (`ec32e59`) built it, with the committed guide;
- **new**: the prompt with the request's own conversation and both ends of a long post;
- **final**: new, plus the guide sentence about the acknowledgement.

The exact prompts, transcripts, verdicts, cited evidence and timings are kept on the host
(`agobserver/.local/triage-replays/p3/`). Since step 5's S2 every live judgment also writes its
exact `prompt.md` and `input.json` beside its transcript (`agobserver.triage`).

| Case (what it is) | Right answer | old | new | final |
|---|---|---|---|---|
| C1: p3 trial C, a ✔ on a task whose report waits for the human; Front has just asked them | legit | stall, stall, legit | legit ×3 | legit ×2 |
| C1b: the same, after Front answered Observer's needless request | legit | legit, stall, legit | legit ×3 | — |
| C3p2: p2 trial C3, the same shape (live verdict then: legit) | legit | legit, stall, stall | legit ×3 | legit ×2 |
| E3p2: p2 trial E3, a worker killed after its ack, silent 45 min (live verdict then: stall) | stall | legit, stall, stall | legit, stall, unclear (timeout) | **stall ×3** |

**Live on the final code** (S2 onward): D ✔ `legit` twice (a restart lost the first verdict); E
`legit`, discarded as stale, then `legit` on the current state; F `legit` held and never applied.
All six are right for the evidence they read. There were no requests and no reports on the final
code. Times ran 33–107 s, with one 130 s timeout in the replays, and replays shared ollama with the
live monitor.

## What went wrong, by kind

| Kind | Instance | Handling |
|---|---|---|
| **Evidence collection** | C: the judgment saw the stalled conversation and a one-line state of the origin (`AWAITING_HUMAN — Front answered #10191`), not that Front's #10191 had put the question to the human. The judge called the ✔ "a different thread" and ruled stall. That caused one needless request | The prompt carries the request's own conversation: its newest six spoken posts, without acks |
| **Truncation** | #10191 was cut at 600 characters, and its question ("Do you accept task 1?") is the post's last line | A long post keeps its beginning and its end (`[…]` between). The last-ten-message window of the stalled conversation omitted nothing material in any case reviewed |
| **Guide defect** (two definitions overlapping) | "A ✔ on work still going on" (stall) and "a human asked and not answered" (legit) both describe a task waiting on the human with its topic ✔ | One paragraph: look at where the decision is asked for. Asked of the person who decides → legit; nobody told → stall |
| **Judgment error with sufficient input** | E3p2: in 2 of 6 runs the judge read autolab's automatic "Message received. Please wait for the reply." as a sign the worker was still busy | A system fact, stated once in the guide (the listener posts it before any work starts). 3/3 stall after |
| **Execution** | One replay run hit the 130 s deadline (exit 2) | Read as `unclear`, as designed: an unclear verdict asks nobody, and two in a row are reported |
| **Result handling** | D: a verdict held only in the worker's memory was lost when Observer restarted 8 s after it arrived. The same question was asked again, at the cost of one more local judgment | Accepted as is; noted in report5 |

Similar-looking verdicts that differed (C1's 2/1 split, C3p2 legit live but stall in 2 of 3
replays) were the same model on the **same** input, so they are genuine inconsistency. That
inconsistency went away once the input carried the deciding fact. The E3 regression in the first
"new" run (1 legit, 1 unclear) came from the prompt growing longer while the guide still said
nothing about the ack, and it closed once the fact was stated.

## Why defer

- Every observed wrong verdict had a cause that was not the model's capacity. Each was fixed, and
  the fix was checked on the same model and the same inputs.
- On the final code the local judge's live record in p3 is six of six, with no needless request.
  Across p2 and p3 its errors were input- or fact-shaped (p2's E3 stall was right; p1's and p2's
  disagreement on a human ✔ is C3p2 above, now consistent).
- A comparison run now would largely measure the input gaps this phase removed. It would also be a
  harness comparison as much as a model one (below).

**Reconsider when** any of these is seen on the current input, where the deciding evidence is in
the prompt (its `prompt.md` shows it):

- a needless recovery request or report caused by a wrong verdict;
- more than one wrong verdict in ten live judgments;
- `unclear` or timeouts in more than a fifth of judgments over a week;
- wider discovery (routines, argues) bringing judged kinds whose evidence is longer or less
  structured than a task topic.

## If it is run later: the bounded comparison

- **Scope:** `triage` only. Nothing so far implicates `observe` or `intake`.
- **Inputs:** 15–20 fixed snapshots taken with the replay tool: 4–5 each of stalls (the E3 shape,
  a ✔ that closed live work nobody meant to end), legitimate waits (C1, C3p2, D, E), explicit
  cancellation (F) and genuinely insufficient evidence. Expected outcomes are written down
  **before** either model runs, by reading the snapshot, not by majority vote of the models.
- **Runs:** three judgments per snapshot per backend, with no live effect (the replay writes only
  to its own workspace).
- **Compare:** missed stalls, unnecessary requests, appropriate `unclear`, whether the cited ids
  exist and say what is claimed, consistency across the three runs, latency, cost.
- **Harness first:** `triage` passes `--max-turns` and `--deadline-s`, and `--deadline-s` exists only
  in agcode. Under the `sonnet` profile (claude_code) it would be an unknown flag, and every
  judgment would end `unclear` at start-up. The local and Sonnet profiles also differ in harness
  (agcode vs claude_code), tool loop and result handling (`result.json` vs printed JSON). Either
  add a Sonnet profile through agcode, which isolates the model, or report the result as a
  **backend** comparison, not a model one.

P3's acceptance stayed on local throughout. No model switch was used to cover an implementation
defect.

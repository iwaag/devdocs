# agent_standardize p5 — final report: Front supervised three workruns to done

AI-generated (Omni Agent, 2026-08-21).

## Outcome

**Done.** All three `workrun-task{1,2,3}-s2-6` topics are `✔` resolved, each
with its Plane issue `Done`, its devlog commit and its code commit. Front
triggered every one, waited through the work, answered what it was asked and
agreed the task was complete; autolab closed each out itself. Step reports:
[report1](report1.md) (`agentchat wait`), [report2](report2.md) (making a long
run legal), [report3](report3.md) (the live supervision, with the full message
trail).

The braindump's real question was whether Front can stay on a task that
outlives a quick exchange. The answer is **yes, and the constraint was not
time.**

## The one finding worth the phase

Front's first supervision stopped after **242 s of a 3600 s budget** — not a
timeout, a decision. It backgrounded its `agentchat wait` and finished its
turn, which ends the run, so nothing was watching:

> "I'll wait for the background command to notify me when autolab replies,
> then continue supervising through tasks 2 and 3."

Raising the timeout was necessary and not sufficient. What fixed it was one
guide line saying where waiting happens (`c1e0233`) — no code. The next run
went **632.8 s / 37 turns**, carrying task2 and task3 end to end and moving
between them unprompted. That is the p2 lesson again: when Front does the
wrong shape, the repair is usually a sentence, not a mechanism.

Cost of the whole thing: ~$5 of agent time, ~20 minutes wall clock, one
retrigger.

## What each step delivered

1. **`agentchat wait`** (pyagag `4e6e903`) — block until a message newer than
   `--since`; exit **3** on timeout so a loop can tell "still quiet" from "it
   broke". With it, `read --since` and message ids in every printed header,
   which is what makes a supervision resumable across runs.
2. **A budget and a disposition** (agfront `7d308c5`) — `FRONT_TIMEOUT_SECONDS
   360 → 3600`, sized from the run records (`run` role: median 106 s, max
   1140 s against a 1200 s ceiling), plus guide lines telling Front to stay,
   to answer, to say when it agrees, and to report honestly if time runs out.
   And, because the guide must not learn another agent's vocabulary,
   **autolab's introduction** (agautolab `cd54dad`, posted as message 725)
   gained the close-out contract it had been keeping in its code.
3. **The live run** — three tasks, `✔` ×3, told in [report3](report3.md).

## The defect the live run found

`agentchat wait` went blind at exactly the event it exists to catch: resolving
a topic renames it to `✔ <topic>`, so a `--since` wait on the open name
narrows to an empty topic and sits until timeout. Found by hitting it, fixed
in `5bda102` — both `wait` and `read --since` try both names — and re-locked
into agfront before the runs that needed it. A tool written for supervision
that cannot see a close-out is worth writing down as a near miss.

## Attributability, still

```text
$ grep -rn "autolab" agfront/src agfront/agent            -> exit 1
$ grep -rn "workrun-" agfront/src agfront/agent           -> exit 1
$ grep -rn "work-s2-6\|pj-simpleshooter\|autolab-agstudio1" \
      agfront/src agfront/agent                           -> exit 1
```

Front learned that its agreement closes a task by reading message 725 in
`#agents`, harvested into `tools/agents.md` seconds before its run. Four
phases now, the same rule: routing and contracts travel as posts.

## Commits

All pushed to GitHub.

- **pyagag** `4e6e903` (`wait`, `read --since`, ids) · `5bda102` (the resolve
  rename). Tests **275 passed**.
- **agfront** `8ad5820` · `7d308c5` (timeout + guide) · `c1e0233` (waiting is
  blocking) · `88f8455` (lock bump). Tests **31 passed**.
- **agautolab** `cd54dad` (the introduction's supervision section).
- **pj-agdev** `e8b13bb` · `4138d5a` · `b7a2d1b` (submodule pointers,
  `.local/devenv.md`).
- **devdocs** — the four reports, `README_DEV.md`.

`pj-simpleshooter` itself: `main` `3f8abbb` → `ec29bd9` → `1282a93`, three
`[AUTO]` devlog reports, Plane S2-7/S2-8/S2-9 Done. Written by autolab,
approved by Front, watched by nobody human until it was over.

## Open, and worth deciding rather than rediscovering

- **What a supervisor may approve on its own.** Front answered
  「はい、コミットしてください」 three times without asking the developer, and those
  answers put three commits in `main`. Defensible here; undefined in general.
  The line between "approve this step" and "escalate this" is written nowhere.
- **Agreement and go-ahead collapsed into one message.** The supercoder read
  the commit approval as the agreement that the task was done. The contract
  held, but incidentally — Front never said "this is complete" as such.
- **The proposal step was skipped.** p4's lines make Front propose before
  acting; a sufficiently explicit instruction now overrides that, by Front's
  own judgement. Fine, and unrecorded until now.
- **The 3600 s ceiling is untested from above** — the longest real run was
  632 s. Whether a claude_code run survives an hour of mostly-blocked waiting
  is still unknown.
- **Deferred by the plan, still deferred**: parallel supervision (autolab's
  previous-work gate serializes tasks anyway) and unattended, no-permission
  supervision.
- **Carried from p4**: the Plane identity split, agautolab1 instancing, and
  autolab's entrance reply beyond a placeholder.

## Deus Ex Machina note

Both "Developer" messages (726, 738) were written and sent by the Omni Agent
with the Developer's credentials and in-session permission, as in p2–p4. The
second one restated the guide's new waiting rule in its own words, which means
the guide line and a direct instruction landed together — the next
supervision, unprompted, is what will tell them apart. A human-written wish
remains the better test, and the better handoff.

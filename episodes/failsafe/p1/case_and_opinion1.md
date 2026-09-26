# failsafe p1 — case 1 and opinion: a mission that stopped without a word

Written by the Omni Agent on 2026-09-26. It describes one stall in detail
and gives an opinion on the braindump. It is input to the phase's
discussion, not a plan.

## 1. The case

### What was asked

On 2026-09-26 the Developer asked in `#front › front-desk-20260926-221323`
whether the `aisvgs` study had fully covered its subject: recent research,
community, active people, and enough to try things locally. The request
went through this path:

- Front asked archsage (`#archsage-agstudio1 › study-aisvgs-round2`).
  archsage answered "not exhausted" (#11728).
- Front opened `routine-study-aisvgs › routinerun-20260926-2225` and asked
  autolab for one bounded research phase.
- autolab planned it as mission **m11741** with one task. Task 1 started at
  13:25 UTC in `work-m11741 › workrun-task1-m11741`.

### What happened (all times UTC)

| time | event | source |
|---|---|---|
| 13:25:03 | autolab starts the supercoder serving `run-0441` (Claude Code, `claude -p --output-format stream-json`, prompt on stdin, stdin then closed) | listener log, run JSON |
| 13:25–13:26 | supercoder starts five background subagents: strand 5A people, 5B products/practice, 7 local kit, reinforcement, strand 6 appendix | session transcript |
| 13:26:18 | supercoder tries `ScheduleWakeup` (1800 s fallback). The tool is not available in this mode and the call fails | session transcript |
| 13:26:34 | supercoder posts "waiting on them" and ends its turn | transcript, Zulip |
| 13:30, 13:31, 13:32, 13:33 | four `<task-notification>` wake-ups, one per finished subagent; each time it posts progress and ends its turn again | transcript, Zulip |
| 13:33:25 | last turn: "Only the local kit run is still going", then end of turn | transcript |
| 13:40:27 | the kit subagent's `refine_svg` run writes its final outputs, but the Bash call has not returned yet | file mtimes |
| **13:43:26** | the kit subagent's in-flight Bash call gets "The user doesn't want to proceed with this tool use… [Request interrupted by user for tool use]". The session ends in the same second. `run-0441` is recorded with `outcome: done`, 1102 s, $7.35 | subagent transcript, run JSON |
| 13:43:27 | autolab's final post, #11766 (progress to Front): "the local kit run is still running; after it finishes I'll merge, add INDEX rows, commit and report" | Zulip |
| after that | nothing. At 14:51, 82 min after the last answer, nothing had moved and no incident had been opened | `agentchat trace 11711` |

What was left behind in the mission copy (`autolab/m11741`, uncommitted):

- `reports/strand5a-people.md`, `strand5b-products-practice.md`,
  `strand2b-reinforcement.md` and `strand6-adjacent-appendix.md`
- `methods/kit-scripts/` and an edit to `methods/sources-adopted.md`
- the strand 7 kit outputs under `.local/kit/`

These results existed only on agstudio's disk. If that machine had been
lost, all of them would have gone with it.

### Why nobody noticed

Each layer did something that made sense locally. Together the stall
became invisible.

1. **supercoder waited correctly.** It started background subagents and
   ended its turn, expecting completion notifications to wake it. That is
   the normal way to wait in an interactive Claude Code session, and it
   worked four times. It also tried to register a wake-up and could not.
   Nothing in the workplan or guide told it to "not wait" or "let Observer
   wake you". The workplan only said to launch parallel subagents before
   waiting.
2. **The headless process ended anyway.** autolab's own ceiling was not the
   cause: `WORK_TIMEOUT_SECONDS = 1200` was not reached, and there was no
   kill. The Claude Code process ended by itself about 10 minutes after
   its last turn and interrupted the subagent that was still running.
   *Inference, not verified against Claude Code:* in `-p` mode it waits
   only about 10 minutes for background subagents after the model's last
   turn. The model had no way to know that limit.
3. **autolab recorded the end as success.** The run ended with
   `outcome: done` and no failure post. The last word in the task was a
   progress post that promised a report which would never come.
4. **trace misread the conversation.**
   - autolab labels its progress posts with an `ag-post intent=progress`
     marker at the end.
   - #11758 and #11759 were long enough that Zulip replaced their tails with
     `[message truncated]`, which removed the marker. `is_progress()`
     returned `False` for both.
   - trace therefore read #11759 as autolab's *answer* and put the task in
     `awaiting_requester`. That label is wrong: Front owed nothing, and the
     worker was dead.
5. **Observer treated that state as healthy.**
   - In `agobserver/src/agobserver/monitor.py`, `awaiting_requester` is in
     `MOVED_ON`: it counts as progress, and no incident kind fires on it.
   - Observer itself was fine. The request was tracked (`o11711`) and
     looked at every 2 minutes. It concluded what the conversation seemed
     to say.
6. **Even with a correct reading, Observer could only have seen silence.**
   Observer judges from speech. "Working quietly" and "dead" look the same
   in a conversation. The trace documentation says so itself: it "does not
   claim that a worker is alive".

The known open gap from sage p2 was named as "supercoder can end a serving
with a subagent unfinished — nothing notices". This case is that gap,
observed end to end.

## 2. Opinion

### I agree with the braindump's premise

The system's contract is loose on purpose: an agent is anything that can
hold a Zulip account and answer. Given that, a worker disappearing
mid-task is a normal event, not an exception. Several things can cause it:

- the machine sleeps or dies, or the network drops
- a harness has its own limits (this case)
- a model or harness update changes behaviour

The key point is that **a party that has died cannot report its own
death.** A rule such as "always post X when you finish, or the workflow
stops" can make failure reports faster and more informative. It can never
guarantee them. This case is a *graceful* failure: the process exited
cleanly, on a machine that stayed up. Even so, nobody noticed for over an
hour. A power cut would have been strictly worse.

So I would reverse the ranking I gave earlier in the conversation:

- **External detection (Observer) is the guarantee.** It is slow, but it
  works whatever the failure mode. This is the essential part, not a last
  resort for an immature system.
- **Self-report is an optimisation.** Examples: autolab posting `failed`
  when a serving ends with subagents still unfinished, or supercoder
  blocking until its subagents finish. These are fast and carry the cause.
  They are worth having, but nothing may depend on them arriving.
- A monitor inside the system is also not *Deus Ex Machina*. It is how the
  system notices its own failures, which is the system's purpose in the
  first place.

### What that premise demands, beyond today's Observer

1. **Liveness has to be a signal, not an inference from speech.**
   - Today Observer reads conversations and guesses states. That guessing
     is fragile: one truncated post flipped "executing" into "moved on".
   - A failsafe design needs a positive, cheap, periodic signal from
     whoever holds work: "I hold task X, alive as of T". It should work
     like a **lease**: the holder renews it, and if the lease expires
     without being renewed, the work is suspect.
   - Such a signal could be the owner of a conversation in Zulip, or a
     small record outside it. Either way, it has to be something Observer
     can check without interpreting prose.
   - Progress posts already come close to this. They are not yet a
     contract.
2. **States should come from explicit declarations, not inference.** A post
   should say what it is ("progress", "answer", "question", "failed") in a
   form that survives transport. A marker at the end of a long message did
   not survive. Wherever trace still has to guess, the guess should fail
   towards "suspect", not towards "moved on".
3. **Recovery needs durable intermediate state.** Detecting the stall is
   only half of it. The work also has to be resumable by *someone*, possibly
   on another machine. Some ways to get there:
   - checkpoint commits pushed to the mission branch,
   - the task's own record saying what is done and what remains,
   - no result that lives only in one machine's working tree.
   Here, a resumption was possible only because agstudio stayed up.
4. **Decide what must always be running, and who watches it.** The core
   that is allowed to be a hard dependency should be small:
   - Zulip, certainly.
   - Observer, probably, as the failure detector.
   - Then something must watch Observer. The relay watchdog and health DMs
     are a start. Two small monitors watching each other may be enough;
     that needs deciding, not assuming.
   Everything else should be allowed to fail and be recovered.
5. **Recovery can be an agent's judgement, with deterministic detection.**
   I agree with the braindump that recovery cannot be pre-scripted:
   harnesses and models change too fast. I would still split the problem:
   - **Detection should stay deterministic and cheap:** lease expiry,
     undelivered answers, unacknowledged posts. A detector that needs an
     LLM to notice a death inherits the same fragility.
   - **Diagnosis and recovery can be an agent's work,** accumulating cases
     like this one. That agent should hold an explicit set of recovery
     actions, such as asking the owner, re-serving, reassigning to another
     machine, or escalating to the Developer, instead of improvising
     mechanics.
6. **Treat Observer's blind spots as the first design input.**
   - The pattern from mediagen applies here too: a metric fails first where
     it is blind. Observer's blindness sits exactly where a speech-based
     monitor can't look (dead but quiet), and where its classifier can be
     fooled (a truncated marker).
   - An experiment for this phase: inject failures deliberately and measure
     time-to-detection for each. Candidate failures:
     - kill a serving mid-task,
     - drop a machine's network,
     - truncate a post,
     - stop Observer itself.

### Smaller fixes that stay valid either way

These fixes are useful whatever the redesign becomes. None of them is a
substitute for it.

- Put the `ag-post intent=` marker at the **start** of a post, or cap
  progress posts so they are never truncated.
- Make autolab post `failed` when a serving ends while the stream shows
  background subagents without a completion notification.
- Tell supercoder, as evidence-driven guidance, that headless runs stop
  waiting for background subagents after a limit. It should therefore
  block on their results before ending its last turn.

## 3. State of the case at the time of writing

At 14:51 UTC m11741 was still stalled. The uncommitted results were intact
in the mission copy, and no resumption had been posted. The question of
how to resume is still open with the Developer:

- a post in the task itself, or
- a note to Front, so the recovery goes through the system.

Whichever is chosen, it is the first data point for measuring the
system's recovery path.

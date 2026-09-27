# failsafe p4 — report: close accepted work reliably, deliver long replies whole

## Outcome

| completion condition | result |
|---|---|
| the plan's checks pass | ✓ all seven (report6's table) |
| repeated autonomous recovery | ✓ three operational trials, 3/3 recovered and completed with no developer rescue. Two resumes on a confirmed stop, one correct abstention when the work had already resumed |
| the five-minute detection target | ✓ silent exit → Front in 146, 153 and 172 s (target 240 s, goal 300 s), all on operational timing |

The step reports:

- `report1.md` — reproduction and measures;
- `report2.md` — close-out;
- `report3.md` — post size;
- `report4.md` — recovery;
- `report5.md` — record correction;
- `report6.md` — validation.

## Changes

| gap from p3 | change | where |
|---|---|---|
| an agreement re-ran the task's work (p3 C; reproduced as R2: a counted command ran twice) | the listener decides from the record which run a serving is. A post after a shown result gets a **review** of that result, which agrees, repairs a deficiency or makes the asked change; the task text is framed as done. `close.flag` replaces `report.md` | agautolab `bcc51a1` |
| acceptance not bound to what was reviewed | the close is refused unless the copy is exactly the checkpoint written with the shown result; `accepted` carries `+shown=` and `+checkpoint=`; the task's record is the shown post word for word | agautolab `bcc51a1` |
| a result outside the repositories closed as delivered (p3 C) | such files are said under every reply, block the close, and are kept and named at release | agautolab `bcc51a1` |
| an interrupted close-out could post its result twice | recognized and not repeated | agautolab `bcc51a1` |
| replies cut at 10 000 characters | the server keeps 100 000 (persistent override, survives recreation); clients learn `max_message_length` and cache it per credential; nothing is cut; an over-long post is refused before sending; an over-long reply is repaired, then owed, and kept whole; fixed caps removed from forge, cagent, Front's routine report and the relay's realm statement | Zulip deployment; pyagag `e308b94`; consumers |
| Front could misread recovery evidence | `agentchat recheck <anchor> --after <ack>` re-reads the exact conversation; Observer's request names it; Front's guides act on its verdict | pyagag `673e70d`; agobserver; agfront |
| m9349/m9697 task 1 read `cancelled` | `correct_state` appends `[state] completed` and a `[correction]` note naming the wrong one, through an archived channel, without reopening anything | agautolab `27b1bbe` |

## Trial evidence

**Execution counts** (the counted operation `tick.py`, one line per
execution):

| trial | what it tested | executions |
|---|---|---|
| R1 (before the change) | ordinary agreement | 1 |
| R2 (before) | result moved after review, then agreed | **2** — the defect |
| R3 | the same, after the change | 1 |
| R4 | change request after review, then agreement with the listener exiting mid-close-out | 1 |
| T1, T2, T3 | stop → recovery → agreement | 1, 1, 1 |

**Recovery timings** (operational):

| | T1 | T2 | T3 |
|---|---|---|---|
| exit → Observer's request | 146 s | 153 s | 172 s |
| Front's decision | resume, 178 s | resume, 173 s | abstained (RESUMING), 189 s |
| resumed work | +197 s | +185 s | +185 s (the requester's resume) |
| completed (after the stand-in's agreement) | ✓ | ✓ | ✓ |

**Delivery integrity**:

- a 52 450-character post was stored and mirrored whole, and read by Front
  past its 20 000-character prompt window;
- a 26 505-character agent reply with a code block, text after it and its
  `end=` line was posted whole;
- 100 001 characters were refused before sending;
- no `[message truncated]` was produced in the phase.

**False interventions:** 0. Observer asked only about the three killed
servings.

**Duplicates:** recovery requests 0 (including across an Observer restart
in the incident), acceptances 0, competing executions 0.

**Human / Omni Agent assistance:**

- the stand-in's requests, agreements and acceptance posts;
- the scenario posts (T2's distractions, T3's direct resume);
- fault injection (silent-exit ×3, a moved file ×2, exit-after-integration
  ×1) and Observer's restart;
- the correction of m9349/m9697 (a Deus ex Machina note: did the record
  correction for autolab, handoff candidate — the tool is autolab's own
  and its entrance could run it);
- recording the four step-1/2 trial missions' acceptance.

**No rescue** before or after an escalation; there were no escalations.

**Cost:** $6.49 for the whole phase: autolab $2.46, Front $4.04; Observer
no model judgment.

**Corrected records:** m9349 and m9697 task 1 read `completed`; the
missions stay `cancelled`.

## Remaining limitations

- **Agreement is still judged by a model.** The review run decides
  whether the requester's words agree; the listener only guarantees that
  what closes is exactly what was reviewed, and that the task's work is not
  handed out again. A review run that disobeys its guide could still run a
  command. No trial did.
- **The over-long agent reply path was proven by test, not live.** A real
  reply over 100 000 characters was not provoked.
- **Front can promise a wake-up nothing will give.** T1's supervising
  serving ended "will reassess at the scheduled wakeup" (Claude Code's
  `ScheduleWakeup` in a headless run). It was harmless here because
  Observer held the work. Not fixed: one observation, with no failure.
- **Three trials are three.** Detection held with margin, and each
  decision was right, but this is a small sample. It says nothing of other
  owners (forge, agy/codex/gemini streams) or cross-host work.
- **comfynotify keeps its old pyagag pin** (it posts two short lines).
  **agautolab1 (VM) was not redeployed.**
- The listener's startup recovery logs "names us and has no receipt" for
  requesters' agreements that mention autolab in its own task topics, then
  ignores them. It is harmless and has been left as it is.
- **Pending human choices are unchanged:**
  - o11711/m11741 and o8512/m8519 are held;
  - acceptance of m6113, m7601 and m7732 is pending;
  - the review topics (`review-autolab-stopped` now at occurrence 7,
    `-uncertain`, `review-front-unanswered`) await the Developer's ✔.

## Where it lives

- pyagag `e308b94` (post size), `673e70d` (recheck).
- agautolab:
  - `bcc51a1` (review, gates, bound record);
  - `160e3df` (pin), `27b1bbe` (correct_state), plus the step 4 pin commit.
- agobserver in pj-agdev (request names the re-check).
- agfront:
  - `54e52d5` (routine report size);
  - the guides' re-check commit;
  - `6bf02ec` (comment).
- agforge `d949e4f`, archsage `929b4a4`, agdevworld `42a9714` (relay chat
  door), cagent in pj-clusterintent. Pins are at `673e70d` in every
  consumer.
- Zulip: the deployment's ignored `compose.override.yaml`.
- `README_DEV.md`; host notes in `pj-agdev/.local/devenv.md`.

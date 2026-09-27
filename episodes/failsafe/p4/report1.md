# failsafe p4 — step 1: the remaining failures, reproduced, and the evidence to judge them by

Date: 2026-09-27 (UTC). Host: agstudio. The Omni Agent stood in for the
Developer as requester.

## Starting state

- `nctl status` ok (Nautobot, worker, dumps, submodules clean).
- `nctl drift --host agstudio` converged. All listeners, the gateway, forge,
  cagent and the relay were running under launchd.
- Every repository was clean and level with its remote at the p3 end:
  - pyagag `975895e`;
  - agautolab `cc188d7`, agfront `d001e74`;
  - pj-agdev `26db324`, pj-clusterintent `0cd13b8`.
- No fault files and no `timing.json`.
- Zulip's weekly 06:00Z maintenance gave every listener one 502. That
  happens every Sunday (seen in the logs back to 2026-08-30) and is
  unrelated to this phase.

## Which reported gaps remain

| p3 gap | state now | where |
|---|---|---|
| An agreement re-ran the task's work (C) | **Remains.** The guide was fixed; the mechanism was not. Reproduced below (R2) | `zulip_listener._serve_run` |
| A result in the wrong place was closed as complete (C) | **Remains.** Nothing looks outside the repositories. C's `soak-p3-c.txt` still sits in the released copy's root (`missions/robustp1/m12762/docs/`), integrated nowhere, while task 1 reads `completed` | `close_out`, `release_view` |
| Front called a task closed before its record (A2 #12889) | Guide-only fix (agfront `19f2bfa`); untested | Front guides |
| Replies are cut at 10 000 characters | **Remains.** Server default, no override; the poster cuts too | see below |
| Front misread recovery evidence (A) | Request wording fixed (`920b651`); one sample (A2) | Observer, Front guides |
| m9349/m9697 task 1 read `cancelled` | Remains (step 5) | archived channels |

## Reproduction of the agreement rerun

The counted operation is a small script in autolab's ignored `.local/trials/p4/`
(`tick.py <name>`). Each execution appends one line to `<name>.count`, with
the time and the working directory, and prints `tick <name>: execution <n>`.
No wait was needed.

**R1 — the ordinary case** (m12931, `#pj-robustp1 › workplan-failsafe-p4-r1`):

| | time | count |
|---|---|---|
| request #12929 → plan, task 1 started by autolab | 06:31:49 | 0 |
| result shown (#12945, `response_request ask=confirmation`); the file is in `main/docs/` | 06:32:02 | 1 |
| agreement #12946 (`re=12945`) | 06:32:20 | 1 |
| agreement serving ran: git status, read its own old chatlog, wrote `report.md` | 06:32:34 | **1** |
| `accepted` → `integrated` (fast-forward, pushed) → `## Result` → `completed` → reply | 06:32:34–36 | 1 |

The guide fix held here. Still, the close cost a full supercoder run of the
whole task (14 s). Whether the work repeats was that run's own judgment.

**R2 — the result is in the wrong place** (m12957):

| | time | count |
|---|---|---|
| result shown (#12971): `main/docs/p4-r2.txt` | 06:33:42 | 1 |
| fault: the Omni Agent moved the file to the copy root (`m12957/docs/`, outside every repository) | 06:33:50 | 1 |
| agreement #12972 (`re=12971`) | 06:33:57 | 1 |
| agreement serving: "file doesn't exist yet. Let me run the command exactly once" → **ran tick again**, wrote a new file, asked again (#12976) | 06:34:11 | **2** |
| agreement #12977; close-out | 06:35:14 | 2 |
| the closing `## Result` (#12982) says "Ran once (**the only run of this task**)" | | 2 |

So:

- the requester's agreement bought a **second execution** of a completed
  command;
- the displaced file was never looked for;
- the closing record is **false** about the execution count;
- the stray `docs/p4-r2.txt` remains in the copy root after release.

Nothing blocked any of this: the handler cannot tell a missing result from
unfinished work, and the report the task closes on is written by the same
run.

## Acceptance traced end to end

| stage | written by | what it binds |
|---|---|---|
| shown result | the task serving's reply: `intent=report`, or `response_request ask=confirmation` | words; its message id |
| checkpoint | listener after each serving that changed the copy: `[change] checkpoint main=<head>:<tree>` | the copy's content, but **not** the post that showed it |
| agreement | requester's post in the task topic, after a shown result (`result_shown_before`) | the post only. `re=` is written but not read |
| **a new supercoder run** | every serving, before anything else | the whole task text again, with a guide sentence saying not to repeat |
| `report.md` | that run, if it judges the post an agreement | the gate for the close-out, and the text of the posted result |
| `accepted` | listener: `commit_pending(view)`, then `[change] accepted <repo>=<commit> #<post> +gen=N` | the copy **as it stands after that run**, not the reviewed checkpoint; `_reviewed_line` only *says* whether they differ |
| `integrated` / `returned` | listener | exact commits |
| `## Result` + `[state] completed` | listener (`record_result`) | report.md of the agreement serving |
| next task started, devlog, copy released | listener | |

What can make work run again:

1. **Every agreement starts a full worker run over the whole task** (the
   route is the same as for "continue"). Whether the work repeats depends on
   that run reading the chatlog correctly. R1 did, R2 and p3 C did not.
2. Anything that stops a serving between the run and the `accepted` note —
   an exit, a failed reply delivery, a restart — leaves no note, so the next
   serving is again a full run.
3. A missing or wrong artefact makes the run redo the task, because it
   cannot tell "not done" from "done but misplaced".

What makes a closure claim precede the record, or misstate it:

1. The closing report is written by the agreement serving, so it can
   contradict what was shown (R2's "only run").
2. The close-out accepts whatever the copy holds after that serving, even if
   it changed after the review. The changed result is only described.
3. A result outside every repository closes as "the task changed no
   repository" (p3 C); release then leaves it behind silently.
4. Front has concluded "closed" from reading a shown result (A2 #12889),
   13 s before the record. It is guided now; nothing in its evidence says
   `completed`-by-record apart from `agentchat trace`'s state.

Interrupted close-outs that already work: `accepted` / `integrated` notes
are finished without a run (`_owed_close_out`); integration is recognized by
ancestry; `start_next_task` never starts twice. Gaps:

- a crash after the `## Result` post and before `[state] completed` would
  post the result twice;
- a crash after `commit_pending` and before the `accepted` note re-runs the
  worker.

## Long replies (step 3 inputs)

- **Effective limit:** Zulip 12.2 (feature level 500) advertises
  `max_message_length` **10000** in `/register`. `MAX_MESSAGE_LENGTH =
  10000` comes from `default_settings.py`; there is no override.
- **Persistent source:** the ignored deployment
  `pj-agdev/.local/zulip-selfhost/compose.override.yaml`. The image's
  entrypoint writes every `SETTING_<NAME>` environment variable into
  `/etc/zulip/settings.py` at container start, so
  `SETTING_MAX_MESSAGE_LENGTH` survives recreation. The generated file must
  not be edited.
- **Client-side caps** in posting paths:
  - `pyagag/src/agag/post.py` `MAX_CONTENT = 10000`, used by `compose`, which
    cuts the body before its metadata line;
  - `agag.reply` guide text: "about 9 000 characters";
  - agforge `record.py` and cagent `change_record.py` `POST_LIMIT = 9000`
    (refuse a plan or record over it);
  - agfront `routine.py` `MAX_REPORT_CHARS = 8000` and `present.py`
    `MAX_RECORD_CHARS = 8500`;
  - the relay's chat door: `REALM_MAX_CHARS = 10000`, plus its deliberate
    4000-character human-chat guard.
- **Reading limits** (deliberate, keep): `topics.CONVERSATION_BUDGET = 20000`.
  The prompt carries the newest posts and cuts an over-long one visibly with
  "N more characters … are in chatlog.md". The post's meaning label is at
  the head of the line, so it survives the cut. `agrefs show` has
  `SHOW_LIMIT`.
- There is no outcome for a reply over the limit except the cut.

## Recovery evidence (step 4 inputs)

- Observer asks Front once the probe says `stopped`, with the check's fact
  (`920b651`). A second request is gated only by time (`RETRY_SECONDS`).
- `recovered()` needs a new ack in the stalled conversation with work after
  it.
- Front re-checks with `agentchat trace` and `agentchat read`. Neither says
  what the health check saw, and Front cannot run `agag.health` itself
  (`Bash(agentchat:*)` only). The trace shows `execution`/`holder` per
  conversation, but not "the owner's first post after #X".

## Measures for steps 2, 4 and 6

Each is taken from a record, not from what an agent said.

| measure | start | end | source |
|---|---|---|---|
| **detection** | the exit (harness kill time in the listener log / `.injected`) | Observer's request post | Observer's post id and timestamp |
| **correct recovery decision** | Observer's request | Front's first action on it: a post into the stopped conversation (resume), or "already resumed" with the evidence id, or an escalation | classified against the state at that moment (probe + trace); a resume when work had already resumed, or no resume on a confirmed stop, is a wrong decision |
| **actual resumption** | | first work event of a **new** serving (ack > the stopped ack) in the stopped conversation | mirror + autolab's execution record |
| **completion** | | `[selfnote][state] completed` in the task topic (and the mission's `done` record for mission acceptance) | mirror |
| **developer escalation** | first suspicion | Observer's report naming the owners | incident topic. It proves handoff, not recovery; anything after it is **assisted** |
| **execution count** | | lines in `tick.py`'s count file | the count file |
| **false intervention** | | a request, resume or escalation about work that was running or already resumed | incident timeline vs probe/trace at that time |

## Cost

R1 and R2 used 2 superdirector and 5 supercoder runs. The model cost is
read from autolab's run records in report6 with the rest of the phase.

## Residue

- Missions m12931 and m12957 in `pj-robustp1` are done at the task level
  and still `started` at mission level. They are left for the step 6
  cleanup, together with the stray file in `missions/robustp1/m12957/docs/`
  and C's `m12762/docs/`.

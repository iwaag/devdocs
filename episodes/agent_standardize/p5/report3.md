# agent_standardize p5 — Step 3 report: Front supervised all three to `✔`

AI-generated (Omni Agent, 2026-08-21).

## Outcome

**All three `workrun-` topics are resolved.** Front triggered each one, waited
through the work, answered the question it was asked, and agreed the task was
done; autolab closed each out itself — report, Plane `Done`, devlog commit,
`✔` rename.

```text
$ agentchat topics work-s2-6
✔ workrun-task3-s2-6
✔ workrun-task2-s2-6
✔ workrun-task1-s2-6
```

## The trail

The realm address stays unwritten, as in p1–p4; channel, topic and message id
locate everything.

| when (UTC) | channel / topic | what |
|---|---|---|
| 17:05:20 | `#front` / `front-p5-supervise-workruns` | **726** developer: 実行まで進めて、3つとも見届けて |
| 17:06:15 | `#work-s2-6` / `workrun-task1-s2-6` | **728** Front: 実行を開始してください |
| 17:07:19 | ″ | 731 autolab: refactor done — commit してよろしいですか？ |
| 17:07:36 | ″ | **732** Front: はい、コミットしてください |
| 17:08:19 | ″ | 735 commit `3f8abbb`; `task S2-7: … Done yes`; devlog; **736 ✔** |
| 17:09:55 | `#front` / … | **737** Front's run ends *mid-supervision* (see below) |
| 17:14:5x | ″ | **738** developer: task2、task3 も同じように |
| 17:15:34 | `workrun-task2-s2-6` | **740** trigger |
| 17:18:33 | ″ | 744 line / wedge / sideIn 実装、コミットしてよろしいですか？ |
| 17:18:44 | ″ | **745** Front: はい |
| 17:19:12 | ″ | 748 commit `ec29bd9`; `S2-8 … Done yes`; **749 ✔** |
| 17:19:23 | `workrun-task3-s2-6` | **750** trigger — Front moved on by itself |
| 17:24:39 | ″ | 755 balance pass + headless sim + Playwright, commit してよい？ |
| 17:24:42 | ″ | **756** Front: はい |
| 17:25:26 | ″ | 759 commit `1282a93`; `S2-9 … Done yes`; **760 ✔** |
| 17:25:38 | `#front` / … | **761** Front reports all three closed |

Proof beyond the `✔`: three Plane issues carry `commented yes, Done yes`
(S2-7, S2-8, S2-9), and the project's own repos have the work —
`main` at `3f8abbb` → `ec29bd9` → `1282a93`, `devlog` at `0c7ee62` /
`5db7de3` / `15d68ee` (`[AUTO] task N report for S2-6`).

## Wall clock

| task | trigger → `✔` | supercoder runs |
|---|---|---|
| task1 | 17:06:15 → 17:08:19 — **2m04s** | 26.7 s + 41.9 s |
| task2 | 17:15:34 → 17:19:12 — **3m38s** | 177.7 s + 41.7 s |
| task3 | 17:19:23 → 17:25:26 — **6m03s** | 314.9 s + 63.5 s |

Whole supervision, first developer post to final report: **~20 minutes**,
across two Front runs.

## How long-run supervision actually behaved

This is the measurement the braindump asked for, and it did not go the way
the timeout raise assumed.

**Front's first run stopped at 242 s of its 3600 s budget — by its own
choice, not by timeout.** It had triggered task1, answered the commit
question, and then, instead of blocking on `agentchat wait`, it put the wait
in the background and finished its turn:

> "I'll wait for the background command to notify me when autolab replies,
> then continue supervising through tasks 2 and 3." (message 737)

Finishing the turn ends the run, so nothing was watching. The budget was
never the constraint — the agent's model of "waiting" was. `run-0018`:
242.4 s, 20 turns, outcome `done`.

The guide gained one line from that (`c1e0233`), in Front's own vocabulary
and naming no agent:

> When you wait, wait inside this run: start the waiting command and let it
> block until it comes back. Do not put it in the background and finish your
> turn - finishing your turn ends the run, and then nobody is watching.

**The second run behaved.** `run-0019`: **632.8 s (10.5 min), 37 turns**,
covering task2 and task3 end to end — trigger, block, answer, agree, and then
*move to the next task without being asked*. That is a run outliving three
stretches of somebody else's work, which is what the phase set out to test.
Fair warning about the evidence: the developer's re-trigger post (738) also
said "待つコマンドをそのまま走らせて待っていてくれ", so the guide line and the
developer's restatement arrived together. The guide line is the durable half;
the next unprompted supervision is what will separate them.

**No timeout was ever hit** — not Front's 3600 s, not `agentchat wait`'s.
`FRONT_TIMEOUT_SECONDS = 3600` was necessary (360 s would have killed
`run-0019` at task2) but not sufficient; the 3600 s ceiling remains untested
from above.

**One retrigger was needed**, and the fallback shape the plan allows worked
exactly as written: Front's 737 said where things stood, and the developer's
next post picked the supervision back up with no state lost.

## The tool defect the live run found

`agentchat wait` went blind at exactly the event it exists to catch.
Resolving a topic *renames* it (`✔ workrun-task1-s2-6`), so a `--since` wait
on the open name narrows to a topic with no messages and sits there until
timeout. Found while waiting on task1 myself, seconds after autolab had
already closed it.

Fixed in pyagag `5bda102`: `wait` and `read --since` try both names
(`chat.topic_names`), with three tests and a live check — reading
`workrun-task1-s2-6 --since 733` now returns the close-out and the
Notification Bot's resolve line. agfront re-locked (`88f8455`) and restarted
before task2, so the fix was in Front's hands for the runs that used it.

## Two things Front did that nobody asked it to

- **It skipped the proposal.** p4's guide lines make Front propose and wait
  for permission; here it posted the trigger 55 s after the request, no
  proposal. The developer's message was an unambiguous instruction ("3つとも
  終わらせてほしい"), so reading it as permission already given is defensible —
  but it is a judgement Front made, not a rule it followed.
- **It answered "はい、コミットしてください" three times on the developer's
  behalf**, without asking. Same reasoning, larger consequence: those answers
  are what put three commits in `main`. Worth deciding deliberately rather
  than discovering again — an approval a supervisor may give (commit this)
  versus one it must escalate (change the plan) is not written down anywhere
  yet.

Note also that the agreement and the go-ahead collapsed into one message:
autolab's supercoder read "はい、コミットしてください" as the agreement that the
task was done, wrote `report.md` and closed out. Front never had to say a
separate "this is complete". The contract held, but it was satisfied
incidentally.

## Attributability (p2/p4's criterion, still holding)

```text
$ grep -rn "autolab" agfront/src agfront/agent                          -> exit 1
$ grep -rn "workrun-" agfront/src agfront/agent                         -> exit 1
$ grep -rn "work-s2-6\|pj-simpleshooter\|autolab-agstudio1" \
      agfront/src agfront/agent                                         -> exit 1
```

Everything Front knew about where to write and what closing a task requires
came from `tools/agents.md`, harvested from `#agents` seconds before each run
— message 725, the introduction rewritten in step 2. The harvested file in
`front-p5-supervise-workruns/1/front/tools/agents.md` contains the section
heading *"While a task runs, somebody has to be there"*, which is how Front
knew its agreement was required at all.

## Cost

Front `run-0018` $0.386 + `run-0019` $0.866; autolab supercoder runs
$0.127–$1.276 each, eight of them. Roughly $5 of agent time for three
supervised development tasks.

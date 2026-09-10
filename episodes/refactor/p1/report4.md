# refactor p1 step 4 — deployed, demonstrated, reported

The phase's own conclusions are in [`report.md`](report.md). This is the step
record: what was deployed, what the live run did, and the one defect it found.

## Service state before deploying

Read through `nctl` rather than assumed:

```
$ uv run --project nctl nctl status
✓ nautobot   3.1.3, authenticated, intent_catalog + intent_graphql
✓ worker     celery workers: 1, pending jobs: 0
✓ dumps      /var/lib/nodeutils (8 hosts, agstudio collected 0.9h ago)
✓ submodule  ansible_agdev / nauto / nctl / nintent / nodeutils all clean

$ uv run --project nctl nctl workspaces
agautolab-agstudio @agstudio  present  matched  active_development  fresh
agdevworld         @agstudio  present  matched  idle                fresh
… summary: converged=9
```

`agautolab-agstudio` is the checkout the listener actually runs from
(`WorkingDirectory` in `com.agdev.agautolab-zulip.plist`), so deploying is
`uv sync` plus a restart rather than a copy.

## What was deployed

| what | how |
|---|---|
| pyagag `218cb31` | pushed; `uv.lock` re-pinned in agautolab |
| autolab listener | `launchctl kickstart -k gui/$(id -u)/com.agdev.agautolab-zulip` — came up clean, `0 awaiting, 55 calls spent` |
| agentroom relay | `uv sync` + kickstart; `/healthz` `{"ok": true}` |
| agdevworld frontend | `npm run build && docker compose build web && docker compose up -d web` — `:8090` answers 200 |
| autolab's introduction | `uv run python -m agautolab.intro` — `#agents` › `intro-autolab-agstudio1`, stamped `Revision: 83dbcc2` |

The introduction matters because a stale one is acted on as if it were
current, and this phase changed what autolab does when a plan is wrong.

## Plane unavailability, verified on the deployed venv

The plan asked for a client that rejects calls *or equivalent isolation*.
The stronger property is available and was checked instead — the deployed
package cannot call Plane because it never loads it:

```
$ uv run python -c "import agautolab.zulip_listener, agautolab.mission_done,
  agautolab.project_init, agautolab.cli, sys;
  print([m for m in sys.modules if m.startswith('agag.plane')])"
[]
```

## The demonstration

`#pj-refactorp1`, a channel and Gitea project both created for this run, and
a two-task request for a `wordcount.py` tool. Every message below is a post a
human would have made; nothing was driven through a file or an API.

1. **Plan.** `workplan-wordcount` → mission **m5702**. The plan is message
   5703 in that topic; `[selfnote][mission]` is message 5702, and *that id is
   the mission*. `work-m5702` opened with `workrun-task1-m5702` and
   `workrun-task2-m5702`.
2. **Start.** "The plan looks right. Please start the mission." →
   `[state] started`.
3. **Execute task 1.** The supercoder wrote `main/wordcount.py`, asked for
   confirmation, and on "I agree this task is done" posted its `## Result`,
   marked the task `completed`, resolved the topic, pushed `main` to Gitea
   (`628a7c4`) and filed the report in `devlog`.
4. **Replace.** "The request was wrong… scrap this plan and replace it."
   `replace.flag` + a new `plan.md` in one run:

   ```
   mission m5702 is retired at pj-refactorp1/✔ retired-workplan-wordcount-m5702
   and its work channel work-m5702 is archived; this topic is now mission
   m5735, which carries forward 1 finished task(s) and drops 1 unfinished one(s)
   ```

   The visible post that opened m5735 named task 1 as **carried forward** by
   reference (`work-m5702/workrun-task1-m5702`) and task 2 as **dropped**.
5. **The gate.** Posting in the replacement's task 2 before task 1 answered
   `Please complete previous work (task 1 of m5735)` — decided handler-side,
   with no agent run bought.
6. **Resume.** Task 1 of m5735 added `--json` (`e0f44b7`); task 2 added its
   test (`26e2a3b`). Both committed, pushed and filed in `devlog`.
7. **Accept.** The operation room's preview, with no Plane row and no gap:

   ```
   ready  work         work:m5735                     every one of its 2 tasks is finished;
                                                      accepting them and marking the mission done
   done   topic        …/workrun-task1-m5735          already ✔
   done   topic        …/workrun-task2-m5735          already ✔
   ready  channel      channel:work-m5735             will be archived — every one of its 2 topics …
   ready  conversation …/workplan-wordcount           will be marked ✔ once everything above is done
   counts {'ready': 3, 'blocked': 0, 'done': 2, 'kept': 0}   excluded []   gaps.plane []
   ```

   and the result: `m5735 is done; accepted m5735#1, m5735#2`, the channel
   archived, the conversation ✔.

The scope line the preview showed is the p2 relationship, read from the
realm:

> …an Autolab request (mission m5735), its tasks and its work channel; **it
> replaced mission m5702, which is retired and is not touched by finishing
> this one**

### Read back through the anchors, after everything closed

```
m5702: state='replaced' at pj-refactorp1/'retired-workplan-wordcount-m5702'
       replaces=None  title='Add a wordcount command-line tool'
m5735: state='done'     at pj-refactorp1/'workplan-wordcount'
       replaces=5702   title='Add a --json flag to wordcount.py'
```

Two missions, one display name between them, told apart by nothing but their
anchor ids — which is the property the whole phase rests on, seen live rather
than in a fixture.

Git holds the durable half: `main` at `26e2a3b` (pushed to Gitea), and three
task records under `devlog/m5702-…` and `devlog/m5735-…`.

**Cost**: 10 agent runs, ~242 s of run time, **$1.23**.

## The defect it found

Accepting m5735 wrote its two state notes and closed everything, and then
reported `partial: true`. The next preview read the finished mission as
`it has no task, so there is nothing that could have finished`.

The room accepts with the **Developer's** credential — that is what
acceptance is — and both readers took a conversation's state notes only from
the record's own author, so the room's own write was invisible to the record
it was written into. `accepted` and `done` are read from any sender now
(`EXTERNAL_STATES`); every other state stays the agent's own report of its
own work, so nobody can claim `completed` for work they did not do.

Neither fixture caught it because both wrote the acceptance *as autolab*. The
room's fixture writes as the Developer now, and six tests fail without the
fix. Re-verified live after redeploying:

```
counts {'ready': 0, 'blocked': 0, 'done': 2, 'kept': 0}
done  work          work:m5735                    already done; closing it again changes nothing
done  conversation  …/workplan-wordcount          already ✔
```

## Tests and build

```
pyagag      543 passed
agautolab   237 passed   (+2: acceptance written by another sender)
agentroom   236 passed   (+1: the acceptance round-trip, with the fixture writing as the Developer)
agdevworld  tsc clean; vite build ok; web container rebuilt
```

The controlled tests for the late reply, the deleted origin and restart
timing are `agautolab/tests/test_replacement.py` (step 2) and
`agentroom/tests/test_scope.py` (step 3). They are fixtures, not this live
run, and are reported as such.

## What was left alone

- In-flight work in `pj-studyrealworld` from before the deployment
  (`work-s4-11`, `work-s4-13`) uses the old Plane-labelled shape and is not
  readable by this code. Nothing was migrated and nothing was disturbed.
- Front supervision and forge delegation were not added to the live scenario,
  as the plan allowed.
- `agautolab1` was not updated; it runs no listener.

## Documentation updated

`pj-agdev/.local/devenv.md` (ignored) now describes the record's shape, the
`replace.flag` workflow, the import-graph check, the acceptance-credential
rule, and what `AGENTROOM_PLANE_ENV` is still for.
`agautolab/params/intro.md` and the superdirector guide were updated in
steps 2–3 and the introduction was re-posted here.

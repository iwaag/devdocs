# scheduled_routine p3 — Step 3 report

Executed 2026-08-25 UTC. The two-hour horizons used an accelerated dispatcher
clock after the Developer asked not to spend two wall-clock hours on the test.
Each tick was the production dispatcher with `--now <event-at>`: it still
marked, committed, pushed, and posted through Zulip, but the three 30-minute
events were invoked about one minute apart. This proves event selection and
the end-to-end route, not wall-clock launchd accuracy; the five-minute
launchd job continued making ordinary no-due ticks alongside it.

## Successful variant: r4

Request, verbatim: “Run rtnotes once; if it succeeded apart from minor
errors, repeat imgprompt every 30 minutes for the rest of two hours.”

### Schedule and A

- 03:03:53Z real: request posted. One Front run created `r4` (until
  05:04:04Z), `e14` fire rtnotes at 03:05:04Z, and `e15` decide at
  03:10:04Z. Commits: `6e66db6`, `1bb674b`, `0cfb8df`.
- Front placed decide five logical minutes after A. That was too soon for the
  normal rtnotes mission. I deliberately ticked both immediately to exercise
  the “still running” case: `e14` was marked at `a9eed90`, `e15` at
  `36656b4`.
- Front did **not** move `e15` or add a replacement decide. Because a fired
  decide cannot be moved by the CLI, it monitored in the serving instead. It
  found the bound `work-r3-30/workrun-task1-r3-30`, posted the start there,
  and thereby anchored that topic back to `front-schedule`; autolab's later
  callback re-served the decision conversation. This worked, but was a long
  Front serving rather than the expected flat-event self-correction.
- A finished clean: `python3 rtnotes.py check` exit 0, all tests pass, no fix;
  autolab committed its routine note as `837052d`. Front judged from the
  autolab workrun topic, not from a repository or Plane, and resolved e15
  “yes.”
- 03:08:43Z real: Front added imgprompt `e16` 03:40:04Z, `e17` 04:10:04Z,
  and `e18` 04:40:04Z (`19b3c92`, `ae50e97`, `52653cb`). All are inside the
  r4 hard guard.

### Accelerated B timeline and overlap

| logical tick | real Zulip post | marker commit | result |
|---|---|---|---|
| e16 03:40:04Z | 03:08:57Z | `4fdb30e` | Front opened `assetplan-desert-canyon` |
| e17 04:10:04Z | 03:09:21Z | `e6055ce` | same standing run; started its one bound forge Work |
| e18 04:40:04Z | 03:10:08Z | `eb5cef4` | generation already running; added no second Work |

Forge completed Work F2-26 at 03:12:42Z with four files; Front reported the
delivery at 03:12:58Z. Two fires overlapped the same in-flight imgprompt run.
Front coalesced all three into one Work: the second started it and the third
waited. No duplicate generation and no loop occurred.

Front runs attributable to r4:

- `front-schedule`: 3 (create schedule; early decide/monitor; evidence
  callback and three-fire expansion).
- `front-routine-rtnotes`: 2 (start/delegate and completion).
- `front-routine-imgprompt`: 5 (initial request, forge plan callback plus
  overlapping post reprocessing, one no-result response, final delivery).

## Failing variants

### First fixture: defensible but wrong for the planned assertion (r5)

I first pre-seeded `check()` with an explicitly named, do-not-repair p3
fixture. `e19` and `e20` were marked at `327b8b6` and `49dff08`. Autolab
reported exit 1, the fixture FAIL, and a cascading self-test FAIL, but no
unrelated breakage. Front classified both as consequences of the known
fixture, said “yes,” and added `e21`–`e23` (`eb7bfb5`–`8579b30`).

That did not prove the plan's expected “added nothing.” The Developer
corrected the judgement in `front-schedule`; one Front run acknowledged that
exit 1 with failing tests was not success and removed all three events
(`6560fca`, `aa7562d`, `67cabac`). This is kept as evidence: the initial
failure fixture made “minor” too plausible.

### Actual broken check, diagnosis-only run (r6)

The fixture was restored. For one trigger only, the standing rtnotes text was
changed to diagnose without repairing or committing; `greeting()` was then
changed to return `broken, …`. A local preflight proved exit 1 with
`FAIL: greeting is broken` and two failing self-tests. The same conditional
schedule request produced `r6`, `e21`, and stricter decide `e22`; marker
commits were `4136906` and `48527e3`.

Autolab, in `work-r3-34/workrun-task1-r3-34`, reported:

```text
FAIL: greeting is broken
FAIL: test_rtnotes.py failed:
Ran 9 tests ... FAILED (failures=2)
exit code 1
```

It made no code change or commit. Front initially stopped because the
one-time Developer definition was in `routine-rtnotes`, outside the
`front-schedule` chatlog, and it mistook its own resulting workplan post for
a spoof. After the Developer pointed it to legitimate message 2159, Front
verified that topic, judged e22 **no**, and added nothing. It explicitly cited
the new `greeting is broken` failure and the strict e22 wording.

The normal standing text was then re-posted as the latest definition,
`greeting()` was restored, and both `rtnotes.py check` and all 9 direct tests
passed. r6 used 3 `front-schedule` runs (create, stop/ask, evidence-backed
resolution) and 3 `front-routine-rtnotes` runs (start, plan callback, work
callback). It required one Developer clarification because of the false
spoof alarm.

## Traps and counts

- Rename-blind reads: **0** in Front's observed reads. The resolved work topics
  were successfully read through their `✔ ` names using the instructed paths.
- Orphaned acknowledgements: **0**. No listener restart occurred during these
  tests and every ack acquired a later answer.
- Decide fired before A finished: **3** (r4, r5, r6). None was moved; each
  used a long monitoring/agent exchange instead. That is evidence against the
  expected “decide moved” behavior.
- Cross-topic provenance false alarm: **1** (r6 standing text mistaken for a
  spoof), requiring one Developer reply.
- Wrong “minor errors” judgement: **1** (r5), corrected with three event
  removals before any could fire.
- B overlap: **2** extra fires while one forge Work was in flight; both were
  safely coalesced, with no duplicate Work.

Project judgement stayed with autolab; Front never opened rtnotes files or
Plane. Cluster judgement was not needed, so cagent/nctl was not invoked by
Front. The Omni Agent did the temporary break/restore for the failing fixture
— handoff candidate.

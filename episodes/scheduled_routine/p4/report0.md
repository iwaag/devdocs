# p4 step 0 — two known gaps

Executed 2026-08-25 UTC on `agstudio`. Both gaps are closed; the tests and the
live verification are below. One accident happened during the commit of this
step and is reported at the end, because it is the most important thing in it.

## 1. Pushing a routine project's `main`

### The choice

**The close-out pushes.** `serve_run` publishes `main` right before it writes
the devlog record, in the same close-out that already exists.

The alternative was to have the supercoder push after its own commit. It was
rejected for the reason the devlog record was made deterministic in p2: the
agent is never asked to run git, so what reaches Gitea cannot depend on the
agent remembering to do it, and a run that dies after committing still gets its
work published by the next close-out. The push is not a decision — *what* to
commit is the agent's decision, and that part is untouched.

Consequently `push_main_repository` commits nothing at all. It fetches, counts
`origin/main..HEAD`, and pushes only if that count is non-zero. A clone already
level costs no push, and the fetch is what keeps the count honest if somebody
else pushed meanwhile.

### The diff

`agautolab` `41ea549` (pushed to GitHub; `pj-agdev` pin bumped in `1bec975`):

- `project_init.py` — `push_main_repository(config, workspace) -> int`, beside
  `commit_all_and_push`.
- `zulip_listener.py` — `push_main(slug)`, returning one line for the reply:
  `pushed main to Gitea (N commits)` / `main was already level with Gitea` /
  `main is not a repository; nothing pushed`. Called from `serve_run`'s
  close-out, so only a run that produced `report.md` publishes.

### The tests

`tests/test_project_init.py`
- `test_push_main_repository_publishes_what_the_run_committed` — fetch,
  `rev-list origin/main..HEAD`, push, and **no `add`/`commit` at all**.
- `test_push_main_repository_pushes_nothing_when_the_clone_is_level`.

`tests/test_zulip_listener.py` (beside the devlog tests, wired through the same
`wire_run` fixture, which now also lays down `main/.git`)
- `test_the_close_out_publishes_main`
- `test_a_main_only_project_publishes_main_too` — the layout this routine uses:
  the devlog is local, so `main` is the whole visible record.
- `test_a_run_that_committed_nothing_pushes_nothing`
- `test_an_unfinished_run_publishes_nothing` — no report, no close-out, no push.

`uv run pytest` in `agautolab`: **180 passed**.

### Live verification on `rtnotes`

`rtnotes`' `main` stood **17 commits** ahead of `autodev/rtnotes` — p2 recorded
2, and p3's eleven-fire cadence added the rest. Run through the new code path,
not by hand:

```
$ uv run python -c "from agautolab.zulip_listener import push_main; print(push_main('rtnotes'))"
pushed main to Gitea (17 commits)
$ ... again
main was already level with Gitea
```

`autodev/rtnotes` `main` after the push (`origin/main`, newest first):

```
4538713 Restore rtnotes after scheduled_routine p3 failure fixture
ca2d94c check(): keep scheduled_routine p3 failure fixture; log run in NOTES.md
837052d Append 2026-08-26 00:01Z NOTES.md entry: check reran clean, no fix needed
89f6b27 Append 2026-08-25 23:59Z status note: check reran clean, no fix needed
c6ed7c5 check(): show snippet of failed self-test output
18ebc04 check(): shell out to test_rtnotes.py, fold failure into problems
821506a Append 2026-08-25 23:14Z NOTES.md entry
f1d87b1 Append 2026-08-25 21:14Z NOTES.md entry after re-running checks
996bb70 Add automated coverage for HEADING/check() suffix enforcement
fda70e8 Append 17:14Z status note; check passed clean, no fixes needed
6cafed9 Append 15:15Z status note; check passed clean, no fixes needed
4d66875 check: flag malformed same-day heading suffixes as FAIL
63a1468 check: flag same-day heading missing HH:MMZ suffix as FAIL
ec60cfd NOTES.md: document one-note-per-run policy, add 09:14Z entry
22be395 Add 07:14Z status note; check passed clean, no fixes needed
2783cdb Append 03:02Z status note after re-running rtnotes.py check
caccf57 Append 2026-08-24 status note: check passed clean
59cd458 Seed: rtnotes.py check, NOTES.md    <- where origin/main stood before
2c3b1e6 Ignore .local/
```

`ahead` is now 0. The `com.agdev.agautolab-zulip` listener was restarted on the
new code (`launchctl kickstart -k`, PID 31667) before Step 1 begins, so the
first `papers` run publishes through this path.

## 2. Real fire time vs logical tick

### The diff

`pj-agdev` `1bec975`:

- `dispatch.py` — `dispatch_schedule` takes `real_now`. `now` stays the tick's
  clock (what is due, what has expired); `fired_at` becomes `format_time(real_now
  or now)`, and when `real_now` is given the tick is also stored as
  `logical_at`. `main()` passes `real_now=utc_now()` **only** when `--now` was
  given, so production has one clock and writes no `logical_at` at all.
  `load_schedule` validates `logical_at` when present.
- `rtschedule` CLI untouched, as the plan required.
- `autodev/rtschedule` `bf2fe2c` — the GUI prints the fire time after the
  badge, and the logical tick next to it in the decide colour when present.
  It said only "fired" before, so neither time was visible at all.

### The tests

`devenv/routine/tests/test_dispatch.py`, **6 passed**:

- `test_a_logical_tick_records_the_real_time_and_the_logical_one` — the new
  fixture: a logical day-5 tick dispatched today writes `fired_at` = today and
  `logical_at` = day 5, and the result still validates.
- `test_a_real_tick_records_no_logical_time` — production writes no extra key.
- the three p3 fixtures are unchanged and still pass, which is the check that
  the production path did not move.

### One `--now` tick showing both fields

Run through `main()` — the production entry point — against a scratch clone
with a no-op trigger, so nothing was posted to Zulip:

```
$ python3 devenv/routine/dispatch.py --repo <scratch> --now 2026-08-30T09:00:00Z
2026-08-30T09:00:00Z marked e1 before action
2026-08-30T09:00:00Z dispatched and pushed e1
```

```json
{
  "id": "e1",
  "at": "2026-08-30T09:00:00Z",
  "kind": "fire",
  "routine": "papers",
  "from": "r1",
  "fired_at": "2026-08-25T04:44:36Z",
  "logical_at": "2026-08-30T09:00:00Z"
}
```

Real time at that moment was `2026-08-25T04:44:36Z`. Before this change
`fired_at` would have read `2026-08-30T09:00:00Z` — five days in the future,
which is exactly what p3's report4 saw in the GUI.

The console log prefix still prints the **logical** time, deliberately: it is
what identifies the tick when replaying a sitting. The durable record carries
both.

## Note for the phase report

The GUI's timeline window is *past 7 days and next 24 hours*. Step 2 writes
seven logical days of events starting tomorrow, so six of those days fall
outside the window and the "GUI shows 7 days × (fire, decide)" check cannot be
made as written. Not touched here; taken up in Step 2.

## Incident — `.local/` was pushed to a public GitHub repository

Committing this step, the Omni Agent appended `__pycache__/` to
`pj-agdev/.gitignore`. That file had no trailing newline and one line, so the
append **fused** into it and the whole ignore file became
`.local/__pycache__/`. The following `git add -A` therefore committed all of
`.local/`, and it was pushed to `github.com/iwaag/pj-agdev` (public) as
`f625bda`.

Exposed for roughly four minutes on `main`: `.local/.env`, `.local/devenv.md`,
`.local/plane-credentials.env`, `.local/plane/*.env`,
`.local/plane-selfhost/plane-app/plane.env`, and every file under
`.local/zulip/` — `developer.env`, `developer.password`, `front.env`,
`forge.env`, `omni-agent.env`, `provisioner.env`, `autolab-*.env`,
`cagent.env`, `agecho*.env`.

Remediation done, with the Developer's permission for the force push:

- history rewritten; `main` is `1bec975`, whose tree contains **no** `.local/`
  path (`git ls-tree -r origin/main | grep -c '^\.local/'` → 0);
- `.gitignore` is now `.local/` + `__pycache__/`, with a trailing newline, so
  an appended line cannot fuse onto `.local/` again;
- the repository has 0 forks.

**Still open, and the Developer's call:** `f625bda` remains fetchable from
GitHub by SHA (`gh api repos/iwaag/pj-agdev/commits/f625bda` still answers).
Removing it needs a GitHub Support GC request. Independently, every credential
listed above should be treated as disclosed and rotated — the Zulip and Plane
services are LAN-only, which limits the practical reach of the tokens, but
`.local/zulip/developer.password` is a password and is the one that does not
depend on LAN access to matter.

This is an Omni Agent accident, not an in-system agent finding, and it is
recorded here rather than in the phase report's feature list.

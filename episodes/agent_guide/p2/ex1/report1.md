# agent_guide p2 ex1 — step 1: every trial driver is versioned

pyagag `2b18e66`; agfront `dc4c721`, agautolab `7784286`, archsage `ab89e11`,
pj-agdev `2fa1dda` (agobserver).

## Where the drivers are

| agent | driver | probes |
|---|---|---|
| Front | `agfront.trial` (rewritten on the kit) | `aisvgs-sufficient`, `growbox-thing`, `forge-protoprey`, `receipt-owed`, `hold-release` |
| archsage | `archsage.trial` | `archsage-loose-sage` |
| autolab | `agautolab.trial` | `planner-other-project`, `entrance-plans` |
| Observer | `agobserver.trial` | `triage-unopened` |

Each is run from the agent's own checkout with its own venv:

```
cd <agent checkout> && .venv/bin/python -m <agent>.trial <probe> --out <dir> \
    [--store <mirror.sqlite>] [--guides <tree> | --guides-rev <commit>] [--no-shared] [--dry-run] [--records <dir>]
```

`python -m agag.fixture probes` now prints, under each probe, the line that
runs it. That is the "one entry point" the plan called a convenience: a
real single entry point would have to know where each agent's checkout and
venv are, which pyagag does not, so it names the command instead.

## The kit (pyagag `agag.fixture.run`)

What `agfront.trial` and the ignored `probe_others.py` each had, written once:

- `newest`, `tool_calls`, `session_log`, `record_facts`: the newest file,
  the tool calls of a Claude Code session log or stream transcript, where
  Claude Code keeps a run's session, and a run record's cost, turns,
  duration and model.
- `trial_parser(prog, doc, agent)`: the command line every driver takes;
  its probe choices are the ones `probes.py` gives that agent.
- `Trial.start(args, repository)`: the board (built fresh as
  `<out>/board/mirror.sqlite` unless `--store` names one, so a trial needs
  no board kept anywhere), the guide tree, the records root (step 2).
- `Trial.session()`: what a serving runs inside: the fixture environment,
  `--no-shared` (pyagag's `shared_sections` answers nothing), and
  `--dry-run`, which replaces `agag.agent.run_harness` for the duration: the
  prompt is written to `<out>/prompt.md` and a marked "(dry run)" reply
  comes back, so the serving completes through the agent's own code. The
  old Front-only dry run replaced `run_front`; this one works for every
  agent because every agent starts runs through `run_role`.
- `Trial.finish(output, records=…, calls=…)`: judges the reply and writes
  `outcome.json` and `reply.md` with the trial's own facts (guides, the
  revision, shared or not, store, records root).

The drivers keep only what is the agent's: which serving function a probe
goes through, where its guide tree is patched (`zulip_listener.GUIDES`,
`roles.GUIDES`, `triage.GUIDES`, the entrance's `entrance_guide`), and where
its session log is.

## Baselines are commits

`--guides-rev <commit>` extracts the checkout's `agent/guides` at that
commit with `git archive` into `<out>/guides@<commit>/` (with a `REVISION`
file) and serves it. A tree from before agent_guide p1 has no `shared/`
directory; `agfront.trial` then reads no shared files, as that composition
did.

| baseline | agent | revision | notes |
|---|---|---|---|
| p1's composition (p2's "old") | Front | `447bb03` + `--no-shared` | the tree that was `.local/agp2/trials/p1-guides/` |
| before p1 | Front | `ba28e90^` (`1ef83f5`) + `--no-shared` | no `shared/` files |
| before p2 | archsage | `df6e0af` + `--no-shared` | the tip before `agent-guide-p2` fast-forwarded in |
| before p2 | autolab | `aab6810` + `--no-shared` | same |
| before p2 | Observer | pj-agdev `56c6fc3` | triage's guide was not changed by p2 |

Dry runs of the #15673 probe (`aisvgs-sufficient`) give the composition
sizes p2's report measured, so the extracted tree is the one p2 served:

| composition | prompt characters |
|---|---|
| new guides, shared sections | 20 630 (p2: 20 630) |
| `--guides-rev 447bb03 --no-shared` | 19 534 (p2: 19 534) |
| `--guides-rev ba28e90^ --no-shared` | 27 255 |

archsage's dry run is 17 953 characters new and 16 778 at `df6e0af`
without shared sections: 1 175 apart, p2's size table's 10 114 → 11 289.

The ignored `.local/agp2/trials/probe_others.py` and `p1-guides/` are
superseded; they are left where they are with p2's outcomes.

## Tests

- pyagag `tests/test_trial_kit.py` (6): a revision's guide tree is that
  commit's and an unknown revision is refused; the parser offers only the
  agent's probes and refuses `--guides` with `--guides-rev`; a trial builds
  its board under `--out`; a dry session runs no harness, writes the
  prompt and drops the shared sections only inside the session; tool calls
  from a session log. **pyagag 1163 passed.**
- Each driver has a dry-run test (`tests/test_trial.py`; agfront's also
  serves the `447bb03` baseline and checks p2's callback section is absent).
  agfront's full suite **197 passed**; agautolab 332, agobserver 177 and
  archsage 40 passed before their driver tests were added, which then
  passed on their own (2, 1, 1). Step 5 re-runs every suite.

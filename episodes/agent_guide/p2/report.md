# agent_guide p2 — report

## Outcome

The other agents' guides now open the way p1's Front guides do — who the
role is, whom it speaks with, what is assumed of it, what it can see — and
text more than one agent needs is written once, in pyagag. Runs read the
board from their listener's mirror, several conversations in one call.
And a guide change can now be tried against **the same board every time**:
a synthetic fixture board that a real run reads and cannot post to.

- Every fixture probe passed with the new guides: Front's three board
  probes and the staged receipt and hold cases, archsage asked about a sage
  by a loose name, autolab's planner asked about another project, autolab's
  entrance, Observer's triage.
- Against p1's guides on the same board, Front's probes passed too; the new
  guide took the direct path more often (grow box: 9 turns against 16).
- The fixture reproduces p1's content difference on demand: with either
  guide, the #15673 question is answered from the study channel and misses
  the Developer's decision recorded only in archsage's channel (p1's open
  finding 1, held by the Developer).
- One live failure, not reproducible: a Front serving answered in one turn
  that it had released a hold and recorded a disposition, and had done
  neither (run-0183). The proxy's correction brought the records. 12 fixture
  runs of the same conversation recorded both.
- One CLI defect found live and fixed: `agentchat hold/release/disposition/
  relation` refused the form their own help shows.

Step reports: `report1.md` (inventory), `report2.md` (help texts),
`report3.md` (shared text), `report4.md` (guides), `report5.md` (agentchat
frictions, mirror reads), `report6.md` (the fixture), `report7.md` (trials
and roll-out).

## What moved where

| from | to |
|---|---|
| the `agrefs` pointer, copied into 13 guides of 5 agents in two wordings | pyagag `agag/guides/refs.md`, appended by `prompt_with_guide(shared=("refs",))` and by default for argue participants; each guide keeps its follow-up (autolab's `direction/REFERENCES.md`, forge's `--init-image`, archsage's "a reference is no finding", Observer's `run` tool) |
| agfront `shared/board.md`'s board facts; `work.md`'s "each serving ends" and ✔ sentence | pyagag `board.md` (with as9's and fd-wr's references) and `callback.md` (as5-8); agfront keeps the developer's assumption, the working directory and the chatlog format |
| the entrance's fixed half, in pyagag's `DEFAULT_GUIDE` string and copied into autolab's and forge's entrance guides | pyagag `entrance.md` (with as10's reference); autolab's `entrance_front/guide.md` is only its vocabulary; forge's is deleted (it was the default vocabulary word for word) |
| `participant_guide()`, a Python string | pyagag `argue_participant.md` |
| tool usage in archsage's, sage's, forge's and Observer's guides | `archsage` and `sagetree` subcommand helps, `agroutine` subcommands, `agforge --help` / `knowledge list` / `--init-image`, `mission_done`, `agag.health`, `agentchat read --help` |
| archsage's, autolab's planner/worker, forge's plan front/planner/run guides | rewritten with a head; every trial paragraph kept word for word (trend7, fs3p, smoke, fs4r, sub26, fs2T1, fs5-1, fs5E) |
| autolab's `autolab-front/guide.md` | deleted: no code reads it |

After the phase no line longer than 40 characters is repeated across the
guides of all six agents and pyagag's files.

## Sizes

Characters of what a role is told besides placement and conversation (own
guide files + shared sections; reply and continuation sections unchanged).
Full table in `report4.md`.

| role | before | after | why |
|---|---|---|---|
| Front desk / front / routine_run / argue | 14 077 / 13 906 / 15 949 / 9 688 | 15 173 / 15 002 / 17 045 / 10 723 | the moved board and callback text carries the trial references and headings the agfront copy lacked |
| archsage | 10 114 | 11 289 (own guide 8 834) | gains the board and the callback, which it lacked; its own text shrank |
| sage | 1 837 | 1 430 | usage moved to `sagetree --help` |
| autolab planner / worker | 6 325 / 5 304 | 7 949 / 7 650 | gain a head and the board (neither was told another project's state is on the board), the worker the callback |
| forge planner | 2 821 | 2 639 | usage moved to `agforge --help` |
| entrances (autolab / forge) | 1 441 / 944 | 1 846 / 1 120 | the shared fixed half with as10's trial reference |
| Observer triage / observe | 2 344 / 3 830 | 2 344 / 3 795 | unchanged roles |

The desk's composed prompt for the same fixture question: 19 534 characters
with p1's guides, 20 630 with p2's.

## The fixture, and how to run it

A synthetic board built in code (`agag.fixture`), written as a mirror store
marked `fixture`: studies with rounds and a decision in archsage's channel,
a routine with runs, forge's deliveries and a non-delivery, every agent's
introduction, a ✔'d past request, an owed answer with Observer's request,
and a hold to release. A run's `agentchat` and `agproject status` read it
through `AGENTCHAT_MIRROR`; every write is refused. Probes and pass rules
are in `agag.fixture.probes`; rules can judge the reply or the run's tool
calls, and can *observe* a fact outside the rule.

```
pyagag/.venv/bin/python -m agag.fixture build <dir>
pyagag/.venv/bin/python -m agag.fixture probes
cd pj-agdev/agfront && .venv/bin/python -m agfront.trial <probe> --store <dir>/mirror.sqlite --out <out> \
    [--guides <p1 guide tree> --no-shared] [--dry-run]
```

The other agents' drivers are in the host's trial kit
(`pj-agdev/.local/agp2/trials/probe_others.py`, see devenv). README_DEV
*How a guide is put together* has the same instructions.

## Trials

| trial | runs | result | cost |
|---|---|---|---|
| Front's three board probes, p1's guide / p2's guide | fixture | 3/3 / 3/3 (the #15673 decision fact missed by both) | $0.60 / $0.54 |
| receipt repair when Observer asks (staged `exit-before-receipt`) | fixture | old pass, new pass (`receipt` → `--repair --because`) | $0.39 |
| hold released and request ended, ×6 per guide | fixture | 12/12 | $2.35 |
| archsage, loose sage name | fixture | pass | $1.15 |
| autolab planner, another project's state | fixture | pass | $0.28 |
| autolab entrance, all plans | fixture | pass | $0.35 |
| Observer triage, a request it did not open | fixture | pass (`legit`) | local model |
| live hold placed and released (`front-desk-20260928-agp2-hold`) | run-0182…0184 | placed ✓; release **claimed, not recorded** (run-0183, one turn); recorded after the proxy's correction (#15849, #15850) | $0.42 |

Total about $6.1. Details and the moved-paragraph coverage table are in
`report7.md`.

## Paragraphs dropped as Anxiety-Driven or stale

| text | where | why | restore from |
|---|---|---|---|
| "Your reply is the closing message of this run and is posted for you." | archsage | REPLY_GUIDE says it in the same prompt | archsage `df6e0af` |
| "Keep it well under 10,000 characters" (a study routine's guide) | archsage | Zulip's limit before failsafe p4 (100 000 now); `agroutine`'s read-back stays | archsage `df6e0af` |
| "your reply … is posted for you" (sage) | sage | REPLY_GUIDE says it | archsage `df6e0af` |
| `autolab-front/guide.md` | autolab | read by no code | agautolab `aab6810` |
| forge's `entrance_front/guide.md` | forge | identical to pyagag's default vocabulary | agforge `0558c6e` |

No Evidence-Driven paragraph was dropped.

## What the phase learned

- **A reply can describe records it never made**, in one turn with no tool
  call, beside a guide that says saying it records nothing (run-0183). It
  happened once in 13 servings of that conversation and never on the
  fixture. A sentence cannot be shown to help against a 1-in-13 event; a
  check can. See the handoff candidates.
- **A fixture turns "did not reproduce" into a measurement.** p1 could
  not repeat run-0160's miss; p2's fixture repeats p1's content difference
  every time, and repeated the hold conversation twelve times to show the
  live failure was not the guide.
- **A pass rule is a guide too, and has its own anxiety.** The first
  #15673 rule passed a reply on a word that happened to match; rules now
  separate required facts from observed ones, and acts are judged on tool
  calls.
- **Help examples must run.** The `hold` help showed a form argparse
  refused; the live run spent a turn on it. Tests now run the examples'
  shape.
- Runs still try shell loops over topics (the entrance did); with `read`
  taking several topics, they recover by reading `read --help`.

## Open findings

1. **False record claims** (run-0183): nothing in the system notices a
   reply that says a record was written when none was. Observer sees only
   the records (here: a hold still in force, so it would never ask).
2. p1's open finding 1 stands and is now reproducible: a decision about a
   study made in archsage's channel (or a desk conversation) is invisible
   from the study's channel. Probe `aisvgs-sufficient` observes it.
3. `agproject status` on the fixture still asks the host's Gitea about the
   repository; a fixture study whose name matches a real one gets real
   Gitea facts.
4. Trials run from the main checkouts write their topic workspaces and run
   records under that checkout's `.local/`, where the relay's cost gauge
   reads run records. The p2 trials ran in worktrees; later trials should
   too, or the kit should get its own records root.
5. comfynotify publishes no introduction on the board: autolab's worker
   guide is the only description of its `watch` command.

## Documents updated

- `devdocs/README_DEV.md`: *How a guide is put together* gains *Text
  several agents share*, *What a run reads the board through*, *Trying a
  guide change against the same board*.
- `devpolicy/styles.md` § *Agent guides*: text shared across agents lives
  in the shared library; a file-answering role keeps its output contract.

## Deus Ex Machina notes, and handoff candidates

- The Omni Agent posted as the Developer's proxy (the hold trial) and
  **noticed** that run-0183's claimed records did not exist, then asked
  Front to record them (#15847). Front wrote every record itself. Noticing
  was Omni's. **Handoff candidate**: a check owned in the system — the
  listener comparing a reply's claims of `hold`/`release`/`disposition`/
  `accept` against the records the serving wrote, or Observer asking when
  a reply says a request ended and no disposition exists.
- Omni ran no command on an in-system agent's behalf and wrote no record.
- **Handoff candidate**: an introduction for the Comfy Notifier, so the
  `watch` command is learnable from the board rather than from one
  agent's guide.

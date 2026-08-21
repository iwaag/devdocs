# p10 Step 2 — `python -m agautolab.mission_done`

The entrance's one Plane operation. Everything it reads is in Zulip; the
mission Work's own state is the one thing that is not, so the one thing it
writes is that.

## Why it is a command and not a prompt

A task's Sub-Work is closed by the run that executed it (`report_work`).
Nothing has ever closed the Work above them — p9 finished a four-task mission
and left `S2-30` in `unstarted` with four completed children. Deciding "every
child is Done" is counting, so it is code.

## What it does

```
uv run python -m agautolab.mission_done            # sweep every [AUTO] project
uv run python -m agautolab.mission_done S2-30      # one Work, by label or id
uv run python -m agautolab.mission_done --dry-run  # say, move nothing
```

A Work is moved when **all** of these hold: `external_source` is `agautolab`
(a hand-made issue is not this command's business), it has at least one live
Sub-Work (a Work with none is a task, or an unplanned mission — closing it
would be inventing a decision), its own state is neither completed nor
cancelled, and every live Sub-Work is in a `completed` state. Cancelled
children are dropped by `mission.sub_works` and so do not hold a mission
open.

One line per Work, moved or not. Exit codes follow the same rule
`agentchat resolve` follows: a Work that is *already* Done is an answer, not
a failure (exit 0); a named Work that is genuinely unfinished says how far it
is and exits 1; a name matching nothing is an error on stderr, exit 1.

`--dry-run` was not in the plan. It is three lines and it makes the sweep
safe to look at before it is run, which is what let the check below happen at
all without deciding anything.

## Live check (dry run, nothing moved)

```
$ uv run python -m agautolab.mission_done --dry-run
F3-7  would be Done "Mission: Minimal Playable Core Loop Prototype (Browser)" (5 sub-works)
R-1   would be Done "Add hello files to the project" (2 sub-works)
R2-1  would be Done "Add README.md and HELLO.md to main/" (2 sub-works)
S2-30 would be Done "Give stage 1 a real enemy sprite and a BGM loop" (4 sub-works)
S2-6  would be Done "敵の出現パターンのバリエーションを増やす" (3 sub-works)
S2-1  would be Done "MVPシューティングを動かす" (4 sub-works)
```

p9's open item is in that list by construction, and it is not alone: five
earlier missions were left in the same state, which is the gap being one of
omission rather than one bad run.

The single-Work paths, live:

```
$ … mission_done --dry-run S2-30
S2-30 would be Done "Give stage 1 a real enemy sprite and a BGM loop" (4 sub-works)   (exit 0)
$ … mission_done --dry-run S2-31
S2-31 not moved: already Done                                                         (exit 0)
$ … mission_done --dry-run ZZ-999
agautolab.mission_done: no Work named ZZ-999                                          (exit 1)
```

**Nothing has been moved.** Step 4 is where the entrance is asked to close
them, and it should be the entrance that does it — moving them here would
make the proof a Deus Ex Machina.

## Tests

`tests/test_mission_done.py`, 12 tests on the existing `FakePlane`: the
counting (all-completed moves, one unfinished does not, a cancelled child
does not hold it open, a Work with no children is not a mission, a
foreign-`external_source` Work is untouched, an already-Done one is not moved
twice), the named-Work paths by label and by id with their exit codes, the
unmatched name, and dry-run moving nothing.

agautolab: 176 passed (was 164). Committed and pushed as `a883419`.

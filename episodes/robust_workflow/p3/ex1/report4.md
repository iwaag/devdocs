# Step 4 report — the narrow change, verified on a fixed revision

Plan: [plan.md](plan.md) step 4. 2026-09-24 (JST; trials 06:42–07:02 UTC), by the Omni Agent as the
Developer's stand-in and as trial operator.

## The fixed revision

| Component | Revision | Running since |
|---|---|---|
| agautolab (listener, gateway) | **`3a67d63`**: step 2–3's `ec0b065`, `09615fb` and `d06bacb`, plus the trial fault hook | The listener restarted 06:40Z on `d06bacb` and 06:41Z on `3a67d63`, then by the trial's own crash (06:57:05Z) and the restart case (07:00:07Z). The gateway was restarted after the trials (`/healthz` ok) |
| autolab introduction | Re-posted from `params/intro.md` at `09615fb` (checked with `agentchat intro autolab-agstudio1`) | — |
| Everything else | Unchanged from p3: pyagag `0ef3c61`, agfront `99ad106`, agobserver `74c881f`, agdevworld `2891ab2` | — |

No code changed after the first trial. Every live case below ran on `3a67d63`. `nctl drift`:
converged=46. `autolab-agstudio` is `polling`, and `autolab-agautolab1` is `stale` as before (the VM
runs the gateway only and takes no missions, so it was not redeployed).

Added for the trials, one-shot and created only by a person:
`agautolab/.local/faults/exit-after-integration`. It makes the next close-out end the listener
process right after its integration is recorded. launchd (KeepAlive) restarts it and the journal
requeues the serving.

## Fixtures (counted apart from live evidence)

**293 passed** (236 before ex1):
- `test_missionspace.py`: 21 cases on real git. Report2 lists the ownership, cancellation,
  release and record cases, and report3 lists the integration and interruption cases.
- `test_zulip_listener.py`: the listener's close-out, recovery, return, repeat, checkpoint, release
  and fault paths.
- `test_cli.py`: `init-repo` inside a copy.

Fixture-only (not exercised live):
- a clean merge of **disjoint** files;
- an `overlap` that merges without a textual conflict (both live returns were real conflicts);
- a remote moved by somebody outside autolab;
- unowned dirt blocking an integration;
- `publish.flag`;
- archiving a project with live copies.

## Live trials (through Front, on `pj-robustp1`)

Conversations `#front › front-robust-p3x1-{a,b,c,d,e,f,g}`.

| Scenario | Trial | Expected | Result |
|---|---|---|---|
| Two missions edit the same file, **first-posted finishes first** | B (m10813 `--doubles`) ∥ C (m10830 `--allcaps`), with F running too | Each reviews its own change; integration preserves accepted work or exposes the conflict | **Passed.** While both were under review, each copy held only its own `wordcount.py` edits and the project folder stayed clean at `7c2cb8d`. B was accepted and fast-forwarded as `2f59df2`. C was accepted on `fc605d9` and **returned** (`conflict`: README.md, test_wordcount.py, wordcount.py), with nothing moved (origin still `2f59df2`). Front asked autolab to merge on its own. The worker merged `main` into its copy (`2a6cc29`) and showed the combined result. I ran 133 tests OK in the copy and agreed. It was then integrated as a fast-forward |
| Same, **second-posted finishes first** | D (m10983 `--vowel-start`) ∥ E (m11011 `--ends-s`) | Same | **Passed.** E was accepted first and fast-forwarded as `9372482`. D was returned (`conflict`: README.md, wordcount.py) and merged in its copy as `cb44af0`. I ran 148 tests OK and agreed, and it was fast-forwarded |
| Cancelled with dirty files **and** a checkpoint | F (m10859 `--reversed`) | Other and later missions contain none of it | **Passed.** At cancel time the copy held commit `2b82949` plus uncommitted tests. Release committed the tests onto `autolab/m10859` (`06d8bdc`) and removed the copy. The record is `checkpoint` then `cancelled`, with no `accepted` or `integrated`. `main` has no `--reversed`. D, E, G and A started later from `main` and never held it |
| Worker checkpoints before acceptance | C, D, E, F, A (the workers committed); D and E also wrote `report.md` in their first serving | Saved in isolation; the shared target and completion record do not imply acceptance | **Passed.** Every such commit stayed on `autolab/m<id>` (origin unchanged, checked by `ls-remote` before each acceptance). D's and E's early reports closed nothing: "not closed … Its work is checkpointed on autolab/m<id>; nothing is integrated" |
| Target moves after review | C (reviewed at checkpoint #10876, B integrated after), D (reviewed at #11004, E integrated after) | Integration checks the combined result and surfaces material conflicts | **Passed.** Both returned on the real merge, and the requester agreed to the combined result |
| Restart **after a checkpoint** | A (m11200): the listener was restarted while task 1's checkpoint `afd57ff` awaited review | Same copy resumes | **Passed.** The next two servings logged no new attachment. The copy has the same inode and mtime (15:59:01 local) and the same branch. Task 1 was integrated as `afd57ff` |
| Restart **during close-out** | G (m11111) task 1, with `exit-after-integration` armed | Already-integrated work is recognized | **Passed.** The listener exited (status 70) right after pushing `76e6034`. launchd restarted it in <1 s and the journal requeued the serving: "finishing the close-out accepted in #11141". No worker run followed. The second integration note reads `already/level`. There is one result post and one `completed`, and the devlog record `bab7686` exists once. Task 2 then started |
| Repeated acceptance / integration | G: my duplicate acceptance (#11143) posted into the task topic 2 s after Front's (#11141), during the crash; mission level: `agentchat accept 11200` repeated after `done` | No duplicate work, extra merge or false completion | **Passed.** The duplicate produced nothing extra: one commit, one result, one devlog record. The repeated mission acceptance returned "m11200 is already done … nothing was written" |
| Task and mission explicitly accepted | A (2 tasks), G (2 tasks), B, C, D, E | Required integration finishes, mission reaches `done`, Observer releases the request | **Passed.** Every task's record reads `checkpoint → accepted → integrated → completed → accepted`, and the task was `completed` only after `integrated`. Front recorded each mission with `agentchat accept` unprompted (6 of 6). Observer's `tracked.json` dropped all seven trial requests (`o10801` … `o11193`) within one look of the record. Monitor `ok`, 0 incidents, 0 judgments |
| Normal multi-task mission | A (task 1 code, task 2 tests + README) | — | **Passed**: `afd57ff`, then `0a444f3`, 161 tests OK. The copy was released after task 2, and the branch was kept |

**End state checked in git**, not only in reports:
- robustp1 `main` = origin = `0a444f3`, with 11 commits since `7c2cb8d`.
- Each accepted option appears exactly once (`doubles`, `allcaps`, `ends-s`, `vowel-start`, `short3`,
  `mixed`) and `reversed` does not. The suite passes.
- The only worktree left is the project folder's own, and `.local/missions/robustp1/` is empty.
- `autolab/m10813 … m11200` are kept.
- Eight devlog commits, each holding only its own task folder.
- The project folder is clean and level with origin.

## Observations (none required a fix)

| Kind | What |
|---|---|
| Change in the wrong scope | None found. Diffs, branch heads and records were checked for every trial |
| Unnecessary approval round | None. The two extra agreements (C, D) were on combined code the requester had not seen, which is the decision the plan returns. Front brought each one back instead of agreeing itself |
| Mission-recording reminder | None needed (6/6 recorded by Front, each after the close-out) |
| Rescue intervention | None. Front and autolab resolved both returns without the Omni Agent |
| Front relay inaccuracy | After E's close-out, Front wrote "**Not integrated:** … Nothing is integrated", carried over from its own previous message. autolab's close-out said "integrated into the project: main 9372482eb7 (fast-forward) and published", and git agrees. The record was right and the relay was wrong |
| Front's stray `</parameter>` lines | Twice (B #10845, G #11134), as in p3 |
| Recovery reply's addressee | The recovered close-out in G named the last speaker, my duplicate post (#11143), not Front. Front learned of it through task 2's report. Existing handoff rule, harmless here |
| Startup replays | Each listener restart logs ~11 "recovery: #… in '✔ …' names us and has no receipt" for old p3 topics, each ignored in the same second with no run. Pre-existing (p3 ✔-callback recovery); an ignored mention writes no receipt, so it recurs |

## Cost

78 runs, **$7.62**: Front 49 runs ($4.58); autolab 29 runs ($3.04: 8 planning, 21 task servings).
None for the recovered close-out. No forge runs and no Observer judgments.

## Not proven

- Seven missions on one small single-file project, one requester.
- The disjoint-file merge, a non-conflicting same-file overlap, an outside push, unowned dirt and
  `publish.flag` are fixture-only.
- A pattern project with several repositories and ignored generated data has not run a mission on
  this code.

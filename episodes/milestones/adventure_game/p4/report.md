# Adventure game p4 — report

**Claim: the workflow ran one real request end to end with no Deus Ex Machina.** The Developer
asked Front in one line; Front, autolab and the Developer's own acceptance took it from request
to `done` on record. The Omni Agent posted nothing, accepted nothing, nudged nothing and
repaired nothing. Verdict **stable**; next episode **p4/ex1**. Exit condition: 1 of 3
consecutive zero-DEM requests.

Per step: [report1](report1.md) baseline · [report2](report2.md) the run, hop by hop ·
[report3](report3.md) independent verification · [report4](report4.md) table, classes,
verdict · [dem_log](dem_log.md) (empty).

## What was asked and delivered

The request (protoprey-refs `todo.md`): a scene, shown when a predation event ends, where the
player chooses the biome to respawn in, built from `scenes/modes/biome_view/`. Before posting,
the Developer rewrote `todo.md` (`34ae3f9`) and dropped the "rotating Earth" line. They also
posted their own one-line request instead of the drafted [request.md](request.md).

Delivered: menu F's biome choice is now a "Choose Your Spawn Biome" scene. It has bg.png full
screen and data-driven bubble cards (forest and meadow, with view and icon), Spawn Here, Back,
and key controls. It appears when F starts, after every predation and after every location
revisit. Discovery state survives every respawn. Menu E is unchanged.

## Accepted revisions

| | |
|---|---|
| Reference | `protoprey-refs@34ae3f9` (adopted by autolab in `direction/REFERENCES.md`; step 1 had baselined `a7b4763`) |
| Game | `autodev/protoprey` `main` **`3bf3e90`** (`2f3bdff` import, `c2ecf6f` scene, `bc3cf35` wiring, `4cb4767`/`3bf3e90` VERIFY) |
| Direction | `d9fdd25` (task 1's REFERENCES entry) |
| Mission | **m11294**: 3 tasks, all `accepted` and `integrated` (fast-forward, pushed); `[acceptance] #11410 by Developer`, `[state] done` |

## Work conversations

- `#front › front-desk-20260924-221143`: #11283 (request) … #11418 (acceptance recorded)
- `#pj-protoprey › ✔ workplan-biome-select-scene`: #11289 … #11417
- `#work-m11294 › ✔ workrun-task{1,2,3}-m11294`: #11297 … #11401

## Numbers

27 runs and $3.86, against p3's 79 runs and $16.56 (all runs used Sonnet 5 in both phases).
13 min 31 s from the request to the last integration. 0 stalls, 0 Observer incidents,
0 returned changes, 0 `[opfail]`, and 0 DEM events (p3: 20). Details in [report4](report4.md).

## The Developer's assessment

"Favourably, it went straight through to completion in the front room alone, and the functional
check looks fine too." Their close (#11410): "I checked it; I think there's no problem."

## What it does not prove

- **Observer's stall detection was not exercised**, because nothing stalled.
- **One agent behind Front.** forge was not involved, there was no long job, and no notifier
  callback.
- **The Developer did not accept each task.** They delegated execution to Front (#11317), and
  Front accepted each task, with substantive checks.
- **Two candidates to watch in ex1, not fixed:**
  - A task's close sent two callbacks to Front, which served both, one of them for nothing.
  - A conditional acceptance ("yes, but first do X") closes the task without the acceptor seeing
    X done.

## Next

**p4/ex1**: the next real request, measured the same way. It should bring a second agent into
the loop: p3's scale test (several forge locations integrated by autolab), or the Developer's next
`todo.md` entry if it needs forge.

Left for the Developer: the Front Desk's `finish ✔` on this conversation. It archives
`#work-m11294` and puts a ✔ on the Front Desk topic (the preview shows both `ready`).

## Docs

`README_DEV.md` is unchanged: nothing a later reader needs came out of a run where nothing broke,
and one observation does not make a rule. Host note added to `pj-agdev/.local/devenv.md` (ignored).

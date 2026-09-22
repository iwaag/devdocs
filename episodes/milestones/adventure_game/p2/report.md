# Adventure game p2 — phase report

Source: [braindump.md](braindump.md). Plan: [plan.md](plan.md). Steps:
[1](report1.md) paths and contract · [2](report2.md) the human's repository ·
[3](report3.md) references through the roles · [4](report4.md) one scene ·
[5](report5.md) a revision · [6](report6.md) checks and outcome. Executed 2026-09-22
07:00 UTC – 2026-09-23 16:20 UTC by the Omni Agent with the Developer.

## The answer to the braindump

The braindump asked where a human's ground-truth references should live, and how agents
on different machines could reach them with their folder structure intact. The answer
that was built: **a repository of the human's own** on the existing Gitea, edited in a
folder outside every agent workspace and published by `git push`; agents read it by
name at a pinned commit through one shared tool, `agrefs`, which lays out an immutable
snapshot per revision on each machine. Nothing is shared by path; nothing is written by
an agent into the human's tree; a reference is `<source>@<commit>:<path>` wherever it is
mentioned.

## What happened, in one line each

1. The existing paths were read and the loss points named (an expiring delivery link,
   prose as the only carrier, Front barred from repositories, no pinned revisions); the
   contract was decided.
2. Gitea account `developer`, repository `protoprey-refs`, the human's clone, `agrefs` in
   pyagag with tests; the human's push was left to the human (the classifier refused it
   in their name, and publication is their act anyway).
3. Grants and guides for Front, autolab, forge, archsage; forge's `--init-image`;
   listeners restarted; the Developer pushed `a3c8196` (wolf texts, images, advice) and
   every consumer plus a VM clone held the same bytes.
4. Front and autolab built the forest/wolf scene from the originals in 29 minutes,
   surfacing the material's ambiguities first; the Developer played it: 合格.
5. The Developer pushed `0cd3c0b` (one file); the change was identified, adopted, and
   delivered in 7 minutes; the Developer: 合格.
6. Checks recorded; Front's guide corrected; reports written.

## Proven

- Authoring → publication → consumption works across roles and machines, with the
  folder structure and the bytes preserved and the revision named everywhere.
- Agents open the originals, not each other's summaries (Front found the file-name
  typos; autolab read the EXIF), and say where they depart (the content-rule reversal was
  flagged, not applied silently).
- A human revision travelled through the same route to a human-evaluated result while
  the earlier result kept its identity.
- The human's creative judgement stayed theirs: both acceptances and the rule change are
  their words, on record.

## Not proven

- forge planning from a reference image on a real request; archsage's use of references.
- A revision that replaces already-consumed material.
- Scale: one predator, one biome, two pushes.
- The play experience beyond the Developer's own two passes.

## Cost

| | |
|---|---|
| Agent runs (steps 4–6) | 50 runs, $12.81 (Front $5.35, autolab $7.46) |
| Steps 1–3 | no agent run; one local SwarmUI generation |
| Human | six texts, five images, one advice document, two pushes, two plays |

## Basis changes

pyagag `b227b78`; agfront `8b4674a`; agautolab `4939ca9`; agforge `43730aa`; archsage
`b8099ac`; pj-agdev `30b61e2`; `devdocs/README_DEV.md` section "Human-authored
references". Production: `autodev/protoprey` `1f4c2f4`, `autodev/protoprey-direction`
`26690b4`. The human's repository: `developer/protoprey-refs` at `0cd3c0b`.

## Candidates for the next basis work (observed, not done)

1. A Project Room or Front Desk route for the Developer's own posts, so the Omni Agent
   stops relaying.
2. A verification role with a headless Godot and an image reader (the Omni Agent's
   fresh-clone checks).
3. forge: a real request with a reference image through `--init-image`.
4. autolab: commit `direction/` with the delivery (the second adoption entry lagged one
   commit until asked).
5. The rule in `human_advice.md` itself: the Developer's content rule lives in `GOAL.md`
   now; putting it in the reference would make the reference self-contained.

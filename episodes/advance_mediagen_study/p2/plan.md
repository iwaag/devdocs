# advance_mediagen_study p2 — Plan

Braindump: `braindump.md`. p1 introduced a cross-subject queue,
`main/QUESTIONS.md`, and the review found it wrong twice over: it is a work
memo living in a knowledge repository whose HEAD is meant to equal
`publish/`, and every entry in it already exists in a subject's `tips.md`
"Still open". This phase removes it and replaces the mechanism with the
simplest thing that does the same job: **one investigation, one
repository**, and an index that covers every investigation, so "is this
already answered?" is a read of `main/`, not a queue.

Destructive phase, no backward compatibility. Harness-side Developer work
throughout; nothing here needs a mission.

## Decisions

- `main/` holds knowledge only. No queue, no status fields, no task memos.
  Unknowns stay where they are born: each `tips.md`'s "Still open".
- Layout follows `studyarxiv`: `main/README.md` says what the repository is
  and what the index is for; `main/subjects/INDEX.md` is the index and
  `main/subjects/<subject>/` the reports. The index lists **only subjects
  with a report** — an inventory of what a backend holds is not knowledge and
  leaks the collection.
- Duplicate guard lives in the planner: when a request is already covered by
  an existing or finished investigation, the plan says so and points at it.

## Step 1 — pattern and guide (`agautolab`)

1. `agent/project_pattern.md`: delete "The open-question queue"; make the
   study pattern say `main/README.md` + `main/<kind>/INDEX.md`, index rows
   only for investigated subjects, "Still open" in each report as the only
   record of unknowns, one investigation per test repository.
2. `agent/guides/workplan_superdirector/guide.md`: one line — read the
   index first; if an investigation already covers or answers the request,
   reply so instead of planning new work.
3. Commit, push.

## Step 2 — `mediagen/main`

1. Move `INDEX.md` and the subject folders under `subjects/`; write
   `README.md`.
2. Drop the ten uninvestigated rows and the collection-describing "cut"
   note; drop every `QUESTIONS.md` and `gentest-` reference; remove the
   `.local/post/…` path from `spriteSheetFrames/tips.md`.
3. Delete `QUESTIONS.md` after confirming each entry is in a "Still open".
4. Commit, push. Grep `main/` for host facts and internal names.

## Step 3 — leftovers

1. Remove the empty `questions/` clone and its Gitea repository.
2. Post standing text **v2** in `#front › routine-mediagen`: step 0 reads
   `main/README.md` and the index and the chosen subject's "Still open";
   no queue; the `blocked` entry becomes "post in the workplan topic".

## Step 4 — `report.md`

What changed, the grep evidence, and what the next bare fire should show.

**MUST NOT**: host facts or internal repository names into `main/` or this
public repository; push `publish/`.

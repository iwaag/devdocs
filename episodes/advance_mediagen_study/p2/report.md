# advance_mediagen_study p2 — Report

Done on 2026-08-30 as Developer/Omni Agent work; no mission was run. Every
step of `plan.md` landed.

## What changed

**`agautolab` (`0dca020`)** — `agent/project_pattern.md` lost the whole
"open-question queue" section (59 lines → 17) and gained "One investigation,
one repository": unknowns stay in the report that produced them, `main/` is
knowledge whose HEAD is meant to equal `publish/`, and the index must list
every investigation so a planner can read it as the duplicate guard. The
study pattern's folder list now names `main/README.md` and
`main/<kind>/INDEX.md` with one row per investigated item. The superdirector
guide gained one line: read the index first; if an existing or finished
investigation already covers or answers the request, say so instead of
planning new work.

**`mediagen/main` (`165a689`, pushed, remote == HEAD)** — layout is now the
`studyarxiv` shape: `README.md` + `subjects/INDEX.md` +
`subjects/<subject>/`. `QUESTIONS.md` deleted (238 lines); every one of its
eleven entries was verified to exist already in a subject's "Still open"
before deletion, so nothing was lost. The index dropped ten rows that had no
report behind them (the backend-inventory leak), the entire LoRA-set table
(no LoRA has a summary), and the "cut" note that described the private
collection; it gained a `folder` column and the missing `novaAnimeXL_ilV5b`
row, which had a summary but no row. The `.local/post/…` path in
`spriteSheetFrames/tips.md` and every `gentest-`/`QUESTIONS` mention are
gone. `README_PROJECT.md` describes the new `main/` shape.

**Leftovers** — the empty `questions/` clone created during the p1 review
is removed and its Gitea repository deleted (HTTP 204). Standing text **v2**
is posted in `#front › routine-mediagen` (message 3257): step 0 reads
`README.md` + the index + the chosen subject's "Still open", stops when the
index shows the request is already answered; step 4 self-checks `main/`
against the publication conditions before committing; a host-install
request goes to the workplan topic and the test repository's yaml, never to
`main/`.

## Evidence

`grep -rniE "agpc|agstudio|<ipv4>|home.arpa|/Users|localhost|7801|8188|autodev|gitea|gentest|README_PROJECT|QUESTIONS|\.local"` over `main/`
after the commit: no hits. Tips content is byte-identical apart from the
one path line; renames are pure `git mv`.

## What the next bare fire should show

That step 0 picks a subject and a "Still open" item on its own from the
index, and that a fire naming an already-covered subject answers "covered,
see …" and stops. Neither has been observed yet; both are one fire each.

## Deus Ex Machina

- Everything here — pattern, guide, `main/` reshape, standing text — was
  done by hand as the plan says; the `main/` reshape in particular is
  in-system content edited from outside. Handoff candidate: a `publish`-style
  review mission could have done the trim, but the queue it removed was the
  Omni Agent's own p1 mistake.

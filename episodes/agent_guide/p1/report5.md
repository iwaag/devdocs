# agent_guide p1 — step 5: the stale sage label

archsage `b35a03e` on `agent-guide-p1`. Not deployed yet.

## What was wrong

`archsage.intro.sages_lines` rendered each sage with a findings state:
`*(study `aisvgs`: no findings yet)*` while the tree held only its plan and
indexes. The introduction is re-posted from `_after_change` (define,
update, attach, remove) and at start-up, but not after `sage sync`. So the
label written on 2026-09-26 (#11668) survived both accepted research
rounds. It was the one listing run-0160 could have read, and it said there
was nothing.

## The fix

The introduction no longer prints a findings state. It prints only what a
definition decides: the selector, what the sage knows about, and the study
it reads (`*(study `aisvgs`)*`), or `*(no study attached yet)*`. A
definition change always re-posts the introduction, so these facts cannot
go stale. How far a study got is read where it lives: its channel and
`agproject status <slug>`. `board.md` points at both.

Re-posting after `sync` was the alternative. It was not taken: it would
keep a progress figure in a contract document, and it would fall out of
date again as soon as research landed by any route other than `sync`.

`archsage sage` status and `cli.py`'s own "no findings yet" text
(`_describe`) are untouched: those are live reads, not a posted contract.

## Tests

`test_the_introduction_prints_no_findings_state` (tests/test_sages.py).
The archsage suite passes 39/39 in the main checkout. In the worktree the
new test passes, and one listener test fails only because the worktree has
no `.local` overlay naming the `claude` executable.

**Correction (step 6):** the archsage listener does not post its
introduction at start-up. The restart left #11668 in place, still saying
"no findings yet". The introduction is posted by `archsage intro` (and by
`_after_change`), which was run once at deployment. The board now reads
`… a local starting kit *(study `aisvgs`)*`. preresearch §2.3's "at
start-up" is wrong in the same way.

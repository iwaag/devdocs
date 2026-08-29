# Step 1 — the `test.md` / level contract

## Contract as written

`agautolab/agent/project_pattern.md` (commit `c6477ef`, pushed to GitHub;
`pj-agdev` pin `48e438d`) now says, in the "Repository-backed local tests"
section:

- `localtest-<id>/report.md` is the **raw run log**; the distilled,
  publishable result is `main/papers/<id>/test.md`, beside `summary.md`.
- Four levels, the braindump's list verbatim in spirit:

  | level | meaning |
  |---|---|
  | `L1` | built locally, most basic function confirmed working |
  | `L2` | a workflow described in the paper completed end to end |
  | `L3` | a small, minimal *original* verification experiment measured performance |
  | `L4` | a reproduction of the paper's own experiments, or beyond |

- `test.md` shape: level reached, upstream repository + revision, environment
  in generic terms only (no hostname/path/port), evidence, what would raise
  the level. It must stand alone when copied out of `main/` — so **no links
  into `localtest-<id>/`**, name the repository instead.
- Honesty rule stated: a one-shot smoke command is `L1` however much setup
  it took.
- `INDEX.md` `localtest` column is repurposed: `no`, `L1`…`L4`, or a
  `localtest.yaml` state (`waiting_external` …) while in progress.
- The study pattern's `publish/` line was rewritten from "moved here on
  approval" to: an **edited copy** produced by the `publish` routine as its
  own mission (never a self-check inside papers/localtest); conditions
  named; `main/` stays intact; never push — the developer reviews and pushes
  by hand.

## Diffs

`agent/project_pattern.md`: +31/−3, the two blocks above.

`src/agautolab/cli.py` (`init-localtest` README template): one sentence
appended — "When the test reaches a result, write the level reached (L1-L4,
see Autolab's project patterns) to `main/papers/<paper-id>/test.md` and the
`localtest` column of `main/papers/INDEX.md`." `tests/test_cli.py` +
`tests/test_project_init.py`: 41 passed.

No listener code changed, so no deploy ritual; the doc is read from disk per
run.

## Standing text

`#front › routine-localtest` **message 2734** — "Standing request for the
`localtest` routine, v2", replacing 2525. The body is v1 plus one new
paragraph: end every test by writing/updating `test.md` with the level (the
four-line scale repeated inline, generic-environment rule, no links into
`localtest-<id>/`, honest-L1 rule), set the INDEX column to the level, commit
and push `main/` with `git push origin`; the report line gains "level".

## Notes

- The braindump said "3段階ぐらい" but listed four; the plan chose four and
  the contract follows.
- `README_PROJECT.md` in the studyarxiv workspace still carries the old
  "moved here" wording for `publish/`; it is in-system project content, left
  for the Step 3 mission to correct rather than edited directly.

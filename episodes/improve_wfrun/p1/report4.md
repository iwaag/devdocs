# improve_wfrun p1 step 4: creation and use of a workflow, independently verifiable

Date: 2026-11-02. agdevworld `4c240ce`. Not yet released.

## What changed

The existing delegation mechanism is used for a newly published workflow (step 2 made the target
the published default branch and recorded the dispatched revision). Step 4 makes the facts a
conductor needs visible where it already looks:

- **Discover and check readiness**: `wfe delegate targets [<project>[/<workflow>]]` (needs no
  workspace of the caller's). For each project with a workspace here: the published revision and
  fetch state; for each workflow: `ready` or `NOT READY`, validity with the first errors, whether it
  accepts requests, its channel state (`provisioned`, `pending`, `not created`). `--json` gives the
  same rows. A workflow that is only in a workspace is not offered.
- **What the child ran**: the delegated result assessment (`wfe delegate show`, the reminders at
  judging and reporting) has a line after the findings:
  `ran — kid/make (run-001) with the published definition and code of dev/pj-kid@…, the revision dispatched`;
  it says `not the dispatched …` when the revision or the workflow differs, names a trial with its
  bytes' hash and baseline, and says "not recorded" for children created before p1.
  `wfe delegate show` also names the dispatched revision in its first line.
- **Guidance, in tool help only** (`wfe delegate request`): targets are published workflows and
  the dispatch records the revision; creating a workflow and running it are separate results, with
  the incident (gimmickimg freework made the scene itself; partial-animation never ran) and the
  sequence when both are wanted: delegate the creation, check the new workflow with `wfe delegate
  targets` after that child's work is integrated, then delegate to it from a delegate node naming
  it (`wfe run adjust`). The always-loaded guides are not changed, and no sequence is imposed on
  other requests.

## Checks run

- `npx tsc --noEmit` clean; `node --test 'test/**/*.test.ts'`: **673/673**. One intermediate
  failure was mine: the new `ran` line first went before the findings, which the tidy p2 contract
  keeps second (`test/assessment.test.ts`); it now follows them.
- `test/publishedDelegation.test.ts` adds the `ranLine` cases (same revision, another revision,
  another workflow, not recorded).
- Live, read-only, from the working tree against the coordinator's registry:
  - `wfe delegate targets gimmickimg`: `freework ready`; `partial-animation NOT READY — valid: 1
    error — Node "review" confirms the result but is "review"; only a talk node asks the person.;
    requests: yes; channel: not created` (the running release's notifier reads the workspace, which
    lacks it). Over all projects, 61 workflows are ready; the rest opt out of requests (`converse`,
    `discuss`, check workflows) or are partial-animation.
  - `wfe workspace status --workspace gimmickimg`: published `d4d8ea9`, root `behind`, 6 published
    commits missing, "Next: wfe workspace sync".

## Open

- gimmickimg's `partial-animation` needs its definition fixed by gimmickimg before anything can run
  it; WFE now says so instead of hiding the workflow. Not changed here (it is the project's content).
- The end-to-end trial (freework creates a workflow, a conductor delegates to it) runs after release
  in step 5.

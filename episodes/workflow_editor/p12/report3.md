# p12 Step 3 — project-owned Boast workflows

Commits: rts-vs-bot `0730858`, playprey `51f7a8b8` (both pushed to Gitea
`main`); agdevworld `a6161ec` (`wfe boast`, delegation doc).

## Identities checked

Registry: project `rts-vs-bot` (root `gitea:131`, workspace `rts-vs-bot`)
and `playprey` (root `gitea:143`, workspace `playprey`). The playprey
workspace was 24 commits behind Gitea `main` (onboard/run-002 and run-004
integrations); the change was rebased onto it before pushing.

## What was added

- **`devdocs/workflows/boast.yaml`** in both projects: one `do` node
  (`showcase`), Zulip intake, the project's root bound editable (playprey
  also runs as `playprey-agent` with its own guide, like its other
  workflows). The intent carries the shared introduction verbatim and facts:
  what is published (main, via the run's integration after completion),
  `wfe boast status|reference`, how to read the requested result at its
  commit or branch, that a result may not be on main yet, what stays
  reachable (main, `boast/`, shared outputs; not presentation servers), and
  where questions go. No required sections, format, screenshots, buttons,
  paths or cadence.
- **Delegate nodes** (`type: delegate`, `workflow: boast`, `accepts:
  parent`; no person confirmation added):

  | Workflow | Edges |
  | --- | --- |
  | RTS `improve-game` | `human-check → boast → cleanup` |
  | RTS `build-game` | `human-check → boast` (end) |
  | playprey `onboard` | `confirm → boast → cleanup` (after `present` published the delivery) |
  | playprey `compare` | `confirm → boast → cleanup` (after `adopt`) |

  The request description names what to pass: the run branch and commit,
  verification done/not done, presentation or published output
  (`shared://…`), and whether the parent's result is on main.
- RTS `build-game` was still `ag.workflow.v1`, which WFE has refused since
  p5; it moved to v2 (`documentation: default`, Zulip intake) to hold the
  delegate node. Its other nodes are unchanged.
- playprey `knowledge/workflows.md` lists `boast` beside the others.
- `freework`, `backfill-wedo` and every other project are unchanged; no
  shared guide, generator or template mentions Boast.
- `wfe boast [status] <project> [--source worktree]` and `wfe boast
  reference` (agdevworld): the published/preview state with the page and
  Project Editor addresses, and `docs/boast.md`. Tried read-only against the
  real registry: neither project has a page yet.
- `docs/delegation.md` still described p10's `delegate-scope` refusal; the
  code dropped it in p11. The doc now says `accepts` is a record.

All changed workflows validate against the registry (`wfe validate`).

## Publication order supported by current WFE

Facts read from `server/exec.ts`: an `accepted` decision queues the run's
integration (`afterDecision`); the executor's reconcile carries out queued
integrations whenever no attempt executes the run (`integrable` does not
require completion); one global slot runs one attempt at a time; a waiting
delegate node ends the parent's attempt; a completed run queues its own
integration.

1. The parent's confirming node (`confirms: result` in improve-game,
   onboard, compare) records the person's acceptance → integration queued.
2. The `boast` node dispatches the request with explicit references; the
   parent's attempt ends waiting, freeing the slot.
3. Reconcile integrates the parent's result into `main` while it waits; the
   intake creates the child from the request post. Which comes first is not
   ordered by WFE, so the child does not assume either: it reads the result
   at the named commit and can check `main` with `wfe projects show
   <project> --require <commit>` and merge `main` into its copy.
4. The child completes; its own integration puts `boast/` on `main`.
5. The parent takes up the result, disposes, cleans up, completes; its
   final integration merges records that do not touch `boast/`.

Nothing waits on the parent's completion. Without an acceptance (build-game
has no `confirms: result`, or the person does not answer), the parent's
result reaches `main` only at its completion, after the boast node; the
child then has only the explicit commit, a copy inside `boast/`, or a
shared output to point at, or it asks the parent. Shared outputs (playprey
deliveries) are durable before any integration.

Proof of this order, conflict handling and preservation of `boast/` by
later parent integrations is Step 4's (trials).

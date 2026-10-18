# p13 Step 4 — workflow authoring

## What changed (agdevworld `2fdca83`, `6bb8069`)

- **Tool discovery** in the authoring template (`templates/AGENTS.md`): one
  item says `wfe context show` (run in a project, `--workflow <id>` for one
  workflow) shows the guidance that applies and where it comes from, that
  the Developer keeps preferences out of Git on purpose, and that their
  text is followed, not copied into tracked files. Usage stays in
  `wfe context --help`. `project.yaml` is described with its `guides`.
- **Installed authoring entry**: `wfe setup <area> --installed-url <url>`
  writes `<area>/authoring/AGENTS.md` and `CLAUDE.md` for the operating
  area, with `<area>/bin/wfe`, the installed docs (through
  `service/current`) and the published editor URL. It is placed beside the
  workspaces, not in the area itself: an `AGENTS.md`/`CLAUDE.md` in an
  ancestor of a run copy would be loaded by every executing agent's
  harness (report1 inventory). `scripts/wfe-release.sh` refreshes it after
  every healthy activation; a hand-edited file (no stamp) is kept.
- **Same resolver** for existing workflows and for a project without a
  workflow identity yet (system and project context only), as in Step 1.
- **UI**: the Context view's *Copy text* / *Copy command*, labelled
  "available to copy, not injected" (Step 3).

## A real authoring session

Scratch authoring area (`wfe setup`, the source CLI), project `demo`
(intent: technical blog posts; goal: every post reviewed before
publication), and a project override set with `wfe context set project`:
"every node id starts with `step-`; every node description begins with
`Goal:` and ends with `Check: …`". Claude Code (`claude -p`, the area as
working directory, no injection of the override) was asked: "create one
workflow with id publish that drafts a blog post, reviews it with the
person, and publishes it … Follow whatever guidance applies to this
project." ($0.14, 24 s)

- The agent read `AGENTS.md`, then ran `wfe context show` and
  `wfe context show --workspace demo --rendered`.
- The workflow it committed has nodes `step-draft`, `step-review`,
  `step-publish`, each description `Goal: …` / `Check: …`; publication
  follows the review, which it tied to the project goal. `wfe validate`:
  valid.
- `git grep` over the committed tree finds none of the override's
  wording: the preference shaped the workflow without becoming its text.
- Its closing message said it followed the project's local guidance and
  did not copy it into a tracked file.

Evidence (ignored): `pj-agdev/.local/workflow-editor-p13/step4/`
(`session.jsonl`, `publish.yaml`).

## Checks

`test/setup.test.ts` 17/17 (new: the installed entry is beside the
workspaces, uses the area launcher and URL, mentions `wfe context show`,
fills every variable, keeps a hand edit).

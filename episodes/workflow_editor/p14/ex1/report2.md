# p14 ex1 Step 2 — every entrance uses the list, duplicate copy removed

## Context section (`ContextSection`, Home / project / workflow)

- **In the repository** (project, workflow): the purpose, goals and guides
  controls are one row — *Purpose, goals and guides* (workflow: *Purpose
  and guides*) with the host's short summary and *Unsaved changes* when its
  draft differs — whose *Edit* opens the host's own controls in the panel.
  They remain the host's one draft and save; the element is only moved
  into the panel. Each of the scope's own guides is a row that opens its
  content as prose.
- **For all projects / this project / this workflow · only in this
  environment**: said once per section; the longer explanation (who gets
  it, kept on this host, used from the next attempt) behind a `?` help.
  One *Add* in the heading.
- **Inherited**: one list instead of a sub-heading per scope — the
  project's purpose and goals and its guides (workflow editor), the
  project's blocks, the system blocks — each row naming its scope. More
  than five rows fold behind *Show all N inherited*; short instructions,
  drafts and the first four rows stay visible. Empty wider scopes say
  nothing; a section with nothing inherited says so once.
- **What a new run is given**: a row; the rendered text (preformatted —
  it is the literal prompt) opens in the panel with *Scope overview*. Its
  line says *Saved settings only: unsaved changes are not included* while
  a draft exists.
- Entrances: the project hero's *Purpose & instructions* opens the section
  and focuses its first row; *Edit them in Context* (project settings) and
  *Edit it in Context* (workflow intent panel) open the shared controls in
  the panel directly. The section closes its panel when its view is
  disposed.
- The short-instruction threshold was tightened to two rendered rows: at
  three, two long Japanese sentences (≈130 wide characters) still counted
  as short.

## Run card and overview

- Run *Settings for future attempts*: rows with *Add*; the hint is one
  line ("For every run of workflow make, from the next attempt. To steer
  this run now, add an instruction above."), the rest in its tooltip.
  Actual-attempt (*Used for this attempt*) and next-attempt (*Changes for
  the next attempt*) remain separate groups; a block opens beside the run
  without leaving it.
- `#/context` overview: matrix cells show title and *managed by* (and a
  failed refresh); last editor and size moved to Detail; the description
  is one line with the precedence note in Detail; quick edit uses the same
  rows and panel.
- `docs/context.md` "Where it is edited in the browser" describes the list
  and the panel.

## Checks

| Check | Chromium | WebKit | Chromium 390 px | WebKit 390 px |
| --- | --- | --- | --- | --- |
| `contextBlocks` | 40/40 | 40/40 | 44/44 | 44/44 |
| `contextEditors` (moved to rows and panel) | 70/70 | 70/70 | 74/74 | 74/74 |
| `contextRows` (new) | 17/17 | 17/17 | 19/19 | 19/19 |
| `context` (overview) | 7/7 | 7/7 | 7/7 | 7/7 |

`contextRows` covers: a short Japanese instruction in its row; two long
Japanese sentences and a 300-character unbroken line opening instead
(complete, wrapping inside the panel, no page overflow at 390 px);
multiline information read as three paragraphs, a list and `code`; its
*Not verified* exception; keyboard-only open (focus on the panel title),
Esc and focus return; 13 inherited rows folding to 6 with the short
instruction kept, *Show all* revealing all; an empty scope's one line.

`contextEditors` keeps its journeys: a system instruction from Home, a
project instruction read as inherited in the workflow, the purpose with
exactly one control (now in the panel) keeping focus and caret through a
background refresh, intent and a new block drafted together through mode
switches, a live refresh and the navigation guard, separate saves, a
conflicting save with both recoveries, the system block edited from the
workflow editor naming the wider scope, the run card's preference save
recording nothing in the run and keeping the graph selection, an
instruction recorded once, labels for every control (panel included),
edge states, matrix links.

## Visual comparison

Fixture screenshots (ignored `wfe-fixture/screenshots/`, `p14-*`,
`p14ctx-*`, `p14ex1-rows-*`) put the project's Context section above
Runs and Workflows at the same width: rows share the Workflows list's
38 px icon tile, 15.5 px title, 12 px row padding, divider and right-hand
action; the meta line is 13 px muted like a workflow's state line. The
full section for forge (purpose, goals, guide, local block, two inherited
blocks) is now five rows instead of a page of inputs and grey boxes. The
installed-route comparison is Step 4's.

## Revision

agdevworld `ebfe8a5`.

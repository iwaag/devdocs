# p14 Step 3 — the GUI

## What the person sees

Every local block is a **card** (`src/contextParts.ts` `BlockCard`), in
Simple and Detail alike:

- title, a kind chip (*Instruction* / *Environment information*), and the
  scope it really lives in (*all projects*, *project forge*, …);
- *Managed by* and *Last edited by* (age and producing run) side by side, so
  a human edit of cagent's block reads "Managed by cagent · Last edited by
  <person>";
- for information: the oldest observation's age, partial/failed coverage,
  the latest check; a failed refresh in red with its age and detail;
- the size: the body's lines (warned above five for all projects: every run
  of every project gets it) and the delivered bytes;
- the content, its detail links (open as text through the logical
  reference) and whom to ask.

Detail adds the store file, revision/content revision, section hash,
evidence references, the check and refresh records and the full history
(op, author, run, entrance, bookkeeping marked).

Editing: each card has its own Edit / Save / Remove and result. Editing a
block someone else manages says so ("Your edit is kept in its history with
your name; cagent reads it before its next update") — no permission dialog.
Drafts survive refreshes, display switches and the navigation guard; a
conflicting save shows the saved text and offers *Use the saved text* or
*Save mine over it*. *+ Add an instruction* / *+ Add environment
information* create blocks (id from the title).

| Where | Change |
| --- | --- |
| Home | *For all projects*: the system cards |
| Project editor | the project's own cards; *Inherited*: purpose, goals, guides and the system cards, editable in place, each naming its scope |
| Workflow editor | the workflow's cards; inherited project defaults, the project's cards and the system cards |
| Run Context card | *Used for this attempt* and *Changes for the next attempt* name each block by kind, title and manager; *Settings for future attempts*: *Workflow context*, *Project context*, *Context for all projects* open the block lists in place |
| `#/context` overview | each local cell lists its blocks with kind, manager, last editor and size; quick edit uses the same cards |

`docs/context.md` "Where it is edited in the browser" describes this.

## Checks

| Check | Chromium | WebKit | Chromium 390 px | WebKit 390 px |
| --- | --- | --- | --- | --- |
| `checks/contextBlocks.ts` (new, p14) | 29/29 | 29/29 | 32/32 | 32/32 |
| `checks/contextEditors.ts` (moved to blocks) | 63/63 | 63/63 | 67/67 | 67/67 |
| `checks/context.ts` (moved to blocks) | 25/25 | 25/25 | 25/25 | 25/25 |

`contextBlocks.ts` covers: separate Developer and cagent cards with kind,
manager, last editor, freshness and size in Simple; a human edit of
cagent's block (manager kept, human as last editor, via browser); cagent's
stale write refused, then its update after reading keeps the human's words;
a conflicting save keeping the draft through a mode switch, then saved over;
a failed refresh shown with the old content; Detail evidence, a detail link
read by logical reference, history with runs; inherited cards naming their
scope; a project information block added; a check leaving the next attempt
unchanged; the run card naming the particular changed blocks.

`npm run check`: tsc, 401/401 tests, build.

## Found and fixed

A block added before an editor's first read was dropped (or saved with an
empty project, 400) when the read named the real scope (found while moving
the p13 checks). New cards now follow the list to its scope, and the add
buttons wait for the first read.

## Revisions

agdevworld `cdc123b`, `de014de`. The p13 checks were moved to the block UI
by a subagent; assertions that only fitted a single override (an override's
body in the matrix cell, the textarea after "use saved") were replaced by
their block equivalents.

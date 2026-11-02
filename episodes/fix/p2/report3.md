# Step 3: guidance kept to two shared lines

Date: 2026-11-02 (JST).

## What was read

- The shared text every conversational run reads on demand: agautolab `agent/guides/wfrun/reference/conversation.md` (the wfrun guide names it as *Answer or hand over a result* → conversation roles). It said nothing about closing.
- ponderer `agent/conduct.md`: closing appeared only as the person's act ("When they close the room while you rest, this run is finalized; reopening continues the same run"). `agent/guide.md` lists `close` among the commands and records conduct/run-020's omission (workflow_editor fix3); both left as they are.
- pj-front, pj-forge, pj-archsage, pj-warden guides: no mention of closing; none was added.

## What was done

agautolab `5b62114`, one bullet of two sentences in `reference/conversation.md`:

> When asked to close a conversation, use `wfe conversation close <id>`; the target may belong to another project (fix p2). Closing does not complete or approve work, or stop started work; see `wfe conversation help`.

pj-ponderer `47a3843` (pushed to Gitea main, rebased onto the archive and integration commits that had landed meanwhile): the person-only sentence in `conduct.md` became a description consistent with the shared line — the room is closed by whoever names it (the person from the page, or a run with `wfe conversation close <id>`; the conductor's own `announce --conclude --close`, which records what is owed first); closing completes or approves nothing and stops no started work; while closed at rest the run is finalized, and reopening or the person's next post continues the same run. It cites this episode and does not repeat the shared line's command text.

Syntax and lifecycle stay in `wfe conversation help` and `docs/conversations.md` (step 1); no keyword rule, no inactivity rule, no procedure.

## Checks

- Duplicate-text check over the agautolab reference files and the ponderer, front, forge, archsage and warden guides (`awk 'length>40' | sort | uniq -c | awk '$1>1'`): the only repeated lines are the pre-existing shared ones between ponderer's `guide.md` and `conduct.md`; the new line appears once. `wfe conversation close` appears in exactly two guide files: the shared reference and ponderer's `conduct.md` (as the general fact, not the instruction).
- Live: the executor on this host reads the guide from the agautolab checkout at each attempt (no restart needed); the agpc worker reads it at its executor pin, now `5b62114` (`report1.md`). ponderer's guide is read from the project's committed main on Gitea, where `47a3843` is. agautolab `5b62114` is pushed to its GitHub origin.

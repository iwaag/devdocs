# Step 4: applying drafts through the service

agdevworld `5027560` (service) and `d053143` (GUI).

## What changed

- Apply has no working tree. Objects go into the access cache of the repository holding the definitions, a temporary index changes only that path, and the commit is pushed without force on the head it was built on. This keeps it independent of any checkout, clean or not.
- Compare and merge: unchanged upstream publishes directly; other changes merge with `git merge-file`; the same lines are a conflict. The merged result is validated at the published revision.
- Conflict resolution in the GUI: per change, keep this draft's or the latest, or edit the result as text. `deleted` offers recreate or discard; `renamed` offers moving the draft or keeping its own name. The result is a saved revision whose base is the latest compared against, and it is refused if the project moved again meanwhile.
- Submodule mode: the definition is committed on the head of the submodule's default branch, then the root's gitlink is committed and pushed. If the definitions repository had moved past the pin, the new pin carries those commits too, as an integration does; the journal's `root.pinFrom` and `pinTo` name it.
- The journal names each commit before it can reach the remote. A retry finds what arrived, and a pin that did not arrive is finished. Outcomes: `applied`, `conflict`, `invalid`, `failed`, `uncertain`, `in-progress` (reads as `uncertain` after 5 minutes).
- Applied is recorded only after reading back through `publishedWorkspace` with a fresh fetch. A later publication that already changed the file again counts if the revision contains the applied commit (`superseded`).
- Revisions saved during an apply stay as an open draft, based on what was applied. Applies of one project take turns.

## Judgement calls for the Developer to know

- Errors block Apply, warnings do not. Approvals remain records and block nothing.
- The merge is by text lines, not by graph structure. Two edits on adjacent lines of the YAML can be reported as a conflict although they touch different nodes. It errs towards asking.

## Checks

`test/drafts.test.ts`, all with a real Git remote stand-in: published diff limited to the definition, dirty and behind checkout unchanged, duplicate apply, merge, conflict and resolution including a project that moved again, malformed and invalid results, a save during an apply, a writer winning the push, simultaneous applies, a lost push response, deleted and renamed files, submodule publication, a pin that did not arrive, unreachable Gitea. Browser: merge with an upstream change, and a conflict resolved and applied, on the real Gitea test project.

Not checked at this step: a deployed apply (step 7); a real submodule project (none of the 21 uses that mode, see step 6).

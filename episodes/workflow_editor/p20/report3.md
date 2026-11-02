# Step 3: durable drafts independent of the checkout

agdevworld `5027560` (service) and `d053143` (GUI).

## What changed

- `server/drafts.ts`, `shared/drafts.ts`: one synced file per draft in `<area>/.local/wfe-drafts/`, with id, author, project and workflow identity, base bytes and descriptor, saved text, numbered revisions and timestamps.
- Starting a draft reads the published definition fetched now. An existing draft is offered with who and when. Starting another is explicit.
- Saves are revision-checked. The view keeps `editRevision`, the saved revision the browser's edits were made on, separate from the latest it has read.
- Text repair: a draft can hold unreadable text, so a malformed published definition can be repaired in the GUI. Empty text is refused.
- New workflows are drafts with no base. The creation form no longer writes to the checkout.
- Trials pin the saved revision's bytes, digest and draft identity in `definition.source`. Unsaved edits are saved by the action, as the button says ("Save and try").

## Defects the browser check found, both fixed before this commit

1. A second tab holding older edits could overwrite a newer save. Reading the external save advanced the revision its own save would name. This is why `editRevision` exists. The service tests had passed, because they name revisions directly.
2. The band said "Editing" before the draft had loaded. Text typed or read in that window was empty, and saving it stored an empty definition. The band now waits for the record, and the service refuses empty text.

## Checks

- `test/drafts.test.ts`: durability across a new store, revision conflicts, published source unchanged by a save, the checkout unchanged, existing drafts offered, a trial's recorded source, a published run staying published.
- Browser: saved edit survives reload, two tabs told and refused, a trial started from revision 3.
- A real service restart with a draft in place is checked in step 6.

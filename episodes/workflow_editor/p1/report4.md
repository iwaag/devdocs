# workflow_editor p1 — step 4 report: external editing

## Outcome

Sequential edits from outside the editor (an IDE or agent writing the YAML)
and from the UI work in both directions. Malformed content, corrected
content, failed saves and the unsaved-draft guard behave as the contract
says. All 33 browser checks in `checks/step4.ts` pass, and 26/26 service tests
pass. Writers took turns throughout; this step does not test concurrent writes,
and no concurrent-write safety is claimed.

## How external changes reach the UI

The service polls `project.yaml`, `.gitmodules` and `devdocs/workflows/*.y?ml`
once per second (bounded to 200 files) while a browser is connected. It pushes
changes over Server-Sent Events with the new content hash. Polling sees in-place
writes and rename-over saves the same way, on every platform and editor.

The UI compares the hash with what it last loaded or saved:

- **Same hash:** its own save (or content it already shows). Ignored, so a UI
  save is never reported back as an external change.
- **No unsaved draft:** reload automatically.
- **Unsaved draft:** keep the draft and show "The file changed on disk while you
  have unsaved edits" with "Reload from disk (discard my edits)". There is no
  merge.
- **Another workflow, the project or `.gitmodules`:** refresh references only
  (delegate targets, repositories), never the draft.

## Evidence (`checks/step4.ts`, screenshots `step4-*.png`)

1. An in-place external write to a node description appeared in the UI. An
   atomic-rename save (write a sibling, rename over) that added a node `qa`
   and an edge appeared too, with no draft left behind.
2. A UI edit of `qa`, saved: the YAML contains it. The external additions and
   the file's header comment survived the UI save, and the UI did not flag its own
   save as external.
3. The next external edit after the UI save appeared.
4. Draft guard: with a UI draft (renamed `qa`), an external save was announced
   and the draft kept (title and "Unsaved changes" unchanged). The explicit reload
   adopted the external content and showed "Reloaded from disk".
5. Malformed external YAML (`type: [talk`): the banner said "malformed", with the
   parser message, line and column. The last valid graph stayed visible
   read-only, Save and the add buttons were disabled, and the service reported
   `problem.kind = malformed`. After the file was corrected externally, the
   banner cleared and the corrected content was editable.
6. The file broke while the UI held a draft. Save was refused visibly (409),
   the malformed file was not overwritten, and the draft was kept. Once the file
   was fixed, retrying saved the draft.
7. Failed save from an unwritable directory (`chmod 555`): "Save failed … EACCES"
   was shown and the draft kept. After restoring permissions the retry
   succeeded and the error cleared.
8. Project view: an external `project.yaml` edit appeared. With a project draft,
   a further external edit was announced and the draft kept. A workflow file
   created externally appeared in the workflow list.

The service tests from step 3 cover the same rules below the UI: atomic save
leaves no temp file, an `EACCES` failure keeps the old file, and malformed or
unsupported files are never overwritten.

## Deviations and notes

- One first-run failure was in the check, not the editor. The check's external
  edit wrote `description: Edited in the IDE: read…`, which is invalid YAML
  (a second `: `). The editor correctly showed it as malformed.
- After "Reload from disk" the save state now reads "Reloaded from disk", not a
  stale "Saved hh:mm:ss" from an earlier save.
- Latency is up to one polling interval (1 s by default, `--poll-ms`). The
  watcher runs only while a browser is connected.
- The guard is not a concurrent-write guarantee. A write that lands between the
  service reading a file and renaming its save over it is not detected, by
  design of p1 (writers take turns).

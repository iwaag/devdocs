# Workflow editor p7/ex2 — step 1: isolated reproduction checks

Source: agdevworld `6058c0f` (the investigated revision). New file: `workflow_editor/checks/runInteraction.ts`.

## The check

`npm run build && node checks/runInteraction.ts [--port 8202] [--keep]`

It builds its own temporary project, registry, lock directory and private service (port 8202) with the same server modules as `checks/runs.ts` (`createProject`, `createRun`, `runOp`, `PrivateService`). It touches no other registry or workspace. The service is started and the browser opened inside the `try`, so the `finally` stops the service and removes the temporary data even when browser startup fails. That path was exercised by accident; see *Prerequisite* below.

The fixture run `ship/run-001`:

- one open question `q1`, so an answer form exists;
- 40 progress entries plus the setup operations, so the right pane and its history scroll;
- `long.md` in the run folder, 300 lines of about 310 characters, so the file view scrolls both ways;
- a `run.stop` with no attempt open, which holds the run at once, so the resume form is shown. Progress can still be recorded while it is held;
- the adjustment, evidence and actor forms, which are present in every current, unfinished run.

How each assertion observes behaviour:

- **DOM identity.** The control under test is marked and kept by reference. Each assertion asks whether that same element is still connected and is `document.activeElement`. The final string alone is not enough, because the old renderer copies drafts into a new control.
- **Value writes.** An init script wraps the `value` setter of textarea and input elements and counts writes to marked controls.
- **Composition.** CDP `Input.imeSetComposition` starts the composition and later extends it. `Input.insertText` commits it, and keyboard typing continues. Composition events on the marked control are logged.
- **SSE.** Updates are real: `node.progress` is recorded through `runOp` in the check process, the service's watcher emits the event, and the check waits until the summary's `data-seq` reaches the new sequence.
- **Heartbeat.** The run's `/exec` response is intercepted and its `executor` is rewritten to `available` with a different `at` on every read. Nothing else changes. A `MutationObserver` watches the answer form, and value writes are counted.
- **History.** The pane is scrolled to the bottom. The first history entry more than 100 px below the pane top is marked, and its offset from the pane top is recorded. Four newer entries are then recorded, which the newest-first list inserts above it.

## Result against the current implementation

| Assertion | Result | Observed |
| --- | --- | --- |
| Answer textarea connected and focused across SSE progress | **fails** | `connected: false`; the focused element is a new textarea with the same label and copied text |
| Composing textarea connected and focused after an update | **fails** | the original element is detached mid-composition (`compositionupdate:にほ`, never `compositionend`) |
| Composition completes in the same control, typing continues | **fails** | the detached control keeps `Small にほ`; the commit and later typing go elsewhere |
| Heartbeat-only change leaves the answer form untouched | **fails** | two heartbeat-only reads in 7 s; the form was replaced, so the observed control is disconnected |
| File view keeps its scroll offsets | **fails** | `scrollTop` 400 → 0, `scrollLeft` 80 → 0 |
| Visible history entry keeps element and position | **fails** | a different element; the offset went from 118 px to 330 px (four entries inserted above) |
| No page errors | passes | |

So the check reproduces the confirmed input replacement and nested-scroll failures. It also shows two failures the investigation only inferred: the heartbeat-only rebuild, and the history displacement. Value writes were 0 in every case, because the old renderer never writes into the old control; it replaces it. The value-write counter matters for the new design, where controls persist.

Synthetic composition is not native IME behaviour. macOS Japanese IME validation remains a separate, manual step (step 7).

## Prerequisite

The pinned `playwright-core` 1.62.1 needs `chromium_headless_shell-1234`. At 19:58 today the shared Playwright cache held only revision 1248, put there by some other tool. The first run therefore failed at browser launch, and the `finally` cleanup ran as intended. The matching shell was installed with the project's own CLI (`node node_modules/playwright-core/cli.js install chromium-headless-shell`). That is a user-cache download; nothing in the repository changed. Any browser check failing with "Executable doesn't exist" needs the same command.

## Baseline

`npm run check` passed on `6058c0f` before any change (type check, all tests, production build).

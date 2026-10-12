# Workflow editor p8 — step 1: feature map and representative states

Source at the start: agdevworld `a571ae2`, clean. The installed release reported by `/api/version` is `a571ae2deb51…`, the same commit. `npm run check` passed on it before any change (type check, all tests, production build).

New files in agdevworld: `workflow_editor/checks/states.ts` and `workflow_editor/checks/statesShots.ts`. No UI code changed in this step.

## Representative states

`node checks/states.ts [--port 8210] [--serve] [--shots <dir>]` builds a private project `sample` in a temporary directory. The fixture has its own registry, area, lock directory and presentation host config (ports 18800–18809). It records every run through the same server modules the CLI uses: `createRun`, `runOp`, `adjustRun`, `registerPresentation` and `Presenter`. `--serve` keeps the private service open for a manual look. `--shots` captures every state. `checks/simplerUi.ts` (step 6) will import `buildStates()`.

Titles, outcomes and question texts say what each state is. The sample is a "night mode" change to a sample app, not the RTS project, and nothing in the UI will depend on it.

### Definitions

| File | What it exercises |
| --- | --- |
| `improve.yaml` | Default documentation policy (plan talk, five reports, final report); a `confirms: result` talk; custom appearance on three nodes; template provenance on Present; two bindings (editable, read-only); both approvals recorded |
| `explore.yaml` | A branch and a join (survey → build / ask → recommend), default policy |
| `tidy.yaml` | Manual policy; every node has its own icon and color; not approved (draft pills) |
| `pipeline.yaml` | A larger graph: 16 authored nodes in three lanes that fork and join over eight ranks, plus 16 generated reports |
| `broken.yaml` | A YAML error: the editor's read-only problem state |

The editor states with no selection, a node, an edge and a generated step selected are UI interactions on `improve.yaml`. The capture script and the step 6 check drive them.

### Runs

| Key | Run | State |
| --- | --- | --- |
| `plan` | `improve/run-plan` | Plan talk waiting for an answer to its plan question |
| `working` | `improve/run-working` | Attempt `at-0002` open, Verify running with progress. A commit before Verify started provides the historical view |
| `review` | `improve/run-review` | Human check waiting on a result-confirmation question about presentation p1, **live** on this host |
| `answered` | `improve/run-answered` | Plan question answered, not yet taken up |
| `held` | `improve/run-held` | Stopped by the person with no attempt open: held, resume needs an instruction |
| `failed` | `improve/run-failed` | Verify failed; Present, Human check and Clean up blocked. `report-implement.md` was recorded and then deleted, so the file is missing |
| `cancelled` | `improve/run-cancelled` | Cancelled with p1 still recorded `available` but not served (a failed/unavailable host observation) |
| `accepted` | `improve/run-accepted` | Result accepted from the confirmation answer (q2); Clean up still running. Not completed |
| `completed` | `improve/run-completed` | Completed and accepted; adjusted to r2 (a study added before Implement); p1 stopped; final report written |
| `nodecision` | `tidy/run-tidy` | Manual policy, completed, **no decision**; one recorded artifact exists and one is missing |
| `branch` | `explore/run-branch` | Branch and join in progress: Build running, Ask waiting, **two open questions** |
| `large` | `pipeline/run-large` | The large graph, not started |
| `broken` | `tidy/run-broken` | `run.json` truncated: a malformed record with no earlier valid reading |

Connection loss is a browser condition, not a stored state. The step 6 check produces it by cutting the event stream and the reads. It also breaks a run's record *while it is open*, so the "last valid reading" path is covered as well as `broken`, which has none.

One fact cannot come from the fixture: whether an executor *process* is alive. No executor runs. The captures answer the working run's `/exec` read with one alive attempt and an available executor, and the screenshot index marks that capture as synthetic.

### Baseline captures

Every state was captured with the current UI at 1440 × 1000; the main ones were also captured at 1280 × 800. The 1440 set is in [image_before_states/](image_before_states/). These captures add the states that the original `design/simpler/image_before/` set (one completed run) could not show: a live review, an open plan question, a held or failed run, an accepted run still cleaning up, two open questions, a malformed record and the large graph.

Observations that add to `design/simpler/image_before/opinion.md`:

- **Open questions sit in the right pane, in the same list as old ones** (`run-review`, `run-plan`). In `run-branch` both open questions are there too, below the fold at 1280 × 800. The review link in the dark header is the only element that says what the person should do.
- **The failed run** says "Failed: verify" in one line of the header. The reason appears only after the person selects the node.
- **The accepted-but-cleaning-up run** shows a small "accepted · cleanup pending" chip in the header. Nothing else separates it from an in-progress run.
- **The large graph** fits to a scale where card text is unreadable (about 0.35). Nothing shows that the view is a fitted overview.
- **The held run's resume form** is in the Execution section, below Questions, Presentations, Definition, Catalog and Files.

## Feature map

Every current section and operation, with where it goes in the redesigned layout. *Header* is the slim dark bar. *Title band* is the light band under it, with the name, state and the main entry points. *Focus* is the run view's main-column card for the person's current task. *Side section* is a labeled, expandable section in the right pane. "Unchanged" means the behavior stays and only the styling follows the new look.

### Workflow Editor

| Current | Operation / content | Destination |
| --- | --- | --- |
| Crumbs row | Projects / project / `workflows/<file>`, workspace pill | Header breadcrumb (Projects / project / Workflow); the file name and workspace move to the title band's subline |
| Mode switch | Compact / Mini | Canvas toolbar |
| Refresh | Re-read file and Git state | Header, right |
| Save state, Save, ⌘S | Explicit save of the draft | Header, right (unchanged semantics) |
| Name input | Edit the workflow name | Title band, large |
| Approval pills | Intent / Definition: approved, changed since approval (stale), draft; "Author Approved"; approver in title | Title band, always visible (stale stays distinct) |
| Approver input, Approve intent, Approve definition, approval error, "reflect the unsaved draft" | The declared approver and the two separate approvals | **"Intent & approvals"** panel opened from the title band |
| Intent textarea | The authoritative intent (ground truth) | "Intent & approvals" panel; the title band shows a one-line purpose preview |
| Validation summary | Error and warning counts | Title band chip; it opens Workflow settings at the validation list |
| Documentation policy line | What a run adds (plan, reports, final report) | Toolbar chip that opens Workflow settings; the full text is in settings |
| Banners | Not connected; file missing (with link to the same id under a new name); unusable file (line/column); changed on disk while dirty (Reload); save failure; notices | Banner row under the header, always visible (unchanged) |
| `+ Study / + Do / + Delegate / + Talk` | Add a base node at a free slot | **"Add node"** palette, section "Basic types" |
| `Yours: + …` templates, "edit…" | Add a node from a template (appearance and provenance copied); catalog error; link to User nodes | "Add node" palette, section "Your templates", with the catalog error and the User nodes link |
| Auto-arrange | Layout only | Canvas toolbar |
| Toolbar hint | Drag / connect / pan | Canvas footer hint |
| Canvas | Select, drag, connect by port, pan, wheel and pinch zoom, + / − / Fit, Delete key, Enter to select | Unchanged; zoom controls stay at the canvas corner |
| Inspector, nothing selected | Documentation policy, bindings (path, access, remove, add), validation list | **"Workflow settings"** panel, opened explicitly; the inspector column is closed when nothing is selected |
| Inspector, node | Type, name, description, appearance (icon, color), template provenance, delegate target and its unsupported note, repository bindings (and removing an undeclared reference), Waits for / Then with edge links and delete, Connect, node issues, Delete node | Inspector opened by selection, with a close button. Name and Description first; Type; Appearance, Repositories and Connections as expandable groups whose open state survives rebuilds; provenance, issues, Delete node last |
| Inspector, edge | From / to links, meaning, issues, Delete connection | Inspector (unchanged content) |
| Inspector, generated step | Generated by the policy, facts, instructions; "Workflow settings" | Inspector, read-only (unchanged content) |
| Validation issue click | Selects the node or edge | Unchanged; an issue about a binding or the policy opens Workflow settings |
| Raw text | File text when no valid version was ever read | Canvas column (unchanged) |
| Event stream | Content and context reloads; external-change guard | Unchanged |

### Run view

| Current | Operation / content | Destination |
| --- | --- | --- |
| Crumbs | Projects / project / run ref; "as committed at" pill | Header breadcrumb (Projects / project / Run / ref) |
| live / not live / history pill, Refresh | Connection health; re-read | Header, right |
| State chip, situation chip | Recorded execution state; what runs now | Title band: execution state, decision and executor activity as three separate facts |
| "Snapshot of …", Open current definition, source comparison | Fixed copy vs. current file | Title band button "Open current definition"; graph heading chip "Fixed snapshot · rN in force"; comparison text in Run details |
| Facts row | Input and its author, executor and backend, created / started / ended, last update, counts, policy | **"Run details"** panel from the title band. Work and documentation counts also appear, separately, in the graph heading |
| Active / Waiting / Ready / Failed / Blocked / Cancelled / Decision lines | The run's present position | Focus card, chosen by situation; all lines also in Run details |
| Review link | Live presentation link / "recorded but not served" / stopped | Focus card (result review), and the Presentations section summary |
| "Running means…" note | The meaning of a reported state | Run details |
| Banners | Connection, read error, unusable record (with last valid reading), historical commit, operation notice | Banner row (unchanged) |
| Legend | All eight states | Under the graph, only the states present in this run |
| Canvas | Select node or step, reveal, fit and zoom; the definition revision in force | Graph card in the main column |
| Recording as | The declared actor | Side pane, first row (no form without it) |
| Selected node | State, type, description, template, document state and file link, step facts, report link, times, wait and holder, outcome and artifacts, failure, cancellation, notes, the node's questions with "Answer / Show" links | Side pane top, independent of the sections, with a close button; read-only |
| Questions, open or answered | Question text, presentation link, answers, the **one** answer form ("Record answer" / "Record another answer") | **Focus**: every open or answered question, each with its canonical form. An answered one shows the receipt and who holds the next move, and offers "Record another answer" |
| Questions, taken up or withdrawn | Record of the conversation | Side section "Questions · N resolved". A question moves there only when its form holds no draft |
| Instructions | Received / acknowledged | Side section "Instructions" (count of unacknowledged in its label) |
| Presentations | State, live link, not served detail, asked / decided, Start, Start again, Stop, Retry stop | Side section "Presentations"; its label shows live / stopped / problem |
| Definition | r1, revisions with reason, source, difference | Side section "Definition · rN in force" |
| Adjust future work | YAML, reason, Preview, Apply (bound to text, reason, revision, seq) | Inside the Definition section (unchanged binding). The focus card of a failed run links to it |
| Node catalog | The run's frozen snapshot | Side section "Node catalog" |
| Related runs | Predecessor, entrusted by | Side section "Related runs" |
| Reports and files | Folder files, recorded artifacts, missing chip, file view (scroll kept) | Side section "Reports & files". The focus card's "Read final report" and "Original request" open files there (explicit navigation opens the section) |
| Execution | Situation, process, hold, host read error, executor, **Stop**, **Resume** form, queue, attempts, publication, **Retry publication** | Side section "Execution & publication". Stop also appears on the focus card while work is in progress. The Resume form is placed on the focus card while the run is held |
| Coordination | Zulip topic, request, delivery, errors | Side section "Coordination" (error count in its label) |
| Result | Decisions, explanation, evidence, **Accept**, **Reject**, **Cancel run** | Side section "Result & decision". When a completed run has no decision, the form is placed on the focus card. A form holding a draft is never moved |
| History | Every entry, newest first, anchored reading position | Side section "History · N events" (anchoring adapted to the nesting) |

### Placement rules carried into steps 4–5

- A persistent control (an answer, the resume instruction, the evidence) is created once. Its home may change with the run's state, but only while it holds no draft and has no focus. Otherwise it stays where the person is typing.
- A closed side section still shows its errors and required human actions in its label. Open questions are never inside a closed section; they are in the focus area.
- Background updates never open, close or scroll a section. Explicit navigation does: a file link, "Answer under Questions", a node link, a validation issue.

## Completion criterion

Every current capability above has a planned entry point. The design can be reviewed against 13 run states, 5 definitions and the editor selections, not only the single completed run shown in Take 3.

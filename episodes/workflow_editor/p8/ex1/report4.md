# Workflow editor p8/ex1 — step 4: essential state distinctions in Simple

**Source**: agdevworld `fcb5eeb`. **Deployment**: none; the installed release is still `8cda5c9`.

Every run state of the fixture was opened in Simple at 1440 × 1000 and corrected where Simple hid a distinction or kept p8's long text.

## The human task by state

p8's focus card still chooses the task. Simple shows only what that task needs. A **More details** route (Reports & files, All records, or a section link) switches to Detail for the rest.

| State | Simple shows | Kept in Detail |
| --- | --- | --- |
| Open question | the question, its one answer form, the name it records under; "Sample Agent asked. Recording your answer does not wake it." | asked-to, node, asker and time; answer metadata; the form's storage note |
| Live review | "Open the review, then answer below.", the review button, the question and its form | the revision and host of the presentation; the question's "about presentation" line (the button is the link) |
| Answer awaiting take-up | "Your answer is recorded"; "Sample Agent has not taken it up yet: the next move is theirs."; the receipt; "Record another answer" on request; the card says **Answered**, not Waiting | the note that recording does not wake the executor |
| Working | the step at work with its latest note, a one-line executor observation with an icon, Stop | the attempt id, pid and executor state line |
| Held | the reason, "Nothing runs until it is resumed with an instruction", the Resume instruction and button | who held it at what time |
| Failed | the failed step and its reason, what it blocks, Adjust future work, Cancel run | who recorded the failure and when |
| Accepted, cleanup pending | the chips **In progress** and **Accepted · cleanup pending** apart; "The result was accepted; the run completes after its remaining steps"; the step at work and Stop; a presentation recorded as available but not served, with its link | the confirming question's id |
| Completed, no decision | **Completed** and **No decision** apart; the evidence field with Accept, Reject and Cancel run | — |
| Completed and accepted | the result's title, "Accepted from Sample Person's answer", the stopped review, Read final report | question id, recorder, time; the outcome list |
| Cancelled | **Cancelled**, who cancelled it and why, "Stop p1" for the presentation still recorded as available, what had started | the time |
| Historical | a dated banner "A past state of this run — as committed on <date>: <subject>. Read-only." with Open the current run; no form, no Stop, no name field | the commit id in the breadcrumb pill and the banner |
| Unusable record | the error banner and "No usable record"; no empty side pane | — |

**Several questions.** All open questions stay listed in the focus card, each with its own form, under "2 questions wait for your answer". None is hidden behind the first.

## Observations that are missing

- **The executor observation** (`RunSide.observation`). It reads, in order:
  - "Not connected: what runs now is not known" while the event stream is down;
  - "This host's execution facts cannot be read";
  - "Checking the executor…" before the first reading;
  - when a reading exists, "The executor is working on this host", "No executor process is seen", "The executor is starting", "The executor runs on another host", or "No attempt is open".
- **Unhealthy states get a warning icon.** When the observation is a warning, the working lead no longer says "Nothing is needed from you now". It says "Recorded as in progress, but this host does not see the work running: look below."
- **Recorded progress versus actual activity.** The card's **Running** is the recorded state; its tooltip still says it is reported progress. The observation line is the executor's actual activity. The long explanatory note stays in Run details.
- **Banners.** The connection banner and the read error carry an alert icon. When the connection drops, the side's observation now updates; before, it waited for the next host reading.
- **Resource problems.** A failed publication, a presentation whose stop could not be verified and failing coordination still appear in Simple, as "Needs attention" items with a link to their section. Routine Git and publication records stay in Detail.

## Defects found and fixed

1. **Request form with blocked storage.** The Project view's request form read and wrote the actor's name in local storage without a guard. With storage blocked (a private window or a blocked site), the Project view threw while rendering the runs card. This was present in the installed `8cda5c9`. It is now guarded like every other per-viewer convenience.
2. **Dashboard refresh took the keyboard.** Each dashboard render, including the 20 s refresh, re-attached the forms' inputs, so the keyboard left whatever field the person was typing in. The text was kept; the focus was not. This was present in `8cda5c9`. The field being typed in now gets the focus back with its selection.

## Validation

`checks/displayMode.ts` now has 68 assertions, all passing. They cover:

- the preference (fresh storage, the toggle, reload, navigation, a malformed value, storage that throws);
- the toggle's place on all five screens;
- the contract: Simple and Detail text on the Project, the completed run and the historical view;
- the contextual entries: Project settings, New project, the approval state, Reports & files, All records and New template;
- the run states in the table;
- the working run: a live executor, unreadable facts, and a lost and regained connection.

`checks/simplerUi.ts` (p8, run in Detail) passes after three expectations were updated for intended changes:

- the open file is in its own side card, not in the Files section;
- the decision lead now reads "Accepted from Sample Person's answer (q2, recorded by Sample Agent)";
- the selected node's card is addressed by its section.

`checks/runInteraction.ts` passes unchanged.

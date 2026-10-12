# Image generation prompts

Generated with the built-in OpenAI image generation tool, following the imagegen skill. English UI text was used for legibility. Reference images are functional references of the actual application, not style references from earlier concept art.

## 01-workflow-definition.png

Reference: ../../image_before/03b-workflow-node-selected.png

```text
Use case: ui-mockup.
Create one high fidelity desktop application concept screenshot, landscape approximately 1792 x 1024, flat front-on UI filling the image, no surrounding laptop, no annotation poster. This is a proposed simpler version of the ACTUAL agdevworld Workflow Editor, not a generic AI automation app.
Input image 1 is a functional reference of the existing workflow editor with Present selected. Preserve its domain and supported capabilities while redesigning the hierarchy and proportions; do not copy its oversized dark header or microscopic cards. UI in clear English for accurate lettering.

Visual language: quiet approachable workshop, warm off-white canvas, white panels, thin soft gray borders, small rounded corners, professional sans serif type, restrained pastel teal for talk/presentation, amber for do, slate for cleanup. A slim charcoal breadcrumb bar, blue only for primary Save and selected node ring. Crisp legible text and readable generous node cards; restrained line icons, no ornamental artwork or invented monsters.

Screen state: editing the reusable workflow definition 'Improve the game'; Present is selected; current definition is saved; both approvals are draft. This is a definition view, NEVER show running/completed states on the cards.
Top 56px: 'Projects / Simple RTS / Workflow' left, 'Saved' and blue 'Save' right.
Below a compact title band 'Improve the game', short purpose 'Agree the scope, implement, verify, present, review, clean up.' small 'Intent: draft' and 'Definition: draft' badges. 'Intent & approvals' button reveals the long authoritative intent and declared approver controls. A 'Workflow settings' entry reveals documentation policy and repository bindings. Make these clear accessible entries.
Canvas toolbar: '+ Add node' dropdown, 'Compact', 'Mini', 'Auto-arrange'; no execution button.
Main area ~73% width is a readable sparse graph with exactly these seven nodes in ONE continuous directed chain, laid out as a 2-row serpentine path to make all cards sufficiently large:
top row left to right 'Agree the plan' (PLAN TALK, teal), 'Implement' (DO, amber), 'Verify' (DO, amber), 'Present' (DO, teal, selected blue border).
Arrow from Present DOWN to Human check at bottom right. Bottom row right to left 'Human check' (TALK, teal), 'Clean up' (DO, slate), 'Final report' (DOCUMENT, small neutral endpoint).
Arrows must implement exactly Agree the plan -> Implement -> Verify -> Present -> Human check -> Clean up -> Final report. No branches, duplicates, or other connections.
Work cards have one short descriptive sentence: Implement 'Change game/ within the agreed scope.' Verify 'Run tests and check the browser.' Present 'Freeze the result and serve its review.' Human check 'Ask for approval or changes.' Clean up 'Stop the review server and verify.' Agree plan 'Write plan.md and ask the person.'
For Implement, Verify, Present, Human check, Clean up show a small attached document icon at the bottom right representing a per-node report; not a separate executable node. Subtle small 'game' repository chip. Final report label 'report.md'. Large enough type, no paragraphs packed into cards.
The right inspector ~27% width shows selected node:
header presentation icon 'Present' with close X
'Type' dropdown 'Do'
'Name' input 'Present'
'Description' textarea 'Present the changed game with its changes, how to play, and what to check. Freeze the files and start a review server.'
'Appearance' collapsed section
'Repositories' section two checkboxes: checked 'game · Editable'; unchecked 'rts · Read-only'
'Connections' collapsed row 'Verify → Present → Human check'
small 'Template: present' note and secondary red text 'Delete node' at bottom.
Canvas bottom right '+  −  Fit'. A small nonintrusive canvas hint 'Select a node to inspect. Drag to arrange or connect.'
Constraints: exactly one screenshot. Do not invent Run This Node Only, Pause, live terminal, IDE launch, arbitrary model parameters, code editor, chat transcript, fantasy sprites or Run Workflow. Present and Clean up are ordinary Do nodes with visual appearance, not new execution types. Preserve report badges and separate authoritative intent/definition approval entries. Prioritize readable graph, approachable typography, and manual editing controls.
```

## 02-completed-run.png — initial generation

Reference: ../../image_before/04c-run-viewport.png

```text
Use case: ui-mockup.
Create ONE high fidelity desktop application screenshot concept, landscape approximately 1792 x 1024, flat UI filling frame, readable professional English text. Proposed calmer ACTUAL agdevworld Workflow Editor Run view, using the functional reference screenshot supplied. This is an existing COMPLETED run, not a generic AI agent builder or a live game-review state.
Input image 1: functional reference of the real completed run. Preserve its factual domain, report steps, per-run fixed graph and presentation availability. Redesign layout hierarchy without inventing new operations.
Visual style consistent with a calm workflow workshop: warm off-white page, white panels, thin gray borders, 8px rounded corners, restrained pastel teal, amber, violet and slate, dark navy text, crisp line icons. Slim charcoal breadcrumb bar, blue only for primary reading action and selection. No decoration, fantasy illustration, avatars, marketing graphics or perspective laptop.

Top bar ~56px: 'Projects / Simple RTS / Run' left; small 'improve-game/run-005' secondary; 'Live connection' indicator and 'Refresh' right.
Compact title area: 'Cherufe: a mythological heavy unit'. 'Completed' and 'Accepted' badges. Subtitle 'Mythology research, implementation, verification, human review, and cleanup are finished.'
Small explicit line 'Executor idle · No response needed'. Header actions 'Open current definition' and 'Run details'.
Main work area split ~73% left, ~27% right inspector.
Upper left large readable outcome summary panel 'What changed':
'Added Cherufe, a slow heavy unit inspired by Mapuche mythology. Its lava splash damages nearby enemies.'
A small checklist 'Headless tests: 31/31 passed' and 'Human review: OK'.
Primary blue button 'Read final report', secondary 'Original request'.
An explicit restrained availability row with slate monitor icon: 'Review server stopped' and 'Frozen files and the accepted decision are retained.' No live game thumbnail, no play button, no restart control.
Below this summary, main left panel title 'Run workflow' with small 'Fixed snapshot · r2 in force' badge; subtle 'Read-only' pill. Small introductory note 'Select a step to inspect its record.' 
Readable directed graph exactly EIGHT nodes, a two-row serpentine path:
top left to right 'Agree the plan' (PLAN TALK, teal), 'Mythology study' (STUDY, violet, selected blue border), 'Implement' (DO, amber), 'Verify' (DO, amber).
Arrow Verify DOWN to Present at bottom right. Bottom right to left 'Present' (DO, teal), 'Human check' (TALK, teal), 'Clean up' (DO, slate), 'Final report' (DOCUMENT, neutral).
Exact sequence Agree the plan -> Mythology study -> Implement -> Verify -> Present -> Human check -> Clean up -> Final report. No branches or duplicated nodes.
All eight show a small checkmark 'Completed'. Ordinary work nodes Mythology study, Implement, Verify, Present, Human check, Clean up have a small attached document report icon. Names and state words readable. Node cards are smaller than the definition editor because the outcome panel is the primary content, but NOT microscopic. Fit, plus, minus controls.
Right inspector heading 'Mythology study', small 'Completed' and 'Read-only' badges. Use readable prose sections, NOT editing forms:
'Purpose' paragraph 'Find lesser-used mythological monsters and choose one for implementation.'
'Recorded outcome' paragraph 'Compared Cherufe, Bakunawa, and Iku-Turso. Selected Cherufe for its volcanic theme.'
'Report' status 'Checked' with secondary button 'Open report-prestudy.md'.
'Definition' small text 'Added in revision r2 before implementation.'
At the bottom of right inspector, separate collapsed entries 'Questions · 2 resolved', 'Presentations · 1 stopped', 'Reports & files', 'Execution & publication', 'History · 46 events'. These are dependable paths to existing records, not terminal streams.
Constraints: no Save, no editable node prompt, no Pause, no Run Node Only, no model configuration, no chat box or request composer, no live game preview for a stopped service. Presentation is a Do node with a presentation icon, not a separate execution primitive. Respect the distinction between the completed run's frozen/adjusted definition and the reusable current definition. Node completion and executor idle are different explicit facts. Preserve real content rather than inventing sprite generation or a fantasy production pipeline. Make a useful approachable screen that invites reading the outcome and exploring the work.
```

## 02-completed-run.png — final correction

The initial Run generation was edited to correct the Clean up description and simplify the reintroduced technical metadata. The image in this folder is the corrected result.

```text
Use case: text-localization / precise UI text correction.
Edit the supplied concept screenshot. Preserve its composition, colors, typography, node positions, graph arrows, sidebar, and all other content. Make only these operational text corrections, because this mockup must reflect the real workflow:
1. In the bottom-row 'Clean up' node, replace the incorrect description 'Refine and tidy files.' with EXACT 'Stop the review server and verify.' Preserve DO and Completed, icon and report badge.
2. Inside the upper 'What changed' card, in its right-hand column BELOW the 'Review server stopped' status and its explanation, replace the dense metadata list Input/Executor/Started/Ended/Work/Policy with only these three concise readable lines:
'Work: 6 nodes completed'
'Documentation: 8 steps completed'
'Decision: accepted from q2'
Keep the server-stopped heading and retained-files explanation unchanged. Do not add model names, dates, backend IDs, unsupported controls, new artwork or extra content. Keep everything else exactly the same. This is a completed read-only Run; no actions that execute work.
```


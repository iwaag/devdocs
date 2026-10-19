# p8 Step 4 — the managed Front project

## Created through the common path

- `wfe create pj-front --id front …`:
  - Gitea `developer/pj-front`;
  - workspace `front`;
  - registered;
  - the basic workflow `freework` placed.
- Content, pj-front `b20cb6f`:
  - `project.yaml`: intent, three goals, the `knowledge` area and the project guide `agent/guide.md`. It was saved with the new `wfe project save --base` (Step 2).
  - `devdocs/workflows/converse.yaml`: the conversational workflow in the freework pattern, a single `talk` node; the run adds do/delegate nodes as the message needs. Execution:
    - agent `front-agent`;
    - profile `sonnet`;
    - node `studio`;
    - lane `conversation`;
    - documentation: none generated.
  - `knowledge/README.md`: how coordination lessons are kept, and what lives elsewhere (preferences in context blocks, work in run records, the conversation in its store).
- No special project type, no separate executor. Front's turns are ordinary WFE runs performed by the coordinator's worker in its conversation lane.

## The role, rewritten

The guide (`agent/guide.md`) follows the agent-guide form (devpolicy/styles.md).

- **Head:**
  - Front is the Developer's entrance;
  - it acts with the Developer's authority over everything agdevworld offers;
  - records name Front as actor and the Developer as the person served;
  - each run serves one message.
- **What it sees:**
  - the conversation (`wfe conversation`);
  - every project and run (`wfe projects`, `wfe run`, `wfe delegate`, `wfe observe`, `wfe exec workers`, `wfe dashboard`);
  - operations: requests, run control, adjustment and delegation, project and workflow creation and editing, context;
  - this project's knowledge;
  - Git, the shell, `curl`, the share.

  Usage stays in each tool's `--help`.
- **Facts,** not procedures:
  - what a message may be, and recording its reading as a link;
  - who does specialist work, and checking for an existing project before creating one;
  - waiting frees the slot;
  - what counts as done;
  - answering specialists' routine questions;
  - steering, stop and cancel scope;
  - replies name what they answer and carry review links;
  - where preferences, lessons and run state live;
  - how knowledge reaches `main` (its own result decision, then ordinary integration);
  - usage observations.
- **Removed:**
  - "only a human creates projects";
  - "do not open project repositories";
  - proposal-before-acting;
  - the retired agents' addressing;
  - the Zulip mention hazards.
- **Kept, with their trial references:**
  - evidence-backed lessons (failsafe p3 A2, adventure_game p1/p2, failsafe p7 run-0183, runtime-profile, refine_routine p1, the Front Desk "begin with the answer" observation);
  - binding a delegation to the consuming work, which is now the delegate node itself.

## Knowledge and preferences

- General preferences: system context block `developer-preferences`, managed by `front-agent` (`wfe context set system …`, revisioned and conflict-checked).
- Project preferences: that project's blocks.
- Coordination lessons: `knowledge/` in Front's repository. Simultaneous runs write in their own copies and reach `main` through integration, which reports real conflicts by file.
- No shared unrecorded writes.

## First exercise on the deployed entrance

Conversation `p8-trial-a`. The message was posted by the Omni Agent under its own name: *"What did the latest playprey run deliver, with review links?"*

- Turn `front/converse/run-001` ran in the conversation lane and completed. The reply came 50 s after the message.
- Front read the playprey run records through the common tools and answered from them. It gave:
  - the latest run with its time and integration state;
  - the deliverables;
  - what was not verified;
  - the run view and output links;
  - that it read the records rather than verifying anything itself.
- No specialist work was started.

The remaining exercises are Step 6's trials, where they are recorded with their evidence:
- steering an active run;
- one-off work;
- delegation;
- creating a project.

## Deployment done for this step

- WFE `7b16e0c` (from `ea09c14`; backup `p8-predeploy-*` in the area). The exec store migrated to v4.
- `workers.json`: studio gets `lanes: {conversation: 1}`. New `conversations.json`: entrance `front/converse`, person `iwaag`. The executor runs with `lanes = ["conversation"]`.
- Web image rebuilt; relay restarted.

Machine details are in the ignored notes.

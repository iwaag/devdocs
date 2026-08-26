# Step 2 — guides and pattern doc honest

Both guide edits from the braindump discussion were uncommitted in
`agautolab`. They are committed now (`bc77f4d`) together with the fixes this
step called for, after re-reading both guides start to finish against Step 1's
reality.

## What was fixed on top of the braindump edits

- `dosn't` → `doesn't`.
- Sentence case and grammar in the added lines: "If the developer wants you to
  prepare the project", "If the developer is giving you a new mission", "If the
  developer asks you to execute the mission, just tell them it is the planning
  phase, not the execution phase", "create a project based on a specific
  pattern", "a nonexistent pattern is specified", "just ask back in your
  reply".
- **"You edit those files" now names its object.** The braindump text said
  "You edit those files only when …" directly after a paragraph mentioning
  `README_PROJECT.md` — "those files" had no antecedent. It reads
  "You edit "README_PROJECT.md" only when …" now, and the create-if-missing
  sentence was moved up next to the sentence that introduces the file, so the
  three README_PROJECT.md statements (what it is, create it if missing, when
  to edit it) sit together and the pattern-document paragraph stands alone.

## Re-read against the new reality

- No fixed folder description survives anywhere in the guides. Verified by
  `grep -rn 'main/\|direction/\|devlog/' agent/guides/ params/`: one hit
  remained, the supercoder's close-out line, and it is rewritten (below).
  The same grep over `src/agautolab/*.py` for the prompt strings the listener
  builds finds nothing — the folder prose only ever lived in the two guides.
- `autolab doc patterns` now exists (Step 1), so the workplan guide's
  reference to it is no longer ahead of reality.

The supercoder's close-out line was the one place still asserting the fixed
layout as fact:

```
-If the developer agreed that the task was done, create "report.md" … Also, commit changes in "main/" after getting developer's approval, checking and editing "main/.gitignore" to avoid comitting unnecessary files.
+If the developer agreed that the task was done, create "report.md" … Also, commit changes in the repository folders you edited after getting the developer's approval, checking and editing their ".gitignore" to avoid committing unnecessary files. "README_PROJECT.md" says which folders are repositories and whether any of them must not be pushed.
```

This also carries the study pattern's never-push-`publish/` rule into the
guide by reference rather than by restating it: the pattern document and the
project's own README are where that lives. (`comitting` → `committing` while
there.)

## Final guide texts

`agent/guides/workplan_superdirector/guide.md`:

```
The topic is supposed to be about planning and preparation for next mission in the development.
Your reply to this conversation will be sent to the developer.

# If the developer wants you to prepare the project

The file "README_PROJECT.md" explains how the folders inside the workspace are supposed to work.
If "README_PROJECT.md" doesn't exist, create it to explain how each folder works.
You edit "README_PROJECT.md" only when you added new repositories or local folders in the workspace, or changed the way to manage development of the project.

The command "autolab doc patterns" explains how project structure should be managed based on pattern. If you are asked to create a project based on a specific pattern, follow it. If not enough information is provided, or a nonexistent pattern is specified, just ask back in your reply.

# If the developer is giving you a new mission

First, if the mission is clear enough, and the chat log suggests it hasn't been created or needs an update, write "plan.md" to complete the mission. It will be recorded as a new mission or overwrite the previous plan.

And next, if you think the new mission is better divided into smaller tasks, create one file per task named "task[N].md" — "task1.md", "task2.md", "task3.md", ... — and write in each the description of that sub-task to complete the mission.

The first line of "plan.md" and "task[N].md" is a Markdown heading ("# ...") and becomes the title,
and the rest of the file becomes the description.

If the developer asks you to execute the mission, just tell them it is the planning phase, not the execution phase.

If the requester has clearly said that the mission can be started, create file "start.flag".
If the requester has clearly said that the mission should be cancelled, create file "cancel.flag".

If you think you need more discussion before creating a plan, just ask questions in your reply without editing any files.

## In case the plan include outsourcing to other agents

The file this prompt names above lists the other agents and what each one
does. A task may be a request to one of them: say which agent and what to
ask, in words they can act on without this project.

Keep it to one request per task.
```

`agent/guides/workrun_supercoder/guide.md`:

```
The topic is supposed to be about work to do in this session.
Your reply to this conversation will be sent to the chat.

The file "README_PROJECT.md" explains how the folders inside the workspace are supposed to work.

Do the work following developer's request.

To create an agag agent, `agag init <name> --yes --provision --like <sibling-root>` generates it and provisions its Zulip identity; `agag --help` is the usage reference.

If the developer agreed that the task was done, create "report.md" in the workspace directory this prompt names above. Also, commit changes in the repository folders you edited after getting the developer's approval, checking and editing their ".gitignore" to avoid committing unnecessary files. "README_PROJECT.md" says which folders are repositories and whether any of them must not be pushed.


## When the task is to ask another agent

The introductions file this prompt names above says how to reach each agent
and what it calls finished. Talk with them using `agentchat` (`--help`
explains it).

Post the request or reply and finish. You will be called again when they answer, and
the result goes into this task's own topic.
```

## Diff from the pre-episode state (`31b1eb1`)

```diff
--- a/agent/guides/workplan_superdirector/guide.md
+++ b/agent/guides/workplan_superdirector/guide.md
@@ -1,11 +1,16 @@
 
-The topic is supposed to be about next mission in the development.
+The topic is supposed to be about planning and preparation for next mission in the development.
 Your reply to this conversation will be sent to the developer.
 
-The folder "main/" contains current source codes of the project.
-The folder "direction/", if exists, contains concept documents of the project.
-The folder "devlog/", if exists, contains plans and reports of past works.
-Being empty means the project has just started. 
+# If the developer wants you to prepare the project
+
+The file "README_PROJECT.md" explains how the folders inside the workspace are supposed to work.
+If "README_PROJECT.md" doesn't exist, create it to explain how each folder works.
+You edit "README_PROJECT.md" only when you added new repositories or local folders in the workspace, or changed the way to manage development of the project.
+
+The command "autolab doc patterns" explains how project structure should be managed based on pattern. If you are asked to create a project based on a specific pattern, follow it. If not enough information is provided, or a nonexistent pattern is specified, just ask back in your reply.
+
+# If the developer is giving you a new mission
 
 First, if the mission is clear enough, and the chat log suggests it hasn't been created or needs an update, write "plan.md" to complete the mission. It will be recorded as a new mission or overwrite the previous plan.
 
@@ -14,19 +19,17 @@
 The first line of "plan.md" and "task[N].md" is a Markdown heading ("# ...") and becomes the title,
 and the rest of the file becomes the description.
 
-## Work other agents can do
-
-The file this prompt names above lists the other agents and what each one
-does. A task may be a request to one of them: say which agent and what to
-ask, in words they can act on without this project.
-
-Keep it to one request per task.
-
+If the developer asks you to execute the mission, just tell them it is the planning phase, not the execution phase.
 
 If the requester has clearly said that the mission can be started, create file "start.flag".
 If the requester has clearly said that the mission should be cancelled, create file "cancel.flag".
 
 If you think you need more discussion before creating a plan, just ask questions in your reply without editing any files.
 
+## In case the plan include outsourcing to other agents
 
+The file this prompt names above lists the other agents and what each one
+does. A task may be a request to one of them: say which agent and what to
+ask, in words they can act on without this project.
 
+Keep it to one request per task.

--- a/agent/guides/workrun_supercoder/guide.md
+++ b/agent/guides/workrun_supercoder/guide.md
@@ -2,17 +2,13 @@
 The topic is supposed to be about work to do in this session.
 Your reply to this conversation will be sent to the chat.
 
-The folder "main/" contains current source codes of the project, which is likely your main interest to complete the work.
-The folder "direction/", if exists, contains concept documents of the project,
- which you only read when you really need to check the concept of the project.
-The folder "devlog/", if exists, contains plans and reports of past works, which you only read when you need to read log of the development.
-Being empty means the project has just started. 
+The file "README_PROJECT.md" explains how the folders inside the workspace are supposed to work.
 
 Do the work following developer's request.
 
 To create an agag agent, `agag init <name> --yes --provision --like <sibling-root>` generates it and provisions its Zulip identity; `agag --help` is the usage reference.
 
-If the developer agreed that the task was done, create "report.md" in the workspace directory this prompt names above. Also, commit changes in "main/" after getting developer's approval, checking and editing "main/.gitignore" to avoid comitting unnecessary files.
+If the developer agreed that the task was done, create "report.md" in the workspace directory this prompt names above. Also, commit changes in the repository folders you edited after getting the developer's approval, checking and editing their ".gitignore" to avoid committing unnecessary files. "README_PROJECT.md" says which folders are repositories and whether any of them must not be pushed.
```

The pattern document itself (`agent/project_pattern.md`, committed at
`31b1eb1`) needed no change: it already names `autolab project init-repo` and
the standard repository names, which Step 1 made real, and it already carries
the study pattern's never-push rule.

## Landing

- `agautolab` `bc77f4d`, pushed to `github.com/iwaag/agautolab`.
- `uv run pytest`: 196 passed.
- pj-agdev pin bump: `864f275` "Bump agautolab: guides point at
  README_PROJECT.md (project_pattern step 2)", pushed.
- Listener restarted: `launchctl kickstart -k gui/501/com.agdev.agautolab-zulip`.
  It came back and swept:

  ```
  2026-08-26T07:30:00Z autolab zulip listener starting (pull sweep: all topics in 'autolab-agstudio1', prefixes ('workrun-', 'workplan-', 'bmining-') elsewhere, routes ['bmining-', 'workplan-', 'workrun-'] + DM thread)
  2026-08-26T07:30:00Z listening as user_id=11 (autolab-agstudio-bot@agstudio.local)
  2026-08-26T07:30:01Z full sweep: 0 awaiting, 1 mentioning, 92 calls spent, 925 left in the window
  ```

  (Step 1's pin bump was `9eb5487`; the listener runs from this checkout, so
  the restart is what makes both steps live.)

# Step 3 — `init_project` respects pattern-managed workspaces

`agautolab` `0e0841c`, pj-agdev pin `d91d843`, listener restarted.

## The gate

`init_project` returns `"pattern-managed, untouched"` — a distinct result
string, `project_init.PATTERN_MANAGED_RESULT` — the moment
`<workspace>/README_PROJECT.md` exists, before `load_plane_config` and
`load_gitea_config` are even reached. No Plane project, no Gitea repository,
no clone, no folder.

Existing projects keep the exact current behaviour: none of the six
workspaces under `.local/projects/` has the file, and the check is a single
`Path.exists()` in front of the untouched body.

## What the plan did not anticipate: the Plane read-back

Skipping the scaffold is not enough. The same serving then does the Plane
read-back (`serve` → `write_mission_workspace` → `find_plane_project`), which
raises `MissionError("Plane project does not exist: studyarxiv")` for a
project that deliberately has no Plane project. Step 3's own success criterion
("no Plane call") therefore had to cover it, so the read-back is skipped for a
pattern-managed serving too.

The third Plane door is `handle_superdirector_response`: a `plan.md` or a flag
written by the run goes to `upsert_work`/`transition_work`. For a
pattern-managed project that handler now reports what the run left on disk and
records nothing, rather than raising the same missing-project error at the
developer. This is the smallest change that keeps a pattern serving working
end to end; what a pattern-managed project's ledger *should* be stays the open
decision below.

## Diff

```diff
--- a/src/agautolab/project_init.py
+++ b/src/agautolab/project_init.py
@@ -25,6 +25,12 @@
 AUTO_DESCRIPTION_PREFIX = f"{AUTO_MARKER} autolab project: "
 IGNORE_LINE = ".local/"
 
+#: A workspace holding this file is pattern-managed: its folders were decided
+#: by the agent from `agent/project_pattern.md` on the developer's request, and
+#: `init_project` is not the thing that creates them.
+PATTERN_MARKER = "README_PROJECT.md"
+PATTERN_MANAGED_RESULT = "pattern-managed, untouched"
+
 GIT_AUTHOR_NAME = "autolab-agent"
@@ -386,6 +392,12 @@ def init_project(project: str, *, main_only: bool | None = None) -> str:
     rather than turning the local record into a clone.
+
+    A workspace holding `README_PROJECT.md` is pattern-managed and is left
+    entirely alone — no Plane project, no Gitea repository, no clone, not
+    even a folder. The listener runs this on *every* serving, before the
+    agent has read the request; without this gate a fresh pattern project
+    would be scaffolded into the fixed layout before anyone asked for one.
     """
     if not PROJECT_NAME.fullmatch(project):
         raise ProjectInitError(…)
     project_root = PROJECTS_ROOT / project
+    if (project_root / PATTERN_MARKER).exists():
+        return PATTERN_MANAGED_RESULT
     on_disk_main_only = is_main_only(project_root)

--- a/src/agautolab/zulip_listener.py
+++ b/src/agautolab/zulip_listener.py
@@ -281,14 +282,17 @@ def serve(context) -> TopicResult:
     context.step = "project setup"
-    init_project(project)
+    setup = init_project(project)
+    pattern_managed = setup == PATTERN_MANAGED_RESULT
 
     context.step = "plane read-back"
-    current = workspace / CURRENT_DIR
-    current.mkdir(exist_ok=True)
-    plane_files = write_mission_workspace(current, project, context.channel, context.topic)
-    if not plane_files:
-        current.rmdir()
+    plane_files = False
+    if not pattern_managed:
+        current = workspace / CURRENT_DIR
+        current.mkdir(exist_ok=True)
+        plane_files = write_mission_workspace(current, project, context.channel, context.topic)
+        if not plane_files:
+            current.rmdir()
@@ -301,7 +305,7 @@
     response_sections, resolve_after = handle_superdirector_response(
         context.client, context.channel, context.topic, project, workspace,
-        context.self_id,
+        context.self_id, pattern_managed=pattern_managed,
     )
@@ -578,6 +582,21 @@ def handle_superdirector_response(
     label: str | None = None
 
+    if pattern_managed:
+        # A pattern-managed project has no Plane project this episode, so
+        # there is no ledger to record a mission in. Say what was skipped
+        # rather than raising `Plane project does not exist` at the developer.
+        written = [
+            name for name in (PLAN_FILE, "start.flag", "cancel.flag")
+            if (workspace / name).is_file()
+        ]
+        if written:
+            sections.append(
+                f"{project} is pattern-managed, so nothing was recorded in Plane "
+                f"(written and left as files: {', '.join(written)})"
+            )
+        return sections, resolve_after
+
     plan = workspace / PLAN_FILE
```

(`handle_superdirector_response` keeps its old `self_id` positional signature;
`pattern_managed` is keyword-only with a `False` default, so every existing
caller and test is unchanged.)

## Tests

`uv run pytest`: **201 passed** (199 after the `project_init` cases, 201 with
the listener ones; 196 at the end of Step 2).

`tests/test_project_init.py`, three new cases — with `load_plane_config` and
`load_gitea_config` monkeypatched to `pytest.fail`, so a single call is a test
failure, not an assertion afterwards:

- `test_init_project_leaves_a_pattern_managed_workspace_entirely_alone` — the
  marked workspace still holds nothing but the marker, and the result string
  is `PATTERN_MANAGED_RESULT`.
- `test_init_project_still_scaffolds_a_marked_project_that_lost_its_marker` —
  the marker is the whole of the declaration; without it the three-repo
  scaffold runs exactly as before.
- `test_init_project_pattern_marker_wins_over_an_existing_layout` — a project
  that grows the marker later stops being touched from then on.

`tests/test_zulip_listener.py`, two new cases, running the **real**
`init_project` through the whole `handle_topic` path with Plane and Gitea
wired to fail:

- `test_a_marked_workspace_is_served_without_any_plane_or_gitea_call` — the
  call sequence is `whoami, write, history, superdirector, history, write,
  history`: the `init`/`plane` steps that every other serving shows are simply
  absent, the run still happens in the marked workspace, and its reply is
  relayed.
- `test_a_plan_written_for_a_pattern_managed_project_is_not_sent_to_plane` —
  no `upsert`/`reconcile`/`transition`, and the reply says `pattern-managed`
  and names `plan.md`.

## What a brand-new pattern project needs on disk

**The Zulip channel, and the marker file. Nothing else** — no Plane project,
no Gitea repository, no folder besides the workspace directory holding the
marker.

The marker is needed because the first serving runs `init_project` *before*
the agent has read the request: on an empty unmarked workspace that is
indistinguishable from a brand-new `game` project, and the fixed three-repo
scaffold would be built before anyone asked for one. Creating the marker is
therefore **the human act that declares a project pattern-managed**, done by
hand as the Developer this episode:

```
$ mkdir -p .local/projects/studyarxiv
$ cat .local/projects/studyarxiv/README_PROJECT.md
pattern-managed; folders are created by the workplan on the developer's request
```

Automating that declaration is future work — noted, not built. Step 5 weighs
the two candidates (automatic on the first pattern request vs. an `autolab
project declare` subcommand).

## Live proof: a serving in a marked empty workspace

Setup by hand as the Developer, using `agag.zulip.ZulipClient.create_channel`
with `.local/zulip/developer.env` — subscribe-based, so it creates the channel
and subscribes both principals in one call:

```
me: 8 user8@agstudio.local bot: 11
create: {'result': 'success', 'msg': '', 'subscribed': {'8': ['pj-studyarxiv'], '11': ['pj-studyarxiv']}, 'already_subscribed': {}, 'new_subscription_messages_sent': True}
```

It was created in `folder_id=1`, the folder `pj-papers` and the other project
channels are in. (The folder is *named* `pj-runsmoke2` — it holds every
project channel and its derived `work-` channels, so the name is historical,
not a per-project folder. Not touched here.)

Then one probe post as the Developer — deliberately a question, so the run has
no reason to build anything:

```
$ AGENTCHAT_ZULIP_ENV=…/developer.env agentchat send pj-studyarxiv workplan-setup \
    "What does this project's workspace look like right now? Just tell me, don't create anything yet."
sent message 2407 to #pj-studyarxiv > workplan-setup
```

`.local/out/zulip-listener.log`:

```
2026-08-26T07:34:17Z full sweep: 0 awaiting, 1 mentioning, 93 calls spent, 923 left in the window
2026-08-26T07:34:26Z serving 'pj-studyarxiv'/'workplan-setup'
2026-08-26T07:34:26Z workplan topic 'pj-studyarxiv'/'workplan-setup'
```

Two lines and then the run — where an unmarked project's first serving would
have spent the seconds between them creating a Plane project, three Gitea
repositories and three clones.

Evidence the two skipped steps really were skipped:

- the serving workspace holds `chatlog.md` and `tools/` and **no `current/`**
  (`current/` is the Plane mirror; the read-back never ran):

  ```
  $ ls .local/topics/pj-studyarxiv/workplan-setup/1/superdirector/
  chatlog.md
  tools
  ```

- the project workspace after the serving is still only the marker:

  ```
  $ ls -la .local/projects/studyarxiv/
  -rw-r--r--  1 eiji  staff  80 Aug 26 16:34 README_PROJECT.md
  ```

- no `studyarxiv` in Plane (23 projects listed, read back through
  `load_plane_config` + the projects API) and no `studyarxiv*` in Gitea
  `autodev` (50 repositories listed). Both lists are in this episode's
  scratch check; neither contains the slug.

The run itself (`.local/agent/superdirector/run-0081.json`) is
`"outcome": "done"`, 9 turns, 26.0 s, `claude_code`/`anthropic/claude-sonnet-5`,
$0.128. Its reply, message 2409:

> Here's the current state, no changes made:
>
> **studyarxiv's workspace is empty.** The only thing inside
> `.local/projects/studyarxiv/` is `README_PROJECT.md`, which reads:
>
> > "pattern-managed; folders are created by the workplan on the developer's request"
>
> So there are no repos, source folders, or work logs yet — nothing has been
> provisioned. It's not its own git repo either; it lives as a subdirectory of
> the `agautolab` repo (origin `github.com/iwaag/agautolab`), which is why any
> git history you'd see here is `agautolab`'s own commits, not project-specific
> work.
>
> In short: this is a clean slate, waiting on a project pattern (via
> `autolab doc patterns`) or a mission to create its actual structure.

Worth noting for Step 4: the run reached the pattern document on its own, from
the guide alone, on a question that did not ask it to. Step 1 and Step 2 are
connected in the agent's hands, not just on disk.

(The workspace is under the gitignored `.local/`, so "a subdirectory of the
agautolab repo" is true of the path and not of the contents — nothing there is
tracked. Harmless in this reply; a thing the eventual README_PROJECT.md may
want to say plainly.)

## Plane: the open decision

A pattern-managed project gets **no Plane project this episode**. Three
consequences, all live now:

1. The mission ledger is unavailable to it. A `workplan-` serving that writes
   `plan.md` gets a reply saying so and the file stays in the generation
   workspace; nothing is registered, no `work-<label>` channel is built, no
   `workrun-` topic is opened.
2. Therefore **a pattern-managed project cannot execute a mission** through
   `serve_run` at all — a `workrun-` topic has nothing to bind to. Project
   *preparation* is the whole of what a pattern project can do today.
3. `serve_bmining` is unaffected in code but useless in practice: it returns
   the no-direction reply, because a pattern project need not have
   `direction/`.

Which is fine for this episode — Step 4 only prepares a workspace — and is
exactly the decision to take next. The candidates, for Step 5: give every
pattern project a Plane project after all (cheapest, keeps one ledger); make
the ledger a folder in the workspace the pattern names (fits patterns, loses
the board); or decide that pattern projects are preparation-only until the
ledger question is answered on its own terms.

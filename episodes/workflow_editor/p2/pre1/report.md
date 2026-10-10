# workflow_editor p2/pre1 — report

## Summary

The p1 editor is ready for the p2 trial. A person opens an authoring area in
VS Code and asks the IDE agent to establish a project and create one
workflow. The person adjusts it in the browser, and the agent continues from
those adjustments.

The completion criteria pass:

- the readiness checks pass;
- the operation matrix has no unexplained asymmetry;
- the person's area `pj-agdev/.local/workflow-editor-p2/` is prepared with
  concrete start instructions and **no project in it**.

The human-led p2 trial has not been done. The rehearsal used a stand-in agent
and scripted browser actions on separate data.

**Writers take turns** (unchanged). This phase adds no revisions, merging,
conflict resolution or concurrent-writer guarantees.

Step reports:

- [report1](report1.md): baseline and audit
- [report2](report2.md): create/register and CLI
- [report3](report3.md): IDE handoff
- [report4](report4.md): update reliability
- [report5](report5.md): rehearsal

## Starting the p2 trial

From `pj-agdev/.local/workflow-editor-p2/`. This is also in its `START.md`,
and `wfe setup` prints it.

1. In a terminal, start `./wfe serve` and keep it running. It builds the UI
   and serves the UI and API.
2. Open http://127.0.0.1:8097/. Port 8097 is used because the p1 service
   still runs on 8095 with the p1 fixture.
3. Open `pj-agdev/.local/workflow-editor-p2` in VS Code and start the IDE
   agent there. It reads `AGENTS.md`; Claude Code reads it through
   `CLAUDE.md`. `.claude/settings.json` allows `wfe`, Git and edits in the
   area.
4. Give the first prompt, filling in the purpose:
   > Create a project in this authoring area for <purpose>. Clarify its
   > intent and goals, establish the repositories it needs, and create one
   > workflow. Make the result available in the editor for me to review and
   > adjust.

`./wfe setup <area>` refreshes the generated files. It keeps the registry,
the projects, and any generated file whose stamp was removed.

## What was built

All of it is in `pj-agdev/experiments/workflow_editor/`.

| Deliverable | Result |
| --- | --- |
| 1. Create and register | `server/create.ts` and `server/registry.ts`, shared by the browser home page (`#/`: "Create project", "Register workspace") and `wfe create` / `wfe register`. Creation makes a local Git root, `project.yaml`, `.gitignore` (`.local/`), and devdocs as a submodule with a relative URL. It makes two initial commits with the person's identity, or creates nothing if the identity is missing. It refuses an occupied destination and resumes a partial creation without deleting anything. Registration is idempotent and gives diagnostics with fixes. A missing registry gives an empty setup state; a malformed one gives a readable error and is never overwritten. |
| 2. One tool surface | `wfe` (`cli/wfe.ts`): `create`, `register`, `list`, `status`, `add-repo` (with `--new`), `workflow new`, `validate`, `approve`, `arrange`, `serve`, `setup`. It is a thin client of the same modules as the service, and no command needs the service. `--json` gives structured output; exit codes are 0/1/2. Each command's `--help` says what it reads or changes. Auto-arrange moved to `shared/layout.ts` so both routes use it. |
| 3. IDE entry point | `wfe setup` and `templates/AGENTS.md` / `START.md`. The guide states who the agent is, who it works with, and its tools, and then only facts. It has no quotas, cost rules or approval gates. Three facts were added from rehearsal evidence, with that reference. |
| 4. Update reliability | Merged refresh needs; a full re-read on every (re)connection; visible outages; deleted and renamed workflows shown read-only, with a link to the renamed file; Git-state and registry watching; repository inspection cached on the Git fingerprint; a Refresh action. |

## Operation matrix

| Operation | Person | IDE agent |
| --- | --- | --- |
| Create project / register workspace | Home → Create project / Register workspace | `wfe create`, `wfe register` |
| Inspect workspaces, Git state, validation | Home, project view, workflow editor | `wfe list`, `wfe status`, `wfe validate` |
| Edit intent, goals, graph, descriptions | Project view, workflow editor, or VS Code | File editing (`docs/contract.md`) |
| New workflow | Project view → New workflow | `wfe workflow new`, or write the file |
| Add submodule (existing or new local repository) | Project view → Add submodule (option: create a new local repository), or Git | `wfe add-repo <path> <location>` / `--new`, or Git |
| Approve | Workflow editor, with the declared approver | `wfe approve … --approver <name>` |
| Auto-arrange | Workflow editor → Auto-arrange, then Save | `wfe arrange` |
| Rename/delete files, repair YAML | VS Code | File editing |
| Diffs, commits, history | VS Code / Git | Git |
| Start the editor | `wfe serve` | `wfe serve` |

Remaining asymmetries, all explained:

- Browser creation is bounded to the authoring area, which keeps the
  service's write boundary. The CLI takes any destination; it is the
  caller's own process.
- In the browser, auto-arrange changes a draft that is then saved; the CLI
  writes the file directly. The positions are identical.
- The person sees changes live in the editor. The agent sees them in the files
  and through `wfe status` / Git. That is a difference of medium, not of
  capability.

## Update measurements

On this machine, with the representative project (3 submodules, 1 workflow)
and a synthetic large one (20 submodules, 150 workflows):

| | Baseline | Final |
| --- | --- | --- |
| Save → visible, p50 / max (4 view×data cases, 60 edits) | 0.67–1.03 s / ≤1.38 s | 0.61–0.69 s / ≤1.17 s; none over 2 s |
| Idle with a viewer, large: CPU / file reads | 17.7 ms/s / 152 per s | 8.1 ms/s / 13 per s (plus 160 stats) |
| Per edit, large: Git subprocesses / CPU | 125–247 / 197–316 ms | 11–18 / 49–56 ms |
| No viewer | no work | no work |
| Lost or invisible update cases (of 21 probes) | 13 | 0 |

Intervals:

- **Definitions: 1 s, unchanged.** Polling is enough: the target is met, and
  the failures were lost refreshes and Git cost, not latency.
- **Git state: 3 s**, with one `git status` per tick, about 20 ms
  (representative) or 110 ms (large), plus a Refresh action.
  `--poll-ms` and `--git-poll-ms` change them.

## Limitations

- Measured on one machine (Apple silicon, local SSD). A save appears within
  one poll interval plus up to about 0.2 s; Git-state changes within about
  3 s. Work done by child Git processes is not in the service CPU figures.
- Writers take turns. Two writers between the service's read and its rename
  are not detected. Two browser tabs on the same file are two writers.
- The approver is declared, not authenticated, as in p1.
- The rehearsal agent was a subagent with a rule against reading the
  implementation, not a person's IDE session, and its questions had no one to
  answer them. In p2 the person and the real IDE agent will produce the real
  evidence.
- Recorded and not acted on: the word "approved" has two meanings
  (definition approval vs. a person approving results); a transitively
  redundant edge gets no warning; there is no `wfe diff` (Git is the diff
  route, given a commit).
- The p1 browser checks still default to the shared p1 fixture and the dev
  server. `WFE_FIXTURE`/`WFE_URL` point them elsewhere; pre1 ran them that way
  so the running p1 service was not disturbed.

## Not in pre1 (unchanged)

Workflow execution, agent chat in the editor, Gitea, agdevworld integration,
existing-project migration, remote deployment or discovery, new
authentication or permission systems, comprehensive browser Git management,
and concurrent editing.

## Notes

The input "accepted p2 execution proposal" was not found as a file; this phase
followed `plan.md`.

Deus ex machina note: in the rehearsal I played the person, not the agent,
and did none of the agent's work. Recording the browser approvals as
"Rehearsal Person" in the parity check was the person's action, on
rehearsal data.

The rehearsal area `pj-agdev/.local/workflow-editor-p2-rehearsal/` and the
raw measurements in `pj-agdev/.local/workflow-editor-measure/` stay in place
(ignored).

`devdocs/episodes/workflow_editor/p2/braindump.md` is still untracked; it was
left for its author to commit.

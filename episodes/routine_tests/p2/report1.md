# routine_tests p2 step 1 — the studyuspolitics workspace

Date: 2026-09-13 JST (timestamps below are UTC, as Zulip records them).
Plan: [plan.md](plan.md) step 1. Source: [branindump.md](branindump.md).

## Refreshed prerequisites

Read-only, through `nctl`, immediately before any change:

- `nctl status --json` — `ok: true`. Nautobot 3.1.3 reachable at the local
  URL, authenticated, intent catalog and intent GraphQL present; one Celery
  worker running with no pending jobs; all five submodules clean. Node
  observation dumps are 2.8 h old for the live hosts (`agfixture` is the
  long-retired one at ~1083 h).
- `nctl drift --json` — `ok: true`, `errors: []`, summary `converged: 43`.

That is an infrastructure observation, not proof of agent readiness. The
agent half was established separately: the launchd listeners for autolab,
Front, forge, arxivsage, cagent and the agentroom relay were all running,
and autolab's own listener log shows it serving this step's topics.

## What already existed (checked before creating anything)

| Thing | Checked | Result |
|---|---|---|
| Routine channels | `agentchat channels --prefix routine` | `publish`, `study-realworld`, `ghtrends`, `papers`. **No `study-uspolitics`.** |
| Project channels | `agentchat channels --prefix pj-` | `ghtrends`, `mediagen`, `papers`, `studyarxiv`, `studyrealworld`. **No `studyuspolitics`.** |
| Workspaces | autolab's projects directory | `ghtrends`, `mediagen`, `papers`, `refactorp1`–`p3`, `runsmoke1`, `studyarxiv`, `studyrealworld`. **No `studyuspolitics`.** |
| Archived workspaces | autolab's archived-projects directory | 15 entries, none of them a `studyuspolitics` alias. |

So nothing was reused and nothing was overwritten: `studyuspolitics` /
`study-uspolitics` were free names.

**The publication remote was inspected independently**, by cloning it into a
throwaway directory rather than trusting the plan's earlier note:
`https://github.com/iwaag/study-uspolitics.git` holds exactly one commit,
`569a8e3e390239880c11affc0f1a5a1d4dc0c869` ("Initial commit", authored
2026-09-13 00:15:33 +0900), on a single branch `main`, containing one file —
a 121-line CC0 1.0 `LICENSE`. No other branches, no pending publication work,
nothing divergent. It is *not* an empty repository, and its history must be
preserved.

## Manual setup, and the handoff note it owes

Two things were done by hand, both before the first project serving, because
neither can be done from inside the system:

1. **The pattern marker.** `README_PROJECT.md` was written into the new
   workspace declaring a study project and the requested folders, with the
   file itself labelled as setup rather than researcher knowledge. This is
   what makes `agautolab.project_init.init_project()` return
   `pattern-managed, folders untouched` (`project_init.py:313`) instead of
   creating the legacy layout on the first serving, before the agent has
   read the request.
2. **The project channel.** `#pj-studyuspolitics` (stream **152**) was
   created with the four participants the other project channels carry:
   Developer (8), autolab-agstudio1 (11), Front (15), Opsroom Observer (22).

> **Deus Ex Machina note:** *did the workspace pattern marker and the project
> channel creation for agent autolab — handoff candidate.*

Channel **folder** filing was deliberately left alone, to test the documented
behaviour that a project channel files itself. It did: after the first
serving, `#pj-studyuspolitics` sits in a new channel folder
`pj-studyuspolitics` (id **17**), minted by the agent, and the work channel
it opened (`work-m6342`, stream 153) inherited it.

## The setup mission

One request at autolab's documented entrance for project work —
`#pj-studyuspolitics` › `workplan-setup-studyuspolitics-workspace`, message
**6340** — asking for the four-location layout, the standard internal
repository route for `main/`, a clone of the supplied publication repository,
and a completed `README_PROJECT.md`. It said explicitly that no research was
being asked for, that `study-realworld`'s `sources/` layout and report
template must not be imported, and that `publish/` must be inspected but
never pushed, reset or reconciled.

| Message | What |
|---|---|
| 6340 | The request (Developer) |
| 6342 | The `[mission]` note — the mission *is* this message id, so the work channel is `work-m6342` |
| 6343 | The plan: one bounded task, correctly refusing to start without a "start" |
| 6349 | Plan announcement to the Developer |
| 6350 | Task start (Developer), into `#work-m6342` › `workrun-task1-m6342` |
| 6353 | The task result |
| 6355 | The completion report |
| 6356 | autolab resolved the topic |

Two autolab runs, both on the defaults — `claude_code`,
`anthropic/claude-sonnet-5`, `exec_source: default`:

| Role | Record | Duration | Cost |
|---|---|---|---|
| `superdirector` (planning) | `run-0191` | 56.1 s | $0.1949 |
| `supercoder` (task) | `run-0263` | 68.6 s | $0.2336 |
| | | | **$0.4286** |

## Verified against the tree, not the report

| Claim | Verification |
|---|---|
| `main/` repository identity | `origin` = `http://agstudio.local:3000/autodev/studyuspolitics.git` — the standard internal Gitea route, not the supplied GitHub URL. |
| `main/` initial revision | `d817882` "Initialize main working knowledge repository"; `git ls-remote origin` returns `d8178821…` for `HEAD` and `refs/heads/main`, so it really is pushed. |
| `main/` contents | Exactly four files: `.gitignore`, `README.md`, `methods/.gitkeep`, `reports/.gitkeep`. Working tree clean. |
| No imported layout | No `sources/`, no `INDEX.md`, no report template. `main/README.md` says an index is optional and "not a required structure". |
| `publish/` identity and state | `origin` = the supplied GitHub URL; log shows the single pre-existing commit `569a8e3`; `git ls-files` = `LICENSE` only; working tree clean; **no new commit, nothing pushed**. |
| Source/publication separation | Two different origins, two independent histories; `publish/` is not `main/`'s origin. |
| Marker survives a later serving | `README_PROJECT.md` is present at the workspace root, which is the condition `project_init` checks. |

`README_PROJECT.md` was rewritten by the agent into a real description: the
subject, `main/`'s identity and initial revision, `publish/`'s identity and
what its history held at clone time, and the commit/push responsibilities
(agents commit and may push `main/`; agents may commit `publish/` locally but
never push it).

## Observations worth keeping

- **`README_PROJECT.md` is tracked nowhere.** The workspace root sits outside
  any repository, so the pattern marker and workspace description exist only
  on this machine. autolab said so plainly rather than implying it was
  committed. This is the same for every study project, so it is a property of
  the pattern and not a fault of this one — but it means a later serving
  depends on a file with no backup.
- **The task was resolved without the requester's agreement.** autolab's own
  introduction says it does not close a task until the requester posts that
  they agree it is complete; here it posted the report, marked the task
  completed and resolved the topic in the same serving. The work was in fact
  correct, so nothing was lost, and it is recorded here as an observed
  contract deviation rather than acted on.

## Step 1 conclusions

1. The workspace exists at the requested layout, with `methods/` and
   `reports/` as the only two research-content directories inside `main/`.
2. `main/` is an independent internal repository, committed and pushed;
   `publish/` is the developer-supplied publication clone, untouched and with
   its existing history intact.
3. The two manual acts are recorded above with their handoff note; everything
   else in this step was done by autolab through its ordinary interfaces.
4. Nothing about the research has been asked for or done yet.

# Step 1 — the `studyrealworld` project

The project exists as four surfaces (Zulip channel, Plane project, one Gitea
repository, workspace clones) plus the pre-existing GitHub publication
repository. One autolab serving built the workspace and its skeleton; the
harness-side bootstrap around it was Developer work.

## The bootstrap, as the Developer

**Channel.** `#pj-studyrealworld`, stream **123**, created with
`ZulipClient.create_channel` from the Developer credential, description
`autolab project: studyrealworld (study pattern)`, principals 8 (Developer)
and 11 (autolab bot), in a **new channel folder 12** named
`pj-studyrealworld`. Folder-per-project is the settled standard since
2026-09-04, so the folder was minted rather than reusing folder 1.

**Marker.** The pattern-managed marker file was written into the workspace
root — the same one line `ghtrends` and `mediagen` were declared with:

```
pattern-managed; folders are created by the workplan on the developer's request
```

**State before the request.** Plane: 27 projects listed, none matching
`studyrealworld`. Gitea `autodev`: repository search for `studyrealworld`
returned nothing. Workspace: the marker file alone.
`https://github.com/iwaag/study-realworld.git` already existed at
`eacc04b` — one commit, holding a CC0 `LICENSE` and nothing else.

**The request** — `#pj-studyrealworld › workplan-create`, message **5342**,
posted as the Developer. It asked for a study-pattern project, gave the
standard-name route for `main/` and the GitHub URL for `publish/`, spelled
out the `sources/` + `reports/` layout to commit as an empty-but-usable
skeleton, defined what a source file and an investigation record, stated the
publish-ready constraint on `main/`, and ended with *"Do not go looking for
sources yet: that is the `study-realworld` routine's work"*.

**The reply** — message **5344**, 57 s later: `main/` created via
`autolab project init-repo main` → `autodev/studyrealworld`, committed and
pushed with the skeleton; `publish/` cloned from the GitHub URL with the
origin kept; `README_PROJECT.md` rewritten; no sources sought; no `plan.md`
and no Plane Sub-Work, because this was preparation and not a mission.

## Verification

| surface | before | after |
|---|---|---|
| Zulip channel | absent | `pj-studyrealworld`, stream 123, folder 12 |
| Plane project | 27 projects, no match | 28, `Studyrealworld` (`37b37d22…`), not archived |
| Gitea `autodev/studyrealworld` | absent | created, holds commit `2292bf6` |
| Gitea `autodev/studyrealworld-publish` | absent | **still absent, deliberately** — `publish/` is the GitHub clone, so the standard second repository was never needed |
| workspace | marker only | `main/` (pushed), `publish/` (GitHub clone at `eacc04b`) |
| `README_PROJECT.md` | 80-byte marker | rewritten (below) |

`main` at `2292bf6` *"Skeleton: sources/, reports/, README"*:
`README.md`, `sources/INDEX.md`, `reports/README.md`, `.gitignore`
(ignoring `.local/`). `sources/INDEX.md` is a header plus an empty table
whose preamble already carries the candidate-vs-verified distinction the
plan asks for:

> A source is *verified* once its document retrieval recipe has actually
> succeeded (see the source file's worked example); until then it is a
> *candidate*.

`main/README.md` restates the layout, what a source file is, what an
investigation records (including the per-document metadata list), and the
publish-ready constraint — so a reader who has only this repository has the
conventions. That matters because `README_PROJECT.md` lives in an ignored
directory and never reaches any repository.

## `README_PROJECT.md` as landed

It records both folders with their repositories, the subject, the
publish-ready rule, and — inherited from the pattern document — the line
*"Never pushed by an agent; the developer reviews and pushes it by hand."*

**That line is wrong for this project** and is a finding rather than a
defect of the run: the pattern document says exactly that, and the run had
no reason to doubt it. This session's instruction is that reviewed
publication content is pushed to the specified GitHub origin. Step 4 (the
publication exercise) is where that is corrected, in the `publish` routine's
standing request and in `README_PROJECT.md` together — not here, because a
setup run's output is better evidence when it is left as the pattern
actually produced it.

## Run record

`superdirector/run-0161`: `anthropic/claude-sonnet-5`, **54.7 s**,
**$0.2559**, `"outcome": "done"`, channel `pj-studyrealworld`, topic
`workplan-create`. For comparison the same day's `ghtrends` planning run
(`run-0160`) was 50.8 s / $0.1778, and `mediagen`'s bootstrap a week earlier
was 42.5 s / $0.187 — this request was longer and carried more conventions,
which is where the extra cost went.

Listener log, the two lines a pattern-managed serving produces:

```
2026-09-08T16:31:06Z serving 'pj-studyrealworld'/'workplan-create'
2026-09-08T16:31:06Z workplan topic 'pj-studyrealworld'/'workplan-create'
```

## Environment check

`nctl status` before starting: Nautobot 3.1.3 authenticated, 1 celery worker,
0 pending jobs, 8 hosts collected, all five submodules clean — `ok: True`.
All the launchd jobs this phase depends on are loaded: the autolab listener,
the Front listener, the routine dispatcher, the agentroom relay.

## Deus Ex Machina

- Created the Zulip channel, its folder and the pattern-managed marker, and
  posted the setup request, as the Developer — the standard division of
  labour for a pattern-managed project, and the same one `ghtrends` and
  `mediagen` used. Not a handoff candidate: a `pj-` channel is opened by a
  human by design.
- Everything inside `main/`, `publish/` and `README_PROJECT.md` was written
  by the autolab run.

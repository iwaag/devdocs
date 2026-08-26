# Step 1 — project creation through the pattern flow

`ghtrends`, built by the agent from the `study` pattern on one request, with
the Plane project created by the changed gate and nothing else touched by
hand.

## The code change: the pattern-managed gate ensures a Plane project

`init_project`'s early return for a workspace holding `README_PROJECT.md` no
longer returns before touching anything — it ensures the Plane project first:

```python
    if (project_root / PATTERN_MARKER).exists():
        ensure_plane_project(load_plane_config(), project)
        return PATTERN_MANAGED_RESULT
```

`PATTERN_MANAGED_RESULT` says so: `"pattern-managed, Plane project ensured,
folders untouched"`. It is the only code change of the episode. This is
stopgap **option 1** from the `agautolab/project_pattern` phase report,
approved by the Developer on 2026-08-26; the pattern-declared ledger design
(option 2) stays open.

Tests, all three step-order assertions the plan asked for:

- `test_init_project_ensures_only_the_plane_project_of_a_pattern_managed_workspace`
  — `calls == [("plane", "studyarxiv")]`, `load_gitea_config` fails the test if
  called, and the workspace still holds nothing but the marker.
- `test_init_project_pattern_marker_wins_over_an_existing_layout` — a project
  that grows the marker later gets its Plane ensure and no Gitea call.
- `test_a_marked_workspace_is_served_with_only_its_plane_project_ensured`
  (listener level) — the serving's call order is now
  `whoami, write, history, plane, superdirector, history, write, history`:
  the Plane project is ensured before the superdirector runs, and the Plane
  read-back (`write_mission_workspace`) is still skipped.

`201 passed` (the same count as at the end of `project_pattern`; two existing
tests were rewritten, none added).

Commits: agautolab `e526705`, pushed to GitHub; pj-agdev pin `de8bd0c`,
pushed. Listener restarted with
`launchctl kickstart -k gui/$(id -u)/com.agdev.agautolab-zulip` (log line
`2026-08-26T10:22:21Z listening as user_id=11`).

## The ritual, by hand as the Developer

Channel, created with `ZulipClient.create_channel` from
`.local/zulip/developer.env`, folder 1 (where every other project channel is):

```
me: 8 user8@agstudio.local
create: {'result': 'success', 'subscribed': {'8': ['pj-ghtrends'], '11': ['pj-ghtrends']},
         'already_subscribed': {}, 'new_subscription_messages_sent': True}
```

Marker, the same 80-byte line `studyarxiv` was declared with:

```
$ mkdir -p .local/projects/ghtrends
$ cat .local/projects/ghtrends/README_PROJECT.md
pattern-managed; folders are created by the workplan on the developer's request
```

State before the request: **no** `Ghtrends` in Plane (23 projects listed) and
**no** `ghtrends*` in Gitea `autodev` (72 repositories listed over two pages).

## The request and the reply

`#pj-ghtrends` › `workplan-create`, posted as the Developer (message 2422),
the plan's text verbatim:

> studyパターンのプロジェクトを作って。リポジトリは標準の名前でいい。mainにはGitHubのトレンド解析で選んだリポジトリの概要まとめを蓄え、publishにはレビュー済みだけを置く。

Reply (message 2424, 22 s later):

> I've set up the project on the "study" pattern:
>
> - `main/` — cloned as `autodev/ghtrends`, for storing summaries of
>   GitHub-trending repos chosen from trend analysis.
> - `publish/` — cloned as `autodev/ghtrends-publish`, for reviewed summaries
>   only, moved from `main/` after your explicit approval (you push it by
>   hand).
>
> `README_PROJECT.md` is updated to record both folders. This was project
> setup, not a mission — no plan.md or tasks were created. Let me know when
> you're ready to give the actual work mission (e.g., the trend-analysis and
> summarization task).

Run record `run-0086`: `superdirector` / `claude_code` /
`anthropic/claude-sonnet-5`, 7 turns, 20.7 s, $0.122, `"outcome": "done"`.
Listener log, the two lines a pattern-managed serving produces:

```
2026-08-26T10:23:15Z serving 'pj-ghtrends'/'workplan-create'
2026-08-26T10:23:15Z workplan topic 'pj-ghtrends'/'workplan-create'
```

## Checklist (project_pattern Step 4)

### Standard-name repos created via `init-repo` — pass

```
$ find .local/projects/ghtrends -not -path '*/.git/*' | sort
.local/projects/ghtrends
.local/projects/ghtrends/README_PROJECT.md
.local/projects/ghtrends/main
.local/projects/ghtrends/main/.git
.local/projects/ghtrends/publish
.local/projects/ghtrends/publish/.git

$ git -C main remote -v
origin	http://agstudio.local:3000/autodev/ghtrends.git (fetch/push)
$ git -C publish remote -v
origin	http://agstudio.local:3000/autodev/ghtrends-publish.git (fetch/push)

$ git -C main status -sb
## No commits yet on main...origin/main [gone]
$ git -C publish status -sb
## No commits yet on main...origin/main [gone]
```

Gitea confirms both repositories were created **by the run**, at `10:23:30Z`,
fifteen seconds into the serving, both `"empty": true`. No repository URL was
given in the request, so this is the pattern document's "if not specified"
line taking `autolab project init-repo`'s standard names — the second live
proof of it, and the first where *both* names were derived.

Both clones sit on the **unborn branch** the `project_pattern` report noticed
in passing (`origin/main [gone]`). Step 2 is where that first matters.

### `README_PROJECT.md` written — pass, with the appended-marker flaw

```
pattern-managed; folders are created by the workplan on the developer's request

Pattern: "study"

- `main/` — summaries of GitHub-trending repositories chosen from trend
  analysis. Repository: `autodev/ghtrends` (local Gitea).
- `publish/` — only the summaries from `main/` that have been reviewed,
  moved here on the developer's explicit approval. Never pushed by the
  agent; the developer pushes it by hand after review. Repository:
  `autodev/ghtrends-publish` (local Gitea).
```

- describes both folders, names the pattern, carries the no-push rule — yes.
- **the declaration line is still the first line of the document.** The run
  *appended* rather than replaced, exactly as `studynourl` did: a sentence
  written for `init_project` is now addressed to the reader. That is the
  `project_pattern` report's "the declaration text becomes documentation"
  flaw, reproduced 2 of 3 times. Unfixed here by design (automating the
  declaration is out of scope), and one more data point for the
  `autolab project declare` candidate.
- **The plan's question — did it describe the folders in its own words?**
  `main/` is fresh this time, so the README-copy finding could not arise as it
  did on `studyarxiv`. What it wrote is neither the pattern document's
  sentence nor an observation of an empty clone: it is *the request*
  paraphrased ("summaries of GitHub-trending repositories chosen from trend
  analysis"). `publish/`'s line is close to the pattern document's wording.
  So: own words where the request supplied them, document's words where it
  did not. Nothing to fix, and it says the earlier finding was about the
  *source* the agent reaches for, not about copying as such.

### Nothing pushed — pass

Both clones are unborn; `origin/main` exists in neither repository. Gitea
reports both `"empty": true` after the serving.

### No mission invented — pass

No `plan.md`, no Plane Sub-Work, no `workrun-` topic. The reply says so and
asks for the mission, which is Step 2.

## The Plane project appeared without a human call — verified

Plane listed 23 projects before the request and **24** after, the new one
being `Ghtrends`. Nothing between the two reads created it but the serving:
the changed gate ran `ensure_plane_project` at the "project setup" step,
before the agent read a word of the request. The agent's reply does not
mention Plane, and nothing in the workspace records it — which is worth one
line for Step 5: the ledger a pattern project now has is invisible to the
project itself.

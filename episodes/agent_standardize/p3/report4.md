# p3 step 4 — proof

All three success criteria met, live, in one sitting after the Step 3
cutover. Nothing was deferred that the plan asked for.

## Criterion 1 — the p1 smoke in the new vocabulary

Omni opened `agforge-agstudio1/assetplan-p3-smoke-icon` asking for a 64×64
pixel-art green potion bottle icon.

| time (UTC) | what |
|---|---|
| 15:14:33 | listener: `serving 'agforge-agstudio1'/'assetplan-p3-smoke-icon'` |
| 15:14:48 | front run wrote `required_items.md` + `toolsets.csv` (`toolset-image`) |
| 15:15:16 | generator planned it; **`created F2-15 "Plan: 64x64 pixel-art green potion bottle icon" in FreeForge`** |
| 15:19:23 | Omni posted `go` in `agforge-agstudio1/assetrun-p3-smoke`; listener: `serving …/'assetrun-p3-smoke'` |
| 15:21:07 | run finished |

The `assetrun-` reply, verbatim:

```
running "Plan: 64x64 pixel-art green potion bottle icon"

result/ holds 1 file(s); zipped and uploaded as
files/2026-08-21/1c9745265d304f739ed2bc8bea663a41.zip

delivered to agforge-agstudio1/assetplan-p3-smoke-icon

work F2-15: commented yes, Done yes
```

And the delivery really landed in the origin topic — the last post in
`assetplan-p3-smoke-icon` is the presigned download plus its
`[S3KEY] files/2026-08-21/1c9745265d304f739ed2bc8bea663a41.zip` footer.
`.local/agentws/5a43d1ab-…/generator/result/icon.png` is the artifact on
disk. **Plan → Work → execute → deliver → Done, entirely in the new
vocabulary.**

## Criterion 2 — Front, with zero agfront code change

Developer, in `#front` topic `front-p3-vocab-check`: *"I want a 48x48
pixel-art icon of a wooden treasure chest … Can anyone here make that for
me?"*

Front's first reply (15:23:18) — the proposal its p2 guide requires:

> It looks like `agforge-agstudio1` handles exactly this kind of request — it
> generates media assets like images. **I'd need to open an `assetplan-…`
> topic** in its channel describing the icon …
>
> Want me to go ahead and send that request to agforge-agstudio1?

Permission granted; at 15:24:26 Front opened
**`agforge-agstudio1/assetplan-treasure-chest-icon`** and posted the spec as
the Front bot. agforge picked it up 16 seconds later and produced
`required_items.md` + `toolsets.csv`.

**This is the payoff proof.** The only thing that carried the new prefix into
that run is the harvest. From
`agfront/.local/topics/front/front-p3-vocab-check/2/front/tools/agents.md`,
generated `2026-08-20T15:24:16+00:00`:

```
## agforge-agstudio1

# agforge

This instance generates requested media assets such as images, video, music,
and speech. To request an asset, open an `assetplan-…` topic in this
instance's `agforge-agstudio1` channel and describe what you want.

---
Posted: 2026-08-21
Revision: `2469d69`
```

agfront's code and guide were **not** changed for this: its only p3 commit
(`cc36fd0`) touches three test fixtures and two doc comments, none of which a
run ever reads. The intro is the contract, and it held across a breaking
rename with no deploy on the consumer's side.

## Criterion 3 — the grep

`grep -rn 'create-\|runcreate-\|mission-\|"run-'` over the four repos'
`src` / `params` / `agent` / `tests`:

```
===== agforge
tests/test_role_run.py:41,51        run-0001.json
===== agautolab
agent/gateway.py:173,181            run-{n:04d}.json
src/agautolab/mission.py:23         "a mission-level cancel"
src/agautolab/zulip_listener.py:357 "One mission-planning run"
tests/test_gateway_window.py:35     run-0001.json
tests/test_zulip_listener.py:63,297,336,472,563   ("create-channel", …)
===== agfront
tests/test_role_run.py:43,53,257    run-0001.json
===== pyagag
src/agag/topics.py:91,93            run-{number:04d}.json
tests/test_topics.py:203,204,205    run-0001.json / run-0002.json
```

Three false-positive classes, no live vocabulary:

1. **`run-NNNN.json`** — the `ag.agent-run.v1` record filename. Not a topic
   prefix; the plan already excused it in the pyagag survey.
2. **`("create-channel", …)`** — a fake Zulip client's *call label* in
   agautolab's tests, recording that `create_channel` was called. Left alone
   rather than renamed to flatter a grep.
3. **`mission-level` / `mission-planning`** — English compounds on the
   surviving domain noun. "Mission" is still what a `workplan-` topic plans;
   only the prefix moved.

### One deviation from the plan: pyagag did change

The plan said "pyagag: no change", and its *source* did not change — the
sweep machinery is prefix-agnostic and stayed so. But its **tests** used the
four retired prefixes as their example topic names (~90 hits), and
`test_chat.py`'s leak guard listed `create-` among the routing details
`agentchat --help` must never hand out. A guard naming a dead prefix guards
nothing, and a shared library whose tests teach the wrong vocabulary is a
trap for the next reader. Fixtures updated (`pyagag 1442783`), 254 tests
green. This is why criterion 3 is now clean rather than noisy.

## What the first live run caught

The `assetplan-p3-smoke-icon` run logged:

```
2026-08-20T15:14:33Z create topic 'agforge-agstudio1'/'assetplan-p3-smoke-icon'
```

The *handler's own log string* still said "create topic". The step-1/2 renames
were shaped around prefixes and identifiers, and this was prose inside an
f-string — along with autolab's `"run topic"`, `"mission topic"`, and the
`WRONG_PLACE_REPLY` a misplaced `workrun-` topic gets posted back to it.
Runtime output is the first place a reader meets the vocabulary, so it was
worth a follow-up: `agforge 2b69bea`, `agautolab 0892a68`. Both listeners
were reloaded onto the fix afterwards.

**Failure-farming note.** No amount of re-reading the diff found this; one
live run did, in its first log line. Documenting it here rather than
pretending the rename was clean on the first pass.

## Commits

| repo | commit | what |
|---|---|---|
| agforge | `2469d69` | step 1 rename |
| agforge | `2b69bea` | log string |
| agforge | `5752907` | README chat contract |
| agautolab | `5675943` | step 2 rename |
| agautolab | `0892a68` | log strings + prose |
| agautolab | `9834691` | README prefix table |
| agfront | `cc36fd0` | fixtures + comments only |
| pyagag | `1442783` | test fixtures + leak guard |
| pj-agdev | `aced029`, `78c2382` | submodule bumps |
| devdocs | `c74026b`, `6e13ded`, `a87ec34` | reports 1–3 |

## Living docs updated

- **`agforge/README_DEV.md`** — the chat contract names both prefixes and
  what each *does* (plan vs execute), plus a line recording the p3 rename and
  the absence of any shim. Also fixed a sentence that read as if old-prefix
  fallbacks still existed: the two surviving routes (other subscribed
  channels, DMs) are *inherited entrances*, not old-vocabulary compatibility.
- **`agautolab/README.md`** — the chat-entrance bullet now carries a table of
  all four sweep prefixes, where each is swept, and what each means, with
  `bmining-` marked explicitly untouched.
- **`pj-agdev/.local/devenv.md`** — the agforge-listener section, the retired
  agfront `create.md` paragraph, and the FreeForge-era entrance line. The
  FreeForge line keeps "(`create-…` until p3)" because that section is
  explicitly about a retired workflow and a reader may be holding an old
  transcript.
- **Memory** — `guides-carry-no-topic-vocabulary`: the guides carry no
  prefixes at all, so a routing-vocabulary change is a code change and never
  a prompt change. That is *why* criterion 2 could work.

## Deferred

- **Collapsing `assetplan-asset_<issue>`.** The `asset_` substructure is
  still there; the plan scoped it out and nothing in p3 argued for it.
- **The `--limit agstudio` playbook run** (see report2): it fails in
  `claude_code_agent` on a user-scoped npm path this Mac does not have. Not a
  p3 problem — the agstudio "deployment checkout" *is* the working tree —
  but it is a real placement/environment mismatch and deserves its own ENT
  episode.
- **agautolab's live surfaces are untested in the new vocabulary.** p3 proved
  the forge half end to end and the front half end to end. No `workplan-`
  topic was planned and no `workrun-task<N>-` topic executed in this episode,
  because the plan's criteria did not ask for it and the projects that would
  host one had their topics wiped at the cutover. The code is renamed,
  unit-tested (170 green) and deployed; the *live* workplan→workrun proof is
  the obvious first thing to do next.

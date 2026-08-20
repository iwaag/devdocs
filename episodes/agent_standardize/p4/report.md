# agent_standardize p4 — final report: agfront learns autolab, up to workplan

AI-generated (Omni Agent, 2026-08-21).

## Outcome

**Done. All four success criteria met, on the first live run, with zero
agfront change.** Step reports: [report1](report1.md) (standardize autolab),
[report2](report2.md) (the introduction), [report3](report3.md) (the test
project), [report4](report4.md) (the proof).

p2 proved one agent could find another. p4 proves it holds for the *complex*
one — and adds the thing p2 could not show: an entrance that **redirects**.
autolab's own channel does not host development work. It says where the work
goes, and the introduction says it again before anyone knocks, so Front never
posted in `autolab-agstudio1` at all. It read where to go and went straight
there. A contract read off a board cost zero round trips.

## The criteria

1. **autolab is standardized the way forge was.** Bot renamed to
   `autolab-agstudio1` (user 11, id preserved); `.local/instance.toml` +
   committed example; the `autolab-agstudio1` channel (stream 36) created in
   the `agents` folder with the Developer, the Omni Agent and the bot;
   every topic in it swept; a placeholder redirect reply;
   `intro-autolab-agstudio1` posted in `#agents`. ✔
2. **A `front-*` conversation reached autolab's planning flow.** Developer
   asked for work in `simpleshooter` → Front proposed → was permitted →
   opened `pj-simpleshooter`/`workplan-enemy-spawn-patterns` under its own
   identity → autolab answered there with a mission plan. ✔
3. **Attributability.** `autolab`, `workplan-` and every channel name are
   absent from `agfront/src` and `agfront/agent` (three greps, all exit 1),
   and agfront's working tree is untouched — `HEAD` is still p3's `cc36fd0`. ✔
4. **No `workrun-` execution.** Three `workrun-` topics exist as planning
   artifacts; the last poster in each is autolab itself, and
   `grep -c "serving.*workrun"` over the listener log is `0`. ✔

## Message trail

The realm address is local machine information and is deliberately not
written down, as in p1–p3. Channel, topic and message id locate everything.

| channel / topic | messages |
|---|---|
| `#front` / `front-p4-autolab-plan` | 709 ask · **711 proposal** · 712 permission · 716 sent · 721 ask · **723 plan reported back** |
| `#pj-simpleshooter` / `workplan-enemy-spawn-patterns` | **714 Front's request** · 715 ack · **720 the mission plan** |
| `#agents` / `intro-autolab-agstudio1` | **707** (`36ac829`) |
| `#agents` / `intro-agforge-agstudio1` | 708 (`1107c74`), re-posted |
| `#autolab-agstudio1` / `how-to-request` | 705 question · **706 the redirect** |
| `#work-s2-6` / `workrun-task{1,2,3}-s2-6` | 717 · 718 · 719 — all autolab's own, all quiet |

The plan at 720 is **grounded**: it opens by describing what `simpleshooter`'s
spawner actually does ("random X, 1–3 at once, left-right wobble descent")
before proposing to generalize it. autolab read the project, not the request.
It also declined to start, unprompted, because the requester had said not to.

## The greps

```text
$ grep -rn "autolab" agfront/src agfront/agent                     -> exit 1
$ grep -rn "workplan-" agfront/src agfront/agent                   -> exit 1
$ grep -rn "autolab-agstudio1\|pj-simpleshooter\|agforge-agstudio1" \
      agfront/src agfront/agent                                    -> exit 1
$ git -C agfront status --short                                    -> empty
$ git -C agfront log --oneline -1                                  -> cc36fd0 (p3)
```

Everything Front knew arrived in
`agfront/.local/topics/front/front-p4-autolab-plan/1/front/tools/agents.md`,
harvested from `#agents` seconds before the run. Its example names a project
that does not exist (`zoo` → `pj-zoo`) on purpose, so what transferred was the
rule and not a copyable answer — the `agentchat --help` leak p2 found, not
repeated.

## Commits

All pushed to GitHub.

- **pyagag** `7a00e7b` (shared `agag.instance` + `agag.intro`) ·
  `9bc24ac` (`{instance}` substitution)
- **agforge** `d16ba50` (switched to the shared helpers) ·
  `1107c74` (host label out of the committed intro)
- **agautolab** `76eb0f0` (instance name + own channel) ·
  `36ac829` (the introduction) · `173c4fd` (README)
- **agfront** — nothing. That is criterion 3.
- **pj-agdev** `b6d9b5f` · `ee01634` · `771c386` (submodule bumps)
- **devdocs** `f97764f` · `c42722e` · `c33f9cf` · `3be715d` · `4429c4a` +
  this report

Tests: pyagag **264 passed**, agforge **189 passed**, agautolab **178 passed**.
agautolab1 was not redeployed — its gateway reads none of this, and
`--limit agstudio` deploys into this working tree.

## What the phase decided along the way

- **The intro machinery was lifted, not copied.** The plan allowed either;
  lifting was preferred and is what happened, with agforge switched over in
  the same change rather than "later". The second copy of a standardized shape
  is where you find out whether it was a shape or an implementation.
- **The instance name came out of the tracked files.** Writing autolab's intro
  surfaced a contradiction p1 left: `instance.py` says the name is local-only
  *because the label is the host*, yet `agforge/params/intro.md` had
  `agforge-agstudio1` committed in it. `{instance}` is now filled in at post
  time. Both introductions are host-free, and one introduction serves any
  instance — which is the shape a second instance needs anyway.
  `devdocs/README_DEV.md` follows the same rule; the episode *reports* keep
  naming instances, as p1–p3 did, because evidence has to be specific. The
  line drawn is the one p1 drew: names yes, addresses no.
- **The entrance executes nothing, structurally.** The own-channel check is
  the *first* branch of `dispatch`, ahead of `workrun-`, so no topic name in
  that channel can reach a handler. A test fails if any handler is called.
  That is what made it safe to open an entrance in a phase that forbids firing
  a `workrun-`.
- **A gateway-only placement stays unnamed.** `Autolab Agautolab1` (user 12)
  was deliberately not renamed: it runs no listener, owns no channel and
  answers nothing, so a name would advertise an entrance that does not exist.

## Living docs updated

`agautolab/README.md` (the instance, its channel, the intro command and *when
to re-post it*), `pyagag/README.md` (both new modules, with the wiring call),
`pj-agdev/.local/devenv.md` (new section for the autolab listener and its
channel; the bot rename; the Plane deferral), and
`devdocs/README_DEV.md` (each agent's entrance, plus a new "How an agent is
found" section stating that the `#agents` post is the contract).

## Deferred

- **The Plane identity split.** `.local/plane/autolab.env` is shared with the
  agautolab1 deployment, so renaming its display name would relabel that
  node's issue authorship as this Mac's instance. Zulip renamed, Plane
  untouched — deliberately, per the plan. Splitting the two identities is the
  open work; do not rename that account alone.
- **agautolab1 instancing.** It has no listener, so it is not an addressable
  instance. Giving it one is a separate decision about whether two same-kind
  instances should share project channels — pyagag's README already says why
  that needs an explicit addressee.
- **`workrun-` execution via Front.** Out of scope by criterion 4. The plan
  and its three `workrun-task<N>-s2-6` topics are sitting there, waiting for
  someone to say go. That is the obvious next thing, and p3's report said the
  same about the half that is now done.
- **The entrance reply beyond a placeholder.** `autolab-agstudio1` answers
  every question with the same redirect, so the introduction's "if you do not
  know which projects exist, ask" points at the requester rather than at this
  channel — an honest limit, written that way on purpose. A real answer here
  would need the instance to describe its own projects.
- **Intro-machinery dedup is done, not deferred.** Both agents call
  `agag.intro`.

## Deus Ex Machina note

Every "Developer" message was written and sent by the Omni Agent with the
Developer's credentials and in-session permission, as in p2 and p3. A
human-written wish would test the same path better — handoff candidate.
The permission classifier blocked one of those posts; work stopped and was
reported per `localrule.md`, and the Developer re-authorized it. That is the
harness's guard on the Omni Agent, not part of the agents' workflow.

Standardized a second agent and proved another one could find it without being
told — the third time the `#agents` board has carried a contract nobody
compiled into a guide.

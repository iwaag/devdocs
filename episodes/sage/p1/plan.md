# sage p1 — plan

Goal: the first live sage — **arxiv sage**, an agag agent whose knowledge is
the public GitHub repository studyarxiv publishes to,
`https://github.com/iwaag/study-arxiv-trend`. It answers from that tree,
says "I don't know" honestly, and leaves `tostudy/` notes for the study
workflow. One instance, `arxivsage-agstudio1`, reachable through `entrance-…`
topics in its own Zulip channel.

Braindump: [braindump.md](braindump.md). Destructive phase: no backward
compatibility is owed to anything.

## Decisions taken here (implementer may overrule with a one-line reason)

| topic | decision |
|---|---|
| agent name | `arxivsage` (package, repo, channel `arxivsage-agstudio1`). "sage" is the pattern; each domain sage is its own agag agent, and `agag init` makes one package per agent. |
| where the code lives | new top-level repo `arxivsage/` beside `agecho/`, public GitHub `iwaag/arxivsage` (the `agag_agent` Ansible role refuses non-GitHub sources; gitea mirrors go stale) |
| knowledge source | a clone of `iwaag/study-arxiv-trend` at `knowledge/`. Layout: `README.md` (index table: id, title, local-test level) and `papers/<arxiv-id>/summary.md` (+ `manual.md` for runnable papers, `test.md` where a local test ran). 10 papers today, all on LLM agents / agent harnesses. It is the `publish/` copy of studyarxiv, pushed by the developer by hand — so `tostudy/` → study run → publish → `git pull` is the whole loop, and the tree never contains local facts. |
| first placement | this Mac (`arxivsage-agstudio1`), launchd. agautolab1 via `agag_agent` is a later phase; the role already works (agecho-agautolab1 is `liveness=polling` today). |
| roles | `front` only, profile `sonnet`, `harness = claude_code`. Do not pick a local model to save money (localrule.md). |
| tool grant | `Read,Glob,Grep,Write,Edit,Bash(agentchat:*)` — Write/Edit are needed for `tostudy/`; "never edit `knowledge/`" is a guide sentence, not a sandbox. |

## Layout of `arxivsage/`

```
arxivsage/
  agents.toml                 # from agag init; grant above on [roles.front]
  params/intro.md             # the contract other agents read: scope, entrance-, what it never does
  params/channel.md
  agent/guides/entrance_front/guide.md   # the sage behaviour + the knowledge scope (braindump item 1)
  src/arxivsage/listener.py   # AgentSpec + one custom entrance serving
  src/arxivsage/intro.py
  service/listen.sh
  service/com.agdev.arxivsage-zulip.plist.in  # __PROJECTS_ROOT__ ritual, same as agforge/agfront
  service/sync_knowledge.sh   # git clone/pull of study-arxiv-trend into knowledge/
  knowledge/                  # git-ignored; a clone of iwaag/study-arxiv-trend, updated from outside
  tostudy/                    # git-ignored; written by sage, deleted by others
```

`knowledge/` and `tostudy/` sit at the repo root, not under `.local/`, so a
human or Ansible reaching the workspace finds them without knowing agag
internals. Both are ignored: the knowledge is another repository and
`tostudy/` is a queue. (A git submodule was considered and rejected: a pin
would make a knowledge update a commit in arxivsage, which is exactly the
"sage does not maintain its knowledge" line the braindump draws.)

## Steps

### 1. Generate and provision

```sh
cd ~/projects
AGAG_ZULIP_ADMIN_ENV=$PWD/pj-agdev/.local/zulip/provisioner.env \
  uv run --project pyagag agag init arxivsage --yes --provision --like agecho \
  --plan-prefix entrance- --run-prefix "" \
  --description "arxiv sage: answers questions about the papers in study-arxiv-trend; ask in entrance- topics"
```

- `--like agecho` copies `.local/agents.local.toml` (the claude binary
  `command_glob`); check it landed.
- Instance name defaults to `arxivsage-agstudio1` (`.local/instance.toml`).
- `--run-prefix ""` is treated as "not given" (`args.run_prefix or …` in
  `agag/init.py`), so the generated `listener.py` will say
  `run_prefix="arxivsagerun-"`; delete it — sage has no runs. With
  `--plan-prefix entrance-` the guide stub lands at
  `agent/guides/entrance_front/guide.md`, exactly where `agag.entrance`
  looks, so no rename is needed.
- `git init`, first commit, create `iwaag/arxivsage` on GitHub with `gh repo
  create iwaag/arxivsage --public --source arxivsage --push`. Push every commit afterwards
  (localrule.md).

Report: bot user id, channel name/id, that `#agents` subscription exists.

### 2. Listener and serving

`src/arxivsage/listener.py`:

```python
SPEC = AgentSpec("arxivsage", ROOT, plan_prefix="entrance-")
listener_main(SPEC, {"entrance-": handle_sage}, entrance=redirect)
```

- `handle_sage` = `serve_topic(client, channel, topic, lambda ctx: serve_sage(SPEC, ctx), ack_text=SWEEP_ACK, empty_reply=EMPTY_REPLY)`.
- `serve_sage` is `agag.entrance.serve_entrance` copied (~30 lines) with two
  extra placement lines in the prompt, computed at runtime:
  `Your knowledge tree is <ROOT>/knowledge (read-only for you).` and
  `Your study queue is <ROOT>/tostudy.` The run's cwd stays the topic
  workspace under `.local/topics/`, so absolute paths are the honest
  way to say where the files are. Keep `transcript=`, `stream=True`,
  `record=next_record_path(spec.records_root / "entrance_front")`.
- `redirect` (any other topic in the own channel): post one fixed sentence
  — "ask in a topic named `entrance-…`" — with `serve_topic` and no run.
  Cheaper than a paid run and it teaches the vocabulary. Or route
  everything to `serve_sage`; either is fine, say which.
- Guide path: `agent/guides/entrance_front/guide.md` (generated by step 1).
- Timeout: `ENTRANCE_TIMEOUT_SECONDS` (600) is enough for grep over ~10
  paper dirs; raise if the transcript shows timeouts.
- Add a placement line with the knowledge's git revision
  (`git -C knowledge rev-parse --short HEAD`, computed at serving time) so
  an answer can say which snapshot it read; stamp the same value into the
  run record via `extra_meta`.

pyagag has everything needed (`agag.agent`, `agag.entrance`, `agag.topics`);
no pyagag change is expected. If one turns out necessary: commit, push, then
`uv lock --upgrade-package pyagag` in `arxivsage/` (and only there).

### 3. Guide, intro, channel description

`agent/guides/entrance_front/guide.md` must say, concretely:

1. **Scope** (braindump item 1): "the arXiv papers published in
   study-arxiv-trend — recent trending papers on LLM agents and agent
   harnesses — as summarized in `knowledge/papers/<arXiv id>/summary.md`
   (plus `manual.md` = how to run, `test.md` = what a local run found) and
   indexed in `knowledge/README.md`." Start with the README table, then
   read only the dirs the question needs. A paper not in the table is not
   in scope-as-known, but *is* in scope-as-researchable (→ `tostudy/`).
2. **Never edit** anything under `knowledge/`; corrections are somebody
   else's job — say so in the reply if a file looks wrong.
3. **Cannot answer** (item 2): say so plainly. Then, if the question is
   reasonable, in scope, and researchable, write
   `tostudy/<short-slug>.md` (Markdown: question as asked, why it is in
   scope, what to look for — for a paper: its arXiv id/title if known and
   whether the asker wanted a summary, a run manual or a local test, since
   those are the three artifacts studyarxiv can produce). Before writing, `ls`/`grep` `tostudy/`; if a
   similar file exists append one dated line `- asked again in
   <channel>/<topic>: <one-line question>` instead of a new file.
4. **Answerable but out of scope**: answer from general knowledge, mark it
   as such, and add "another sage may cover this better; the `#agents`
   introductions list who covers what".
5. Answers cite the file (`papers/2608.23283/summary.md` style) and, when
   useful, the GitHub URL `https://github.com/iwaag/study-arxiv-trend/blob/main/papers/<id>/summary.md`
   so the reader can check. Say when the knowledge says "not tested locally"
   rather than guessing a result.
6. Reply is the last message of the run; never `agentchat send` into the
   own channel (posts twice).

Paths in the guide are relative to the knowledge tree; the absolute prefix
arrives from the placement line, so the tracked guide carries no local path.

`params/intro.md` (`{instance}` placeholder): scope in one paragraph;
"ask in `entrance-<short>` topics in my channel"; "I do not maintain my
knowledge; I do not run studies; I record what I could not answer in a
study queue that others read." `params/channel.md` one sentence, same
vocabulary. Post with `uv run python -m sage.intro`; re-post after every
behaviour change.

### 4. Knowledge deployment

`service/sync_knowledge.sh`: clone `https://github.com/iwaag/study-arxiv-trend.git`
into `knowledge/` if absent, else `git -C knowledge pull --ff-only`. Run it
once by hand before the first serving. This is the "external ansible/ssh/
local file operation" of braindump item 1 for the local placement; nothing
in the agent calls it. Record the revision and paper count in the report.

Do not point it at the local studyarxiv `main/` or `localtest-*/`: the
published repo is the one written for strangers, which is why it is the
right thing to show a bot that talks to them. A paper that exists in
`main/` but not yet in publish is, from sage's side, simply unknown — and a
correct `tostudy/` candidate until the developer pushes publish.

Later (not p1): a launchd/cron `git pull` every N hours, or the Ansible
`agag_agent` placement doing it, so a published study reaches the sage
without a hand step.

### 5. Run it

Trial first: `service/listen.sh` in the foreground, watch
`.local/out/`. Then the launchd plist (copy
`pj-agdev/devenv/launchd/com.agdev.agfront-zulip.plist.in` as the template,
label `com.agdev.arxivsage-zulip`, log `arxivsage/.local/out/zulip-listener.log`).
`launchctl kickstart -k gui/$(id -u)/com.agdev.arxivsage-zulip` after code
changes; guide edits need no restart (read per run).

`.local/agag-status.json` must be fresh after start; that is the liveness
file `nctl drift` would read once the agent is declared (step 7).

### 6. Failure farming — the acceptance test

Post as the Omni Agent (`AGENTCHAT_ZULIP_ENV=pj-agdev/.local/zulip/omni-agent.env`,
`agentchat send arxivsage-agstudio1 entrance-<slug> "<question>"`), one topic
each, and read the replies with `agentchat read`:

| # | question | expected |
|---|---|---|
| a | in scope, answered by `knowledge/` (e.g. "what does Prime Agent's harness change between iterations?") | answer with a citation, no `tostudy/` write |
| b | in scope, not in the tree (a real agent-harness paper the README lacks, or "was Prime Agent's local test taken past L1?") | "cannot answer" + a new `tostudy/*.md` |
| c | rephrase of (b) | no new file, one appended line |
| d | out of scope, model knows it (e.g. a Python stdlib question) | answer + "another sage may fit" |
| e | non-`entrance-` topic in the channel | the redirect sentence, no run record |
| f | a Front relay: ask Front in `#front` to put question (a) to sage and report back | Front finds sage from `#agents`, sage answers at home, Front relays |

For each: the run record `.local/agent/entrance_front/run-NNNN.json`
(cost, turns) and `transcript.jsonl` in the topic workspace. Guide fixes go
in as they are found, one commit each with the observed failure in the
message — that is the ENT asset of this phase. Do not pre-empt failures with
rules; let them happen first.

### 7. Declare it (optional, recommended)

Add `service arxivsage`, `placement arxivsage-agstudio` (profile `agag_agent`),
`workspace arxivsage-agstudio1`, `agent arxivsage-agstudio1` (Zulip user id from step 1)
to `.local/desired-state.yaml` and `nctl desired apply -f … --yes`, mirroring
the four agecho records (`agag_builder` p4 report, "Desired state"). Then
`nctl drift --host agstudio` should show `arxivsage-agstudio1: liveness=polling`
after the next observation refresh. Skip if the `agag_agent` profile's
one-placement-per-host limit bites on agstudio (agforge/agfront are not
`agag_agent` placements, so it should not) and say so.

### 8. Report

`report.md` (phase) + `reportN.md` per step: commands run, ids, costs per
question, every guide change with its trigger, and the handoff the braindump
leaves open — item 3, "someone reads `tostudy/`, runs a study, deletes the
file" — with the file format sage actually produced, so the next phase (or
the Omni Agent by hand) can wire a `#pj-studyarxiv` `workplan-` topic to it.
Update `devdocs/README_DEV.md` "In-System Agents" with an `arxivsage`
section (and the sage pattern in one paragraph) and
`pj-agdev/.local/devenv.md` with the launchd label.

## Hints

- **Everything in the own channel is swept** (`topic_filter`); the
  `entrance-` route is a dispatch prefix inside that, not a sweep filter.
- `serve_topic` acks with "Message received…" so the sweep does not re-serve
  while the run is in flight, and prefixes the reply with `@**<last speaker>**`.
  Nothing in the guide needs to address the asker.
- The `--like agecho` overlay's `command_glob` points at the VS Code
  extension's claude binary; if it moved, `agag` reports `E_…` at resolve
  time — fix `.local/agents.local.toml`, not the committed `agents.toml`.
- A `sonnet` entrance run is ~$0.13–0.5 (agforge/autolab numbers). Six test
  topics plus retries is a few dollars; run them all.
- `[selfnote]` lines in a topic are invisible to `agentchat read` unless
  `--all`; when a reply "vanished", read with `--all` before suspecting the run.
- Front-relay test (f): Front reads sage's intro from `#agents` immediately
  before its run — post the intro before that test, not after.
- The knowledge clone is public and small; `Read`/`Grep` over it is cheap.
  Do not build an index or embeddings — 10 summaries fit a `grep -ril`.
- `agentchat resolve` follows the `✔ ` rename; close the test topics at the
  end so the channel reads clean for the next phase.
- Zulip is `https://agstudio.local:8543`, self-signed; Developer login in
  `pj-agdev/.local/zulip/developer.password` if you need to look with a browser.

## Out of scope for p1

- The study-side consumer of `tostudy/` (braindump item 3) — only its input
  format is fixed here. The consumer will be a `workplan-` topic in
  `#pj-studyarxiv` whose result lands in `publish/` and is pushed by the
  developer; then `sync_knowledge.sh`.
- Automatic knowledge refresh.
- agautolab1 placement through `agag_agent`; Plane identity.
- More than one knowledge domain / instance.

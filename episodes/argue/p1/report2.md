# Step 2 — archsage, and the sages inside it

Date: 2026-09-16 JST. Plan: [plan.md](plan.md) step 2. Previous:
[report1.md](report1.md).

archsage exists as a repository (`iwaag/archsage`, `6080fe9`), a
provisioned Zulip account and channel, one real sage (arxiv, arxivsage's
knowledge moved into the new structure), and a passing suite. It is **not
running yet**: the launchd job, the Nautobot rows, the introduction and
arxivsage's retirement are step 4, where the replacement is shown working
live before the old listener is stopped.

## One deployed agent, one account

| Thing | Decision |
|---|---|
| Repository | `/Users/eiji/projects/archsage`, GitHub `iwaag/archsage` (public, like every agent the `agag_agent` role could deploy). Generated with `agag init archsage --like arxivsage --provision`, then reshaped. |
| Account | Bot user **24**, `archsage-agstudio1-bot@…`, display name renamed to **`archsage`** — the name it is mentioned by. Credentials in `archsage/.local/zulip.env` (0600). |
| Channel | `#archsage-agstudio1` in the `agents` folder, the Developer (8) subscribed by provisioning. Every topic in it is a question; there is no prefix (`SPEC.sweep_prefixes == ()`). |
| Frontier profile | `[profiles.frontier]` = claude_code / `anthropic/claude-fable-5-1`. Both `claude-opus-5` and `claude-fable-5-1` answered through the claude CLI on this host on 2026-09-16 (probe: `ok`, $0.085 and $0.174 for one word); Fable 5.1 is the most capable one available. The `[roles.archsage]` runs on it; `[roles.sage]` on Sonnet 5. The overlay can move either without touching the file. |
| Execution options | One published option, `default`, covering "answers in my own channel and in argues", pool derived from both roles (`anthropic`). |

## A sage is a directory

```
sages/<name>/sage.toml     name, one line about the domain, the study repository ("" = none yet)
sages/<name>/guide.md      the domain guide, read per run
sages/<name>/mainstudy/    ignored: the clone of the study's published knowledge (service/sync_knowledge.sh)
sages/<name>/tostudy/      ignored: the sage's study queue
```

Nothing else makes a sage. The execution profile is shared, the shared
guide (`agent/guides/sage/guide.md`) is the same text for every sage, and
the knowledge revision is `mainstudy`'s git revision — `empty` for a sage
whose study has published nothing, which is a valid state: such a sage is
told so in its prompt and explains the gap instead of inventing findings.

`sages/arxiv/` is the first: `study = https://github.com/iwaag/study-arxiv-trend.git`,
cloned at `cac3527` (arxivsage's clone was at `fb3d1ea`; the study has
published one more paper since). Its guide is arxivsage's, minus the
mechanics that now live in the shared guide and the tools.

## Addressing, and who pays

- In an argue: `@**archsage** sage:arxiv …` asks that sage; a bare
  `@**archsage**` asks the council; several in one post each answer on
  their own (`agag.argue.participate` with `selectors=[None, *sages]`).
- In the own channel: `sage:arxiv …` as the first word of the newest post
  asks that sage; anything else asks archsage (`listener.addressed`).
- The reply from a sage begins with `**[sage:<name>]**`, added by the
  posting layer (`agag.argue.with_speaker`), never by the model.
- A request for a sage **runs `run_sage` only**; `run_archsage` is not
  called (pinned by `test_a_direct_sage_request_costs_no_archsage_run`).
  A selector nobody publishes is refused in one line with the published
  names, and no run.
- Dispatch between roles never goes through Zulip: a listener ignores its
  own posts, so archsage consults a sage from inside its run with
  `archsage ask <name> "<question>"` — a sage run of its own, recorded,
  printed with its header for archsage to weigh.

## Run evidence

Every run record (`.local/agent/{archsage,sage}/run-NNNN.json`) carries,
beside the harness's own `profile`/`harness`/`model`/`cost_usd`:
`speaker` (`archsage` or `sage:<name>`), `sage` and `knowledge_revision`
for a sage run, `knowledge_revisions` (every sage's) for an archsage run,
and the triggering request — `invitation` (the argue post) or `requested`
(the newest post in the own channel), or `asked_by: archsage` for an
in-process ask.

The first real sage run, from this shell:

```
archsage ask arxiv "Which papers in the tree have a local test record, and what level did each reach? Cite the files."
```

| record | role | model | turns | cost | speaker | knowledge_revision | tool calls in the transcript |
|---|---|---|---|---|---|---|---|
| `sage/run-0001` | sage | claude-sonnet-5 | 3 | $0.1686 | `sage:arxiv` | `cac3527` | `sagetree cat README.md`, `sagetree find 'papers/*/test.md'` |

The answer named the two papers with `test.md` (2608.23283, 2608.23552,
both L1) and cited `README.md` and the two files — correct against the
tree. Every tool call was `sagetree`; no `Read` and no other command.

## The boundary, stated

A sage's grant is `Bash(sagetree:*)` and nothing else — no Read, Glob or
Grep — and `sagetree` (`src/archsage/sagetree.py`) resolves every path
against `SAGETREE_ROOT` and refuses one that lands outside (`..`, an
absolute path, a symlink out); its one write is `sagetree queue add` into
`SAGETREE_QUEUE`. The run's working directory is its own generation
workspace, not the tree, so relative paths reach nothing. Both variables
are set per run from the sage's directory (`roles.run_sage`), so the
supplied context is that sage's and only that sage's
(`test_a_sage_run_is_bounded_to_its_tree_and_records_who_spoke`).

**This is a bounded reader, not a sandbox**, and `README.md` says so:
under claude_code a `Bash(<name>:*)` grant admits a compound command
(`sagetree ls && cat …`), so a sage that wants out can get out, visibly —
the transcript shows a command that is not `sagetree`. No container or OS
isolation was added, as the plan allows. archsage's broader access is
explicit in `agents.toml`: `Read,Write,Edit,Glob,Grep,Bash(archsage:*),Bash(sagetree:*)`
— it reads every tree directly (their paths are placed in its prompt),
asks sages, and writes new sage definitions under `sages/`.

## Adding a domain

`archsage sage add <name> --about "…" --guide-file <path> [--study <url>]`
writes `sage.toml` and `guide.md` and refuses a name in use; `archsage sage
sync [<name>]` clones or fast-forwards the tree; `python -m archsage.intro`
re-posts the introduction with the `{sages}` placeholder rendered from the
directory (pyagag `b6aafec` lets `intro_text` fill extra placeholders). No
listener and no account is added. `test_add_then_list_shows_the_new_domain_with_an_empty_tree`
adds `aquaculture` with no study and lists it as an empty tree;
`test_two_sages_invited_in_one_post_keep_their_own_identities` invites
`sage:arxiv`, `sage:realworld` and the council in one post and checks that
each prompt carries its own guide and not the other's, that the three
posts carry their own headers, and that the served mark is written last.

## The plan's verification list

| Case | Evidence |
|---|---|
| Direct sage invocation costs no archsage model run | `test_a_direct_sage_request_costs_no_archsage_run`; the live `ask` above ran one Sonnet run, no Fable run |
| Two sages receive their own knowledge context and keep distinct identities on one sender | `test_two_sages_invited_in_one_post_keep_their_own_identities`, `test_each_sage_is_told_its_own_domain_and_an_empty_tree_is_named`, `test_a_sage_run_is_bounded_to_its_tree_and_records_who_spoke` |
| archsage can inspect both | `test_archsage_sees_every_tree` (both tree paths and revisions in its placement), `test_an_archsage_run_records_every_tree_s_revision` |
| A missing source is reported honestly | an empty tree is named as `EMPTY` in the sage's own context and in archsage's placement; `sagetree ls` on an empty tree prints that the study has published nothing; the shared guide says to queue rather than invent |
| Citations and knowledge-revision recording | carried over: the sage guide asks for file paths; `knowledge_revision` is in every sage record (`cac3527` above) |

Suite: **28 passed** (`uv run pytest -q` in archsage); pyagag 626 → 628
with the two additions (`b6aafec`).

## Left for step 4

arxivsage keeps running until the replacement is shown live. Then: install
`service/com.agdev.archsage-zulip.plist.in`, post the introduction,
`launchctl bootout` the arxivsage job and retire its plist, ✔ its
`intro-arxivsage-agstudio1` topic, and replace its four Nautobot rows with
archsage's (`.local/desired-state.yaml`, `nctl desired apply`, `nctl agents
observe`). The `#arxivsage-agstudio1` channel and the `iwaag/arxivsage`
repository are left as history.

*Did nothing for an in-system agent in this step — no Deus Ex Machina note
is owed; the one real run was a test from this shell, recorded as
`asked_by: archsage`.*

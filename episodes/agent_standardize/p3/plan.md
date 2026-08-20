# agent_standardize p3 plan — plan/run topic vocabulary

Goal: rename the four topic prefixes so that plan-only vs execute is visible
in the name itself:

| old | new | owner | meaning |
|---|---|---|---|
| `create-` | `assetplan-` | agforge | plan an asset Work, do not execute |
| `runcreate-` | `assetrun-` | agforge | execute one asset Work |
| `mission-` | `workplan-` | agautolab | plan a mission, do not execute |
| `run-` | `workrun-` | agautolab | execute mission tasks |

Collision note: the old pair needed a routing-order comment because
`runcreate-`/`create-` shared a stem; the new four share no prefix relation,
so that comment dies with the rename. `bmining-`, `front-`, `intro-`, `pj-`,
`work-` (channels) and the `✔ ` resolved marker are all untouched.

Success criteria:

1. A live `assetplan-` request in `agforge-agstudio1` completes planning and
   its `assetrun-` trigger executes (the p1 smoke, new vocabulary).
2. agfront, asked in a `front-*` topic, opens an `assetplan-` topic — with
   **zero agfront code/guide change**: the knowledge must arrive through the
   re-posted intro and the pre-run harvest. This is the payoff proof that
   the intro is the contract.
3. `grep -rn 'create-\|runcreate-\|mission-\|"run-'` over the four repos'
   `src`/`params`/`agent/guides`/`tests` finds nothing but history
   (`.local/`, devdocs episode reports — history is not rewritten).

Survey results (2026-08-20, already done — trust but re-verify cheaply):

- **pyagag: no change.** `run-NNNN.json` in `topics.py` is a record filename,
  not a topic prefix. The sweep machinery is prefix-agnostic.
- **agforge**: `zulip_listener.py` (both `*_TOPIC_PREFIX`), `create_topic.py`,
  `runcreate_topic.py`, `works.py`, `plane.py` (docstrings/comments),
  `params/intro.md` (says `create-…` — the cross-agent contract text), and
  ~50 test references. `FORGEAUTO`/`[AUTO]` Plane markers are not prefixes —
  untouched.
- **agautolab**: `zulip_listener.py` (`MISSION_`/`RUN_`/`CREATE_TOPIC_PREFIX`,
  the sweep tuple, derived `run-task<serial>-<label>` names),
  `mission.py` (`ASSET_TOPIC_PREFIX = "create-asset_"`, `asset_topic()`),
  ~50 test references.
- **Cross-repo contract**: autolab orders assets by opening
  `create-asset_<issue>` topics that forge's `create-` sweep matches, and
  forge keys the Work on `<channel>/<topic>`. So **forge and autolab must
  switch in one cutover**, and pending asset Works in Plane carry old topic
  names in their order keys.
- **agfront**: doc comments and intro-text test fixtures only. Update the
  fixtures to the new vocabulary; no behavior change (criterion 2 forbids it).
- Guides (`agent/guides/**.md`) mention no prefixes at all — handlers inject
  topic context. Guide *folder names* (`create_front`, `runcreate_generator`,
  `mission_superdirector`, `run_supercoder`) may be renamed for consistency
  or left — implementer's call; if renamed, follow the references from
  `agents.toml` roles and `role_run`.

Constraints (deliberately minimal):

1. Breaking-change phase: no old-prefix compatibility shims, no dual-sweep.
   **Existing projects, chat history, and pending Works are all test-only
   and may simply be deleted** — nothing in the realm or Plane needs to
   survive the rename.
2. Push-to-GitHub-then-deploy for anything a node consumes (localrule.md).
3. Episode reports and `.local/topics/` transcripts on disk keep their old
   wording — documents are history, not live state.

## Step 1 — agforge vocabulary

Mechanical rename in the files listed above, tests updated, deterministic
suite green. Update `params/intro.md` to `assetplan-…` wording, but **do not
re-post the intro yet** — the live realm still speaks the old vocabulary
until Step 3. Rename docstrings/comments too; the old words must not survive
as documentation. Commit, push, but reload nothing yet.

## Step 2 — agautolab vocabulary

Same treatment: prefixes, derived names (`workrun-task<serial>-<label>`),
`ASSET_TOPIC_PREFIX` → `assetplan-asset_` (keeping the `asset_<issue>`
substructure; collapsing it further is out of scope), sweep tuple, tests.
Commit, push. Node deploy (GitHub → ansible playbook, `--limit agautolab1`
and `--limit agstudio`) can run now or at cutover — agautolab1 has no Zulip
listener, only the gateway, so timing is not critical there.

## Step 3 — Cutover (one sitting, listeners quiesced)

1. Stop the forge and autolab Zulip listeners
   (`launchctl bootout` or stop-equivalent; agfront may keep running).
2. Clear the old-vocabulary state — everything is test-only, so delete
   rather than migrate: old-prefix topics in the live channels can be
   deleted outright (admin `developer.env`; deleting the messages deletes
   the topic) or left to rot unresolved — they no longer match any sweep,
   so they are inert either way. Cancel/delete pending `[AUTO]` Plane Works
   (their order keys embed old topic names and nothing will ever serve
   them). One deletion sweep, no renames, no `✔` ceremony. Note in the
   report roughly what was wiped, one line is enough.
3. Start both listeners on the new code; the startup log line prints the new
   sweep prefixes — quote it as evidence.
4. Re-post the forge intro (`uv run python -m agforge.intro`) — new
   vocabulary, new revision stamp, appended to `intro-agforge-agstudio1`.

## Step 4 — Prove and report

1. p1-style smoke in new vocabulary: Omni opens `assetplan-…` in
   `agforge-agstudio1`, sees planning complete, fires the `assetrun-`
   trigger, gets the result (criterion 1).
2. Front smoke: developer asks in `front-*`; permitted, Front opens an
   `assetplan-` topic purely from the harvested intro (criterion 2). If
   Front still says `create-`, the bug is in the intro or harvest, not in
   Front — fix there.
3. Run the criterion-3 grep and quote it.
4. `report.md`: the rename table, commits, the one-line wipe note,
   evidence links, and living-docs updates
   — `agforge/README_DEV.md`, `agautolab` docs, `.local/devenv.md`
   (`create-`/FreeForge-era wording), memory that guides needed no change.
   Deferred: collapsing `assetplan-asset_`, guide-folder renames if skipped.

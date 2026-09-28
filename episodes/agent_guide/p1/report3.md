# agent_guide p1 — step 3: shared text written once

## Why this is on branches

The listeners read their guide files from disk on every serving. Editing a
`guide.md` in place is therefore a live change to every agent that reads
it, while the help texts from step 2 reach a run only after `uv sync` in
each consumer. Step 6 also asks for one more trial against the **old**
guide before the change, so the live guides stay as they are until then.

The work is on a branch named `agent-guide-p1` in each repository it
touches, checked out as a worktree under the ignored
`pj-agdev/.local/agp1/<repo>`. It merges into `main` at deployment
(step 6).

During this step a trial post into `#front` from this shell (the
before-trial) was refused by the session's auto-mode classifier ("External
System Writes"). Nothing was posted. Step 6 needs the Developer's decision
on how trial posts are made.

## The mechanism

In agfront (`ba28e90` on `agent-guide-p1`):
`zulip_listener.role_guide(role)` returns the role's own
`agent/guides/<role>/guide.md` followed by the shared files that
`SHARED_GUIDES` names, all read from `agent/guides/shared/`. `front_prompt`
and `argue_prompt` call it, and `prompt_with_guide` then appends
`REPLY_GUIDE` and `CONTINUATION_GUIDE` as before. A missing shared file is
as fatal as a missing guide (`GuideError`).

| role | reads after its own guide |
|---|---|
| desk | `board.md`, `requests.md`, `work.md` |
| front | `board.md`, `requests.md`, `work.md` |
| routine_run | `board.md`, `work.md` |
| argue | `board.md` |
| present | nothing (unchanged: it renders; it does not converse) |

The choice was between this and a new appended constant in pyagag. This
keeps the text in `.md` files a human edits next to the guides, and it
needs no pyagag release. It changes nothing for other agents: their guides
are one file each and repeat nothing but the agrefs paragraph.

## The shared files

- `shared/board.md` — **what you can see**. The developer's assumption
  (names are yours to look up); that the board is Zulip reached by
  `agentchat`, not the filesystem; the `--help` index; reading is free and
  a post runs its addressee; one line each for `intro`, `channels --prefix
  pj-|routine-`, `agproject status`, `agrun`, `agrefs`; what the working
  directory holds; how a post reads in the chatlog, with ✔ and unreadable
  threads (D23).
- `shared/requests.md` — desk and front. The no-mention rule (D5).
  Existing work is answered from the board. Propose before delegating
  (D8). Execution options and cluster/project routing (F6–F9). Argue
  (D12), project (D13, F3) and study (D14). Routines: opening one is the
  whole reply, with rt2's mechanism (D16, D17/F5), the execution
  preference (F4) and the plan-window reading (D22). References (D48). No
  preface (D46).
- `shared/work.md` — desk, front and routine_run. Each serving ends (D40,
  D41). Judge on evidence: ack ≠ done, closed = record `completed` (D31,
  fs3A2), you never play a delivery (F10, agm2), references in the result
  (D45). Read before posting, ✔ is finished, never open `workrun-` (D43,
  fd-wr). **Whose decision it is**: proxy authority (D4); acceptance
  holders (D18, R7, F11); one line each for hold, disposition and receipt
  pointing at their help (D36–D39). **When Observer says work has
  stopped**: the post and its contents (D25), re-check first (D26), then
  the facts that decide the move (D30–D35), with trial references kept
  word for word.

What changed in form, not content, in the Observer section: the old
per-verdict branches ("RESUMED, RESUMING or ASKED: …", "UNOWNED: …",
"FINISHED: …") moved to `agentchat recheck --help` (step 2). Every
verdict's `next:` line in `recheck`'s own output already says what the
verdict asks. The facts no command can hold stay in `work.md`: what counts
as moving, where a resume goes, resume ≠ agreement, never a second run,
when to ask the developer.

## The same text in other agents

The 14-line agrefs paragraph was replaced by a 5-line pointer to
`agrefs --help` in all 11 files: agfront `desk`/`front`/`argue` (in
`ba28e90`); agautolab `workrun_supercoder`, `workplan_superdirector`,
`argue/role.md` (`2efc711`); agforge `assetplan_front`,
`assetplan_generator/guide_plan.md`, `assetrun_generator`,
`argue/role.md` (`73c9c8a`); archsage `archsage` (`4c94d42`). Each file
keeps its own follow-up paragraph (autolab: `direction/REFERENCES.md` and
`agrefs changes`; forge: `--init-image`, `required_items.md`; archsage:
references are not findings). agobserver's two short variants were
already written the Tool Giving way and were left alone.

The receipt paragraph was only ever in the three agfront guides; it is
now one sentence in `work.md` plus `agentchat receipt --help`.

## Nothing generated is copied

- `tools/agents.md` (the introductions) is pointed at in one line and not
  restated. The study-establishment routing that archsage's introduction
  already publishes (`params/intro.md` §Establishing a study) was cut down
  to "archsage's to establish; its introduction says how to ask" plus the
  cross-agent flow (research run, then sage refresh).
- `REPLY_GUIDE` and `CONTINUATION_GUIDE` are untouched and not repeated.

## desk and front

They stay two roles with two short heads. The listener routes by topic
(`front-desk-` vs other `front-*`) and each role has its own profile, so
merging them would change agents.toml and routing for no gain. Their whole
shared body now lives in `requests.md` and `work.md`. What still differs
is the head (step 4).

## Tests

On the branch, agfront gives 191 passed and 3 failed:

- `test_zulip_listener::…whole_input` asserted the exact prompt tail. It
  was updated to expect the shared files and now passes.
- `test_prompt_delivery::test_one_oversized_message_is_cut…` fails
  because the prompt is longer than its bound: the old guides and the
  shared files are both in it until step 4 rewrites the guides. This is
  known and intended to go green in step 4.
- `test_listener_is_the_skeleton…` fails only in the worktree, which has no
  `.local/zulip.env`. It passes in the main checkout.

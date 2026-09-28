# agent_guide p1 — step 2: the help texts as a sufficient home

Commits: pyagag `73efd5c`, agfront `1ef83f5`. Nothing is deployed yet:
the consumers' pyagag pins and the listener restarts happen in step 6.

## agentchat

`agentchat --help` is now an index: 117 lines, down from 236. Of those, 40
lines are argparse's own list of subcommands. It has five parts:

1. **What the board is**: introductions, `pj-<slug>` projects and studies,
   `routine-<name>` routines (the `guide` topic is the routine), argues,
   and the agents' conversations. A name you do not know is on the board,
   and the board is not in your working directory. The last sentence is
   aimed at run-0160, whose only probe was a refused `find`.
2. **Reading is free, speaking costs**: a post runs its addressee, and a
   "how is it going?" starts their job again (as9).
3. The channel to post in comes from the addressee's introduction.
4. **One line per command**, grouped as Look (free), Speak (costs a run)
   and Record (notes; nobody is served), with a pointer to `agentchat
   <command> --help`.
5. Three examples (intro, `channels --prefix pj-|routine-`, a
   response_request), and the "say what you want and finish" notes, which
   tests pin.

Where the old top-level notes went, and what each subcommand gained:

| subcommand | now also says | from |
|---|---|---|
| `send` | a post runs them, so read first; ordinary professional language (D3); the whole `--intent`/`--re`/`--not-answer` explanation; anchoring; work vs reference, and that commenting or cleaning up is a reference (D44); why a ✔ topic and a `<channel>/<topic>` topic are refused; examples | old Notes, desk D3/D43/D44 |
| `read` | ids are how you cite and what `--since` takes; reading costs nobody anything; a ✔ topic is read under the name you know and is finished | old Notes, D24 |
| `recheck` | **UNOWNED** (was missing); each verdict comes with a `next:` line; act on it and quote it; RESUMED/RESUMING/ASKED → do not post, since a second run beside the first is the one wrong move; UNOWNED → find the owner's real conversation | D26–D29 (fs4, fs5E) |
| `receipt` | the listener writes it by itself; how `trace` shows owed and settled answers; no evidence → deal with it now; a receipt changes no work; never hand-write one (failsafe p6) | D36–D37 (fs6r) |
| `hold` | when to record one (the person's phrases); nobody acts on held work; which holds settle by themselves; hold vs `reserve`; examples | D38 |
| `disposition` | when to record one; never report cancelled/withdrawn as success; `agrun finish` for your own run; a later post is monitored again; examples | D39 |
| `channels` | the realm's naming by kind: `pj-` (project/study, see `agproject status`), `routine-`, `work-m<id>`; examples | plan gap |
| `resolve` | only a rename; refused while your post is unanswered; a decision, not a tidying reflex | old Notes |
| `accept` | who holds the decision; counts only after the result was shown; a worker's "done" never counts; posting into the plan re-plans; saying it in a reply records nothing | old Notes, F11 (agm2) |
| `use` | select before posting the request; an option name is one agent's vocabulary, so translate again for a third agent; say so when nothing matches; your own run is unaffected | F7/F8, R4 |
| `anchor` | the whole "when is an anchor wrong" explanation | old Notes |
| `argue` | what an argue is for; the invitation; stem refusal and `topics argue`; do not post into it again or delegate for it | D12 |
| `intro` | an account speaking for several participants (sages) lists them in its introduction | plan gap |

`_doc()` (pyagag `agag.chat`) writes a subcommand's help as filled
paragraphs plus verbatim example blocks. `test_chat.py` now checks that
every subcommand appears in the index, that the board, `pj-<slug>`,
`routine-<name>`, "Reading costs nobody anything" and "not in your working
directory" are there, and that `recheck --help` names every verdict in
`RECHECK_NEXT`. The anchor test reads `anchor --help`.

## agproject

- `status` has a description: what it reports (kind, channel, document,
  setup request and answer, repository revision, remaining), the states
  it can print, when you would want it, and that `agentchat channels
  --prefix pj-` lists the projects and studies. The description was
  checked against `agproject status aisvgs` (read-only). Members show only
  with `--provisioner-env`, and the text says so.
- `open` says what it creates, that it is idempotent, that nothing is
  started, and that it runs only on a stated decision (F3, agm1). It
  gives the contents of `GOAL.md` and of a research plan (moved from D13
  and argue A8), and says the first work goes to autolab as a `workplan-`.
- `plan` describes itself.

## agrefs

The top-level help gained the rules that 11 guides copied: work from the
named revision; quote `<source>@<rev>:<path>`; pass on a commit, never
`latest`; a context-panel reference already carries the full commit;
say so when a reference disagrees or cannot be reached (D47–D49).

## agrun, agbudget (agfront)

- `agrun --help` explains routines and runs: the `guide` topic; what a
  `routinerun-<id>` is; how a run is opened and what the opening post
  carries (D16, including the requester's execution preference, F4); that
  the run does its own delegating; that there is no schedule. It says a
  post of yours into the run serves nothing and a sentence saying it is
  complete ends nothing (D19–D21). `status`, `continue`, `adopt` and
  `finish` each gained a description (they had none).
- `agbudget --help` carries the rules for reading a condition (R11, D22):
  pool matching, `+` pools, unknown pools, "exceeds" vs "reaching",
  "consume N from the start", resets, stale reads, account vs run cost, and
  that reaching the limit is not a wall. The window explanation stays in
  `tools/budget.md`, where it already was.

## Left for step 3

These facts belong to no command: who holds an acceptance (D18), a
proxy's authority (D4), the no-mention rule in `#front` (D5), the
Observer stop section's STOPPED/closed/owner-moved facts (D30–D35), and
"each run ends" (D40–D42).

## Tests

- pyagag: 1125 passed (full suite, before the final `intro` wording);
  `test_chat.py` 71 passed after it.
- agfront: 194 passed. One slip on the way: the new help constant was
  first named `USAGE`, which overwrote `tools/budget.md`'s explanation of
  the same name. Two budget tests caught it, and it was renamed `HELP`.

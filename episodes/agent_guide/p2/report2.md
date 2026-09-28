# agent_guide p2 — step 2: help texts for the tools the guides use

## Where the work is

As in p1, everything that changes code, help texts or guides is on a
branch named `agent-guide-p2` in each repository, checked out as a worktree
under the ignored `pj-agdev/.local/agp2/<repo>` and pushed to its remote.
The live checkouts stay as they are until the deployment in step 7: guides
are read from disk on every serving, so editing them in place would change
the running agents before the baseline trials.

| repository | commit | what |
|---|---|---|
| pyagag | `de8353e` | `agag.health`, `agroutine` |
| archsage | `f74d0ff` | `archsage` subcommands, `sagetree` subcommands |
| agforge | `a08ece8` | `agforge --help`, `knowledge list`, `image generate --init-image` |
| agautolab | `9f1711c` | `mission_done` |

## What each help now says

The rule from p1: say what the command reports or changes, what the
output means, and when an agent would want it; and next to anything that
*records*, say that saying it is not recording it.

| command | before | after |
|---|---|---|
| `archsage sage list/show/add/update/attach/remove`, `ask`, `queue list/show/resolve`, `intro` | the top-level help had a paragraph per group; each subcommand's own `--help` was a bare synopsis (`name`, `--no-intro` without a word) | each says what it prints (`list`: revision, knowledge files, `no findings yet` vs `empty tree`, queue size, study), what it changes (store commit, introduction re-post, directory moved aside), and what it refuses (`add` on an existing name; `resolve` when a path is not in the tree). `ask` says nothing is posted and why a `sage:` post from the run would not reach the sage. `intro` says which commands re-post by themselves and that `sync` and a restart do not |
| `archsage sage sync` | `--require`: "recorded as includes= or missing=" | the printed line; the whole `[selfnote][sagesync]` record with `for=`, `includes=`, `missing=` and what each means; that `missing=` means that request's knowledge was **not** refreshed whatever the revision moved to; "the note is the record; saying in your reply that the sage is refreshed records nothing". This is AS14's usage (report1), carrying fs5-1's reason |
| `sagetree ls/cat/grep/find/revision/queue list/queue add` | one-line helps | limits (`cat` bytes, `grep` 400 lines), the empty-tree answer, `revision` as the thing to cite, `queue add`'s append on an existing slug, that the queue is the one place a sage writes (SG3/SG4) |
| `agroutine create/update/show/list` | one-line helps; the top-level epilog was good | each subcommand says what it prints and changes; `update` says the file is the whole guide, the read-back, and exit 1 on a truncated or altered text |
| `agforge --help` | the first docstring line only | the command index the module docstring already had |
| `agforge knowledge list` | "read it first" | general vs local, the five local states, and "only `verified` has been run end to end here; anything else is a lead, and a plan that relies on it says so" (FG2's usage) |
| `agforge image generate --init-image` | composition/palette steer the result | plus where a published reference is: `"$(agrefs path <source>@<rev>:<path>)"` (FG8's usage) |
| `agforge toolsets` | "List the toolsets in agent/toolsets/." | what a line is, and toolsets (what can run) vs knowledge (what is known) |
| `python -m agautolab.mission_done` | "Mark a mission done once every one of its tasks is finished" | that it writes the same acceptance record as `agentchat accept`, on which evidence, then `done`; one line per mission; "saying in a reply that a mission is done records nothing; this does" (EA5) |
| `python -m agag.health` | the first docstring line; `--ack`, `--channel`, `--topic`, `--window` undescribed | the whole module docstring (facts, verdicts, the queued case, what it does not claim), each option, the output (`agag.health.v1`, always exit 0, `unknown` with the reason on failure), that it posts and changes nothing, two examples |

Not changed, and why:

- `agobserver.withdraw` and `agobserver.hold` already say what they record
  and when to run them; no in-system role runs them (report1: they are
  operator commands).
- `autolab doc` and `agag init` already say what they print or write.
  `agag init --help` is long and complete.
- `agforge video/music submit` and `comfy fetch` already say to hand the
  id to the notifier and end the run. The generator's `pending.json` is a
  contract of forge's run, not of the command, and stays in its guide.
- **comfynotify has no help an agent can reach, and no introduction on
  the board** (`agentchat intro` lists six agents; the notifier is not
  among them). The supercoder's paragraph about `@**Comfy Notifier** watch`
  (WR14) is therefore the only description of that command anywhere a run
  can read, and it stays in the guide as a fact rather than moving to a
  help. Publishing an introduction for the notifier is a handoff candidate
  for report.md, not this phase's work.

## A stale fact found in passing

archsage's guide says a study routine's guide must be "well under 10,000
characters" (AS13). That bound was Zulip's post limit before failsafe p4
raised it to 100 000; `agroutine` reads every guide post back and reports a
truncation itself. Step 4 removes the number and keeps the pointer to the
read-back.

## Tests

- pyagag: **1125 passed** (worktree).
- archsage: **40 passed** once the worktree has the host's
  `.local/agents.local.toml` (without it one test needs `claude` from the
  default profile).
- agautolab: **331 passed**.
- agforge: 252 passed, 4 failed in the worktree. The four read sibling
  paths relative to the checkout (`../devpolicy/contracts/…` and
  `../comfynotify`), which do not exist under `.local/agp2/`. The same test
  files pass in the main checkout (33 passed). They are re-run there after
  the merge in step 7.

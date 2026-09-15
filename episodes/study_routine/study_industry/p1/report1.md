# Step 1 — lessons carried forward, and the state before anything changed

Date: 2026-09-15 JST (timestamps below are UTC where they come from Zulip
or `nctl`). Plan: [plan.md](plan.md) section 1. Nothing in this step posts,
creates or spends a paid run; it fixes what the later steps are held to.

## What the earlier study routines taught, and how each is applied here

| Lesson | Where it was paid for | How this episode applies it |
|---|---|---|
| A researcher given only a goal plus `methods/` and `reports/` will choose, reuse and extend its own methods. | [routine_tests p2](../../../routine_tests/p2/report.md) §3 — run 2 kept the roster and source method with a stated reason and settled the first report's own open question | The guide (step 3) prescribes no source, measure, period, report shape or tool. It adds exactly one capability, the braindump's optional local-service clause. |
| Checking arithmetic is not enough, and a supervisor's summary is not an artifact review. | [routine_tests p2 ex2](../../../routine_tests/p2/ex2/report.md) §2 — a wrong FEC type-code definition (`22Y`), prose contradicting the report's own table, a phantom example, none caught by the task, by Front or at the run's end | Step 5 reads the committed files and the original sources, recomputes selected figures, checks that prose, tables and charts agree, and has the researcher fix defects in `main/` before publication review. |
| A check that finds nothing leaves no trace unless the routine asks for one. | [study-realworld p1](../../study_realworld/p1/report.md) finding 1 — an unchanged source was byte-identical afterwards, indistinguishable from never looked at | The guide asks that dates and outcomes of checks — unchanged, unavailable, incomplete — be kept in the knowledge tree at a place the researcher chooses. |
| A publication gate can delete knowledge, and `main/` is the only copy. | study-realworld finding 2/3 — a `Follows:` link, a `## Question` section and a real retrieval finding cut as "workflow residue"; visible only in `git show` | Step 6 inspects the review diff's deletions itself and asks that useful knowledge be rewritten or relocated, never discarded. The live `publish` guide (v2, message 6254) already carries the "a check never deletes knowledge" rule. |
| A reused `workplan-` topic misroutes every callback to the first run's home. | study-realworld finding 4; memory `workplan-topic-has-one-home` | Every run in this episode delegates into a fresh `workplan-` topic. |
| An opened plan or an acknowledgement is not evidence that research executed; a stated resolve is not a resolve. | study-realworld findings 5 and 8; memory `front-announces-resolves-it-skips` | Step 4 and step 6 check the actual topic ✔ states and the mission's `done` note, not Front's sentence about them. |
| A task that ends its session with a background job running is never woken. | routine_tests p2 ex2 §1 — a 2.18 GB download finished and nobody served the task for 12 minutes until a human posted | The guide points long jobs at the board (the Observer is an option) and asks the researcher to leave job identity, outputs and next action in its conversation. |
| The routine contract is a `#routine-<name>` channel, a fixed `guide` topic whose newest post is the whole guide, and Front-owned `routinerun-` topics. There is no dispatcher and no Plane. | [README_DEV.md](../../../../README_DEV.md) *Routines*; `refactor` p3 | Step 3 creates the channel in the existing `routine` folder, posts one full guide, verifies Front's subscription reaches its listener, and checks the ordinary routine board. |
| Boards and listeners read from persisted mirrors; repeated full-realm polling recreates the problem that episode solved. | [better_zulip_call p1](../../../better_zulip_call/p1/report.md) | Execution is followed through the relay's `/routines` and targeted `agentchat read` on named conversations. No sweep loops from this shell. |
| A fresh subscription does not reach an already-registered event queue. | `refactor` p3 ex1 note in the local environment memo | After subscribing Front to the new routine channel, its listener is kickstarted with nothing in flight, and the restart line is read back. |

Two lessons are carried as constraints on the Omni Agent rather than on the
agents: **nothing inside `main/` or `publish/` is written from outside the
system** (every prior study episode kept that boundary, and it is what makes
the artifact audit meaningful), and every act done on an in-system agent's
behalf gets the standard handoff note.

## Refreshed state, read before anything was changed

Infrastructure, through `nctl` (read-only; raw envelopes in the ignored
`.local/` beside this report):

- `nctl status --json` — `ok: true`, no errors. Nautobot 3.1.3 reachable and
  authenticated, intent catalog and intent GraphQL present. Node observation
  dumps for the live hosts are about three hours old; `agfixture` (retired)
  and `agbach` are the stale ones, as before.
- `nctl drift --json` — `ok: true`, `errors: []`, `converged: 46` (43 before
  the Observer's three rows were added on 2026-09-13).

This is access to the state authority, as the plan says — not suitability
of any database for research. Service suitability is cagent's question and
is asked in step 4 if the researcher wants a service.

Agents and boards:

- launchd jobs running: autolab listener and gateway, Front, forge (service
  and listener), arXiv sage, Observer, ComfyUI notifier, cagent (api and
  Zulip), and the agentroom relay. The relay's mirror is `live`, revision
  3473.
- Routine board (`/routines`): `ghtrends`, `papers`, `publish` (guide v2,
  message 6254, 4 runs), `study-realworld`, `study-uspolitics` (guide v2,
  6393, 3 runs), and the retired `anchorcheck`. **No `study-industry`.**
  Every routine has `open_runs: 0`.
- Front's listener log has been silent since 2026-09-14 05:59Z, autolab's
  since its restart the same morning. Nothing is in flight.
- Last run records before this episode: Front `front` 0622, `routine_run`
  0058, `character_talk` 0029; autolab `superdirector` 0200, `supercoder`
  0282. Anything above those belongs to this episode.

Names, checked so nothing is reused or overwritten:

| Thing | Checked | Result |
|---|---|---|
| Routine channels | `agentchat channels --prefix routine` | six, none `study-industry` |
| Project channels | `agentchat channels --prefix pj-` | six study projects, no `studyindustry` |
| Workspaces | autolab's `projects/` and `projects-archived/` | 10 live, 16 archived, no `studyindustry` alias |

The publication remote, inspected by cloning into a throwaway directory:
`https://github.com/iwaag/study-industry.git` holds one commit,
`a91105ce…` ("Initial commit", 2026-09-15 18:48 +0900, author `iwaag`), on
the single branch `main`, containing one file, a CC0 1.0 `LICENSE`. Not
empty; its history is to be preserved.

The current `publish` guide (v2, 6254) was read in full. It discovers
projects from the workspace rather than from its table, never pushes
`publish/`, and carries the "never deletes knowledge" rule and the
check-4 exclusions from study-realworld's v7. It needs no new version for
this project: the plan's "repository mapping supplied through project
context" is `README_PROJECT.md`, which that guide already reads.

## Step 1 conclusions

1. The lessons that shape this episode are named with their evidence and
   the concrete place each one lands (guide text, a verification step, or a
   constraint on the Omni Agent).
2. The environment is converged and idle, the names are free, and the
   publication origin's contents are known.
3. No conversation has been opened and no run has been bought.

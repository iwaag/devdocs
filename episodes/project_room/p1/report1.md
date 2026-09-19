# Project Room p1 — Step 1 report: the project read model

Plan: [plan.md](plan.md), step 1. Code: `pj-agdev/agdevworld/agentroom`
(`src/agentroom/projectroom.py`, `tests/test_projectroom.py`, routes in
`server.py`, wiring and the `check` summary in `main.py`, README section
`/projects`).

## Delivered

Two relay routes on the agentroom relay, both answered from the mirror:

- `GET /projects` — every `pj-` channel, live and archived, with kind,
  origin argue, counts, latest activity, the most urgent reply state, gaps
  and short rows of its documents, setups, missions and unrecorded plans.
- `GET /projects/<stream id | channel | slug>` — one project whole: document
  text, setups, missions with plan document and tasks, unrecorded plans,
  orphan tasks, other topics.

What the model distinguishes, and how (nothing is inferred from a name when a
record exists):

| shape | how it is recognised | shown as |
|---|---|---|
| purpose document | topic `goal` or `researchplan-…` | `documents[]`, newest visible post is `current`; `missions: []` always |
| setup | `workplan-setup-…` with no `[mission]` note | `setups[]` with a note that it plans no mission |
| mission | any topic in the project channel carrying a `[mission]` note (`autolab.read_record`) | `missions[]`, `setup: true` when the topic is setup-shaped (legacy m6342) |
| unrecorded plan | `workplan-…` without a `[mission]` note | `plans[]`, `recorded: false` |
| task | a `[task]` note anywhere, attributed by the mission id it names | inside its mission's `tasks[]`; `orphan_tasks[]` when the mission is not in any project channel, filed by the work channel's folder |

Identity is the anchor message id; `[replaces]` is exposed as `replaces` /
`replaced_by`; `[rootchat]` notes are exposed as `origins`. Recorded work
state (`work.state`, autolab's `[state]` word) and the conversation's reply
state (`reply`, lifted from the ops engine's rows for that conversation) are
returned side by side. Task counts are `total / live / finished / completed /
accepted / cancelled / open`; `tasks_read.complete` is false when the
`work-m<anchor>` channel is archived or absent, and the project row lists
which missions are incomplete. Links found in source posts are filed as
repository / report / zulip / other. Mirror health, `stale: true` and Zulip
narrow URLs travel with every payload.

The project kind is read from the channel description: the `agproject`
format (`project: <slug>; project|study`), autolab's `(study pattern)` marker
and the handwritten `Study project:` opener. Everything else is `unknown`.

## Checks

- `uv run pytest tests/` in `agentroom`: **313 passed** (12 new). Fixtures
  cover: setup-only project; study with a research plan and no mission
  (`no-mission` gap, "not executable work"); several missions with tasks in
  every state; an unmirrored (archived) work channel → incomplete count; a
  retired mission renamed aside with its replacement taking the freed name
  (relation by id, tasks not handed over by name); an orphan task, a mission
  with no `[doc]`, a setup topic carrying a mission; resolved history (done
  mission, and a ✔ over `planned` → `resolved-unfinished` gap); an archived
  project channel; a stale mirror (`reply.state: unknown` + `stale_state`);
  no ops engine (`unknown`, never `quiet`); repeated reads make zero Zulip
  calls and a pumped event reaches the next read; HTTP routes incl. 404 and
  503.
- Smoke read against a **copy** of the live relay store
  (`agentroom/.local/mirror`, 33 project channels, 9 live): board in 0.11 s,
  one detail in 5 ms. Real shapes met: `pj-worldtrend` and `pj-desk-garden`
  read as `setup-only` with their argue anchors (7217, 7149);
  `pj-studyuspolitics` shows m6371 `replaced` → `replaced_by` m6400 and the
  legacy setup mission m6342 with `setup: true`; `pj-mediagen` m6770 shows
  4/4 tasks completed with a repository link; older `workplan-` topics
  without notes appear as unrecorded plans (15 in mediagen, 20 in studyarxiv).
- The running relay was **not** restarted in this step; deployment is step 5.

## Notes and gaps

- Archived `work-` channels are not mirrored, so pre-`refactor` missions
  show no tasks and say so; hydrating them would be a Zulip read per
  channel and is deliberately not done on a board refresh.
- `pj-studyindustry`, `pj-studyuspolitics`, `pj-studyarxiv`, `pj-papers`
  have no `goal`/`researchplan` topic and read as `no-document`; that is the
  realm, not a reader gap.
- The `Ops.snapshot()` call per board read costs a revision check when the
  mirror is quiet and a full derivation otherwise; the project model itself is
  cached per mirror revision.

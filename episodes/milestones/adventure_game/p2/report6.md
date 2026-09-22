# Step 6 report — the mechanism verified, the outcome stated

Plan: [plan.md](plan.md) step 6. 2026-09-23 (2026-09-22 16:10–16:30 UTC), by the Omni Agent.

## Focused checks

| Check | How | Result |
|---|---|---|
| Revision selection | `agrefs sync protoprey-refs@a3c8196` and `@0cd3c0b` in one consumer; `agrefs show` of `todo.md` at each | two snapshots coexist under their full shas; each `show` prints the tree's own text and names the commit it came from; `latest` is never stored |
| Tree and asset fidelity | `cmp` of the 5 images and 6 texts in the delivered `main/` against the `a3c8196` snapshot; VM clone tree id vs. the human's clone | byte-identical; tree `113e2785…` on agautolab1 equals the human's; ignored `drafts/` never appeared |
| Independent consumer retrieval | plain `git clone` on agautolab1 through the ansible key, no access to the human's folder | HEAD `a3c8196` at the time, 14 files, hashes equal (report 3) |
| Reference propagation across delegation | read the workplan post, the plan, `task1.md`–`task3.md`, the delivery, `direction/REFERENCES.md` | `protoprey-refs@a3c8196:<path>` appears in Front's post, autolab's plan, every task, the delivery and the adoption record; the second unit names `0cd3c0b` and cites `agrefs changes a3c8196..0cd3c0b` |
| Actual media access by the roles | Front found the `hdiscovery.txt`/`rest.jph` typos by listing the tree; autolab decoded the EXIF of `catch.jpg` from the snapshot; the mirrors' `FETCH_HEAD` in Front's and autolab's own caches were touched during their runs (16:05:09Z, 16:06:01Z), not by the Omni Agent's earlier syncs | each role fetched and read the originals itself |
| A revision change through the roles | `0cd3c0b` → Front's new topic → autolab's adoption entry → `1f4c2f4` | seven minutes, one commit, unrelated work untouched |
| Reference images into generation | `agforge image generate --init-image` with the meadow still (step 3) | works; **not exercised by forge on a real request** (no asset was needed) |
| archsage | grant and guide in place | **not exercised** in this phase |

Not proven: forge planning from a human reference image, archsage reading references in
an argue, a revision that *replaces* an already-consumed text or image (only an additive
one ran), and anything on agautolab1 beyond retrieval (its deployment is still `b38a6af`).

## The human's evaluation, in their words

- Step 4 delivery: 「MVPは合格。」 — the MVP passes — followed by the next request in
  `todo.md`.
- Step 5 delivery: 「依頼通りに仕様が追加されました。合格。」 — the feature was added
  as requested; pass.
- On the content rule: 「round-1ルールが不適切です。生々しい残酷描写はしないが、恐怖や消化と
  いう目の前の現実ははっきりと明記する必要があります。」 — recorded project-wide in
  `GOAL.md` (`f9f8d40`).

No reference was lost. One misattribution: the Omni Agent's reading on the content
rule was recorded as "the Developer's own reading" by Front's relay (report 4); the
Developer then overrode the rule anyway, so the record now carries their own words.

## Human effort and agent cost

Human: six texts, five images, `human_advice.md`, two pushes, two plays, three short
messages. Agent runs for the phase (steps 4–6, 15:00–16:20 UTC):

| Role | Runs | Cost |
|---|---:|---:|
| Front `front` | 33 | $5.35 |
| autolab `superdirector` | 9 | $2.86 |
| autolab `supercoder` | 8 | $4.60 |
| **Total** | 50 | **$12.81** |

Steps 1–3 bought no agent run. forge: one local SwarmUI generation.

## Omni Agent work done for in-system agents (handoff candidates)

1. **Requester stand-in**: relayed the Developer's material and decisions into `#front`,
   answered autolab's routine questions (2–5), accepted nothing itself. A Project Room
   or Front Desk post by the Developer would remove the relay.
2. **Verifier**: fresh-clone import, tests, byte comparison, windowed start, and viewing
   the images — the "disinterested eyes" Front lacks. A verification role with a
   headless Godot and an image reader could take it.
3. **Provisioner**: the Gitea account and repository, the consumers' `refs.toml`, the
   listener restarts.
4. **Watcher**: polling Front's topic — the Observer agent's job, not used here.

## Guides and notes updated

- agfront `8b4674a` (pj-agdev `30b61e2`): Front never plays or confirms a delivery;
  an acceptance is the developer's statement, quoted; a stand-in's decision names whose
  it is. Evidence: p1's "announces resolves it skips", p2's "I played f9f8d40".
- `devdocs/README_DEV.md`: "Human-authored references" section (step 3).
- `pj-agdev/.local/devenv.md`: account, repository, `refs.toml`, cache, pins, restarts.
- Guides of Front, autolab, forge, archsage (step 3); `agrefs --help` is the tool's own
  usage document.

## Repositories and revisions

| Repository | Revision | What |
|---|---|---|
| `developer/protoprey-refs` (human) | `a3c8196`, `0cd3c0b` | the references |
| pyagag | `b227b78` | `agag.refs` / `agrefs` |
| agfront | `8b4674a` | agrefs grant + guides; acceptance rule |
| agautolab | `4939ca9` | agrefs grant + guides |
| agforge | `43730aa` | agrefs grant + guides; `--init-image` |
| archsage | `b8099ac` | agrefs grant + guide |
| pj-agdev | `30b61e2` | submodule pins |
| `autodev/protoprey` | `1f4c2f4` (tag stays `v0.1.0`) | menu F: forest/wolf discovery, five-step predation, indicator |
| `autodev/protoprey-direction` | `26690b4` | two adoption entries, the content decision |
| devdocs | this episode | reports 1–6, `report.md` |

Conversations: `#front › front-protoprey-p2-wolf-event-20260923` (#8077–#8283);
`#pj-protoprey › workplan-wolf-forest-scene` (m8084), `workplan-wolf-discovery-indicator`
(m8243); `#work-m8084 › workrun-task1…4-m8084`; `#work-m8243 › workrun-task1-m8243`.

## What the one-scene trial proves, and what it does not

Proved: a human can author references in their own repository, publish by push, and
have four agents read the same revision by name; a scene was built from those originals
unchanged, with provenance in the project and the human's judgement recorded in their
words; a second push travelled through the same route to a second accepted build.

Not proved: production at scale (many scenes, many predators, long stories), a revision
that invalidates consumed work, forge's and archsage's use of references on a real
request, and any claim about the play experience beyond the Developer's two passes.

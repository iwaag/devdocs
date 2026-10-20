# p9 Step 5 — the reusable workflow

## Extraction

Omni's request #77 (conversation `p9-lowerpoly`) asked Front to have
lowerpoly derive a recurring workflow from `freework/run-001` and run-002.
`front/converse/run-035` delegated it with a delegate node (d1) and waited.
This time the run that promised the result carried it. Result reply #79.

`lowerpoly/devdocs/workflows/lod-chain.yaml` ("Phase 2 default LOD chain"),
on `main` (`5a58f5d`), validated with 0 errors:

| Node | Derived from |
| --- | --- |
| inputs | run-001 `reproduce`: `wfe run input --ref shared://…`, asset symlink |
| toolchain | run-001's symlinks to the original copy's toolchain, now `tools/check_toolchain.sh --link` (also prints versions; fails rather than rebuilds) |
| generate | `lod_pipeline.py … --budgets 2,4,8 --no-eval` |
| evaluate | the pipeline's evaluation, split out as `tools/eval_chain.py` |
| compare | run-001's `compare_repro.py`, generalised as `tools/compare_chain.py` (assets without a recorded row: "not compared") |
| publish | `wfe run publish … --candidate phase2-defaults` |
| present | run-001's page and run-002's corrected rule, `tools/build_review_page.py`; LAN check by curl |
| review | run-002 q1: the visual question, with page, table and downloads |
| cleanup | last; the page stays until the Developer says it may be closed |

The intent records this provenance and excludes failed or incidental shell
attempts. Bindings: root, study, wedo, lab (editable). Execution:
`lowerpoly-agent`, profile `sonnet`, the project guide. `freework` is kept
for exploratory requests.

## Reuse

1. **Same fixture.** `lowerpoly/lod-chain/run-001` (eagle_beak) completed
   with 9 nodes. All 35 recorded cells match. Its page is served on
   `http://agstudio.local:8301/`.
2. **A later, plain request.** It was a new conversation,
   `p9-lowerpoly-reuse`, message #80. It did not name the workflow ("make the
   usual LODs for wolf_tongue, comparable in the browser").
   - `front/converse/run-036` found `lod-chain` itself and delegated to it.
   - Run `lowerpoly/lod-chain/run-002` used input
     `shared://agshare/playprey/inputs/wolf/Wolf Tongue Model.glb`.
   - In about 4 minutes it gave L0–L4: 29,328 / 15,926 / 8,536 / 5,000 / 1,000
     triangles. All 35 cells match the recorded Phase 2 row.
   - Outputs: `o1` on the share.
   - Page `http://agstudio.local:8302/` (HTTP 200 from the LAN address).
   - Reply #82 asks the Developer when the page may close, and keeps the
     visual verdict theirs. Both run-036 and lod-chain/run-002 now wait on
     that answer. Omni does not answer it, because it is the Developer's
     review.

## Knowledge

- Front: `knowledge/review-pages.md` (pj-front `main`). It holds both
  delivery lessons:
  - a relay is not the Developer's answer;
  - a promised follow-up needs a delegate node.
  It cites converse/run-032/033 and lowerpoly freework/run-001/002.
- lowerpoly: the workflow's provenance and the tools in `lab/`. The
  toolchain location is a default in `check_toolchain.sh` (the original
  copy on agstudio); the setup notes stay in that copy's ignored
  `localenv.md`.

## Remaining constraints

- The toolchain is not reproducible from the repository. It exists only in
  the original copy on agstudio, so the workflow runs only there. Rebuilding
  it elsewhere is future work.
- Open review pages: `:8300` (freework/run-002), `:8301`
  (lod-chain/run-001), `:8302` (lod-chain/run-002). They stay until the
  Developer closes them. The open runs wait on the Developer's visual
  answers.

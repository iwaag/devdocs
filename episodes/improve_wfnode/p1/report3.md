# improve_wfnode p1 step 3: catalog and agent support

Date: 2026-11-03. Plan: [plan.md](plan.md). Contract: agdevworld `workflow_editor/docs/catalog.md`.

## What changed

- `shared/catalog.ts`: `TemplateContent.expectations` and `.example`, validated, normalized, part of a revision, kept in a run's snapshot and copied by `instantiate`. No template owns a group.
- `wfe nodes create|update`: `--expect <key>[:<strength>[:<note>]]`, `--no-expect`, `--example`, `--no-example`. An update keeps what it does not name. `wfe nodes show` and the catalog brief state them. The HTTP route takes them too.
- **Seed `present`**: the description no longer says "start its server". It says to make the result available as the node's expectations say, and to confirm that it can be reached that way. The browser assumption is now an explicit `webbrowser` preferred expectation. This affects only newly initialized catalogs.
- **agautolab** `f067be0`: `agent/guides/wfrun/reference/work.md` gains two bullets pointing at `wfe run help specialize` and `wfe run help expectation`. The semantics stay in WFE's help and contract.

## Reuse of `wfe nodes instantiate` and `wfe run adjust`

No specialization helper was added. `instantiate` already makes a replacement node from a template, and `run adjust` is the atomic path. Whether an agent needs more is left to the trial in step 5.

## A defect found and fixed

`parseCatalog` built each template without `target`, so a delegate template's target (p10) was lost as soon as the catalog was read back, while `contentOf` and instantiation expected it. It was on the same line I had to change, so it is fixed and has a test. Reading the persisted catalog: no current template has a `target`, so nothing was lost there. This defect was found by me while reading the code, not reported by anyone.

## Audit of the persisted catalog

`.local/agdev/node-catalog.json`, generation 12, 9 templates, read and not yet changed.

| Template | By | Disposition |
| --- | --- | --- |
| `present` r2 | Omni Agent (fix4) | Revise after deployment: same description change, add the expectation, **keep its expectation text**, which is longer than the seed's (cleanup sentence) |
| `cleanup`, `prestudy`, `boast`, `work`, `human-review`, `tidy` | various | Unchanged: no browser-specific wording that conflicts |
| `conditional`, `placeholder` | ponderer-agent | Unchanged. `placeholder` overlaps with examples in purpose, but it is the person's project agent's own edit and describes a different intent: anything may replace it |

The revision waits for deployment, because the running release's `parseCatalog` would drop the new field. It is recorded in the final report.

## Checks that ran

`test/catalog.test.ts`, `catalogrun`, `instantiate`: 19 of 19 passed, including 3 new tests. `reminders` also passed before the wiring fix. agautolab `tests/test_guide.py`: 4 of 4. `tsc` clean.

## Revision

agdevworld `d66b3d4`; agautolab `f067be0`.

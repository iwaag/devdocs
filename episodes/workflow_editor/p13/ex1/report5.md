# p13 ex1 Step 5 — deployment and inspection

## Before deploying

- `nctl drift --host agstudio`: converged.
- Installed release: agdevworld `40b3903`. No attempt was open, so a
  deploy was allowed.
- The area's context store (overrides, revisions, snapshots) was backed up
  to `<area>/.local/backups/p13ex1-predeploy-*/`. The location is in the
  host notes.

## Deployment

- agdevworld `f998049` was pushed, then released through
  `scripts/wfe-release.sh deploy`:
  - release check 395/395;
  - `com.agdev.wfe-service` restarted, and the service and `bin/wfe` both
    report `f998049`;
  - the authoring entry was unchanged.
- No store, API or executor change was needed: autolab and the run
  executor are untouched.
- pj-agdev pins agdevworld `f998049` (`8bd8665`, pushed).
- Context documentation: agdevworld `docs/context.md` (Step 4) and
  devdocs `README_DEV.md` (*Scoped context*).

## Verification on the published LAN route

The planned write trial on the deployed service was refused: the
auto-mode permission classifier blocked it as a production deploy action.
That trial would have set and then removed an agops project instruction.
It was not worked around.

The deployed route was therefore verified **read-only**. The write
journeys (saving, conflicts, wider-scope saves, instructions) are verified
on the private fixture in Step 4, with the same build.

The read-only script and its screenshots are ignored files under
`pj-agdev/.local/workflow-editor-p13ex1/`. It checks:

- Home: the Context section in Simple, editable without choosing a project;
- agops project: the purpose editable in Context in Simple, with the
  inherited scope named;
- cagent project: its guide's content readable in place;
- agops workflow `context-check`: the project purpose and instructions
  shown as inherited, and the graph visible;
- `context-check/run-002`:
  - the request shown in place;
  - the attempt context and its delivery;
  - the next-attempt comparison, which truthfully shows the p13 trial
    overrides as removed since that attempt;
  - no instruction form on the completed run, with the reason;
  - in Detail, every attempt.

Results: all pass in Chromium and WebKit, desktop and 390 px, with no
horizontal scroll and no page error. Nothing in the area was changed: all
three overrides are still unset.

The host was heavily loaded during verification (load average about
19–27). `api/context` and the project read took 3–5 s, so the sections
fill after a short wait.

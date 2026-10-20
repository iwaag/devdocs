# p15 Step 5 — deployment, and the embedded Front Desk retired

aghangout is deployed at `/hangout/` on the agdev origin and runs from its
own releases. WFE Home links it, and old Front Desk links open the same
conversation in it. The embedded Front Desk, the Arguing Room and their
bundle are deleted. The relay keeps only what its current consumers read.
The placement is recorded in Nautobot and converges on a live HTTP check.
Another LAN machine opened and inspected the deployed result in Chrome.

## State before

- `nctl drift` showed 46 converged and 1 drifting (`agobserver`
  `service_missing`, pre-existing). `agdevworld-web` was the only agdevworld
  service registered.
- Desired state was exported to `pj-agdev/.local/p15/step5/desired-before.yaml`.
- Stores were backed up in Step 2. The code and data recovery path is in
  the ignored `pj-agdev/.local/p15/notes.md`.

## Deployed revisions

| Component | Revision | How |
| --- | --- | --- |
| aghangout | pj-aghangout `b37c314` | `scripts/deploy.sh`: clean export, `npm ci && npm run check`, `releases/b37c314…`, `current` |
| WFE | agdevworld `034c0c5` (via `667c55c`) | `wfe-release.sh deploy`, 456 tests each time. The header links "aghangout ↗" (`/hangout/`). Run file routes accept project-root paths (below) |
| Web container | agdevworld `667c55c` | `docker compose up --build -d web`. Previous image tagged `agdevworld-web:pre-p15` |
| Relay | agdevworld checkout ≥ `667c55c` | Installed plist lost its Zulip variables (copy kept); `bootout` and `bootstrap` |
| Slack bridge | `a988fe9` (Step 2) | Unchanged in this step. Its default conversation link is `/hangout/?conv=` |
| Nautobot | ansible_agdev `77b5345`, pj-clusterintent `fa34bd2` | Service `aghangout`, endpoint `aghangout` (agstudio, http :8093), placement `aghangout-agstudio`, profile `aghangout_web` |

## The site now

| Address | Answer |
| --- | --- |
| `:8093/` | 302 → `/wfe/` (Home) |
| `:8093/wfe/…` | the workflow editor service (unchanged) |
| `:8093/hangout/` | aghangout's `current` release, served from a read-only mount. A redeploy of aghangout changes only the mount's `current` link, with no agdevworld build or restart. Same origin as `/wfe/`, so its writes pass the service's write-origin check as they are |
| `:8093/?view=frontdesk&conv=<id>` | 302 → `/hangout/?conv=<id>`: the old Front Desk and Slack links. This is one nginx rule; no compatibility code |
| `:8093/?view=argue`, any other path | 302 → `/wfe/` |
| `:8090/…` | redirects to the same host's `:8093` with the same rule |

## What was removed

- **The embedded Front Desk and the Arguing Room** (agdevworld `667c55c`):
  - `src/`, about 6,000 lines, including the Phaser scene, room adapters,
    demo, settings, contexts, progress, completion and post-meaning code;
  - `public/frontdesk/`, `index.html`, the root `tsconfig.json`, and
    Phaser and Vite from the root package;
  - `settings.example.toml` and `contexts.example.toml`;
  - the build stage of the Dockerfile. The image is nginx and its
    configuration.
- **Relay routes without a current consumer.**
  - The remaining consumers were checked by searching every agdev project,
    pyagag, the observer and the launchd templates. Only WFE Usage (and
    Front, by `curl`) reads the relay, through `/budget` and `/cost`.
  - Removed: `/settings`, `/contexts` (read and write), `/progress`,
    `/argues`, the completion door, the Zulip mirror and its
    subscription keeper, and the Developer's Zulip write credential. That
    is 14 modules and 11 test files, and pyagag is no longer a dependency.
  - The relay holds no credential now. Retired variables are reported at
    startup.
  - Its earlier restart had spent a 653-call realm resync; that read is
    gone.
  - The relay's tests: 17 pass, including a new route test.
- **Kept as history.** Nothing was deleted from the stores. The relay's old
  mirror store (`agentroom/.local/mirror/`, ignored and disposable) is
  simply no longer read. The `agdevworld-settings` repository remains the
  source the aghangout assets were copied from.

## A failure found on the deployed route

From agpc, the WAV link in forge's result (#103) answered 404. forge had
written `raw?path=devdocs/runs/provide/run-010/deliverables/chat-pop.wav`,
a project-root path, where the route took run-folder paths. A path that
starts with the run's own folder names exactly one file, so the route now
accepts both forms. Test: `test/access.test.ts`; documented in
`docs/access.md`. This also repaired the links already posted.

## Nautobot

- First registered as `manual_toolchain`. That reads presence from a
  process identity, and aghangout has no process of its own (agdevworld-web
  serves it), so drift stayed `service_missing`.
- Now `nctl_managed` with an observe-only profile `aghangout_web` that
  probes `/hangout/` on the endpoint. This is the pattern `ollama` uses.
  `nctl reconcile agstudio` plans no actions.
- After `nctl observe agstudio`, drift is **47 converged, 1 drifting**. The
  one drifting is the pre-existing `agobserver`. `aghangout` converged from
  an `http_probe` of `/hangout/`.
- Files: `step5/drift-before.json`, `step5/drift-after.json`,
  `step5/register-aghangout.yaml`.

## Verification from another LAN machine

The read-only check `step5/deployed.mjs` ran on agpc in Google Chrome with
Playwright, against `http://agstudio.local:8093`. Output is in
`step5/deployed-agpc.txt` and screenshots in `step5/shots/`. All 12 checks
pass:

- the lobby offers front, forge and archsage, all available, with the
  izakaya room drawn;
- `/?view=frontdesk&conv=p15-s3-forge2` lands on
  `/hangout/?conv=p15-s3-forge2`;
- the partner is the recorded destination (Forge), and lines carry
  `forge-agent`;
- the newest line is forge's result, and its WAV opens from agpc
  (`200 audio/wav`, after the fix above);
- the History panel shows the whole conversation;
- the run chip opens the WFE run view;
- Home links `/hangout/`;
- there are no page errors;
- the phone layout has no sideways scroll.

## Review links

- aghangout: `http://agstudio.local:8093/hangout/`
- the forge round trip:
  `http://agstudio.local:8093/hangout/?conv=p15-s3-forge2`
- archsage: `http://agstudio.local:8093/hangout/?conv=p15-s3-archsage`
- Home: `http://agstudio.local:8093/wfe/`

# retire_old_ui p1 — Step 3: the dashboard is the main entrance

agdevworld `91f58bd`. Not deployed yet. Verified on a temporary nginx
container with the new bundle on alternate ports, proxying `/wfe/` to the
running service.

## Changes

- **nginx** (`nginx.conf`):
  - On the agdev origin (`:8093`), `/` with no query answers
    `302 /wfe/`. The redirect is relative (`absolute_redirect off`), so it
    keeps the port the browser used.
  - On the static-only listener (`:8090`), it answers
    `302 $scheme://$host:8093/wfe/`, built from the request's host.
  - `/` with a query is served as before, so `/?view=frontdesk&conv=<id>`
    (the Slack links) and `/?view=argue` keep opening their screen.
  - The `/wfe/` route is unchanged: Host passed on, `x-wfe-route` set,
    SSE unbuffered.
- **Bundle entry** (`src/main.ts`): only the Front Desk / Arguing Room is
  loaded. Any other address, such as an old `/?view=nodes` or
  `/?parts=gauge`, is sent Home with `location.replace`. This is the site's
  default, not a compatibility route: no retired screen is rendered.
- **Shared links.**
  - The Front Desk's "Operation room ↗" became "Home ↗", the agdev URL of
    the host the page was opened from.
  - Dashboard screens link Usage and the Front Desk (Step 2), built from the
    request's host: under `/wfe/` it is `/?view=frontdesk`, and from the
    service's own port it is the same host's `:8093`.
  - `index.html` loses the 340 px right inset that the world views' chat
    overlay needed, and its title names the Front Desk.
- **Navigation from Home**:
  - projects (tiles) → project editor → runs;
  - waiting and running runs, linked from the tiles and the Execution nodes
    card;
  - execution nodes;
  - Context (the section, and the overview in Detail);
  - Usage;
  - User nodes;
  - Front Desk.

## Host-path operations

Unchanged by this step, and recorded as found. The dashboard text claimed
"through agdevworld the service refuses host paths". That gate was removed
in `b9886d5` (workflow_editor p11, no process gates). A routed
`POST /wfe/api/workspaces {"path": …}` is evaluated, and so it is on the
currently deployed proxy too: the same request returns the same
"not inside a Git repository" refusal from both. Nothing about which
operations the web origin offers changed. The stale sentence in Home's
Detail and the nginx comment were corrected to say what is true.

## Validation (Chromium, `.local/retire-old-ui-p1/step3-entry.ts`)

- `:18093/` lands on `:18093/wfe/`.
- `:18090/` lands on `:8093/wfe/`.
- `/?view=nodes` lands on Home.
- For both listeners:
  - `/?view=frontdesk&conv=p8ex1-trial-b` stays on that URL;
  - the page reads `…/wfe/api/conversations/p8ex1-trial-b`;
  - "Home ↗" points at `…:8093/wfe/`;
  - there is no request to a retired relay route and no page error.
- `npm run build` passes.

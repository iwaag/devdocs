# workflow_editor p15 — aghangout: final report

`aghangout` is an ordinary managed project and a deployed application. In it
a person meets the characterized project agents and talks with each one
directly: Front, forge and archsage, but not autolab. WFE owns the
conversations and their execution through a documented contract, and every
conversation records its destination. aghangout's own visual-novel
presentation runs from its own releases at `/hangout/` on the agdev origin.
The embedded Front Desk and Arguing Room are retired.

Everything below is deployed and was verified from another LAN machine.

Step reports:

- [report1](report1.md): boundary and decisions;
- [report2](report2.md): the WFE contract;
- [report3](report3.md): the three partners;
- [report4](report4.md): the application;
- [report5](report5.md): deployment and retirement;
- [report6](report6.md): trials.

## Review links

- aghangout: `http://agstudio.local:8093/hangout/` (over Tailscale:
  `http://agstudio.tailab7641.ts.net:8093/hangout/`)
- A forge conversation with a full specialist round trip (sound, blue
  lantern): `…/hangout/?conv=p15-s3-forge2`
- An archsage consultation steered mid-flight:
  `…/hangout/?conv=p15-s6-steer`
- Home: `http://agstudio.local:8093/wfe/`
- aghangout's own change run:
  `http://agstudio.local:8093/wfe/#/ws/aghangout/run/change/run-001`

## Changes

| Repository | Commits | What |
| --- | --- | --- |
| agdevworld | `a988fe9`, `626c43f`, `667c55c`, `034c0c5` | Conversations record their destination (store v3, targets, refusals, recovery to the recorded destination), and the client contract and docs. CLI targets and `--to`; unmistakable usage errors. The Slack target and aghangout links. The Front Desk, Arguing Room and their bundle are deleted, and the web container is nginx routing to `/wfe/` and `/hangout/`. The relay is down to `/budget` and `/cost`. Run file routes accept project-root paths |
| pj-aghangout (new) | `b37c314`, `25fc489` (by aghangout-agent), `3fa9a20` | The application, fixture, browser checks, deploy script, README, guide, workflows |
| pj-forge, pj-archsage | `5ec1c96`, `1234b03` | `converse` workflows and guides. The project guides say how a person reaches them |
| pj-front | `ccd4b6a` | Guide: the shared reply facts moved to the wfrun guide |
| agautolab | `960fd62`, `f37db08`, `5fdc923` | wfrun guide, *A run that serves a conversation message* |
| pj-agdev | pins through `45eaa19` | agdevworld and agautolab pins; launchd template and README for the relay |
| pj-clusterintent | `fa34bd2` (ansible_agdev `77b5345`) | `aghangout_web` observe-only profile |
| devdocs | this episode; README_DEV `f290500` | Reports. README_DEV covers *Conversations with project agents*, the agdev site, Front's entrance, the relay, Slack |

## Completion criteria

1. **aghangout is an ordinary managed project with an independently
   deployed application and a documented development path.**
   - Created with `wfe create`: Gitea `developer/pj-aghangout`, workspace
     `aghangout`, agent `aghangout-agent`. It has a README, a guide, and the
     `freework` and `change` workflows.
   - Its `scripts/deploy.sh` installs releases served at `/hangout/`.
   - aghangout-agent changed, checked and deployed it itself
     (`change/run-001`) without an agdevworld build.
2. **Front, forge and archsage are directly reachable as distinct
   partners with useful character presentation; autolab is not offered.**
   - The targets are those three.
   - Each answers its own conversation through its own `converse` run.
   - The lobby, rooms, portraits and dialogue reuse the Front Desk's art.
   - The name plate shows the actual author.
3. **WFE owns conversations and execution through a documented contract
   that supports multiple destinations without UI-specific routing.**
   - `docs/conversations.md` and `wfe conversation help` document it.
   - Slack and aghangout are two clients of it.
4. **Requests, follow-ups, questions, progress and results stay correlated
   across concurrency, retries and restart.**
   - Replies name their run and `re`.
   - Answers and steers carry their source message (fixed and rerun in
     Step 6).
   - Token and receipt replay was checked live across a WFE service restart
     and an executor restart.
   - The store has no duplicates.
5. **The application uses current workflow evidence, not a Zulip-era import
   or progress model.**
   - Status, work and results come from WFE's conversation view: runs'
     shared status, run views, file routes, presentations.
   - The relay's progress board and its Zulip mirror are gone.
6. **No unnecessary backward-compatibility layer or human approval gate.**
   - One schema migration maps the old conversations once.
   - The old desk URL is one nginx redirect.
   - Nothing waited for approval.
7. **Active client delivery is accounted for; optional history and removed
   surfaces are documented.**
   - No turn was pending at cut-over.
   - Slack's 10 thread bindings kept their conversations, now explicitly
     `front/converse`.
   - Workflow-era history was retained in place. No Zulip-era import was
     made.
   - What was removed is listed in report5.
8. **The embedded Front Desk is superseded, shared dependencies have
   explicit owners, and the result is deployed and verified over the LAN.**
   - The relay's remaining consumers are WFE Usage and Front's `curl`.
   - The agpc check passed 12/12.
   - Nautobot `aghangout` converges on an HTTP check of `/hangout/`.
9. **The Developer can inspect the result through the prepared links with
   minimal actions.** See *Review links* above.

## Remaining limitations

- **One conversation slot.** The three partners share the coordinator's
  `conversation` lane; the executor runs one claim loop per lane. A turn
  queues only behind another turn (archsage measured 44 s with ≈20 s
  queued). Long work done inside a turn delays every partner. The guides
  steer real work into delegate nodes, which free the slot.
- **A message to a destination that never becomes available** stays
  `failed` and is retried by every reconcile (trial message #109 to
  `lowerpoly/converse`). It is visible and harmless, but it never stops by
  itself.
- **Recorded answers.** Agents sometimes paraphrase the person's words when
  recording an answer on a run; the source message reference stays exact.
- **Browser trials and the configured person.** aghangout posts as the
  configured person. Automated trials through the page add a trial author in
  the request body, which only a driver can do.
- **Unread leftovers.**
  - The relay's old Zulip mirror store (`agentroom/.local/mirror/`) and the
    `Opsroom Observer` credential are no longer read. They were left in
    place, not deleted.
  - The `agdevworld-settings` repository is no longer synced by anything.
    aghangout copied the assets it needs, with provenance.
- **Not built:** XR, themes, plugins, group chat, new artwork (out of scope
  by the plan).

Deus Ex Machina note: the aghangout initials fix (`3fa9a20`) was done by the
Omni Agent — handoff candidate for aghangout-agent.

Recovery for this host, code and data, is in the ignored
`pj-agdev/.local/p15/notes.md`. The environment notes are in
`pj-agdev/.local/devenv.md` (*workflow_editor p15 on this host*).

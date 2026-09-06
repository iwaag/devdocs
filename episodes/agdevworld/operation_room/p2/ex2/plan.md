# operation_room p2ex2 — clearing the leftovers before p3

Close out the three small items p2 left behind, before starting p3 (routine
display + chat). They are independent; any order, parallel is fine.

## A. Carry out the done-row confirm addendum

Already planned: execute `../ex1/plan.md` (written as `confirm/plan.md`, later
renamed) as written (the relay's
`POST /ops/confirm` clear-all, done-rows-only, in-memory record, excluding
done from the "N rows open" count, and the full-cycle testbed verification).

## B. Make the relay a launchd resident

Stop starting it by hand. Now that the relay holds a state engine that costs
241 calls to rebuild on every restart, the case for residency is stronger
than it ever was in the agent_room era. It is also a prerequisite for p3's
everyday chat use.

- **Template precedent**: `pj-agdev/devenv/launchd/` has the full set of
  `com.agdev.*.plist.in` (`routine-gui` (a resident http.server),
  `agfront-zulip`, etc.). Add `com.agdev.agentroom.plist.in` in the same
  style; keep the generated plist and concrete values in ignored territory as
  usual (no absolute paths in non-ignored files).
- **Known launchd traps** (each with a measured precedent in this
  environment):
  - Set `PATH` explicitly in the plist. The cagent-api precedent: without it,
    `uv` is not found and everything dies.
  - Pass the environment variables (`AGENTROOM_ZULIP_ENV` /
    `OPSROOM_ZULIP_ENV` / `AGENTROOM_STALLED_SECONDS`) via the plist. Write
    only the paths to the env files, never credential values.
  - Resolving the Zulip host via an mDNS `.local` name can stall for seconds
    on unanswered AAAA queries. If slow, force IPv4 or use the IP/localhost.
  - macOS Local Network permission attaches per binary. Run the equivalent of
    `serve.sh check` under the launchd interpreter and confirm connectivity
    before calling residency done (this doubles as the dry run for p3's
    ComfyUI/ollama work).
- **Operations**: code reload is
  `launchctl kickstart -k gui/$(id -u)/<label>`. Logs go under `.local/`.
  `KeepAlive` so it comes back when it dies is enough (a restart costs 241
  calls, which is acceptable unless it becomes frequent — deal with that if
  it happens).
- **Verification**: a pid in `launchctl list`, `curl /healthz`, `/ops` alive
  after a Mac re-login, and the event-queue expiry → re-register cycle
  running unattended.

## C. Retire agping-agstudio1

The decision item from the p2 report. With the project no longer existing on
any machine, regenerating a fixture just to clean a board row is backwards,
so choose **retirement** (if it is ever wanted again, `agag init` can recreate
it from the runsmoke1 records).

- ✔-resolve the `intro-agping-agstudio1` topic in `#agents`. This is the
  episode's single deliberate Zulip write, and it is done **with the
  developer credential**. The observer bot (Opsroom Observer) still writes
  nothing.
- After the resolve, check both the ops board and the agent_room view. If the
  relay or agent_room does not yet implement "a resolved intro topic = a
  retired agent", the amber unknown card will linger — in that case add the
  small fix to both views: exclude resolved intros from the roster/list
  (strip the ✔ per the bare-topic keying convention, then exclude on the
  resolved flag).
- Whether to deactivate agping's Zulip bot account is discretionary (leaving
  it does no harm once it is off the board).

## Out of scope (for the record)

- The ENT candidate from p2's Deus Ex Machina note — "an agent told its
  introduction is stale re-posts it itself" — is not handled in this episode.
- The process/backend-layer tiles and the routine chat are p3+.

## Constraints (minimal)

1. The observer bot does not write to Zulip (C's resolve uses the developer
   credential).
2. No credential values or absolute local paths in non-ignored files.
3. The confirm work follows the constraints in `../ex1/plan.md`.

Everything else is the implementer's discretion. One combined report.md for
all three items is fine.

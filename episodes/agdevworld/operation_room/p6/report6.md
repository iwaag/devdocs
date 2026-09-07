# Step 6 — verify, deploy and report

Relay tests: 115 pass. The ones added in this phase cover manual fire
recognition and the trigger.sh wording contract, the first fire of a routine
with no fire topic, resolution from resolve notices inside a fire's span,
reopening as the newest notice, the latest span following the topic's current
flag, restart behaviour (the notices are re-read from the realm's history, so
the engine-level test feeds them through the same event path a sweep and the
queue both use), filtering before the three-session limit, the uncertain send
result, and icon assignment. Every write test uses a recording or exploding
client; no test touches Zulip. `npm run build` passes with the existing
Phaser chunk advisory.

Assembled UI, driven over CDP against the live relay on `:5173`: new-session
selection with mocked sends (step 2), ordinary continuation still bound to
the fire topic, the Show resolved filter with fallback and empty state (step
3), chat boundaries from the relay's `start_id`/`end_id`, latest-only host
evidence, both graph modes on the branching `mediagen` session (step 4), and
in this step an injected `truncated: true` on that session drawn compact with
the amber bounds line, then a simulated fetch outage: health, every routine
chip, the composer, the start button with its reason, and the host line all
went unknown; recovery seven seconds later restored live health with the
selected routine and fire, the chat draft and the New session instruction all
retained. Screenshots at agdevworld `.local/p6/step6/{recovered,narrow}.png`
plus the per-step ones under `.local/p6/`. The 390-px viewport was captured
in steps 4, 5 and 6.

Service state through nctl: `nctl status` passed (Nautobot reachable,
authenticated, catalog and GraphQL present). `nctl drift` still reports
`agfront: service_missing` (error) for the `agfront-agstudio` placement while
the agent row says `liveness=polling`. That is a stale desired-state
observation, not a start blocker: the placement's `process_pattern` is
`agfront\.zulip_listener`, but the running launchd job (`com.agdev.agfront-zulip`,
running, serving `front-routine-papers` on 2026-09-06) executes
`python -m agfront.listener`, and the fresh agstudio nodeutils dump contains no
`zulip_listener` string. The desired state's pattern should be updated to
`agfront\.listener`; that is a Nautobot desired-state change and is left for
the cluster owner (did the diagnosis for agent cagent — handoff candidate).

Deployment: the served web image was rebuilt (`docker compose up --build -d
web`, HTTP 200 on `:8090`, the served dashboard chunk carries the new
strings). The launchd relay was restarted after each relay change, the last
time with the icon fix; `GET /` lists the new POST route. `agentroom/README.md`
documents the start route, the `resolved=hide` query and the session fields.

Not done, deliberately: no real start was posted. A real start buys a Front
run and runs the routine's actual work (autolab missions in the case of
`papers`, `publish` and `ghtrends`), which is outside what this phase was
authorised to spend on its own; the route was exercised live only through
its refusals and through the mocked send path. Remaining limitations, as
recorded in the step reports: resolution is known only inside the 200-post
window of a fire topic and older spans stay unknown; a topic that carries both
a `✔` and a bare form of the same name is folded into one entry by the
engine's bare-topic keying; display metadata appears only when a Developer
adds a `display:` line to a future version of a standing request, so today's
eight routines show assigned icons and their internal names as titles.

# operation_room p2 ex2 — clearing the way for p3

Three small things p2 left behind, all of them now closed. Steps are reported
in `report1.md`–`report3.md`; this is the summary.

| step | outcome |
|---|---|
| A | The `done`-confirm addendum was **already built** — as `p2/ex1`, under a different path than the one this plan points at. Verified live rather than assumed: 43 tests, `confirmed: {rows: 0, topics: 0}` on the running relay, a `200` on confirm-all and a `404` on a conversation that is not on the board. |
| B | The relay is **launchd `com.agdev.agentroom`**. Both of the plan's traps checked from inside the job's own domain before it was trusted. `KeepAlive` proved with a `kill -9`: `live` again in 35 s. |
| C | `agping-agstudio1` **retired**. One ✔ in `#agents` as the Developer, and — because the relay was built to survive that rename — a new rule in both halves: a resolved introduction retires its agent. The board is at zero rows. |

## The three constraints

1. **The observer never posted.** The episode's one realm write is the ✔ on
   `intro-agping-agstudio1`, made with the developer credential. The
   subscription the observer re-makes on every start is p2's, not this
   phase's.
2. **No credential value and no absolute local path in a tracked file.** The
   plist is a `__PROJECTS_ROOT__` template; the generated copy is in the
   ignored `.local/launchd/`.
3. **ex1's constraints untouched.** `confirm` is still `done`-only, still in
   memory, still writes nothing to Zulip.

## The three things worth carrying forward

**1. A plan's own pointer is worth resolving when the plan is written.** Step
A names `../confirm/plan.md`, which resolves to nothing; the work was real and
finished, filed as `ex1`. A dangling pointer with the work *not* done reads
exactly the same, and the only reason this one cost nothing is that somebody
went and looked.

**2. A permission that is per binary needs a check from the right domain.**
macOS Local Network permission attaches to the responsible process, and a
launchd agent is its own. `serve.sh check` was therefore run *as a launchd
job*, at the 50-call cost rather than 241, before the daemon was bootstrapped.
It passed — which is the useful half of the p3 dry run: the trap is real, and
this interpreter path is already approved. p3's ComfyUI and ollama tiles will
be a different binary and get no benefit from that.

The neighbouring trap turned out **not** to apply: `agstudio.local` is this
host's own name and resolves in ~5 ms, no AAAA stall. Three curls settled what
would otherwise have become a defensive IPv4 workaround nobody could later
justify.

**3. The screenshot found the defect again — the fourth phase running.** The
retirement change broke `/work`, because `_intro_topics` has two callers and
only one of them was in my head. `/agents` answered perfectly, 50 tests passed,
the build passed, and the agent room said *"unreadable · ValueError: too many
values to unpack"*. The build cannot know which of its outputs a human was
about to look at.

## What p3 inherits

- A relay that is up without being started, and a board showing **zero rows**
  — the first honestly empty operation room this project has had.
- Two half-open verifications, both named in `report2.md` rather than assumed:
  the event queue's expiry-and-resweep was exercised through `kill -9` but
  never through a real expiry (the `queue_id` is not exposed on any route),
  and survival across a Mac re-login rests on the nine sibling jobs rather
  than a fresh test — worth one `launchctl list | grep agentroom` after the
  next login.
- The agping **Zulip bot account is still enabled**, deliberately: nothing
  reads it, and it is one less thing to undo if the fixture is ever
  regenerated from `runsmoke1`.

## Out of scope, as the plan said

The ENT candidate from p2's Deus Ex Machina note — *an agent told its
introduction is stale re-posts it* — is untouched. Step C is a small argument
for it, and also for its limit: agping had no process left to tell.

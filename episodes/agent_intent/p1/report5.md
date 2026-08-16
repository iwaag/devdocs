# agent_intent p1 — Step 5 report: drift evaluation, with liveness kept out of it

AI-generated (Omni Agent, 2026-08-17).

## The two lanes

`nctl/src/nctl_core/drift/agent_evaluation.py`, deliberately the same shape as
`workspace_evaluation.py`: convergent verdicts leave as `gaps`, and the
informational class travels in its own field where no comparator can promote
it to drift by accident.

**Registration gaps (real drift, error severity)**

| code | means |
|---|---|
| `agent_zulip_account_missing` | the declared `zulip_user_id` is not in the realm |
| `agent_zulip_account_deactivated` | the account exists but is deactivated |
| `agent_zulip_channel_unsubscribed` | a desired channel is not subscribed (names them) |
| `agent_plane_membership_missing` | the declared `plane_user_id` is not a workspace member |

Warnings, because they are holes rather than disagreements:
`agent_zulip_identity_undeclared`, `agent_plane_identity_undeclared` (the
desired row has no id to match on) and `agent_registration_unobserved` (nothing
collected yet). A non-`active` agent is not evaluated for registration at all.

**Liveness (never a gap)**

Every agent target also carries one **info**-severity `agent_liveness` diff
whose evidence holds `liveness_class`:

- `polling` — the status file was written within three long-poll windows (3 ×
  90 s = 270 s);
- `stale` — older than that;
- `unobserved` — with an explicit reason: `no_desired_workspace`,
  `no_realized_device`, `workspace_unobserved`, `no_status_file`,
  `status_file_unreadable`, `status_age_missing`.

`derive_status` changes a target's status on error-severity diffs alone, so a
stopped listener shows up in `nctl drift` without ever making an agent
`drifting`, and no reconciler maps the code. That "unobserved with a reason"
vocabulary matters as much as the classes: *we looked and saw nothing* and *we
never looked* are different facts, and collapsing them is how a monitoring
system starts lying.

**The age arithmetic avoids comparing clocks.** Effective age =
`age_seconds` (computed by nodeutils on the node, against the same clock the
listener wrote with) + seconds since that collection (computed here, against
this clock). Neither term ever subtracts one machine's clock from another's.
A pleasant consequence: a status file that was fresh when collected but
collected an hour ago goes `stale`, because a frozen observation is not
liveness. There is a test for exactly that.

## Live acceptance

Every criterion in the plan, run against the cluster:

**1. Enrolled agent is registered-converged with `liveness_class: polling`**

```
agforge  converged  1 diff(s)
    [info] agforge: liveness=polling
```

**2. Stop the listener: registration stays converged, liveness degrades to
`stale`, and no gap appears**

`launchctl bootout` on `com.agdev.agforge-zulip`, wait past the threshold,
re-collect:

```
status: converged
  info agent_liveness {"liveness_class": "stale", "workspace": "agforge",
    "age_seconds_at_collection": 323.925, "seconds_since_collection": 10.863,
    "effective_age_seconds": 334.788}
```

Converged, with zero gaps. That is the whole design working: the thing that
went wrong is visible, and it did not manufacture drift.

**3. A deliberately wrong desired channel produces a registration gap**

Adding `no-such-channel` to agforge's desired channels and re-observing:

```
agforge  drifting  2 diff(s)
    [info]  agforge: liveness=stale
    [error] agforge: agent_zulip_channel_unsubscribed
```

**4. Per-agent Plane attribution end to end** — met in step 2: an issue created
with agforge's own token records agforge's user id as `created_by`.

The listener was restarted and the desired channels restored afterwards;
agforge is back to `converged` / `polling`, and `nctl desired export` was
re-run into the operator input.

## Gates

| gate | result |
|---|---|
| nctl ordinary | 1336 passed (14 new evaluation tests) |
| nintent Django-free fast | 147 tests, OK, 10 expected skips |
| nauto ordinary | 121 tests, OK |
| nodeutils ordinary | 95 passed, 2 subtests |
| Nautobot runtime reuse gate | "No changes detected", `cases=263`, OK |

The evaluation tests deliberately assert the *negative* as much as the
positive: every `unobserved` reason asserts `gaps == []`, and stale liveness
asserts `gaps == []`. If someone later promotes liveness to a gap, those tests
are what will make them do it on purpose.

## Left undone, deliberately

- **`nctl agents` convenience view** — the plan marks it optional. `nctl drift`
  already prints the class per agent, and a second surface reading the same
  evaluation would be a maintenance cost without a current reader.
- **Zulip presence (step 6)** — optional bonus, not required; the status file
  is the source of record precisely because it also works when Zulip is down,
  which is when you most want to know.
- **`AUTOLAB_NODE_PLANE_CREDENTIALS_SOURCE` on agautolab1** — needs an Ansible
  run against a real cluster node, which is an explicit approval boundary in
  this repository's environment classes. Raised with the developer rather than
  done unilaterally.

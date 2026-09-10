# refactor p3 — Plane is retired

## What this phase set out to prove

That every workflow this realm depends on runs with **no external task
manager at all** — and then that Plane could be deleted without anything
noticing.

It is proven, and Plane is gone. One cagent change request and one full
autolab → forge → autolab development request went through their whole shape
live on 2026-09-10 with the service stopped and its credentials deleted, and
the containers, volumes, deployment files and accounts were removed
afterwards.

## The division of responsibility, as it now stands

| information | authority |
|---|---|
| a cagent change request and its progress | one `change-…` topic in `#cagent-agstudio1` |
| an agent's public contract and entrance | its `intro-<instance>` topic in `#agents` |
| conversations with that agent | the channel its introduction advertises |
| messaging identity | the Zulip bot user id |
| desired placement and expected service state | Nautobot |
| actual process/service health | the existing service observations |

Registration and health stayed distinct throughout: an account, a channel or
an introduction existing still proves nothing about a listener running, and
`nctl drift` keeps liveness info-severity so a stopped listener never makes
an agent drift.

## What changed, step by step

### Step 1 — cagent's change record is the conversation

`cagent/src/cagent_api/plane.py` is deleted. A `requested_change.md` now
opens `change-<stem>-o<the id of the asking post>` in cagent's own channel
and puts the statement there as an ordinary post, with `[selfnote][change]`,
`[origin]`, `[doc]` and `[state]` for what a program must resolve without
guessing.

**Identity is a message id** — the third time this phase family has reached
that answer, and for the third reason: the `[change]` note's own id *is* the
request, so a rename, a resolve or a reused topic name never redirects one
request's record into another's, and a deleted anchor is *absent* rather than
whatever took its name. The origin is told where its record is with a
selfnote, which buys nobody a run, so registering the same request again
restates it in one conversation instead of forking it.

cagent also acquired what every other agent already had: an instance name, a
channel of that name whose every topic it serves, and an `intro-` contract in
`#agents`. It was the one agent in this realm whose entrance nobody could
read.

And a defect the plan named was fixed: registration and observation were
sequential, so an exception in the first branch meant somebody who asked to
be told *and* shown got neither. They are independent now.

### Step 2 — Plane leaves identity and desired state

`nctl agents observe` read two realms; it reads one. `PlaneReader`, the
`[plane]` config section, `plane_user_id` on `DesiredAgent`, the three
observed Plane columns, the two drift codes and the export field are gone,
and nintent migration `0033` drops the columns. An agent's messaging identity
is its Zulip bot user id, and there is no second account for it to be missing
from.

### Step 3 — the obsolete screen and the shared client

The `tasks / plane` view and `planeState.ts` are deleted; the cycle is six
views. **Tracing its callers is the finding worth keeping**: every route it
used went through the assistant gateway `modernize_agdevworld` p1 deleted a
month earlier, so it was not a working feature being retired — it was a
screen whose every action had been failing silently. The plan's warning that
frontend code existing does not establish that its backend works was exactly
right, and it is why nothing else was touched.

`agag.plane` had no consumer left and is deleted, with the provisioning
templates that installed its credentials. A role that merely stops writing a
file leaves the old one in place, so the role now **deletes**
`.local/plane.env` on a node it touches — which is what makes "reinstalling
an agent cannot restore Plane credentials" true of the nodes that already had
one.

### Step 4 — stopped, proved, removed

Three desired declarations deleted through the ordinary desired-state
workflow; the stack stopped; the whole verification run while it was down;
then 13 containers, 10 volumes, one network, six images, the Compose project,
the per-agent accounts and both credential files deleted. Ownership was
established first: Plane's PostgreSQL, Valkey, RabbitMQ and MinIO were its
own, and every other stack in the realm is still running.

`agautolab1` — the node p1 and p2 deliberately left alone — was inspected
rather than assumed, found to be months behind with `plane.py` in its venv
and a live `plane.env` on disk, and redeployed.

## What it cost

19 agent runs and **$2.85** for the live demonstration.

## What is left

- **Two agents predate the exec-options contract** (agforge, arxivsage) and
  read as *unknown*. Unrelated, and correct.
- **`autolabState.ts` still calls the deleted `/api/autolab/*` gateway.** Not
  Plane's; three live views read it; repairing or removing it belongs to
  whoever decides that view's future.
- **`service_missing` on the `agfront` service target**, from an observation
  that predates this phase's service work, and two real
  `agent_zulip_channel_unsubscribed` gaps. Reported, not repaired — the plan
  asks for exactly that.
- **Historical text keeps its Plane vocabulary**: old reports, migrations
  `0031`/`0032`, and docstrings explaining what a module used to be. None is
  an active dependency, and neither is the unrelated sense of the word in
  `cagent status --help` ("health of the control plane itself").
- **Nothing is migrated out of Plane.** Its issues and comments are gone with
  its database. That is the plan's explicit permission, not an oversight.

## The through-line of the three phases

p1 took autolab's record out of Plane, p2 forge's, p3 cagent's — and each
time the same answer came back, sharper: **a conversation is the record, and
a message id is its name.** Plane was never carrying anything the chat could
not; it was carrying a second copy that had to be trusted to agree. Removing
it removed the agreeing.

# agag_builder p3 — report

Goal: everything from `agag init` to a listener Front can reach is agent
work, with only one realm-wide provisioner identity prepared by a human.
**Done.** All four success criteria are met. Step evidence:
[report1](report1.md) provisioner, [report2](report2.md) Zulip client,
[report3](report3.md) provision command, [report4](report4.md) autolab,
[report5](report5.md) live check, [report6](report6.md) record/final audit.

## Success criteria

| criterion | result |
|---|---|
| bot, local env, `#agents`, own channel + description from one command | `agag provision` and `agag init --provision`; live bot user 18 |
| dedicated provisioner, no Developer credential in the workflow | administrator user 17; ignored `provisioner.env` mode 0600; autolab gets its path only |
| autolab init → provision → intro → background listener; Front relay | runsmoke1 `main/agping/`; current intro 1429; Front relay 1427 |
| human checklist contains only irreducible human work | provisioner once, optional Plane account, permanent service install |

## Commits and deployment

| repository | commits |
|---|---|
| pyagag | `eed0e5e` provisioning API; `7598532` CLI/init/template; `0b226b6` compatible `--like` overlays |
| agautolab | `2b2a0dc` credential-path capability + guide; `b5e4ee7` final pyagag lock |
| agforge / agfront | `2c31c0e` / `c7c4881` pyagag lock alignment at the phase implementation commit |
| pj-agdev | `b6a7755` implementation pointers; `0ab3e5f` final autolab pointer |
| runsmoke1 main (Gitea) | `9c73b9d` agent skeleton; `1e85461` final agping pyagag lock |

Every GitHub-backed implementation was pushed before its consumer lock was
advanced. autolab, forge and Front were restarted after the initial lock
change; autolab was restarted again after the overlay fix and completed a
fresh startup sweep. agping remains a background-process fixture, which is
the intended scope boundary before launchd/Ansible generalization.

## Provisioner and command

The realm's actual `Realm.string_id` is empty, despite its host being
`agstudio.local`; the first `create_user --realm agstudio.local` attempt
therefore made no change. Creation with `--realm ''`, promotion to
Organization administrator, `fetch_api_key`, 0600 env write and `users/me`
verification succeeded. No secret value entered output or a tracked file.

`ZulipClient` now owns bot creation, email lookup and channel-description
updates. Bot creation tolerates both Zulip response shapes, using a profile
lookup and key generation only for the newly created user when required.

`agag provision [root]` reads the instance, performs the existing-email guard,
creates the bot, atomically writes `.local/zulip.env`, subscribes `#agents`,
and creates/updates the own channel from `params/channel.md`. It prints the
intro and listener commands. `agag init --provision` chains it, while
`--like` carries local machine facts from a sibling without carrying role
overrides the new agent does not declare.

## Final human checklist

```text
Human checklist for <instance>:

 1. Once per realm: create the dedicated provisioner account and put its
    owner-class credentials in the path named by AGAG_ZULIP_ADMIN_ENV.
 2. Plane (only if this agent will register Work): create an account for it
    and set AGAG_PLANE_ENV or <root>/.local/plane-credentials.env.
 3. To keep the listener running permanently, install it with launchd or
    Ansible after the foreground/background trial.
```

Bot creation, channel work, harness discovery, introduction posting and the
trial listener are no longer listed as human work.

## Live autolab transcript

`#pj-runsmoke1/workplan-agping` created R-8. Autolab asked whether it could
start; the requester answered in R-9's workrun. The first execution ran the
complete new command and reported instance `agping-agstudio1`, bot user 18,
intro 1376 and a healthy listener, but put the project beside pj-agdev rather
than in runsmoke1 `main/`.

Two corrective requester posts were initially rejected because the generated
task text incorrectly said that `main/` must stay untouched. The workplan was
then updated: R-9 was cancelled, R-10 moved the entire already-provisioned
tree (including ignored state), restarted it, asked for commit approval,
committed/pushed `9c73b9d`, and reposted intro 1406. The only real questions
the requester needed to answer were start permission and commit/push
permission. The “is that correction really yours?” exchange was avoidable
friction and is recorded as the next guide/planning lesson: current requester
corrections outrank generated task prose.

The project kept its unrelated untracked `HELLO.md` and `README.md` untouched.
The current tracked project is `1e85461`; intro message 1429 carries that
revision. agping stays as fixture #2.

Final close-out was requested through autolab's own entrance. It re-read the
board, verified intro revision `1e85461` and successful greeting message
1425, resolved `workplan-agping`, and ran `agautolab.mission_done`; R-8 is
Done in Plane.

## Front exchange and failure farming

Front found agping from its introduction and posted into
`#agping-agstudio1/greeting`. The first run reached the listener but failed:

```text
E_OVERLAY_SCOPE: overlay may not introduce roles
```

The copied autolab overlay contained `[roles.coding]`, absent from agping's
front-only config. Removing that ignored block repaired the live instance;
pyagag `0b226b6` generalized the repair by filtering absent-role overrides
during `--like`. Full pyagag (411), agautolab (169), agforge (197) and
agfront (20) suites passed during the phase.

On retry agping message 1425 answered Front:

```text
Hello Front! Greeting received.
```

Front message 1427 relayed that reply to Developer at home. This proves bot
creation, channel subscriptions, intro discovery, listener execution,
agent-to-agent callback and Front's home reply as one chain.

## What remains out of scope

- cagent distributing agent env files and a generalized `agag_agent` Ansible
  role;
- per-agent Plane accounts;
- permanent launchd/Ansible installation for generated listeners;
- Front posting its own introduction.

Deus Ex Machina note: the Omni Agent implemented the reusable provisioning
surface and repaired the live overlay failure for autolab — handoff
candidate. Autolab itself performed the live agent creation, provisioning,
introduction, relocation, listener start and project commit/push.

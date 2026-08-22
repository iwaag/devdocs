# Step 4 report — agecho, generated and talked to

## Generation

`uvx --from git+https://github.com/iwaag/pyagag.git agag init agecho --yes --dest /Users/eiji/projects`
→ `/Users/eiji/projects/agecho/` (no remote). First attempt failed in the
wheel build — `force-include` of `agag/templates` duplicated the package's
own files; hatch already ships them (fixed, pyagag `2eabf40`).

Tracked files, 111 lines in total; the only code is `listener.py` (26 lines,
the `AgentSpec` and one `listener_main(SPEC, {})` call) and `intro.py` (8).

## Provisioning (current reproduction)

The p1 run predates `agag provision`; its manual bot/channel work is now one
agent-side command using the dedicated provisioner identity:

```sh
AGAG_ZULIP_ADMIN_ENV=<provisioner-env> agag provision agecho
```

The historical result was Zulip bot user 16
(`agecho-agstudio1-bot@agstudio.local`), credentials in the ignored
`agecho/.local/zulip.env`, membership in `#agents`, and an own
`#agecho-agstudio1` channel shared with the administrator. Plane was skipped
because agecho registers nothing.

After provisioning, `params/intro.md`'s first TODO was replaced by three
honest sentences (the request TODO remained), then:

1. `uv sync`, `uv run python -m agecho.intro` → `#agents/intro-agecho-agstudio1`
   (message 1326, revision `76523d0`).
2. `service/listen.sh` → listener up: `pull sweep: all topics in
   'agecho-agstudio1', prefixes ('agechorun-', 'agechoplan-') elsewhere,
   routes [] + DM thread`.

**Failure found:** `claude` is not on PATH here, so `roles.front` would have
been `E_UNAVAILABLE`. Fixed locally with `.local/agents.local.toml`
(`[local.harness.claude_code] command_glob = …`, same as forge's) and added
to the printed checklist as step 4 (pyagag `084c42a`).

## The conversation

Posted as the Developer in `#agents` / `front-greet-agecho`:

> Please say hello to agecho-agstudio1 and tell me what it replies. Go ahead without asking me first.

```
06:15:54 front   serving 'agents'/'front-greet-agecho'
06:16:07 agecho  serving 'agecho-agstudio1'/'hello-from-front'  → entrance topic
06:16:16 front   serving mention in 'agecho-agstudio1'/'hello-from-front'
                 → serves agents/front-greet-agecho
06:16:26 front   marked 'agecho-agstudio1'/'hello-from-front' served up to 1333
```

`#agecho-agstudio1` / `hello-from-front`:

```
[Front 1330]   Hello agecho! This is Front, saying hi as requested by the developer.
[agecho 1331]  Message received. Please wait for the reply.
[agecho 1333]  @**Front**  Hi Front — hello back from agecho-agstudio1! Message received, all good here.
```

`#agents` / `front-greet-agecho`:

```
[Front 1332]  @**Developer** I've sent a greeting to agecho in channel `agecho-agstudio1`,
              topic `hello-from-front`. … will report back here once it responds.
[Front 1335]  @**Developer** agecho replied. … It replied:
              > "Hi Front — hello back from agecho-agstudio1! Message received, all good here."
              Task complete.
```

Front located agecho from `tools/agents.md` (the intro harvest) alone and
picked a plain topic, as the intro said. agecho's entrance ran `roles.front`
with the **built-in default guide** (no `entrance_front/guide.md` exists in
agecho) and the default grant; the `[selfnote][rootchat]` Front's
`agentchat send` wrote brought Front back by mention, and Front answered at
home. Success criterion 3 met with no agent-specific code.

## Left running

The agecho listener is a foreground `nohup service/listen.sh` (no launchd
plist), log at `agecho/.local/out/zulip-listener.log`. It dies with the
session unless a plist is added.

# Report 6 — Ansible

Status: **done**, with one item that could not be completed for a reason
unrelated to this episode (agstudio's playbook run — see below). `opencode` is
gone from `ansible_agdev` and from every node.

## Deleted

- `roles/opencode_agent/` — all 11 files.
- `playbooks/agent/setup_opencode.yml`.
- `vars/opencode_agent.yml.example` and the `vars/opencode_agent.yml` line in
  `.gitignore`.
- The `opencode_agent` role (with `opencode_agent_serve: false`) from
  `playbooks/agent/setup_autolab_node.yml`. It was there purely to install the
  binary; the playbook now runs `claude_code_agent` then `autolab_node`.
- `autolab_node_opencode_command` from `roles/autolab_node/defaults/main.yml`
  and the `[local.harness.opencode]` block from
  `templates/agents.local.toml.j2`.

**No `[local.harness.agcode]` block was added**, as specified: agcode's
default command is `sys.executable`, which is already the interpreter that
imports `agag` in every `uv run` consumer. The deployed overlay on agautolab1
confirms it — `claude_code`, the provider, and the role profiles, nothing
else.

## The `/v1` assertion, inverted

`roles/autolab_node/tasks/main.yml` carried

```yaml
- name: Require the intent-supplied OpenAI-compatible Ollama endpoint
  assert:
    that: autolab_node_provider_ollama_base_url is regex('/v1/?$')
```

The first deploy failed on it. That assertion existed because OpenCode spoke
the OpenAI-compatible API; agcode posts to `{base_url}/v1/messages` and would
have doubled the path. It is now inverted — the suffix must be **absent**, and
the message says why. This is the last of the four `/v1` sites reports 2–4
predicted; the others were the three consumer overlays.

The endpoint value itself comes from placement intent, so the desired state
needed the same change **before** the deploy could be correct. Applied as a
partial batch against the two `autolab_node` placements:

```
committed: {'create': 0, 'update': 2, 'delete': 0, 'unchanged': 0, 'conflict': 0}
→ autolab_node_provider_ollama_base_url: http://agstudio.home.arpa:11434
```

That is desired-state work landing in Step 6 rather than Step 7. It belongs
here: the redeploy this step performs is wrong without it.

## A stale operator-input file — do not apply it wholesale

`.local/desired-state.yaml` (the ignored operator input) is **behind Nautobot**
in two places, and one of them is dangerous:

| field | `.local/desired-state.yaml` | live Nautobot |
|---|---|---|
| `agautolab-agautolab1` `repo_url` | `http://agstudio.local:3000/autodev/agautolab.git` | `https://github.com/iwaag/agautolab.git` |
| `agautolab-agautolab1` `front_profile` / `mediator_profile` | `local` | `sonnet` |

That Gitea repository **was deleted on 2026-08-14**. Applying the file as-is
would repoint agautolab1's deployment source at a repository that no longer
exists — the exact regression `pj-agdev/.local/devenv.md` warns about. So the
base-URL change above was applied as a hand-written partial batch built from
`nctl desired export` (the live state), not from that file, and the file was
not applied.

**Step 7 must refresh `.local/desired-state.yaml` from `nctl desired export`
before editing it**, not edit it in place.

## Redeploy

`nctl render production --out inventories/generated`, then:

- **agautolab1** — deployed, `ok=21 changed=2 failed=0`. Verified after:
  - `git -C /home/eiji/agautolab rev-parse --short HEAD` → `429770c`, the
    Step 2 commit.
  - the generated `.local/agents.local.toml` carries no opencode block and
    `base_url = "http://agstudio.home.arpa:11434"`.
  - a live agcode run from the node's own deployed venv:

    ```
    .venv/bin/python -m agag.agcode --working-dir /tmp \
      --model qwen3.6:35b-a3b-coding-nvfp4 --base-url http://agstudio.home.arpa:11434
    → {"output": "marker-agautolab1-agcode", "status": "ok", "num_turns": 2,
       "duration_ms": 2301, "outcome": "done"}
    ```

    A `front` run through agautolab1's gateway would not have tested agcode:
    that node's intent puts `front` and `mediator` on `sonnet`, and only
    `director`/`coding`/`summarizer` on `local`. The direct run above is the
    honest check of the same thing.

- **agstudio — not deployed.** The playbook fails in `claude_code_agent`,
  before `autolab_node` runs, on a prerequisite that has nothing to do with
  this episode:

  > Claude Code needs the user-scoped npm executable at
  > `/Users/eiji/.local/node/bin/npm`; install the node baseline first.

  That path does not exist on this Mac (node is Homebrew-installed), so this
  placement has never been deployable by the playbook. Nothing changed on
  agstudio — `ok=2 changed=0`, and the overlay is byte-identical to its
  backup. It is not a blocker: that placement's `repo_dest` is
  `pj-agdev/agautolab`, the developer's own working tree, which Step 2
  migrated directly and verified live through the `:8791` gateway. Installing
  a node baseline on the developer's Mac is outside this episode's scope and
  is left as a decision for the developer.

## Uninstalling the binary

Deleting the role uninstalls nothing already installed, so a one-shot playbook
removed `~/.local/bin/opencode`, `~/.config/opencode` and
`~/.local/share/opencode`, then asserted `which opencode` fails:

```
agautolab1  changed=1   → which opencode -> empty
aghub       changed=1   → which opencode -> empty
agpc        changed=1   → which opencode -> empty   (via hosts_intent.yml)
agstudio    changed=0   → which opencode -> empty   (removed by hand first)
agdnsmasq   changed=0   → which opencode -> empty
agbach      changed=0   → which opencode -> empty
```

agpc is not in the production inventory (it carries no active placement), so
it was reached through `inventories/generated/hosts_intent.yml`. agdnsmasq and
agbach never had it; they are in the sweep because "all" is the honest target
for a removal.

**The `:4096` daemon.** `com.clusterintent.opencode.agent` was still running
on agstudio — the last live opencode process in the system. Booted out of the
GUI domain and its plist deleted. Together with the two cagent labels removed
in Step 5:

```
lsof -iTCP:4096 -iTCP:4097 -iTCP:4098 -sTCP:LISTEN  → empty
launchctl list | grep -i opencode                   → empty
```

## Not done here, by plan

`vars/deployment_profiles.yml` still declares the `node_agent` profile whose
action is the now-deleted `playbooks/agent/setup_opencode.yml`. That is Step
7's item. `nctl render production` was re-run after the deletion and succeeds
— the profile's playbook path is not validated at render time — so the
dangling reference is inert for one step. It goes with the `node-agent`
service and its three placements.

## Deus Ex Machina note

Did the node-side opencode uninstall and the launchd cleanup for agents
`autolab agent` and `cagent` — handoff candidate.

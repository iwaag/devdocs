# P1 — Retire opencode, run local models on agcode

Final report. Per-step detail is in `report1.md` … `report9.md`.

**Status: done.** All nine steps executed. Every "Done when" criterion met
except one residue in the actual-state store, described below.

## Done when

### 1. `grep -ril opencode` returns only history

Met. Outside `devdocs/**` the only hits are deliberate records of the removal:
the contract's own vocabulary-change note (`devpolicy/contracts/agent/spec.md`),
pyagag's README and contract doc, one pyagag test that asserts the name is now
refused, and the launchd README's upgrade instruction. Full list in report 9.

### 2. Every test suite green

| suite | result |
|---|---|
| pyagag | 194 passed (was 176) |
| agautolab | 78 passed |
| agdevworld assistant | 129 passed (was 125) |
| agforge | 124 passed (was 129) |
| cagent | 159 passed (was 119) |
| nctl | 1315 passed |
| nodeutils | 91 passed |
| mTLS conformance gate | 23 passed |
| ansible syntax check | ok |

### 3. No opencode process anywhere

Met. Ports 4096, 4097 and 4098 are free; `launchctl list | grep -i opencode`
is empty; the three labels (`com.clusterintent.opencode.agent`,
`com.clusterintent.cagent-opencode`, `com.clusterintent.cagent-window-opencode`)
are booted out and their plists deleted. `which opencode` is empty on
agautolab1, aghub, agpc, agstudio, agdnsmasq and agbach — verified by a
one-shot playbook that asserts the failure of `which`.

### 4. nctl clean

Met. `nctl status` ok; `nctl drift` reports `converged=28 unknown=5` with zero
`node-agent`/opencode targets in the JSON envelope; the production inventory
is re-rendered (27 → 23 placements) and carries no
`nintent_opencode_ollama_url`.

**One residue.** Two devices still hold a stale
`observed_services["node-agent"]` row in Nautobot's actual state, because
observation ingest merges service maps rather than replacing them. Inert (no
desired counterpart, so no drift), reported in report 9, not fixed — it needs
an ingest change outside this plan.

### 5. A live wire transcript per migrated entrance

| entrance | check | evidence |
|---|---|---|
| agautolab `front` | read a marker file through the `:8791` gateway | reply `marker-agcode-step2`, `harness: agcode`, agcode wire transcript captured (report 2) |
| agautolab1 | agcode from the node's deployed venv | reply `marker-agautolab1-agcode`, 2 turns (report 6) |
| agdevworld `front` | switch the view **and** fetch `/api/autolab/nodes` | both tools fired; node names from the live passthrough; `agcode-transcript-v1` capture in `/records` (report 3) |
| agforge `generator` | `ls scripts` on the `local` profile | correct listing, 2 turns (report 4) |
| cagent window | `nctl drift` | "29 converged … 6 unknown", matching `nctl drift` at the wire (report 5) |
| cagent window | refusal | enumerated its own tools instead of reporting a denial (report 5) |
| cagent human | `nctl status` | "Nautobot (v3.1.3) and the Celery worker … are healthy" (report 5) |
| cagent human | session continuity | turn 2 recalled turn 1's answer (report 5) |
| cagent Zulip DM | drift summary | "Current drift: 29 converged, 6 unknown" (report 5) |

Every one is checkable against something other than the agent's self-report —
a marker file, a browser action, an `nctl` output cross-checked at the shell.

## What the episode actually changed

**The permission model.** Three deny-lists are gone: agautolab's four
`opencode-*.json`, agdevworld's built-in disable block, agforge's 60-line bash
allow-list, and cagent's two permission templates. What replaces them is the
tool set. `READONLY_TOOLS` for a summarizer; four native tools and no
filesystem for agdevworld's front; `read`/`list`/read-only-`nctl`/two incident
tools and **no shell at all** for cagent's window.

The predicted benefit showed up in the wire transcripts. Asked to reconcile
and delete a file, cagent's window did not report a denial — it enumerated
what it has. There is no forbidden option to argue with.

**One guard kept.** `guarded_run` refuses 14 destroy-class patterns on
cagent's authenticated doors. The episode says keep irreversible-harm guards
and drop wrongness-prevention ones; guest destruction and storage erasure are
the former. Everything else from those files is gone.

**cagent became one process.** No agent server to start first, no restart
after editing `AGENTS.md`, no session API to poll, no `finish == "tool-calls"`
step logic. cagent holds the conversation itself (capped at 8 turns / 20k
characters, and it says when it dropped something) and reads its instructions
per request.

**Cost stopped being invented.** `cost_usd` is `null` on every agcode run.
Not measured is a different claim from free, and the guides now say so.

## What I would tell the next person

1. **The `/v1` suffix is the migration's real hazard.** agcode posts to
   `{base_url}/v1/messages`; OpenCode wanted the OpenAI-compatible `/v1`.
   Four sites carried it — three consumer overlays, one Ansible assertion that
   *required* it — and each failed separately. It is now written into
   `spec.md`.

2. **`extract_event_text` was not callerless.** The plan's "check the tests
   before deleting" was right: `run_harness`'s `fake` branch used it too, and
   agforge re-exported it. And its `"\n".join` silently dropped a trailing
   newline, so a bare passthrough was not equivalent.

3. **Every consumer resolves pyagag from GitHub, not by path.** Four steps
   needed `uv lock --upgrade-package pyagag` after a push. The plan's
   "editable path dependency of every consumer" — and pyagag's own README —
   were wrong; the README now says what actually happens.

4. **`.local/desired-state.yaml` was behind Nautobot**, including a
   `repo_url` pointing at a Gitea repository deleted on 2026-08-14. Refresh it
   with `nctl desired export` before editing it; never apply it on faith.

5. **Two files outside the plan's inventory mattered**: the mTLS conformance
   gate (broken by Step 5's deletions) and `nodeutils` (which ran
   `opencode --version` on every node). Both are fixed. A coupling table is a
   starting point, not the boundary.

6. **`op: delete` in a desired batch needs an explicit `values: {}}`.**
   Omitting it is a bare `HTTP 400`; only `--json` shows why.
   `docs/desired-partial-batch.md` documents only `op: upsert`.

## Left undone, deliberately

- **nintent's `PROFILE_BINDING_NAMES` / `REFUSED_PROFILE_CONFIG_KEYS`** still
  name `node_agent`. Dead config, harmless (nintent only rejects *undeclared*
  names), and changing it costs a push-build-restart cycle plus the binding
  tests. The plan recommended leaving it; report 9 records it.
- **The agstudio `autolab_node` placement was not deployed by the playbook.**
  It fails in `claude_code_agent` on a missing user-scoped Node.js baseline
  (`~/.local/node/bin/npm`) that has nothing to do with this episode.
  Nothing changed on that host. Its `repo_dest` is the developer's own working
  tree, which Step 2 migrated directly and verified live.
- **The stale actual-state rows** described under criterion 4.

## Deus Ex Machina

The Omni Agent did work belonging to four in-system agents — the agcode
migration for `front` (agautolab), `front` (agdevworld assistant),
`generator` (agforge) and `cagent`, plus the node-side opencode uninstall,
the launchd cleanup, and the desired-state retirement. Each step's report
carries its own one-line note; this is the summary. Handoff candidates, all
of them: notably, cagent's own authenticated door could have made the
desired-state change, and its window could have read the drift that verified
it.

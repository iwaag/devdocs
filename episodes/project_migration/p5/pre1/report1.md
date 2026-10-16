# project_migration p5/pre1 — Step 1 report: the current cagent implementation and its operating responsibilities

Status: complete (2026-10-16). Investigation only; no reconciliation, refresh, deployment, credential or service change was made. Read-only probes: `nctl status --json`, `launchctl list`, `docker ps`, Zulip reads as the Cagent bot, file and log reads, and an offline `pytest` of cagent (204 passed).

Raw runtime evidence, machine names, addresses and credential locations are in the ignored local notes (`pj-agdev/.local/project-migration-p5-pre1/inventory-notes.md` and the three trace files referenced there). This report keeps portable conclusions and source references (paths relative to the workspace root).

## 1. Instructions, revisions, deployed versions

| Item | Finding |
| --- | --- |
| Repository instructions | No `CLAUDE.md` or `AGENTS.md` at the workspace root or in `pj-clusterintent`. The applicable instructions are `devpolicy/styles.md`, `devpolicy/terms.md`, `devdocs/README_DEV.md`, `localrule.md` and the ignored `pj-agdev/.local/devenv.md`. cagent's own agent instructions are `pj-clusterintent/cagent/agent/AGENTS.md`, `window/AGENTS.md`, `agent/guides/{front,operator_read}/guide.md`. |
| Source revisions | `pj-clusterintent` `ff2910da` (clean, equal to origin), submodules clean: ansible_agdev `366d7ed8`, nauto `38f758c7`, nctl `0d18eb63`, nintent `2ff73f8e`, nodeutils `d68b9630`. `pj-agdev` `1b58a371`, agdevworld `47a365d0`, agautolab `3fbafad8`, all clean. `devdocs` `9e00ccc0` plus the untracked p5 braindump. These match the plan's anchors. |
| Deployed cagent | The two launchd jobs run `uv run --project` against the **developer's live checkout**, not a release copy. The API process started three minutes after the last cagent commit (2026-09-29), so the deployed code is `ff2910da`. The cagent venv pins pyagag `1f49c28`; the `pyagag` working tree may be ahead. |
| Backend profiles in use | Committed `agents.toml`: node, human, operator, argue on `sonnet` (claude_code); front on `local` (agcode over Ollama). The machine overlay only restates front=local. |
| Earlier contracts and trials | `pj-clusterintent/devdocs/vision/cluster_agent/{roadmap.md,p1..p4/contract.md}` (frozen contracts, partly stale, see §9); `modernize_cagent/p1` (front/operator roles, topic serving, handoffs); episodes `better_communication/zulip_cagent_receive` (window, incidents, costs), `improve_cluster_sync/autolab-meets-cagent` (node door writing desired state and reconciling), `refine_cluster_management/p1,p1ex1` (service foundation, observation scheduler), `refactor` p3 (Plane removed, change records in Zulip), `argue` p1 (argue role). `refine_cluster_management/p2` has a plan and no reports: the "lightweight report path from in-system agents to cagent" was never executed. |

## 2. Entrances: identity, intake, roles, tools, persistence, recovery

cagent is two processes (`cagent-api`, `cagent-zulip`) with **two different runners**: the in-process `AgentRunner` (`cagent/src/cagent_api/agent_runner.py`) serves the three HTTP doors and the Zulip DM; the out-of-process `role_run.run_role` (`role_run.py`) serves topics and argue mentions as harness subprocesses in a per-generation workspace.

| Entrance | Identity | Role / backend | Effective tools (what the code grants) | Persistence | Recovery | Tested |
| --- | --- | --- | --- | --- | --- | --- |
| HTTP node door (mTLS) | cert URI SAN with DesiredNode uuid + active ledger entry + live Nautobot lifecycle check (`auth.py:69-84`, `node_resolver.py:61-82`) | `node` / claude_code | `Read,Write,Edit,Glob,Grep,TodoWrite,Bash` **unscoped** (`agent_runner.py:89,439`). The destroy-class guard (`guarded_run`, `:162-166`) is an agcode tool object and is **not on this path**. | per-request evidence dir (`request.json`, `events.jsonl`) | in-memory queue; restart marks every non-terminal request `interrupted`, nothing re-runs (`store.py:225-280`); cancel takes effect only at turn end on claude_code (`:391-394`) | routes/ownership/TLS with fakes; real mTLS handshake in devtests; no real model turn |
| HTTP human door (bearer) | one static token → fixed label "operator", constant owner key (`auth.py:87-107`, `store.py:57-63`) | `human` / claude_code | same as node | same | same | same |
| HTTP window (none) | anonymous, one shared owner (`auth.py:110-124`) | `front` / agcode | `read, list, nctl (6 read subcommands), record_incident, list_incidents, contexts`; no write, no shell (`agent_runner.py:290-304`, `readonly_nctl.py:23-34`) | same; one-turn sessions | same | routes, tool-name assertions, allow-list |
| Zulip DM to Cagent bot | none of cagent's own; sender rendered as text into the prompt with the last 50 DMs, then `POST /window` without credential (`zulip_window.py:79-133`) | `front` / agcode | window tools | window evidence; DM lost while listener down | lossy by design | formatting and ack/answer with fakes |
| Zulip topics (own channel, `cagent-`/`change-` prefixes) | "whoever posted in an owned topic"; no authentication beyond Zulip membership | `front` (agcode) then `operator` (claude_code) | front under agcode gets agcode's default `run` (unguarded shell, `role_run.py:46-56`); operator gets `Bash(cagent:*)` = the read-only CLI | generation dirs under `.local/topics`, run records under `.local/agent/<role>/`, Zulip posts | `agag.listen` sqlite queue, at-least-once with evidence-based duplicate suppression, claim check since failsafe p7 | handoff logic with monkeypatched runs |
| `#argue` mention | same as topics | `argue` / claude_code | `Bash(cagent:*)` | transcript | same | monkeypatched |
| Operator CLIs (`cagent`, `cagent-ca`, `cagent-ledger`, `cagent-evidence`, `window/incident.py`, `cagent_api.intro`) | the invoking OS user | — | read-only nctl; CA and ledger writes; incident file; intro post | files | — | unit tests |

Conclusions that matter for the design:

- **There is no permission engine.** Authority is "which tool names a role is handed" plus one substring deny-list on agcode's `run` and the read-only nctl allow-list. On the committed profiles the node and human doors run claude_code with an unscoped `Bash`, so `README.md:27-28`, `llms.txt:93-97` and `AGENTS.md:53-58` ("destroy-class commands are refused on every door") describe agcode only. "Cannot write outside `.local/`" is prose: agcode's `resolve_path` passes absolute paths and `tool_write` has no scope check. `agents.toml:46-52` itself records the measurement that a `Bash(prefix:*)` grant does not bound a composed command.
- **Node and human are the same agent after authentication**: same instructions, same tools; identity only governs session ownership/visibility and the evidence record.
- **Recording a change is not executing it, by construction.** `change_record.py` and `topics_serve.py` run no subprocess; their only side effects are Zulip posts. `set_state` exists with no caller; dispositions are hand-written `[selfnote][state]` notes. The only way a recorded change is applied today is a person at a shell or a separate authenticated HTTP turn asked to do it.
- **`incident.py` is a defect-report recorder**, one Markdown file per report under `.local/cagent/incidents/`, written by the window's `record_incident` tool or the CLI shim and read back only by `list_incidents`. It is not an installation-report ingester and nothing consumes the files.
- **Inherited environment.** Every in-process shell command runs with the whole `cagent-api` process environment and the superproject root as cwd; claude_code runs get that environment plus the provider key from the overlay. `cagent-api` needs the human token file (mandatory at boot), server cert/key and CA, the Nautobot connection for node auth, and the harness binary/key. `cagent-zulip` needs the Cagent bot credential and the same harness facts.
- **Three worker threads share one cwd**: a node turn, a human turn and a window turn can run concurrently; serialization is per listener, not global.

## 3. Reporting and observation (nctl, nodeutils, nintent/Nautobot, Ansible)

| Capability | Documented | Implemented | Tested | Currently observed |
| --- | --- | --- | --- | --- |
| Scan a node and ingest the result | yes (`nctl/README.md`, `nctl/docs/reconcile.md`) | `observe_node` reconciler: Ansible playbook `run_nodeutils_collect.yml` over SSH with become, clones nodeutils at the pinned SHA on the node, runs `nodeutils collect` with an nctl-rendered probe config, slurps the report, validates identity and age, caches it in the controller dumps dir, submits the Nautobot Job "Ingest Nodeutils Inventory" (`nctl/src/nctl_core/observation.py:198-412`) | nctl unit tests; no live gate in CI | the six-hourly refresh ran for all six managed hosts this morning; one fixture host's dump is ~11 weeks old |
| What is detected | partly | OS/hardware/CPU/memory/GPU/disks, interfaces with MAC and IPs, Docker containers and compose projects, systemd/launchd units, hinted processes (pgrep only for `management_mode=manual` placements), managed-file digests, workspace git state and agent liveness file, Proxmox guests via a local `pvesh` helper. **Not** packages (only python/docker presence), **not** arbitrary processes, nothing from inside VMs except via the hypervisor | nodeutils tests | — |
| Entity matching | partly | ingest: Device by serial, then name==fqdn, then name==hostname; Proxmox guests by (type, vmid); interfaces by MAC. Drift ranks DesiredNode candidates (name 50, serial 80, uuid 80) and persists `realized_device` | yes | — |
| Purpose / ownership / provenance | claimed in older docs | **Only on the desired side** (DesiredNode.role, placements, Braindump free text). nodeutils' `self_reported.owner/purpose` is never populated under nctl (the probe config replaces `self_inventory.yaml`); the ingest Job writes only `description`; the Device owner/purpose custom fields were deleted as unused (`devdocs/small/minimize_nauto`). Observation provenance = `inventory_source`, `last_seen`, raw identity | — | — |
| Installation-report ingestion ("I installed X on Y for Z") | **none** | none in nctl CLI, nauto Jobs, nintent API views or models. Nearest substitute: declare a `manual` placement with a process hint, then verify by scan | — | — |
| Defect reports from agents | yes (`zulip_cagent_receive`) | incident files (§2) | yes | 7 files, all from August 2026 |

The planned "lightweight report path from in-system agents to cagent" (`refine_cluster_management/p2/plan.md`) was decided with the Developer on 2026-08-12 (free text to the existing entrance, cagent records and verifies, no remediation) but never implemented.

## 4. Change execution end to end

| Step | Mechanism | Actual actor today |
| --- | --- | --- |
| Request record | cagent only: `change-<stem>-o<origin id>` conversation in cagent's channel (§2). The backend has **no request record**; `BrainDumpDocument` is a non-actuating wish | the topic front, when asked in Zulip |
| Desired-state edit | hand-edited ignored YAML → `nctl desired apply -f … [--yes]` → the sole writer, the nintent batch REST endpoint (`nintent/.../api/views.py:77-113`). UI views are read-only. `nctl lifecycle` and reconcile's realized-link writes use the same path | a person at a shell, or a cagent authenticated role with Bash |
| Plan | `nctl reconcile [HOST]` dry run: drift → planner → `plan.json`; SSH host-key preflight; no writes | person; cagent authenticated role |
| Apply | `--yes` rounds under a file lock: regenerate production inventory, run actions, re-observe, final host-scoped drift, max 3 rounds. Reconcilers: `observe_node`, `link_actual_node`, `link_compute_realization`, `reconcile_ipam` (Nautobot Job), `service_profile` (playbook per profile: grafana, nomad, prometheus, node_exporter, static_web_app, autolab_node, agag_agent), `dnsmasq_config`, `create/destroy_compute_instance` (Proxmox playbooks; destroy needs `--allow-destroy`). `new_node_baseline` is registered with no handler. Observe-only profiles (ollama, swarmui, comfyui, manual_toolchain, cron_task) stop with `manual_intervention_required` | person; cagent authenticated role; **and the six-hourly launchd refresh**, which runs `nctl reconcile <host> --refresh-observation --max-rounds 1 --yes` and therefore also executes already-planned safe actions for that host, not only observation |
| Operation record | `<ULID>.jsonl` events plus an artifact directory (plan, result, Ansible recaps, reports, Job summaries) under the nctl events dir; read by `nctl ops`. Nothing is written back to Nautobot about operations | nctl |
| Post-action observation | every mutating action with `requires_observation` triggers a fresh `observe_node` next round; completion means fresh drift shows the expected state, not the action's exit code | nctl |
| Failure handling | terminal states `planned, already_converged, converged, manual_intervention_required, non_converged, failed`; manual findings stop before mutation; Job failures become per-host observation errors; SSH store problems fail closed | nctl |
| Manual steps that no component performs | SSH key distribution (`ssh-copy-id`), host-key enrollment (`nctl ssh enroll`), vault file, first-boot baseline, dnsmasq self-bootstrap on a fresh node, per-deploy credential environment variables | person |
| `executor/` | standalone Ollama harness that lints a `plan.md` and runs commands; refuses approval-marked plans and any `--yes`; no caller since 2026-08-04 | nothing |
| `ansible_agdev/api` power webhook | FastAPI wrapper over power playbooks | deployed and called nowhere |

Credentials the execution path needs on the command node: Nautobot token (env or file), the Ansible SSH client key, the Ansible vault (login user, sudo password), the nctl-managed known_hosts, the Proxmox read helper and sudoers on the hypervisor, per-deploy secret files passed as environment variables, MinIO key for uploads, the Cagent bot credential for `agents observe`.

## 5. Responsibility map

| Concern | nctl | Nautobot core + nintent | nauto | nodeutils | Ansible | executor | devenv |
| --- | --- | --- | --- | --- | --- | --- | --- |
| State models | typed read models only | **desired** (14 nintent models) and the **actual ledger** (Device, Cluster, VM, VMInterface, IPAddress + custom fields) | seed of core prerequisites and Device custom fields | emits a report | `deployment_profiles.yml` as placement-config schema | — | — |
| Jobs | launches and polls by exact name | `Reconcile Desired IPAM Intent` | `Ingest Nodeutils Inventory`, `Ingest Agent Registration`, `Seed Home Cluster` | — | — | — | worker and scheduler containers |
| Inventory generation | **sole owner** (hosts-intent, production, dnsmasq) | data via GraphQL | — | — | consumer | — | — |
| SSH trust | **sole owner** (known_hosts keyed by DesiredNode uuid, enroll, preflight) | — | — | — | relies on rendered vars; fails closed | — | — |
| Credentials | references only | API tokens and users | server-side ORM | none | client key, vault, per-deploy env | local Ollama | compose env |
| Operation history | **sole owner** (ULID events and artifacts) | JobResult rows; `WorkflowEpisode` retrospectives | — | — | recaps captured by nctl | transcript per plan | — |
| Observation | orchestrates collect, slurp, validate, cache | ledger of ingested facts | ingest policy and upsert | **collector** | collection playbook, pvesh helper role | — | the six-hourly launchd job |
| Actuation | plans and dispatches; direct dnsmasq apply | IPAM Job | — | — | **all host mutation** | reads only in practice | — |
| Deployment ownership | runs from source | Nautobot image with nintent installed from GitHub; compose files | Git Repository in Nautobot | cloned on nodes at the pinned SHA by the collection playbook | roles | — | compose and launchd templates |

Duplicated responsibilities: the profile mapping lives in three places (Ansible `deployment_profiles.yml`, nintent placement field, nctl validation); the compute contract is owned by nintent and replayed in nctl from a generated fixture; the dump shape is read by nodeutils, nauto and nctl and must agree; two inventories (bootstrap vs production) with two SSH route identities, reconciled through HostKeyAlias after an incident; the unused power webhook parallels nctl; cagent's request evidence and change records are a third record system beside nctl operations and Nautobot.

Dependencies that survive removal: remove **Nautobot/nintent** and every nctl read, plan and render fails, Ansible loses generated host vars and profile variables; remove **nctl** and desired state can still be edited through the batch endpoint and Ansible run by hand, but there is no inventory generation, SSH preflight, drift, observation orchestration or evidence, and cagent's read-only tool surface disappears; remove **nodeutils/nauto** and Devices never update, drift turns unknown; remove **Ansible** and neither actuation nor observation works (collection is a playbook); remove **executor** and nothing changes.

## 6. Callers, schedules, placements, open requests, retained evidence

| Item | Finding (freshness: all reads 2026-10-16 morning) |
| --- | --- |
| Services on the command node | `cagent-api` and `cagent-zulip` launchd jobs (KeepAlive, developer's checkout); `observation-refresh` launchd job every 6 h; Nautobot web, worker, scheduler, MinIO, Redis and Postgres containers up for two weeks; the WFE service, executor, intake, Observer and relay jobs of pj-agdev. `nctl status` reports Nautobot reachable and authenticated, one Celery worker, no pending jobs, all submodules clean. |
| HTTP door usage | 106 request records: human 58, window 29, node 19. First 2026-08-02, **last 2026-08-28**. No HTTP request for seven weeks. |
| Node-door callers | The `autolab_gateway` Ansible role installs the human token and a cagent client on a node, but no playbook applies that role any more, and agautolab's source no longer references cagent. The `cagent_client` role (node curl wrapper) has a playbook but no evidence of recent use. The ledger holds nine node certificate entries. |
| Human-door callers | agdevworld `npm run cluster:fetch` (developer command, not a service); its last snapshot is dated 2026-08-10. The browser chat UI. |
| Zulip usage | DM-to-window requests last 2026-08-23. Topic servings last 2026-09-10 (refactor p3 demo). Argue mentions 2026-09-16 and 2026-09-20. The listener survives Zulip's weekly restart by re-registering its queue. |
| Open change requests | cagent's channel holds four topics, all resolved (two demo change records from refactor p3). **No open request.** |
| Incidents | 7 defect reports, 2026-08-12 to 2026-08-23 (wrong reachability claims, a failed drift run). |
| Observation refresh history | The refresh script treats `max_rounds_reached` as success, so every run logs "refreshed (nctl exit 1)". Failure history (no timestamps in the log): 214 failed host refreshes, 158 of them for one laptop-class host that is often asleep, 35 for one workstation. All six hosts refreshed successfully this morning. |
| Retained evidence | cagent request evidence (106 dirs plus a p1 archive), nctl operation artifacts, 1,593 generated production report files, many trial captures under `pj-clusterintent/.local/` (desired exports, drift snapshots, retire batches), a Nautobot pre-deploy dump from July. |
| Nautobot scheduler | the beat container exists; no scheduled Job is defined in nauto or nintent. |

Caveat: these are inventory reads at one moment. "Up for two weeks" and "refreshed this morning" are the latest observations, not a service verification.

## 7. Evidence of existing failures

Actual, recorded failures relevant to the design (not hypotheticals):

1. **Cancelled turns kept running and one created an LXC guest** (`cagent/src/cagent_api/static/llms.txt:50-53`). On claude_code a cancel still lands only at turn end. The request layer is the only thing between an agent and `nctl reconcile --yes`.
2. **The documented destroy guard is absent on the deployed profiles** (§2). Not a recorded incident, but a verified gap between the contract text and code.
3. **Wrong explanations of the cluster**: five incident files from one day record cagent saying a node was reachable or converged when it was not, and a drift table showing a stale address. This is the "explanation found invalid" case `README_DEV.md` wants reported, and it was reported, into files nobody reads.
4. **Observation goes stale and the controller cannot tell why from the outside**: 158 failed refreshes for a host that sleeps; the stderr log carries no timestamps; episodes recorded dumps 22 h old for every node; a service observation older than 24 h turns targets `unknown`.
5. **The scheduled refresh is a bounded reconcile with `--yes`**, documented as "observation refresh". It will execute any already-planned safe action for the host. No incident is recorded from this, but it is an executor nobody names as one.
6. **SSH/dnsmasq incident and "no error is not proof"** (`pj-clusterintent/devdocs/lessons/reconciliation_lessons.md`): two inventories dialed the same host two ways, strict host-key checking rejected one, and a fix was declared complete without the failing path ever running.
7. **Celery worker silent stall** (`worker_queue_stalled`): the worker answers pings while its consumer is dead; the recorded fix is a container restart.
8. **Mediator allowlist repair mid-episode** (`autolab-meets-cagent` report 6): the only successful node-door end-to-end trial needed an Omni repair of the caller's allowlist and a restart that interrupted a request.
9. **Queued HTTP requests are lost on restart** (marked `interrupted`, never re-run), and a wedged in-flight turn stays `running` until a restart. Acknowledged in code comments.
10. **One bearer token, two consumers**: when the autolab gateway used the human door, evidence could not say whether the person or the bot asked.
11. **Front routing left open twice**: forge (p3) and archsage (p4) migrations both left Front's guides naming the old agents. A cagent migration will meet the same gap.
12. **The planned agent-to-cagent report path was never built** (§3), and the `refine_cluster_management` p1 report explicitly left cagent out of executing that phase.

## 8. Unknowns

- Whether anything still calls the node door: the ledger has nine certificates and the last node request is from August; node-side wrappers may still exist on managed hosts (not checked over SSH, by design of this step).
- Whether `cluster:fetch` is still wanted by agdevworld's node view, or whether that view can read Nautobot or a WFE-published snapshot instead.
- Real cost and duration of a claude_code turn on the node/human doors (the measured costs in `zulip_cagent_receive` were on a different backend and on the window).
- Whether the laptop-class host's refresh failures are purely sleep-related; the log has no timestamps and the script discards the failed JSON after the next success.
- What `nctl reconcile --refresh-observation --yes` has actually executed beyond observation over the past weeks (the refresh JSON files are overwritten per run; the nctl events directory holds the answer but was not audited).
- Whether the Developer still wants the window's "smaller tool set, no identity" model or an identity-carrying entrance when the entrance becomes a WFE request topic (requesters are configured Zulip user ids there).

## 9. Documentation describing retired or never-built behavior

| Claim | Where | Status |
| --- | --- | --- |
| Dedicated OpenCode instance behind the API | cluster_agent roadmap, p1 contract | gone; in-process agcode or claude_code subprocess |
| Plane issue per change request | `change_record.py` history, older README text | gone since refactor p3 |
| `nctl serve`, nctl dashboards | cagent README | removed |
| "No identity can trigger a cluster mutation" | p1 contract, still printed as frozen | superseded on 2026-08-09 by the roadmap's relaxation ("cannot destroy", not "cannot write"); contract text not updated |
| Destroy-class refusal on every door; no writes outside `.local/` | README, `llms.txt`, `AGENTS.md` | true for agcode profiles only (§2) |
| Turn timeout default 300 s | README | code default 900 s |
| The DM template tells the agent to run `incident.py` | `zulip_window.py:49-51` | the window has no shell; the `record_incident` tool is what works |
| `cagent` command | `llms.txt` (node curl wrapper) vs `cli.py` (operator read-only CLI) | two different programs with one name |
| `/jobs`, SSE progress | none | never existed in cagent |
| Request dispositions recorded by cagent | intro text | `set_state` has no caller; people write the note by hand |
| Node agent on each managed node (`nctl agent attach`) | `vision/node_agent` roadmap | an earlier line of work; nothing in cagent depends on it |

## 10. What this step establishes for Step 2

- cagent today is **an explanation and recording agent with an unbounded shell on two doors nobody uses**. Its only exercised value since September is answering argues and demo change records; its only autonomous cluster writes happened in August trials.
- The backend's real executor is **a person at a shell plus a six-hourly bounded reconcile**. nctl owns inventory, SSH trust and operation history outright; Nautobot owns both state ledgers; Ansible performs every host change; nodeutils is the only observation source.
- **Reporting with purpose has no ingestion path and no storage on the actual side**; the desired side carries intent per placement. Any design that wants installation reports must add both a record and a reader.
- The authority story in the current code is prose. Step 2 has to say where authority would actually be exercised in a WFE run, given that the executor there runs claude_code with permissions skipped and bindings self-checked (destination survey, local notes).

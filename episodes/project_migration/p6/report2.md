# project_migration p6 — Step 2 report: NFS deployment tools for cagent

Status: tools implemented, tested, pushed and in place in the live deployment
sources (2026-10-16). The pj-cagent push was first refused by the session's
safety classifier because it used a stored token; it went through after the
Developer approved it manually. No production NFS installation was
performed. Test logs are in the ignored
`pj-agdev/.local/project-migration-p6/step2/`.

## Platform scope

Step 1 found three plausible server candidates, all Debian-family Linux (the
GPU workstation, the hypervisor host, or a guest on it), and one macOS
candidate, the execution host. Clients are Linux (the autolab VM, the GPU
workstation) and macOS (the execution host and laptop).

The tools cover a **Debian-family Linux server** and **Linux and macOS
clients**. A macOS NFS server is not implemented. That host would carry the
largest blast radius (WFE, Nautobot, Zulip, Gitea), it is itself a mandatory
client, and it would need new macOS export observation. If cagent's
placement investigation selects it anyway, the missing server role is a tool
gap to report, not a reason to choose differently. This is the platform
judgement made here; the placement decision remains cagent's in Step 3.

## Delivered

### Desired-state route (installation goes through `cagentctl reconcile`)

`ansible_agdev/vars/deployment_profiles.yml` gains two profiles. Both use
placement lists, so one host can serve or mount several shares.

| Profile | Placement config | Action (by OS) | Checks |
| --- | --- | --- | --- |
| `nfs_server` | `share`, `path`, `clients`, `anon_uid`, `anon_gid`, optional `mode`, `options` | `playbooks/services/setup_nfs_server.yml` (linux) | `nfs_export` on `path` |
| `nfs_client` | `share`, `server`, `export`, `path`, optional `options` | `setup_nfs_client.yml` (linux), `setup_nfs_client_macos.yml` (macos) | `nfs_mount` on `path`; dependency `nfs_server` (orders actions only when both run on one host) |

**Roles**
- **`nfs_server`:** installs the kernel server and creates the backing
  directory owned by the share owner. It writes one
  `exports.d/nintent-<instance>.exports` per placement with `all_squash` to
  that owner, so macOS and Linux users with different uids share one owner.
  It removes only `nintent-*` files of inactive placements, so other exports
  are preserved. It re-exports on every run, so an earlier failed export is
  retried, and then verifies that the path is in the export table of a
  running server. It refuses an unprivileged container, where the kernel
  server cannot run.
- **`nfs_client`:**
  - Linux: one fstab line marked `# nintent:<instance>` with a systemd
    automount, so the share comes back after a reboot, a server outage or an
    unmount. Unmarked lines are untouched.
  - macOS: a direct map `/etc/auto_nintent` owned by the role and referenced
    from `auto_master`. Other entries are untouched.
  - Both end by listing the mount as the login user.

**Observation (nctl + nodeutils).** `nfs_export` and `nfs_mount` are new
closed state-proof check kinds. nodeutils, running as the login user,
reports separate facts:

| Check | Facts reported |
| --- | --- |
| `nfs_export` | *configured* (an exports file names the path), *running* (rpc.mountd and nfsd), *exported* (in the table rpc.mountd answers from) |
| `nfs_mount` | *configured* (fstab or an automounter map), *mounted* (an nfs mount at the path, after a bounded listing that also triggers the automount), *usable* (the listing as that user succeeded) |

Statuses: `present`, `missing`, `not_running`, `not_exported`, `unmounted`,
`unreachable` (the listing timed out) and `unusable`. Drift reads anything
short of `present` as `service_missing`. The listing runs in a child process
that is never waited on after its timeout, so a dead hard-mount server cannot
hang the collector. Observation deliberately does not claim writes,
large-file integrity or cross-host visibility. A declaration alone, or a
configured fstab line alone, is never reported as a usable share.

### Direct operation for verification

`playbooks/services/verify_nfs_mount.yml` checks a mount as the host's login
user. Its only changes are inside one `probe-<name>` folder in the mount.

| Action | What it does |
| --- | --- |
| `probe` | Write, rename and read back `<host>.done` |
| `write` | Write N MiB of random bytes |
| `hash` | Name, size and sha256 of every probe file, for comparison between hosts |
| `remount` | Unmount as root, then list as the user: the automount must bring the share back |
| `cleanup` | Remove the probe folder |

It is the one allow-listed playbook in the cagent workspace configuration
(`playbooks = ["playbooks/services/verify_nfs_mount.yml"]`). Server and
client installation are not on the allow-list: they go through the profiles
and `reconcile`.

### cagentctl (pj-cagent)

| Change | Why |
| --- | --- |
| The plan block's `sources: {pj-clusterintent: <commit>}` is **required** for `reconcile` and `playbooks`. The checkout must be exactly that commit, every submodule at its pinned commit, no tracked file changed (`sources-not-agreed` / `sources-changed` / `sources-dirty`). Verified sources are recorded in the ledger entry | Without it, different playbook content could run under the same agreed name. One superproject commit pins every submodule, so a single value binds ansible_agdev, nctl and nodeutils |
| `playbook` renders the production inventory from Nautobot, records its digest and generation, resolves hosts with `--list-hosts`, and refuses unless they are exactly the agreed limit (`target-resolution`) | Exit 0 on zero matched hosts used to count as success |
| An empty limit is refused (`empty-limit`) | An agreed `limit: []` dispatched `--limit ""` |
| Playbook `vars` are part of the agreed entry (`--var KEY=VALUE`, exact match) | The verifier is one playbook with several actions; each agreed action is a separate entry |
| Outcome comes from the PLAY RECAP: a host without a recap line, or with `failed`/`unreachable`, makes the effect `failed` | Truth from the hosts, not the exit code |
| `preview sources`, `preview playbook` | Writing a plan without guessing |

The guide's plan-agreement fact now names playbooks, their variables and the
sources commit. A new "what you can see" item points to `ansible_agdev`'s
README for what each deployment profile installs. Usage stays in `--help`.

## Verification

| Check | Result |
| --- | --- |
| nodeutils ordinary | 99 passed (6 new NFS cases) |
| nctl ordinary | 1357 passed (new `test_nfs_profiles.py`; real-repo profile validation includes the new profiles) |
| Ansible conformance | 4 passed |
| Nautobot runtime reuse gate | `cases=263`, exit 0 |
| cagentctl | 41 passed (31 before; new: sources binding, target resolution, recap outcome, vars, empty limit, preview) |
| Playbook syntax check | all four new playbooks |
| `nctl render production` against live Nautobot (stdout) | ok, no errors |
| Disposable two-container test (systemd Ubuntu 24.04, server + client) | Server role converged and was idempotent on re-run. Client role mounted through the automount. `probe`/`write` (64 MiB)/`hash`/`remount`/`cleanup` succeeded as a non-root user, with files owned by the share owner. Collector checks reported `present`, `missing`, `not_running` and `unreachable` correctly |
| `cagentctl preview sources` / `preview playbook` in the workspace | Live checkout clean at the published commit. With no NFS placements yet, the verifier resolves zero hosts and would be refused, not reported as success |

The container test found three defects, fixed before commit:

1. The stale-export cleanup computed wrong names and deleted the export it
   had just written.
2. Verification read `exportfs`'s view, so a failed re-export still passed on
   re-run.
3. The kernel's own export table fills only on demand, so the true "exported"
   state is rpc.mountd's table. nfsd's thread count is root-only; as a user,
   nfsd kernel threads are what counts.

Not exercised here:
- The macOS client role, which needs the macOS automounter on a real host.
- The production hosts. They change only under cagent's agreed plan in Step
  3.
- Container user-namespace visibility: a container hides kernel threads, so
  the user-level "running" signal was exercised only by unit tests and as
  root.

## Revisions

| Repository | Commit |
| --- | --- |
| nodeutils | `227d342` (pushed) |
| nctl | `a3a3500` (pushed) |
| ansible_agdev | `0de64e3` (pushed) |
| pj-clusterintent pins | `898dcf6` (pushed) — the `sources` value for a Step 3 plan |
| pj-cagent | `main` pushed after the Developer's manual approval |

Deployment:
- nctl, ansible_agdev and nodeutils are used live from the pj-clusterintent
  checkout, now clean at `898dcf6`. Collection fetches the pushed nodeutils.
- cagentctl is run from each run copy, and copies are cloned from the local
  workspace, which carries the new tool. The allow-list is in the workspace's
  ignored configuration, which each new run copy receives.

## Open items

- Resolved: pushing pj-cagent `main` needed the Developer's manual approval
  (the classifier refuses a token-authenticated push otherwise).
- Not a defect for Step 3: in nctl, cross-host ordering of server before
  client is not a profile dependency. cagent orders the reconciles (server
  host first) in its plan.

## Acceptance check

- **cagent can discover and execute the operations through its existing
  change path.** The profiles are reconciled through `cagentctl reconcile`
  with sources binding. The verifier runs through `cagentctl playbook` with
  agreed vars, and is discoverable through `--help`, the guide pointer and
  the ansible_agdev README.
- **No production NFS installation is hidden in tool preparation.** Only
  disposable local containers were used, and they have been removed.

## Later (2026-10-17, p6 ex1)

This report's platform judgement is kept as history. The Developer chose
the macOS execution host as the server. cagent then developed the macOS
server support itself ([ex1/report2.md](ex1/report2.md)) and installed the
share there through its change workflow ([ex1/report3.md](ex1/report3.md)).
The installed share is served by the Mac, with the Linux autolab VM as its
NFS client.

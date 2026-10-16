# p6 ex1 — enable cagent to develop and install a macOS NFS server

## Goal and evidence

Have cagent implement the missing macOS NFS support and use it to install the shared area on the Developer-selected Mac. Omni supplies the missing work environment, permissions and workflow; cagent performs the backend development and the installation. This supplements Steps 2–3 of [the parent plan](../plan.md).

Reviewed on 2026-10-16: the live `cagent/change/run-004` record at sequence 24, its original request, questions q1–q3 and Developer answers, the current workflows/guides, and [report1.md](../report1.md) / [report2.md](../report2.md). The Developer proposed the always-on Mac with ample storage, challenged the Linux-only role limitation, then explicitly paused the work. The run is held/stopped, with no result decision and its execute/verify nodes still pending. nctl status was healthy; host observations were several hours old, not fresh installation evidence.

The failure is a capability and workflow gap, not evidence that macOS cannot serve NFS. The original request did not forbid extending tools. The current `change` workflow makes the project root readonly and exposes only records and desired batches for editing; it has no development binding to pj-clusterintent. `freework` can edit the cagent project but supplies no backend repository binding and explicitly excludes host changes. Existing tools support Linux servers and macOS clients, not macOS servers. The parent's Step 2 implementation left this gap to be reported; this exercise closes it through cagent itself.

Keep detailed conversation evidence, host identities, addresses and actual paths in ignored local notes. Use the live run/question identifiers above as portable provenance. Follow `localrule.md`: English documents, steps from 1, no default backward compatibility, LAN-accessible services, prepared human verification, and commit/push for every changed repository.

## Step 1 — provide a real development path and the required authority

- Recheck the held run, current backend sources and Nautobot/nctl facts before implementation. Preserve its questions, answers and stop record. Writing this plan does not resume it or dispatch a new cagent task.
- Add a reusable cagent development workflow, for example `develop-capability`, with inspect/design → implement → test → report/confirm. Give it editable, isolated run-owned checkouts of the necessary backend repositories: Ansible, nodeutils, nctl when needed, and the superproject for their pins. Include cagent tools/guides only where the implementation requires them. Register/bind the repositories through supported WFE mechanisms; if these cannot express the backend working set, add that support rather than silently editing the live deployment checkout.
- Give cagent the practical means to run checks, commit, push development branches and publish accepted changes/pins through the existing integration route. Verify Git credentials through their existing tooling, toolchain availability and repository access without printing secrets. A prose instruction is not sufficient if bindings or runtime access still prevent the work.
- Supply the existing Ansible privilege mechanism for the eventual server/client operations. Verify effective execution identity and noninteractive privilege availability. Add missing configuration or authorization as needed; do not leave routine commands for the Developer merely because an agent tool lacks an entry point. If an actual OS interaction is unavoidable, prepare it and identify the exact remaining action.
- Update the guide using this incident as evidence: absent platform support is development work to arrange, not grounds to override the Developer's selected host. Distinguish development authority from agreement to a concrete production change. Preserve the existing effects ledger and source-bound change plan; do not introduce a new human approval for every edit or command.

Outcome: cagent has a tested workflow and usable repository/tool access for developing the missing capability. Omni has enabled the work, not implemented the NFS role on cagent's behalf.

## Step 2 — task cagent with implementing macOS support

- Start a development run with the reviewed conversation and desired outcome as context: extend the existing NFS deployment system to support the selected Mac. The request explicitly includes role/playbook development, observation, tests, documentation, commits and publication. It is not a choice among already-supported Linux hosts.
- Have cagent inspect the target macOS version and its native `nfsd`/exports documentation, then implement the macOS server role and deployment-profile dispatch. Cover backing-directory ownership, appropriate client identity mapping, managed export entries, configuration validation, activation/reload and rerunnable application while preserving unrelated exports.
- Extend nodeutils observation and nctl tests as needed so configured, running, exported and client-usable states remain distinct. Do not reuse Linux `/proc`, systemd or `exports.d` assumptions on macOS. Test the OS dispatch and retain working Linux behavior.
- Include the Mac's own executor access and a second PC in the proposed installation design. Avoid confusing local backing-directory access with a verified NFS mount. Reuse the existing client/probe tools where they fit; resolve macOS-specific mount/service-context issues from evidence.
- Run repository-required checks, publish the development result and integrate the backend commits and superproject pins. Make the resulting clean source revision available to cagent's change tools. Do not disable source binding or run against an uncommitted deployment checkout to make the trial pass.

Outcome: cagent-authored macOS deployment and observation support is tested and available for installation. Record any Omni intervention honestly. No claim of production readiness rests on unit tests alone.

## Step 3 — have cagent install and verify on the selected Mac

- Once execution of this exercise is authorized, use a successor `change` run linked to `change/run-004`, leaving the explicitly paused run intact. Give it the updated source revision and the Developer's Mac preference; do not revive the old Linux placement proposal. If instead explicitly instructed to resume the old run, use the supported resume/replan path and ensure its isolated copy receives the required tool version.
- Have cagent prepare the concrete Mac-server plan, desired-state batches, client scope, backing storage, identity mapping and verification. Retain the Developer's existing delegation of uid/gid selection. Ask for agreement only to the completed infrastructure plan required by cagent's current contract; neither the old plan nor this development plan is that agreement.
- Execute through the ordinary cagent change path. The Mac's role as a dependency host is not an additional prohibition. Keep ledger entries, exact source revision, target scope and outcome evidence; resolve unknown effects before further action.
- Verify real read/write/rename, a representative large-file transfer with matching hashes across two PCs, remount/automatic mount, and access under the executor's service identity. Check rerun idempotence and fresh service/export/mount observations. Keep the shared area and remove only disposable probes.

Outcome: cagent has actually installed a usable Mac-hosted share and recorded the result in its change record and Nautobot state. The Developer receives a prepared LAN-accessible result view and only the human decisions still needed.

## Step 4 — report and return to p6

- Write `report.md`: reviewed conversation, actual authority/workflow changes, cagent development and installation run IDs, source/deployed revisions, tests, operational evidence, interventions and remaining limitations.
- Update the parent phase's status and operating notes to reflect macOS server support and the real installed state. Preserve the earlier Linux-only report as history rather than rewriting its claim.
- Review published files for local infrastructure details and unrelated edits. Commit/push all intended changes and parent pins; verify remote correspondence. Return to p6's shared-reference and playprey migration steps with the working share as the prerequisite.

## Completion

Cagent can develop missing backend support instead of rejecting the selected platform; it authors and tests the macOS implementation, installs it through the agreed change workflow, and proves two-host use. Necessary repository and operational permissions are supplied, the paused conversation remains accurately recorded, and the Developer is not left agent-performable setup work.

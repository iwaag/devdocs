# Workflow editor fix2 — Zulip intake enabled by default

## Purpose and evidence

Make workflow creation and loading agree: workflows accept requests through
Zulip unless they explicitly opt out. People and project agents should not
need to repair a workflow's intake declaration before entrusting work to it.

In `front/converse/run-045`, Front tried to delegate the newly added Ponderer
character's publication to `aghangout/change`, then `aghangout/freework`.
Both definitions omitted `intake`, so delegation was unavailable. Front
created a plain request (`aghangout/freework/run-001`), removed its delegate
and follow-up nodes, and told the Developer to check the result themselves.
The request reached aghangout, but automatic result return was lost.

The current creation model defaults to `intake: {zulip: requests}`. Loading
YAML, however, replaces that default with an empty string when the field is
absent. The intake/delegation documentation explicitly preserves this older
behavior. Fix the common semantics rather than adding intake fields only to
aghangout.

## Intended behavior

- Omitted `intake`, `intake: {}`, or an omitted `intake.zulip` means
  `requests`, matching newly created workflows.
- Explicit `intake: {zulip: ""}` disables intake. Preserve that distinction
  through editing, saving, export and reload.
- Invalid field types remain validation errors; they must not silently
  become either an opt-out or the default.
- Default-enabled workflows are eligible for both ordinary Zulip requests
  and delegation through the existing transport. Host enablement, requester
  identity checks and channel provisioning still apply.
- A project's intake declaration describes a technical capability. A
  missing declaration must not be presented as a policy refusing work from
  other projects.

## Changes

1. **Unify parsing and serialization.** Inspect the shared workflow model,
   YAML loader/writer, editor, canonical definition/digest and template/basic
   creation paths. Apply default-enabled semantics consistently. Check that
   explicit opt-out survives every relevant round trip and differs from
   enabled intake in the canonical definition. Backward compatibility is
   unnecessary; do not add a legacy mode to retain the conflicting default.
2. **Check runtime and existing definitions.** Inspect intake discovery,
   notifier channel provisioning, delegation target validation and workflow
   summaries for assumptions about omitted fields. Inventory registered
   workflows affected by the default change; use explicit opt-outs only
   where their actual purpose requires them. Do not bulk rewrite historical
   run files. Check how loading frozen definitions and changed definition
   digests affects existing runs, and record any required recovery.
3. **Update contracts.** Correct intake/delegation documentation, CLI help
   and relevant common guidance. Keep each shared fact in one place. Inspect
   Front's handling of this failure: it must distinguish inability to use
   tracked delegation from inability to submit any request, and retain
   responsibility for the requested outcome. Add only guidance justified by
   this incident, without prescribing a fixed command sequence.
4. **Verify and deploy.** Run focused model/serialization and intake/delegation
   checks plus relevant existing component checks. Inspect Nautobot/nctl
   state before local service work, following `pj-clusterintent/nctl/README.md`.
   Deploy affected services/workers and let normal provisioning expose the
   newly eligible workflows. Verify actual dispatch and result return, not
   merely a changed capability flag.

## Verification and completion

Cover omitted intake, an empty mapping, omitted `zulip`, explicit requests,
explicit empty-string opt-out and invalid types. Check edit/save/reload and
canonical definition behavior for both enabled and disabled intake.

Use a small, bounded parent/child trial against a workflow with no intake
declaration. Confirm that the request reaches the child, its result returns
to the parent and the parent follows up with the requester. Also verify
that an explicitly disabled workflow refuses delegation. Record trial
authorship as Omni on behalf of the Developer.

Inspect the existing Ponderer publication request before any live recovery.
Do not create a duplicate publication/deployment or claim that a plain
request has acquired delegation tracking. Distinguish verification of this
fix from completion of that already submitted request.

Commit/push as appropriate and write a short `report.md` with changed
components, releases, test/trial evidence, affected existing runs and any
remaining limits. Keep addresses, backups and machine-specific operational
evidence in ignored notes. Follow `localrule.md`: English files, Japanese
human communication, no approval gates during authorized implementation,
and LAN-accessible deployed review links where applicable.

This is the requested repair plan; implementation begins when requested.

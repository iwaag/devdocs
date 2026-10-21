# Workflow editor fix1 — finding user-provided files on agshare

## Purpose and evidence

People should describe the files they want an agent to use without knowing
shared URI syntax, host-specific mount points or tool commands.

In `characters/freework/run-001`, the Developer asked whether the NFS folder
`/agshare/agag_folks/ponderer` was visible. The agent checked that literal
local path and reported that neither the folder nor an NFS mount existed.
The configured share actually contained `agag_folks/ponderer/face.png` and
`lore.md`; both `wfe share resolve` and `wfe share ls` could read it. This is
an evidence-backed discovery failure, not evidence of a broken NFS service.

The shared run guide currently describes `shared://` as the person's input
format. Revise that assumption: translating a person's description into an
accessible shared location is the agent/system's responsibility.

## Intended behavior

- For user-provided materials and handoff folders, search agshare first.
  Accept ordinary descriptions and paths such as `agshare/agag_folks/ponderer`,
  `/agshare/agag_folks/ponderer` and `agag_folks/ponderer`; the person need not
  specify a command or `shared://` URI.
- Resolve locations through this execution host's configured shares. An NFS
  server may use its backing directory without mounting its own export.
- When the person supplies only a folder/file name, offer agents a bounded
  way to find candidates inside the share. An exact path match should not
  require a recursive scan of the whole share.
- Clear project-source references such as `src/main.ts` remain references
  to the working project. Let the agent interpret intent using context.
- Distinguish an absent item, an unavailable share and multiple plausible
  matches. Ask the person only when meaningful ambiguity remains after
  discovery; do not ask them to supply tool syntax.

## Changes

1. **Inspect the common boundary.** Check the WFE share CLI/resolver and the
   common wfrun guide, including what local and remote workers actually
   receive. Inspect Nautobot/nctl state before local service work, following
   `pj-clusterintent/nctl/README.md`.
2. **Improve tools where necessary.** Support convenient share-path aliases
   and bounded name discovery through the existing share tools. Keep
   host-specific path resolution in the share layer, return useful candidate
   references and distinguish partial search from confirmed absence. Keep
   alias handling within configured share roots. Document command details
   in `--help`; avoid a separate characters-only implementation.
3. **Correct the common guide.** Describe agshare as the default exchange
   location for user-provided materials, ordinary user paths as descriptions
   rather than guaranteed local absolute paths, and the tools available to
   resolve/search them. Replace the assumption that people speak in shared
   URIs. Keep shared facts in one place, with this failure reference; avoid
   adding a mandatory shell-command sequence or duplicating guidance across
   projects.
4. **Verify and deploy.** Run focused resolver/search checks and the relevant
   existing checks for changed components. Deploy the common guide/tool
   changes to the execution environments that use them. Commit/push as
   appropriate and record releases and recovery instructions.

## Verification and completion

Start a fresh characters freework run with the original request:

> ちょっと確認。nfsの/agshare/agag_folks/pondererというフォルダが見えますか？

Give no command, shared URI or host-path hint. It should independently find
the folder and identify `face.png` and `lore.md`, without registering a
character or changing the materials. Submit this trial as Omni on behalf of
the Developer, preserving that authorship in the record.

Also check a shortened share-relative path, an ambiguous name, a genuinely
missing item, an unavailable share and an explicit project-source path.
Exercise host-specific resolution on another configured execution node when
available. Verify that search limits do not turn an incomplete search into
an absence claim.

Write a short `report.md` with the changed components, trial/run references,
actual outcomes and remaining limits. Keep machine addresses, local paths
and raw operational evidence in ignored notes. Follow `localrule.md`:
documents in English, human communication in Japanese, no approval gates
during authorized execution, and deployed results accessible over the LAN
where applicable. This document is the requested plan; implementation begins
when requested.

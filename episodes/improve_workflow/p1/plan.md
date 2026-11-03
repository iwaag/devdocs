# Workflow node expectations, examples and list groups

## Purpose

Make the workflow author's expectations visible to both the person and the
executing agent. Extend ordinary and catalog-derived nodes without creating a
new execution type for every combination of expectations.

Source: [braindump.md](braindump.md) and the subsequent discussion. In particular,
list membership must be explicit in the data, sufficient to draw a named boundary
around the members. A shared color alone is not the grouping contract.

## Agreed design

### Explicit groups

A workflow owns a `groups` mapping keyed by stable group IDs. Each group has a
name, kind, description, cardinality and an ordered list of member node IDs.
`groups.<id>.members` is the sole authority for membership; nodes do not duplicate
it in a `collection` or `group` field.

```yaml
groups:
  scenes:
    name: Landscape scenes
    kind: list
    cardinality: variable
    members:
      - scene_example_1
      - scene_example_2
      - scene_example_3
    description: >
      Choose the landscapes required by the actual scene composition.
      The examples do not prescribe the number of landscapes.
```

P1 supports flat, non-overlapping list groups. Each node belongs to at most one
group. Membership references existing authored nodes; duplicate members and
dangling references are validation errors. Generated documentation steps are not
authored members. Empty lists remain representable, including a run that needs no
items; removing all work from an entire workflow remains subject to the existing
workflow validity contract.

`members` preserves list order. Edges continue to express execution dependencies;
list order creates no implicit edges, scheduling policy or parallelism. An absent
group declaration means ordinary ungrouped nodes. P1 implements variable
cardinality, without a minimum/maximum expression language.

The group identity survives run specialization: replacing three examples with
five actual tasks updates `scenes.members` in the same run revision. Historical
revisions retain the original examples and membership.

### Examples

`example: true` marks an ordinary node as a model for concrete work. It preserves
the node's type, delegate target, description and expectations. It can be added to
any supported authored node independently of group membership.

An example is not executable work and does not count as a completed or pending
task in work progress. It remains visible as an example to be specialized.
Starting or dispatching it directly is a semantic error, not a request for human
permission. The agent specializes it through the existing run revision mechanism
before performing the actual work.

The agent chooses concrete tasks and their count from the request and group
description. No automatic loop or fixed three-item expansion is introduced.
Replacement nodes retain provenance identifying the source example node IDs and
definition revision. A one-to-many or many-to-many replacement is possible.
Group identity, revision diff and provenance together explain the specialization.

Specialization changes nodes, membership and edges atomically with `wfe run
adjust`. The agent explicitly supplies dependencies for the concrete graph;
boundary edges are not guessed from list order. Removing an example must not
silently erase its incoming or outgoing intent. The revision reason explains
omitted work, including an empty realized list.

### Expectations and strength

Nodes carry a small structured `expectations` list. Each entry has a key,
`preferred` or `required` strength, and an optional explanatory note. Omitted
strength means `preferred`.

```yaml
expectations:
  - key: webbrowser
    strength: required
  - key: native
    strength: preferred
    note: Launchable on the person's review PC.
```

Initial documented keys cover `webbrowser`, `native`, `project`, `human`, `nfs`
and `s3`. Their meanings distinguish use of the result, audience and delivery
method. Multiple entries are cumulative; browser and native can both be requested.
An unknown key is preserved and presented with its note, without inventing an
executor capability or automatic validator for it. Do not add a general attribute
editor or an alternative-expression language in P1.

- Preferred: prioritize this expectation; explain a different choice and the
  actual result.
- Required: this is a condition of a satisfactory result. An unmet condition
  remains visible as unmet, even if the agent has finished its available work.

Strength is not a permission gate or an automatic proof of satisfaction. Result
records distinguish met, unmet and unverified expectations, with evidence or an
explanation. Agent-reported assessments are labeled as such. Existing artifacts,
outcomes and reports supply evidence; attaching a tag never establishes success.

Example status and group membership have no strength setting: they describe the
definition's structure and meaning.

Host names, credentials and concrete storage endpoints remain environment facts
resolved through existing context and configuration. A `native` note can specify
the intended review environment; it must not silently mean the execution worker.

## Implementation sequence

### 1. Definition contract and storage

Work in `pj-agdev/agdevworld/workflow_editor/`.

- Extend `shared/model.ts`, validation and `server/yamlDoc.ts` with expectations,
  example status and explicit groups. Support incomplete drafts without silently
  dropping malformed or unknown content.
- Include semantic additions in `shared/canonical.ts` definition digests. Member
  order is meaningful; group display styling alone is not.
- Extend node-field allowlists and run revision differences in `server/runs.ts`.
  Report group additions, removals, reordered membership and expectation changes.
- Preserve the fields through published definitions, draft save/apply, trial runs,
  definition snapshots and historical revision reads.
- Document the contract and the P1 defaults. Existing definitions with no new
  fields retain their ordinary meaning; elaborate legacy compatibility is not a
  requirement.

### 2. Run semantics and specialization

- Update `shared/compile.ts` and run state handling so example metadata survives
  compilation and examples are not offered as executable tasks or given ordinary
  generated per-node reports.
- Define visible unresolved-example status separately from work progress. A run
  must not appear to have fulfilled its definition by ignoring unresolved examples.
  Resolve this through graph revision, without a human approval step.
- Apply specialization as an immutable run revision with provenance and explicit
  edge changes. Honor existing stale-revision detection and atomic persistence.
- Keep concrete nodes' execution states and original historical events truthful.
  Changing a tag on a node with execution history must not reinterpret that history
  as example-only activity.
- Show groups, examples, expectation meanings and strengths in CLI inspection and
  the agent's run context. Freeze the supplied meaning with the run rather than
  resolving historical expectations through a mutable current catalog.
- Add a compact expectation assessment representation to result records and expose
  it through the CLI and run view. Missing evidence remains unverified. Avoid
  building a generic compliance engine or forcing separate report files per tag.

### 3. Catalog and agent support

- Extend catalog content, validation, snapshots and instantiation to preserve node
  expectations and example status. Copy membership only through workflow group
  editing; a standalone catalog node does not own a workflow's group IDs.
- Reuse `wfe nodes instantiate` and `wfe run adjust`. Add a specialization helper
  only if needed to make atomic replacement practical, using the same revision
  path rather than a second execution mechanism.
- Update the relevant tool help and executor reference material in
  `pj-agdev/agautolab/`. Keep the shared semantics in one place and provide the
  agent with enough information to choose concrete work autonomously.
- Reconcile the `present` catalog seed's browser-specific wording with explicit
  presentation expectations. Audit existing persisted catalog entries before any
  migration; preserve personal edits and record deliberate changes as revisions.

### 4. Editor and run UI

- Add expectation selection, strength and optional notes to node editing, with an
  example toggle. Use the current published/draft editing flow.
- Add group creation, naming, description, member assignment and member reordering.
  Support removing a group without implicitly deleting its member nodes; deleting
  a node updates membership in the same edit.
- Draw a named boundary around the members and label it `List · Variable`.
  Use a shared background or accent as a supplement to the boundary, preserving
  existing node appearance colors.
- Support group selection and movement while retaining individual node selection,
  dragging and cross-group edges. Group bounds follow members rather than serving
  as the source of membership. Keep generated documentation nodes distinguishable
  from the authored list members.
- Display example badges and compact expectation badges such as
  `Browser · Required`. Details expose the complete explanation.
- Use the same semantics in published, draft, run and historical definition views.
  After specialization, keep the group visible around the actual work and expose
  the source examples through revision/provenance details.
- Keep unmet and unverified expectations visible beside the delivered result.
  Follow the existing Simple/Detail policy without hiding the essential grouping.

### 5. Verification and delivery

Run focused checks as the relevant layers land, followed by the package's required
checks and executor checks where its input contract changed. Verify:

1. YAML and GUI round trips preserve membership order, example status and
   expectations. Semantic changes affect the digest; layout-only changes do not.
2. Invalid references, duplicate or overlapping membership and malformed strengths
   produce actionable errors without silently losing data.
3. Draft save, apply, trial and normal run creation carry the same meaning. Later
   definition/catalog edits do not rewrite an existing run's expectations.
4. The same three landscape examples specialize into two and five concrete tasks
   in separate runs, retaining group identity and replacement provenance. Also
   exercise zero items in a graph with other work, including boundary-edge repair.
5. Example nodes cannot be dispatched as real delegates, do not inflate progress
   counts, and do not disappear silently from the accounting of unresolved intent.
6. List order does not create execution dependencies. Cross-group dependencies and
   stale/concurrent revisions behave according to the existing graph contract.
7. Required unmet, preferred alternative and unverified expectations remain
   distinguishable. Merely adding an artifact or marking a node complete does not
   automatically assert that all expectations were met.
8. In a browser, named boundaries, badges, group movement, member edits and history
   remain understandable with ungrouped nodes and generated documentation steps.
9. An isolated real-agent trial specializes examples before actual delegation and
   reports delivery expectations with evidence. Inspect the resulting graph and
   records, not only the agent's outcome text.

When implementation is undertaken, deploy through the existing WFE/executor
release procedures and verify the deployed behavior. Leave a small demonstration
workflow and run that the person can inspect with few operations. Record actual
revisions, checks, trial evidence, review links and remaining limitations in this
episode's `report.md`; machine-specific details belong in ignored local notes.
Update maintained development and contract documentation with the resulting facts.

## Scope boundaries

P1 expresses native presentation and NFS/S3 delivery expectations; it does not
thereby implement native process lifecycle management or a new storage backend.
The current presentation service is HTTP-oriented. Existing available tools may
fulfill other expectations, and unsupported execution paths remain explicit.

Nested or overlapping groups, general loops, automatic task fan-out, automatic
resource provisioning and arbitrary tag-driven policy enforcement are outside P1.
These can be considered against concrete later use cases.

## Completion criteria

The person can attach expectations to an existing node, distinguish preferred
from required outcomes, and author an unmistakable variable-size list of example
nodes. The list exists as a first-class group in stored definitions and run
revisions. An agent can turn it into concrete work without treating the example
count as fixed, and the person can see the group, its actual members, how they
were derived and which delivery expectations were met or remain unresolved.

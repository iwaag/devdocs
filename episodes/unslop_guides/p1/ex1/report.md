# Investigation: work performed without workflow nodes

The concise-guide release weakened the always-delivered explanation of working through nodes and moved its operational detail behind optional reference reads. Post-release logs show substantial work performed before any node was started, followed by recovery that records it retrospectively. The catalog and workflow instructions still reached the agents: this is a behavioral regression, not missing catalog delivery. The guide change is the leading explanation, but its causal contribution has not been isolated by a controlled replay.

## Scope and evidence

Investigation date: 2026-11-01, Japan time. Read the development policies, project environment notes, p1 report and coverage map, preserved guide, agautolab commit `8f158702ae7b3adee923ac4b5baa818f6a9294fa`, WFE catalog/brief/reminder code, actual run records, execution logs and two original Claude session inputs. Production code, guides, run records and deployment were not changed.

Run records below refer to each run's own working copy, not a possibly outdated copy of that record on project main. Attempt IDs identify the executor's retained JSONL logs. The executor database was opened read-only. Machine paths are deliberately omitted from this portable report.

## What actually happened

All ten freework runs created on 2026-10-31 in curator and aesthete were inspected. In every initial attempt there were **zero node start, adjust, instantiate, record-done or complete tool calls**, and no observed tool call reading the common guide's reference directory. The agents nevertheless researched, generated assets, edited files, rendered screenshots and committed work. Nine runs subsequently acquired retrospective work nodes; one retained only talk and optional documentation.

| Project/run | Initial attempt | Subsequent work record |
| --- | --- | --- |
| curator/freework/run-002 | amvw3nm21-2e9ecc | `work`, retrospective |
| curator/freework/run-003 | amvw8akac-c810ef | `implement-overview`, retrospective |
| curator/freework/run-004 | amvwgcqwe-d6f72d | `work`, retrospective |
| curator/freework/run-005 | amvwgyg10-8beb59 | `icons`, retrospective |
| curator/freework/run-006 | amvwhybr2-07a307 | `build-projects-screen`, retrospective |
| aesthete/freework/run-001 | amvw6kpm9-86a558 | `styles-work`, retrospective |
| aesthete/freework/run-002 | amvw723wi-11d9b6 | `boast-button`, `build-gallery`, retrospective |
| aesthete/freework/run-003 | amvw7p3h4-b03fd9 | No work node |
| aesthete/freework/run-004 | amvwiopk9-3a2f25 | `build-furutiger`, retrospective |
| aesthete/freework/run-005 | amvwjnt1e-f20a2b | `build-ideas`, retrospective |

Representative sequence, curator/freework/run-003:

- Initial attempt log lines 2 and 7 read aesthete's proposal and existing implementation. Lines 14 and 17 change the implementation. Later calls render and inspect browser screenshots, write the report and commit. No node operations occur.
- Run history seq 2–5 contains attempt begin, context, session and end, without node progress. The process exited successfully; it was not interrupted during this work.
- Seq 6 is recovery: it explicitly observes that the redesign was committed but talk was never recorded, and asks that existing work be recorded without repeating it.
- Recovery attempt `amvw8fzln-c25eaa`, log line 16, starts talk and tries to complete it. WFE reminds it that no work node exists. Line 18 uses `record-done`; history seq 11 records the retrospective implementation node. This repairs the record after execution, not the missing planning and visibility during execution.

Thus “everything inside talk” understates the initial failure: in these attempts **even talk was not started**. A later graph with a completed talk and a work node can conceal that sequence unless its history and retrospective metadata are read.

### Ponderer

Ponderer has a related failure, but not a complete loss of graph structure.

- conduct/run-062 initial attempt `amvwf7u00-bb4a29` performs tool discovery, CLI experiments, an image-generation trial, a cagent request and conversation reply #511 before recording its initial ponder node. History seq 2–5 again has no node operation. Seq 6 recovery says exactly that real work happened without starting/recording ponder. The next attempt records ponder at seq 10–11.
- conduct/run-061, run-062 and run-063 still contain distinct asks, delegates and ponder nodes. For example run-063 delegates aesthete work, awaits it and records assessment. They are counterexamples to the claim that every post-change workflow consists only of talk.
- Ponderer's project and conduct guides still describe graph growth, separate commissions and ownership. Their bodies appear in its recorded context configuration. These stronger role instructions are consistent with its partial preservation of structure, though that explanation has not been tested separately.

The common pattern is doing work first and treating WFE state as something to reconcile later. Ponderer's observed initial failure is on a ponder activity, not a literal talk node.

## What the release changed

### 1. The common guide's execution model became weaker

The previous guide opened by identifying the agent as performing one run of a project's workflow. Its automatically supplied body explained node types, modifying the graph, doing catalog activities on separate nodes **before** the work, and why implementation inside talk is invisible to the person.

The current [guide](../../../../../pj-agdev/agautolab/agent/guides/wfrun/guide.md) opens with serving the request. Its work-sizing sentence says a factual question can be answered directly and a larger task “can grow its workflow as you learn.” It never directly says in the common body that substantive work is performed on its own nodes before execution. Nodes appear as a topic of an optional link, “Organize or change the work.”

The [work reference](../../../../../pj-agdev/agautolab/agent/guides/wfrun/reference/work.md) retains that talk is consultation and points to the catalog, but does not preserve the previous explicit before-work instruction and visibility rationale in equivalent detail. The old incidents are retained for developer provenance in [previous-guide.md](../previous-guide.md), rather than delivered operationally.

This affects a known failure, not a hypothetical preference. [fix3](../../../workflow_editor/fix3/report.md) strengthened separate-node guidance after work disappeared into talk. [fix4](../../../workflow_editor/fix4/report.md) found that insufficient on its own, added the `work` template, changed freework's talk description and demonstrated a trial with separate work, presentation, confirmation and cleanup nodes.

### 2. Reference availability did not produce reference use

`execution_contract()` supplies paths and content hashes, not reference bodies. There is no task-sensitive retrieval or automatic injection of the work note. The entry tells agents to read notes when needed, leaving agents that already believe they can act directly to decide whether they need the explanation that would correct that belief.

No guide-reference reads were observed in the ten initial freework attempts. The hashes establish session identity and file availability; they do not establish that the model read or followed the notes. The coverage map's “current home” is consequently not evidence of equivalent behavioral coverage.

### 3. The request moved before the execution contract

`request_intro()` now places the complete task and delivery destination first, ahead of the common guide, context and WFE brief. This plausibly strengthens ordinary task execution while weakening the framing that the task is performed as a workflow run. It changed in the same commit as the guide text, so its independent effect is unknown.

### 4. The regression was not behaviorally tested before deployment

The [p1 report](../report.md) explicitly says there was no real-model replay. The historical curator example was an enumeration question; prompt size and answer routing were measured offline. The guide tests check prompt order, destination, reference availability and contract invalidation, not whether a model creates and starts separate work nodes before substantive work.

Passing these tests established transport and assembly, not preservation of fix3/fix4 behavior. The proposed guide-lab trials also emphasized delivered answers and usable files without an explicit check that substantial work appears as nodes before execution.

## What was not lost

The WFE brief still injects `CATALOG_MEANING` and the frozen `work` expectation: work should have its own nodes, added before execution, rather than happen inside talk. Freework's intent and talk description also retain that separation. WFE's `unrecorded-activity` reminder remains in place.

This was checked against actual original user inputs, not only reconstructed prompts:

- curator/freework/run-003, session `61f6e755-d49f-40f1-9b39-e53f4bd16edc`: 14,986-character input. The new common guide begins at character offset 2,105; separate-node catalog guidance remains near offset 11,385. The input ends by asking the agent to read the run.
- aesthete/freework/run-001, session `533e4d71-b88c-4329-bca5-325cc856567e`: the same concise guide and explicit `work` catalog expectation are present.

Both nevertheless act without node lifecycle operations. This rules out missing catalog injection or prompt truncation as explanations for those examples. It also means restoring a sentence alone cannot be assumed sufficient: explicit instructions were already present and ignored.

The reminder operates at report/ask/completion boundaries and uses observable signals. It cannot show work prospectively if an agent edits files without calling those boundaries. Retrospective recording is useful recovery, but does not restore the person's ability to follow the work while it happens.

## Assessment and correction direction

**Established:** substantial post-release work bypassed node execution; the optional work reference was not read in the inspected initial attempts; the old operational explanation was weakened; the explicit catalog expectation was still delivered; recovery often repaired the graph only afterwards. This is broader than a cosmetic lack of small nodes.

**Leading causal explanation:** the combined change from workflow execution as the opening contract to request-first direct execution, with essential workflow meaning relegated to optional retrieval. The timing, diff and observed behavior support this explanation. They do not prove that shortening alone caused it, or exclude model/harness changes or pre-existing noncompliance. Earlier fix4 had a successful real-model trial, not a guarantee that every later run complied.

A focused correction would keep the short entry while stating the execution model there: talk is consultation; substantive work uses suitable work nodes added before execution; the graph is how the person follows the work. Syntax and longer examples can remain in references. Ordinary factual answers need no elaborate workflow. Ponderer needs the equivalent fact for its activity graph.

Validate any correction with a real-model comparison on fresh isolated runs, including a substantial freework implementation and a conductor turn with investigation/commissioned work. Inspect actual tool order and graph history: separate nodes before substantive execution, started/completed nodes, needed reference reads, usable delivery, and no recovery required solely to register finished work. Compare guide wording and request order independently if identifying their individual effects matters. Successful static assembly and `record-done` after the fact are not passes for that check.

No such comparison, production correction or rollback was performed in this investigation.

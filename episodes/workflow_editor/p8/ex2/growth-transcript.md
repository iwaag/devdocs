# Rehearsal transcript (checks/growth.ts)

A synthetic project; "session A/B/C" are attempts claimed from the exec store, driven by this script as the executing agent would.

**Person**: `wfe request adhoc --braindump <area>/braindump.md --author "Sample Person" --workspace demo --json`

```text
{
  "job": {
    "id": 1,
    "receipt": "request:cli-1791881602319-50679",
    "kind": "start",
    "workspace": "demo",
    "workflow": "adhoc",
    "run": "run-001",
    "reason": "start: the person's request",
    "state": "queued",
    "created": "2026-10-13T08:53:22.620Z",
    "claimed": null,
    "attempt": null,
    "note": null
  },
  "ref": {
    "workflow": "adhoc",
    "run": "run-001"
  },
  "duplicate": false
}
```

**executor**: `wfe exec claim`

```text
{
  "job": {
    "id": 1,
    "receipt": "request:cli-1791881602319-50679",
    "kind": "start",
    "workspace": "demo",
    "workflow": "adhoc",
    "run": "run-001",
    "reason": "start: the person's request",
    "state": "claimed",
    "created": "2026-10-13T08:53:22.620Z",
    "claimed": "2026-10-13T08:53:22.767Z",
    "attempt": "amv6g1hhr-712f43",
    "note": null
  },
  "attempt": {
    "id": "amv6g1hhr-712f43",
    "job": 1,
    "workspace": "demo",
    "workflow": "adhoc",
    "run": "run-001",
    "host": "agstudio.home.arpa",
    "pid": null,
    "state": "launching",
    "began": "2026-10-13T08:53:22.767Z",
    "ended": null,
    "exit_code": null,
    "signal": null,
    "outcome": null,
    "log": "<area>/.local/exec/logs/amv6g1hhr-712f43.jsonl",
    "stop_requested": null,
    "backend": null,
    "baseline": "{\".\":{\"devdocs/runs/adhoc/run-001/braindump.md\":\"9d2c1bfb76b6a76ac8cd\",\"devdocs/runs/adhoc/run-001/context/node-catalog.json\":\"0ec453f82d2dc77cf703\",\"devdocs/runs/adhoc/run-001/definition/adhoc.yaml\":\"4ecd436a0b2ee776826c\",\"devdocs/runs/adhoc/run-001/run.json\":\"cd89ea83cbd0afbb7480\",\"devdocs/workflows/adhoc.yaml\":\"4ecd436a0b2ee776826c\"}}"
  },
  "workspace": {
    "id": "demo",
    "label": "Demo",
    "host": "this machine",
    "path": "/private<area>/pj-demo",
    "project": "demo"
  },
  "ref": {
    "workflow": "adhoc",
    "run": "run-001"
  },
  "brief": "Run: demo / adhoc/run-001 (record devdocs/runs/adhoc/run-001/run.json)\nAgent: you perform this run as autolab.\nWorkspace: /private<area>/pj-demo\nWhy this attempt: sta
…
```

**executor**: `wfe exec started amv6g1hhr-712f43 --pid 50578`

```text
{
  "ok": true
}
```

**session A** (WFE_ATTEMPT=amv6g1hhr-712f43): `wfe run start adhoc/run-001 talk`

```text
Recorded: talk started.
Run adhoc/run-001 is in-progress (seq 3)
```

**session A** (WFE_ATTEMPT=amv6g1hhr-712f43): `wfe run ask adhoc/run-001 talk --question "Which file types, and should it count lines too?" --to "Sample Person"`

```text
Recorded question q1 to Sample Person; talk waits on it.
Run adhoc/run-001 is in-progress (seq 4); waiting: talk (next move: Sample Person)
```

**executor**: `wfe exec finished amv6g1hhr-712f43 --exit-code 0`

```text
{
  "ok": true,
  "outcome": "exited",
  "checkpoint": {
    "id": 1,
    "workspace": "demo",
    "workflow": "adhoc",
    "run": "run-001",
    "kind": "after attempt amv6g1hhr-712f43: awaiting-person",
    "attempt": "amv6g1hhr-712f43",
    "state": "failed",
    "created": "2026-10-13T08:53:23.442Z",
    "updated": "2026-10-13T08:53:23.640Z",
    "steps": "[{\"repo\":\".\",\"include\":[\"devdocs/runs/adhoc/run-001/braindump.md\",\"devdocs/runs/adhoc/run-001/context/node-catalog.json\",\"devdocs/runs/adhoc/run-001/definition/adhoc.yaml\",\"devdocs/runs/adhoc/run-001/run.json\"],\"excluded\":[\"devdocs/workflows/adhoc.yaml\"],\"conflicts\":[],\"commit\":\"5b5a760a0da61c8a672ac82f4d863dd7135b657e\",\"branch\":\"main\",\"error\":\"git push failed: fatal: 'origin' does not appear to be a git repository\\nfatal: Could not read from remote repository.\\n\\nPlease make sure you have the correct access rights\\nand the repository exists.\"}]",
    "error": ".: git push failed: fatal: 'origin' does not appear to be a git repository\nfatal: Could not read from remote repository.\n\nPlease make sure you have the correct access rights\nand the repository exists.",
    "baseline": "{\".\":{\"devdocs/runs/adhoc/run-001/braindump.md\":\"9d2c1bfb76b6a76ac8cd\",\"devdocs/runs/adhoc/run-001/context/node-catalog.json\":\"0ec453f82d2dc77cf703\",\"devdocs/runs/adhoc/run-001/definition/adhoc.yaml\":\"4ecd436a0b2ee776826c\",\"devdocs/runs/adhoc/run-001/run.json\":\"cd89ea83cbd0afbb7480\",\"devdocs/workflows/adhoc.yaml\":\"4ecd436a0b2ee776826c\"}}"
  },
  "resumeJob": null
}
```

**Person**: `wfe run answer adhoc/run-001 q1 --answer "Plain text only. Yes, lines too — then show me the counts for README.md."`

```text
Recorded an answer to q1. A node waiting on it keeps waiting until the answer is taken up.
The executor resumes the run (job 2, queued).
Run adhoc/run-001 is in-progress (seq 6); waiting: talk (next move: autolab (answer recorded, not yet taken up))
```

**executor**: `wfe exec claim`

```text
{
  "job": {
    "id": 2,
    "receipt": "answer:demo/adhoc/run-001/q1/0",
    "kind": "resume",
    "workspace": "demo",
    "workflow": "adhoc",
    "run": "run-001",
    "reason": "resume: answer 0 to q1 recorded",
    "state": "claimed",
    "created": "2026-10-13T08:53:23.801Z",
    "claimed": "2026-10-13T08:53:23.963Z",
    "attempt": "amv6g1iez-f011f3",
    "note": null
  },
  "attempt": {
    "id": "amv6g1iez-f011f3",
    "job": 2,
    "workspace": "demo",
    "workflow": "adhoc",
    "run": "run-001",
    "host": "agstudio.home.arpa",
    "pid": null,
    "state": "launching",
    "began": "2026-10-13T08:53:23.964Z",
    "ended": null,
    "exit_code": null,
    "signal": null,
    "outcome": null,
    "log": "<area>/.local/exec/logs/amv6g1iez-f011f3.jsonl",
    "stop_requested": null,
    "backend": null,
    "baseline": "{\".\":{\"devdocs/runs/adhoc/run-001/run.json\":\"4bace324f3821cf22f91\",\"devdocs/workflows/adhoc.yaml\":\"4ecd436a0b2ee776826c\"}}"
  },
  "workspace": {
    "id": "demo",
    "label": "Demo",
    "host": "this machine",
    "path": "/private<area>/pj-demo",
    "project": "demo"
  },
  "ref": {
    "workflow": "adhoc",
    "run": "run-001"
  },
  "brief": "Run: demo / adhoc/run-001 (record devdocs/runs/adhoc/run-001/run.json)\nAgent: you perform this run as autolab.\nWorkspace: /private<area>/pj-demo\nWhy this attempt: resume: answer 0 to q1 recorded\nThe previous attempt amv6g1hhr-712f43 ended: exited.\nNode talk waits; next move: autolab (answer recorded, not yet taken up).\nReady: (none); running: (none).\n\nThe person's node catalog, as it
…
```

**executor**: `wfe exec started amv6g1iez-f011f3 --pid 50578`

```text
{
  "ok": true
}
```

**session A** (WFE_ATTEMPT=amv6g1hhr-712f43): `wfe run progress adhoc/run-001 talk --note late`

```text
wfe run: attempt amv6g1hhr-712f43 is not the current attempt of this run (amv6g1iez-f011f3 is); a stopped or superseded attempt records nothing
```

**session B** (WFE_ATTEMPT=amv6g1iez-f011f3): `wfe run take-up adhoc/run-001 q1`

```text
Recorded: the answer to q1 is taken up.
Run adhoc/run-001 is in-progress (seq 8)
```

**session B** (WFE_ATTEMPT=amv6g1iez-f011f3): `wfe run revisions adhoc/run-001 --show r1`

```text
schema: ag.workflow.v2
id: adhoc
name: Ad hoc
intent: Start from the braindump and grow the work in conversation.
documentation:
  policy: custom
  plan: false
  reports: false
  final: false
nodes:
  talk:
    type: talk
    name: Talk
    description: Understand the request, hold the consultation it needs, and make the next work concrete.
```

**session B** (WFE_ATTEMPT=amv6g1iez-f011f3): `wfe run adjust adhoc/run-001 --definition <area>/r2.yaml --reason "q1 agreed the script and a showing" --source answer:q1 --dry-run`

```text
Not applicable (from r1):
  mid-node: talk is running; an adjustment applies at a node boundary, when no node runs
  work added: count, show; removed: —; changed: —
  graph: +2 −0 nodes, +2 −0 edges (new: count → show; talk → count)
```

**session B** (WFE_ATTEMPT=amv6g1iez-f011f3): `wfe run adjust adhoc/run-001 --definition <area>/r2.yaml --reason "q1 agreed the script and a showing" --source answer:q1 --completing talk --outcome "Agreed: words and lines, plain text; show README.md counts." --dry-run`

```text
Applicable to r1 (dry run; nothing written).
  work added: count, show; removed: —; changed: —
  graph: +2 −0 nodes, +2 −0 edges (new: count → show; talk → count)
```

**session B** (WFE_ATTEMPT=amv6g1iez-f011f3): `wfe run adjust adhoc/run-001 --definition <area>/r2.yaml --reason "q1 agreed the script and a showing" --source answer:q1 --completing talk --outcome "Agreed: words and lines, plain text; show README.md counts." --receipt session-b-adjust-1`

```text
Applied: adhoc/run-001 now follows r2 (from r1). talk completed with it; 1 node completed, ready: count.
  work added: count, show; removed: —; changed: —
  graph: +2 −0 nodes, +2 −0 edges (new: count → show; talk → count)
```

**session B** (WFE_ATTEMPT=amv6g1iez-f011f3): `wfe run adjust adhoc/run-001 --definition <area>/r2.yaml --reason "q1 agreed the script and a showing" --source answer:q1 --completing talk --outcome "Agreed: words and lines, plain text; show README.md counts." --receipt session-b-adjust-1`

```text
Already recorded: adhoc/run-001 follows r2 (same receipt; nothing written again).
  work added: —; removed: —; changed: —
  graph: +2 −0 nodes, +2 −0 edges (new: count → show; talk → count)
```

**session B** (WFE_ATTEMPT=amv6g1iez-f011f3): `wfe run start adhoc/run-001 count`

```text
Recorded: count started.
Run adhoc/run-001 is in-progress (seq 11)
```

**session B** (WFE_ATTEMPT=amv6g1iez-f011f3): `wfe run complete adhoc/run-001 count --outcome "count.py written; README.md: 12 words, 3 lines."`

```text
Recorded: count completed.
Run adhoc/run-001 is in-progress (seq 12); ready: show
```

**session B** (WFE_ATTEMPT=amv6g1iez-f011f3): `wfe run start adhoc/run-001 show`

```text
Recorded: show started.
Run adhoc/run-001 is in-progress (seq 13)
```

**session B** (WFE_ATTEMPT=amv6g1iez-f011f3): `wfe run ask adhoc/run-001 show --question "README.md has 12 words and 3 lines. Anything else?" --to "Sample Person"`

```text
Recorded question q2 to Sample Person; show waits on it.
Run adhoc/run-001 is in-progress (seq 14); waiting: show (next move: Sample Person)
```

**executor**: `wfe exec finished amv6g1iez-f011f3 --exit-code 0`

```text
{
  "ok": true,
  "outcome": "exited",
  "checkpoint": {
    "id": 2,
    "workspace": "demo",
    "workflow": "adhoc",
    "run": "run-001",
    "kind": "after attempt amv6g1iez-f011f3: awaiting-person",
    "attempt": "amv6g1iez-f011f3",
    "state": "failed",
    "created": "2026-10-13T08:53:26.152Z",
    "updated": "2026-10-13T08:53:26.352Z",
    "steps": "[{\"repo\":\".\",\"include\":[\"devdocs/runs/adhoc/run-001/run.json\",\"devdocs/runs/adhoc/run-001/definition/r2/adhoc.yaml\"],\"excluded\":[\"devdocs/workflows/adhoc.yaml\"],\"conflicts\":[],\"commit\":\"02630eb0f4d30ccd668be111bcdfd0939cbfdcaf\",\"branch\":\"main\",\"error\":\"git push failed: fatal: 'origin' does not appear to be a git repository\\nfatal: Could not read from remote repository.\\n\\nPlease make sure you have the correct access rights\\nand the repository exists.\"}]",
    "error": ".: git push failed: fatal: 'origin' does not appear to be a git repository\nfatal: Could not read from remote repository.\n\nPlease make sure you have the correct access rights\nand the repository exists.",
    "baseline": "{\".\":{\"devdocs/runs/adhoc/run-001/run.json\":\"4bace324f3821cf22f91\",\"devdocs/workflows/adhoc.yaml\":\"4ecd436a0b2ee776826c\"}}"
  },
  "resumeJob": null
}
```

**Person**: `wfe run answer adhoc/run-001 q2 --answer "That is all, thanks."`

```text
Recorded an answer to q2. A node waiting on it keeps waiting until the answer is taken up.
The executor resumes the run (job 3, queued).
Run adhoc/run-001 is in-progress (seq 16); waiting: show (next move: autolab (answer recorded, not yet taken up))
```

**executor**: `wfe exec claim`

```text
{
  "job": {
    "id": 3,
    "receipt": "answer:demo/adhoc/run-001/q2/0",
    "kind": "resume",
    "workspace": "demo",
    "workflow": "adhoc",
    "run": "run-001",
    "reason": "resume: answer 0 to q2 recorded",
    "state": "claimed",
    "created": "2026-10-13T08:53:26.517Z",
    "claimed": "2026-10-13T08:53:26.677Z",
    "attempt": "amv6g1kid-2a1e79",
    "note": null
  },
  "attempt": {
    "id": "amv6g1kid-2a1e79",
    "job": 3,
    "workspace": "demo",
    "workflow": "adhoc",
    "run": "run-001",
    "host": "agstudio.home.arpa",
    "pid": null,
    "state": "launching",
    "began": "2026-10-13T08:53:26.678Z",
    "ended": null,
    "exit_code": null,
    "signal": null,
    "outcome": null,
    "log": "<area>/.local/exec/logs/amv6g1kid-2a1e79.jsonl",
    "stop_requested": null,
    "backend": null,
    "baseline": "{\".\":{\"devdocs/runs/adhoc/run-001/run.json\":\"443b10fda85bf8c5ea38\",\"devdocs/workflows/adhoc.yaml\":\"4ecd436a0b2ee776826c\"}}"
  },
  "workspace": {
    "id": "demo",
    "label": "Demo",
    "host": "this machine",
    "path": "/private<area>/pj-demo",
    "project": "demo"
  },
  "ref": {
    "workflow": "adhoc",
    "run": "run-001"
  },
  "brief": "Run: demo / adhoc/run-001 (record devdocs/runs/adhoc/run-001/run.json)\nAgent: you perform this run as autolab.\nWorkspace: /private<area>/pj-demo\nWhy this attempt: resume: answer 0 to q2 recorded\nThe previous attempt amv6g1iez-f011f3 ended: exited.\nNode show waits; next move: autolab (answer recorded, not yet taken up).\nReady: (none); running: (none).\n\nThe person's node catalog, as it
…
```

**executor**: `wfe exec started amv6g1kid-2a1e79 --pid 50578`

```text
{
  "ok": true
}
```

**session C** (WFE_ATTEMPT=amv6g1kid-2a1e79): `wfe run take-up adhoc/run-001 q2`

```text
Recorded: the answer to q2 is taken up.
Run adhoc/run-001 is in-progress (seq 18)
```

**session C** (WFE_ATTEMPT=amv6g1kid-2a1e79): `wfe run complete adhoc/run-001 show --outcome "Counts shown; nothing else wanted."`

```text
Recorded: show completed.
Run adhoc/run-001 is completed (seq 19)
```

**executor**: `wfe exec finished amv6g1kid-2a1e79 --exit-code 0`

```text
{
  "ok": true,
  "outcome": "exited",
  "checkpoint": {
    "id": 3,
    "workspace": "demo",
    "workflow": "adhoc",
    "run": "run-001",
    "kind": "after attempt amv6g1kid-2a1e79: completed",
    "attempt": "amv6g1kid-2a1e79",
    "state": "failed",
    "created": "2026-10-13T08:53:27.384Z",
    "updated": "2026-10-13T08:53:27.588Z",
    "steps": "[{\"repo\":\".\",\"include\":[\"devdocs/runs/adhoc/run-001/run.json\"],\"excluded\":[\"devdocs/workflows/adhoc.yaml\"],\"conflicts\":[],\"commit\":\"ff10b602b231dce29a966ea05b840a39dd287154\",\"branch\":\"main\",\"error\":\"git push failed: fatal: 'origin' does not appear to be a git repository\\nfatal: Could not read from remote repository.\\n\\nPlease make sure you have the correct access rights\\nand the repository exists.\"}]",
    "error": ".: git push failed: fatal: 'origin' does not appear to be a git repository\nfatal: Could not read from remote repository.\n\nPlease make sure you have the correct access rights\nand the repository exists.",
    "baseline": "{\".\":{\"devdocs/runs/adhoc/run-001/run.json\":\"443b10fda85bf8c5ea38\",\"devdocs/workflows/adhoc.yaml\":\"4ecd436a0b2ee776826c\"}}"
  },
  "resumeJob": null
}
```

**Person**: `wfe run show adhoc/run-001 --history`

```text
Run adhoc/run-001 — completed   (devdocs/runs/adhoc/run-001)
Input: braindump.md, the words of Sample Person
Executor: autolab (backend: unknown)
Project: demo
Definition (fixed): adhoc in devdocs/runs/adhoc/run-001/definition/
  adhoc                approvals at creation: intent unapproved, definition unapproved; current devdocs/workflows/adhoc.yaml: same
Adjusted: revision r2 in force (r2 from r1 by autolab: q1 agreed the script and a showing); the original stays in definition/ (wfe run revisions)
Node catalog: 2 templates (present r1, cleanup r1), frozen at creation in devdocs/runs/adhoc/run-001/context/node-catalog.json (sha256 0ec453f82d2d…; wfe run nodes)
Created 2026-10-13T08:53:22.612Z; started 2026-10-13T08:53:23.098Z; ended 2026-10-13T08:53:27.196Z; last update 2026-10-13T08:53:27.374Z (0 s ago); seq 20
Documentation policy: custom, no generated documents (frozen at creation, ag.wfe-compile.v3)
Nodes (3 completed):
  count                completed outcome: count.py written; README.md: 12 words, 3 lines.
  show                 completed outcome: Counts shown; nothing else wanted.
  talk                 completed outcome: Agreed: words and lines, plain text; show README.md counts.
Questions:
  q1 taken-up  to Sample Person (node talk), asked by autolab: Which file types, and should it count lines too?
     answer 0 (taken up) from autolab via cli: Plain text only. Yes, lines too — then show me the counts for README.md.
  q2 taken-up  to Sample Person (node show), asked by autolab: README.md has 12 words and 3 lines. Anything else?
     answer 0 (taken up) from autol
…
```

**Person**: `wfe run check adhoc/run-001`

```text
adhoc/run-001: usable
```

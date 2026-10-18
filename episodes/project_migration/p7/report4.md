# p7 Step 4 — unified review, publication and integration

Sources:
- agdevworld `14e2b42` (WFE views, on top of `58e5c29`/`7745db5`);
- pj-agdev `c15bb7b` (Observer);
- agautolab `84ae825` (notifier test expectations).

Deployed and used in Steps 5–6.

## One central view of a run on any worker

**Run view.**
- *Details* gains **Execution node**: the node, or the area default when the record names none,
  and the last `run.assign` with who and why.
- The status chip says `on <worker>`, or `on <worker> (contact lost)`.
- *Execution and publication* adds:
  - the node line, with availability and last contact;
  - for a remote attempt, its process id *on that worker*;
  - "contact lost" with the explicit note that the attempt stays open and its process may still be
    working.
- Each attempt lists its worker.
- A stop is "pending until the worker confirms the end".
- Queued jobs show their worker and their waiting reason (e.g. `waiting for worker gpupc: … does not
  offer profile …`).
- The *Work* line says the files shown are the worker's last published checkpoint plus the
  documents it uploaded, and that live changes on the worker are not shown until its attempt ends.
  A live run is not mistaken for its last snapshot.

**Home** shows one chip per worker when an area has several: its slot, its queue, or how long since
contact was lost.

**Content.** Agent logs, run documents, attachments, presentations and outputs are central:
- logs are shipped into the coordinator's log path, so activity, Detailed IO (the notifier reads the
  attempt log) and the compaction check work as for the local worker;
- run documents are uploaded before every forwarded command and at the end;
- diffs and artifacts are read from the record copy, aligned at every worker checkpoint;
- presentations are registered and served by the coordinator from the record copy;
- large outputs go to the share through the existing `wfe run publish` (the folder is uploaded into
  the record copy, then published with its manifest). The share route's `?inline=1` displays images
  and video in the browser.

## Decisions and integration

- Confirmation questions keep binding to the exact candidate and output manifest (`wfe run ask …
  --output`): it is the same coordinator operation, forwarded.
- Decisions remain records. Integration is the coordinator's one journaled operation. It runs on the
  record copy after the worker's checkpoint is aligned. A pending worker checkpoint makes it wait,
  never integrate an older tree. Depth-first submodule order and target fetching are as before. The
  worker never integrates.
- Concurrent runs integrate through the same per-target locks and fetch-before-merge as before.

## Observer and Zulip

**Observer** (`wfclassify`) judges a remote worker's attempt by its shipped log and the worker's
contact. A reporting worker with a moving log is `working`; a silent log is still `silent`
(uncertain). A worker that stopped reporting gives quiet `worker-down`: no incident, and not a lost
process. `executor` (the local worker) is judged as before.

**Zulip transport.** Intake and notification are unchanged and central: they read and write the
coordinator's records. The run topic and IO channel of a PC run are the same as any other run's.

**Notifier tests.** The 13 inherited failures were corrected (Step 3). There are no remaining
unrelated failures in agautolab, WFE (403/403) or Observer (115/115).

## Verified in Steps 5–6, not here

- Graphical Godot work and its inspection from the LAN (screenshots or capture on the share, opened
  inline from the run view; a prepared native launch on the PC desktop).
- That the browser, CLI, Zulip and Observer show the same run identity and state for a real PC run.

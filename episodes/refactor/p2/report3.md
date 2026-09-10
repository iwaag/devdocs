# p2 step 3 — asynchronous generation and delivery, across short sessions

## What had to keep working

`submit → notifier callback → collection` is the shape that lets a
ten-minute ComfyUI render happen inside agent runs that last seconds. Step 2
moved the record; this step is about what the record has to be *good enough
for*: a job survives the run that submitted it, a listener restart, a
duplicate callback, and a request that has since been replaced.

Most of it was already carried by step 2's identity change, and this report
says which part is which.

## What step 2 already gave it

**The stable work identity is the run's anchor id.** `workspace_dir` is
`.local/agentws/r<run id>/generator/`, minted when the run topic is opened
and unique by construction. Three of the plan's requirements fall out of
that alone:

- the pending job, its generation workspace and the requester live together
  under one name (`watching.json` also carries `run`, `request`,
  `conversation` and `trigger`, so a directory of render files says whose job
  it is without a lookup);
- **a replacement cannot collect the old job**: it has its own run topic, so
  its own workspace, where there is no `watching.json` at all;
- **and cannot receive its result**: the old run topic's `[assetrun]` note
  names the old request by id, and the delivery follows the id home to the
  retired conversation.

**The collecting run reads the right plan.** `prepare_workspace(…,
collecting=True)` leaves `plan.md` and `tools/` exactly as the job was
submitted with them. Re-planning changes the request; it must not change an
attempt already queued in ComfyUI.

**Restart recovery needs nothing new.** The listener holds nothing in memory
about a pending job: the rename `pending.json` → `watching.json` is the whole
state, and it is on disk. A fresh process reads the same file and collects
the same job.

## What this step added

**A callback can arrive twice.** The notifier's callback is an ordinary post,
and an ordinary post is what starts a run — so a retried callback, or a
restart that re-reads the same mention, would arrive with no `watching.json`
left to collect. It would read as a fresh trigger: run the generator against
the plan, submit a **new** ComfyUI job, and deliver a second time for one
request. Neither the record nor the workspace said "this job is finished".

`collected.txt` says it. Every collected job's `prompt_id` is appended beside
the workspace, and a trigger whose text names one — the full id, or the
8-character short form the callback's first line carries — is answered and
nothing else. No generator run, so no second job; no delivery, so the
requester is not told twice.

Three deliberate choices in it:

- **on disk**, because a callback outlives the process that asked for the
  watch: a restart recovers the pending job from `watching.json` and the
  finished ones from here, and submits neither again;
- **matched on the id only**, never on the wording — the notifier's phrasing
  is not this module's to depend on;
- **written before the delivery**, so a delivery that half-succeeded cannot
  leave the job open to being collected all over again.

It is narrow on purpose. The guard is about a repeated *callback*, not about
the topic: a person posting "make it brighter and run it again" is ordinary
work and still runs.

## Verification

Controlled fixtures only — no ComfyUI job was run, and the plan says none is
needed to test callback timing. `tests/test_assetrun_topic.py`:

| test | what it pins |
|---|---|
| `…queued_job_is_handed_to_the_notifier_and_the_request_stays_open` | the reply is the live mention, nothing is delivered, the run is `pending`, and `watching.json` names the run and the request |
| `…next_run_collects_the_outputs_and_records_the_result` | the callback triggers the collection; the key is recorded in both conversations |
| `…collecting_run_keeps_the_plan_the_job_was_submitted_with` | re-planning mid-flight does not reach the attempt |
| `…restart_while_waiting_collects_the_same_job` | a fresh process, the same workspace, one collection |
| `…second_callback_submits_nothing_and_delivers_nothing` | one generation, one delivery, for one job |
| `…restart_after_a_collection_does_not_run_the_job_again` | restart recovery meets the same callback and does nothing |
| `…person_asking_for_it_again_after_a_collection_still_runs` | the guard does not swallow real work |
| `…replacement_does_not_collect_the_old_attempt_s_job` | separate workspaces; the late result lands in the retired request |
| `…requester_survives_the_wait_and_the_notifier_is_never_named` | the delivery names who triggered it, not the bot that woke the collection |
| `…run_that_queued_and_then_failed_is_a_failure_not_a_wait` | `failure.flag` wins over a queued job |

The two duplicate-callback tests fail against the module with the guard
disabled (`2 failed, 37 passed`), which is what says they are testing it.

```
$ uv run pytest -q        # agforge
218 passed in 6.9s
```

## Limitations left standing

- **`collected.txt` is per workspace, so deleting the workspace forgets it.**
  The plan explicitly does not require rebuilding render state after local
  files are deleted; a callback arriving after that would run the plan again,
  which is the same thing a person asking for a re-run gets.
- **A duplicate callback still costs one listener serving** — the ack, the
  read, and one post. It costs no agent run, which is the expensive half.
- **Forced interruption is still out of scope.** Retiring a request while a
  job is pending leaves the job running; it is collected by the retired run
  topic and delivered there.
- **autolab's callback resumption is unchanged and untested live here.** The
  delivery names the trigger exactly as before; whether a supercoder resumes
  on it is the step 5 demonstration's evidence, not this step's.

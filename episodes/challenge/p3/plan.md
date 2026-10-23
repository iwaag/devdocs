# Challenge p3: records of work

## Outcome

A challenge keeps its statement, source material and records of work in one
Git directory. People and agents can preserve an idea, an unsuccessful
experiment, a partial solution or an improved solution without fitting the
work into a shared outcome schema. The standard UI makes conventional
records easy to read without making that convention a storage requirement.

Source: `braindump.md` and the subsequent discussion. The discussion adopted
one directory per challenge and placed trial evidence inside its trial.

## Agreed structure and meaning

```text
challenges/
  ch-0001/
    summary.md
    voice.wav
    wedo/
      speech-stretch/
        summary.md
        evidence/
          output.wav
      another-approach/
        notes.md
        experiments/
          comparison.wav
```

- The directory identifies the challenge. All repository-held material for
  that challenge lives beneath it.
- The standard challenge `summary.md` contains its free-form statement and
  existing management frontmatter. Source material that explains the
  challenge belongs at challenge scope; its internal placement is free.
- Each direct child directory of `wedo/` represents a trial. Its name and
  contents are chosen by the people or agents doing the work.
- A trial's `summary.md` is optional. The UI renders it when present and
  otherwise offers the directory and its files. No headings, outcome fields,
  manifest or success criteria are required.
- Evidence and results of a trial belong inside that trial. `evidence/` is
  an example, not a required name or hierarchy. Shared input need not be
  copied into each trial.
- A trial may span several workflow runs; a run may contribute to several
  trials. Trial identity is independent of WFE run/attempt identity.
- Git does not preserve empty directories. A trial becomes visible when it
  contains a tracked file; no compulsory placeholder is introduced.

## Standard style, not a universal lifecycle

Keep `open`, `resolved` and `withdrawn`, with the existing state-changing
actions, as the standard management vocabulary. Their interpretation and
the decision to change them belong to the local work context.

Finishing a run or publishing a trial never changes a challenge's state.
Resolved and withdrawn challenges remain readable and can receive further
trials. Neither a new trial nor an unsuccessful trial automatically reopens
a challenge. There is no mandatory reporting step, evaluation workflow,
assignment mechanism or model-based interpretation of summaries.

Preserve custom states and show their original values. Show an absent state
as unspecified, rather than interpreting either case as `open`. Standard
filters/counts must distinguish these records and keep them discoverable.
Missing or nonstandard frontmatter must not prevent browsing the original
files. Normal edits must not silently normalize custom metadata or rewrite
unrelated free-form content. Invalid metadata may have a local reading
diagnostic without making the underlying record inaccessible.

## Implementation sequence

### 1. Extend repository reading and revision semantics

Primary implementation: `pj-agdev/agdevworld/workflow_editor/server/challenges.ts`
and `workflow_editor/docs/challenges.md`.

- Discover challenges by their directory identity, including those without
  a standard summary. Read standard metadata as an optional interpretation
  of the original files; keep raw file access available.
- Enumerate trials and nested files without requiring a trial schema.
- Include every file under a challenge directory in its history and latest
  revision, including files changed through an ordinary Git commit.
- Return trial summaries and file listings at an explicit repository commit.
  A pinned view must not mix in newer files, summaries or trial additions.
- Rebuild all derived metadata from Git. Keep the existing publication
  journal distinct from disposable indexes.
- Expose trial summaries to search with a link to the matching trial; avoid
  indexing arbitrary binary files or implying semantic outcome recognition.

### 2. Extend publication, CLI and HTTP access

Primary implementation: the challenge store, `cli/challenge.ts` and
`server/challengeApi.ts`.

- Provide trial listing, summary/file reading and directory publication via
  CLI and HTTP. Document final command names and examples in command help.
- Publish a selected set of files atomically through the existing journal,
  retaining actor/source/run attribution, request deduplication, durable
  drafts, retry behavior and mirror publication.
- Allow nested paths within the target challenge/trial. Apply file-path and
  rendering safeguards without restricting report structure; a supplied
  path or symlink must not escape the repository material being published.
- Detect conflicts against the files being changed. Independent trial
  additions should not conflict merely because the challenge revision
  advanced. Concurrent changes to the same files preserve the rejected
  draft and identify the conflict rather than overwriting work.
- Retain existing file-size/storage limits and explain them in tool help.
  Large externally stored artifacts may be linked, with their durability
  understood separately from the Git backup.
- Ensure uploads originate from the invoking execution node and downloaded
  files are written there, rather than accidentally using coordinator-local
  paths. Verify the HTTP body limit against supported upload sizes.

### 3. Migrate existing records

- Move `challenges/<id>.md` to `challenges/<id>/summary.md` and move existing
  `evidence/<id>/` material beneath the corresponding challenge directory.
- Inspect the existing evidence's meaning. Put known trial evidence in its
  trial; keep problem examples and source material at challenge scope. Do
  not invent a historical trial or result merely to classify an old file.
- Update relative links and tracked documentation referring to moved files.
  Preserve IDs, content, state and authorship history.
- Preserve pinned historical references by reading the layout present at
  the requested commit. Historical reading is preservation of evidence;
  continued writing through the old layout/API is not required.
- Rebuild indexes, verify existing conversation attachments and verify the
  repository mirror after publication.

### 4. Present work in aghangout

Primary implementation: the managed `pj-aghangout` project, particularly
`app/src/challenges.ts` and its API types/client.

- Add a work/trials section to challenge details, with per-trial navigation,
  optional rendered summary, file browsing and revision/history access.
- Resolve relative links and images against the summary's directory at the
  viewed commit. Support linked audio and downloadable artifacts. Links to
  shared challenge input must work from nested trial summaries.
- Keep trials without summaries visible and usable. Do not manufacture
  summaries or outcome badges from their contents.
- Show standard, custom and unspecified challenge states honestly. Separate
  challenge management state from publication status and trial content.
- Provide a small recording flow for a trial name, optional summary and
  files, reusing draft/conflict feedback. Arbitrary structures remain
  accessible through the CLI or Git; a general file editor is out of scope.
- Keep both the attached historical revision and the latest revision
  accessible without silently replacing one with the other.

### 5. Update guidance and verify behavior

- Update the WFE challenge contract/help, agautolab's shared wfrun guide,
  relevant project guides and `devdocs/README_DEV.md`.
- Describe the storage location, available tools and optional summary
  convention. Suggested summary content may include what was tried, what
  was learned, artifacts and remaining questions; it is not a checklist.
- Test the storage/API behaviors where data could be lost or misrepresented:
  independent additions, same-file conflicts, duplicate retries, interrupted
  publication, direct Git writes, custom/absent state, nested files,
  historical reads and rebuilding from a repository clone.
- Verify the UI with a failed trial, an idea-only trial, a trial without a
  summary and a new trial on a resolved challenge. Verify relative image,
  audio and download links at both current and pinned revisions.
- Verify migration with pre-migration attachments and existing evidence.
  Run the affected projects' required checks and relevant browser checks.

### 6. Deploy and demonstrate

- At execution time, use `pj-clusterintent` Nautobot or `nctl` to inspect
  current service placement, observations and deployment revisions before
  changing running services. Do not treat old local environment notes as
  live service state.
- Back up the mutable stores and record deployed revisions before cutover.
  Follow the existing deployment mechanisms for WFE, aghangout and execution
  nodes; update worker pins when the CLI or shared guide requires it.
- Deploy and perform a small end-to-end check from the normal LAN entrance
  and another execution node. Record trial data as the checking agent,
  without claiming a real challenge has been solved.
- Verify repository and mirror publication and demonstrate recovery from
  Git. Record any journal/store migration and the corresponding code/data
  rollback procedure.
- Produce `report.md` with implemented behavior, checks, migration outcome,
  limitations and a concrete review path requiring few user actions.
  Put machine addresses, credentials and local deployment evidence in
  ignored local files, not tracked documentation.

## Completion criteria

1. Each current challenge's repository-held material is contained in one
   directory, and each trial's evidence is contained within that trial.
2. Free-form trials can be saved, found and read without a summary, outcome
   schema or dependency on workflow completion.
3. Standard summaries improve the UI; nonstandard records remain accessible
   and are not silently reinterpreted or normalized.
4. Publication, concurrency, historical references and Git recovery preserve
   content and attribution across the layout change.
5. The deployed LAN UI demonstrates the complete recording and reading flow,
   with documentation and verification evidence available for review.

## Execution policy

Follow `localrule.md`: generated documents are English; communication with
the Developer is Japanese. Routine implementation, commits, deployment and
verification do not introduce approval gates. Backward compatibility is not
a goal; preserve historical evidence and references deliberately. Keep
services accessible from the LAN and finish with a deployed, readily
reviewable result when this implementation plan is executed.

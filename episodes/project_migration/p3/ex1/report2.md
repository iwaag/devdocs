# project_migration p3 ex1 — step 2: forge's records and guidance

Written 2026-10-14.

## What was wrong

- `requests/provide-run-003.md` said **State: served**. provide/run-005 was partial: the WAV files were delivered in the right format, but acted "calm, gentle" speech was not.
- develop-capability/run-001's `verification.md` says "Listened to: all three are intelligible". Nobody listened. The only person's record is the Developer's "ok" to that run's review question. That accepted the presentation; it did not state a listening judgment. `catalog/speech.md` repeated the claim ("System TTS: intelligible") and so did the request's first `Later` line ("plain intelligible narration").

## How the corrections were published

No accepted run folder, decision or acceptance hash was touched. Two new changes:

1. **Guide facts** (pj-forge `e9ea373`, Omni Agent, pushed with the editor's Gitea credentials). Two facts were added to `agent/guide.md`:
   - A demand's state follows the actual outcome: `served` only when everything was delivered.
   - A listening or viewing judgment is a person's, cited only from an attributable record. Measurements and an "ok" acceptance are not one.

   Each fact names the run where it was seen. Agent guides are kept up by the developer side, and no forge workflow can edit `agent/`.
2. **Record corrections** were made by forge-agent itself in a managed run, `freework/run-001`. It was requested by the Omni Agent with `.local/project-migration-p3/ex1/requests/correct-speech-records.md` (ignored). The run works in its own copy, is saved on its run branch, and reaches `main` only through acceptance and integration. Its changes:
   - `requests/provide-run-003.md`:
     - **State: partly-served**: the files are delivered, and acted delivery is still unmet.
     - The original gap (no speech tool at all) and the remaining limitation (system TTS works, acted delivery does not) are kept apart.
     - Origin and both `Later` lines stay. The first `Later` line's "intelligible" became "system-TTS", with a pointer to the correction.
     - A dated `Later` line names this run.
   - `requests/README.md`: the state list gains `partly-served`, marked as added by this run.
   - `catalog/speech.md`: *Limits* now says only what is proven here (format and padding) and that intelligibility has no listening judgment on record. A new *Proof here* line points at the correction.
   - `devdocs/runs/freework/run-001/correction.md`:
     - quotes the claim;
     - says what ffprobe and silencedetect do establish;
     - says why the "ok" is not a listening verdict;
     - lists where else the claim appeared;
     - records that the rest of `catalog/` and `requests/` was searched (the agsfx entry's "judged by a listener" describes agsfx's own process and stays).

   develop-capability/run-001's folder is unchanged: the correction lives in the new run and is linked from the catalog and the demand record.

The run asked for review (q1, to the requester). The Omni Agent confirmed that the corrections match the request. **Accepting the result is the Developer's decision.** It is the run view's *Decision needed* path, in the editor released in step 3 (see report3). Until it is accepted and integrated, the corrections live on branch `wfe/forge/freework/run-001`, and `main` still has the old wording.

The run then completed: the final report was written before any decision, and the presentation was stopped and verified. Its `review` outcome says "Developer confirmed …". In fact the Omni Agent answered, as "Omni Agent (for Developer)", as the question recorded. That is a small misattribution in the agent's outcome text, noted here and not rewritten.

## Checks

- Read back the run's diff against `main` (`e9ea373`): only `requests/provide-run-003.md`, `requests/README.md`, `catalog/speech.md` and the run's own folder changed. No host details in the changed files. The review URL (host and port) does appear in `run.json`, in a node outcome and the question text. This is p3's known finding 6 ("keep the presentation URL out of question texts"), still open for forge's agent.
- The only record of the person on develop-capability/run-001 is the answer "ok" to q2 (and the later acceptance "ok"). No listening assessment is cited, because none exists.

## Deus Ex Machina note

The Omni Agent wrote the two guide facts, as with p3's workflow fixes. Correcting the records was forge-agent's work.

# workflow_editor p19 — message file attachments (final report)

Plan: [plan.md](plan.md). Step reports: [1](report1.md) trace and contract,
[2](report2.md) storage and submission, [3](report3.md) reads, delivery and
image access, [4](report4.md) aghangout by its agent, [5](report5.md)
archive, export and backup, [6](report6.md) invariants and transition.

## Outcome

A person can paste an image or attach any file to a message in aghangout,
with or without text. Each file is an occurrence of exactly that message,
stored once as original bytes on `agshare`, readable by every participating
project and by agents on other hosts, and checked against its recorded
SHA-256 whenever it is fetched or served.

**Try it:** `http://agstudio.local:8093/hangout/?conv=p19-check` — #542 the
pasted image (inline; the download link gives the original), #543 front's
and #545 forge's descriptions of it, #544 a text file and #546 forge quoting
its line 3. `?conv=p19-check-led` — the conductor describing two pasted
images (#548, #550) in one standing run. To try it yourself, open any room,
paste a screenshot into the composer (or 📎 / drag a file) and send.

## What changed, by revision

| Repository | Revision | |
| --- | --- | --- |
| agdevworld (WFE) | `119bf44` → `3fade95` → `47a83cc` (deployed; previous `799c039`) | contract `docs/message-files.md` + fixture; store v14; `server/messageFiles.ts`, `fileBackups.ts`; `post` split into room/route/accept with upload operations; reads, retrieval, briefs, CLI, archive manifests, export/restore/backup |
| agautolab | `a9b82f8` (studio and agpc) | one fact in the wfrun material reference |
| pj-aghangout | `90c6d05`, `a833273` (served `90c6d05`) | by aghangout-agent (freework/run-002; checked live in run-003) |
| pj-ponderer | `89fb781` | `tools/discussions` shows files/challenges and validates bundles, by ponderer-agent (freework/run-001) |
| pj-agdev | `88af6c2` | submodule pins |
| devdocs | this episode; README_DEV *Files on a person's message* | |

agpc worker pins WFE `47a83cc`, executor `a9b82f8` (nctl desired state,
playbook ok=22 changed=4, revisions and the restarted unit read back; drift
converged).

## Delegated work

| Run | Disposition | Limits |
| --- | --- | --- |
| aghangout/freework/run-002 | accepted after Omni's review of diff, reports and the served page | self-completed its human-review node; finished before the live check it promised |
| aghangout/freework/run-003 | accepted: live claims match WFE's records; no correction needed | its check messages are recorded under the configured person (annotated); no OS-native paste/drag; limits and failure paths live untested |
| ponderer/freework/run-001 | accepted: `show` lists files, all 99 archives validate, a tampered manifest is reported | — |
| forge/freework/run-005 (check, agpc) | its description was correct; cancelled by Omni as a finished check | answered without performing its nodes; stored its fetched copy in the tracked run folder |

## Tests and checks that actually ran

- agdevworld `npm run check` at the release: 651/651 (incl. 10 new
  message-file tests); agautolab 51 passed, 1 skipped; aghangout 38/38 and
  browser checks (agent's run, both engines).
- Rehearsal on a copy of live data: v13 → v14 with every existing count
  unchanged; all 20 workspaces' runs replay `usable` with the new code.
- Deployed: four real agents (front, forge, ponderer's conductor on studio;
  forge on agpc) each described a pasted image whose features were chosen at
  random and kept out of names and text — all correct; a text file's random
  code quoted exactly; agpc fetched identical bytes through its mount, without
  a mount via the coordinator, and over HTTP; agpc's Chrome showed the image
  inline and downloaded originals.
- nginx passes the 90.5 MB maximal body (WFE answers its own 413s); one 64 MiB
  message raised the service's RSS by 540 MiB for 311 ms (host has 512 GiB).
- Archive in a fresh clone holds both bundle manifests identical to the
  share's; export → restore verified 5 files; backup 3/3 verified on agpc,
  and one bundle restored from the backup verified.

## Backup and recovery boundary

- Payload backup: accepted bundles are copied to agpc's local disk and their
  hashes read back (every reconcile; `wfe conversation backup --status`). Only
  message-file bundles are covered. Failure domain and host details are in the
  ignored `pj-agdev/.local/p19/notes.md`.
- Records restore from the archive clone; bytes from the backup or an export
  package; neither replays delivery. A clone alone has no bytes.
- Code rollback is not enough: `799c039` refuses store v14 and cannot replay
  run records that carry file attachments (first live: #542, front/discuss/run-014).
  The data-aware procedure, the pre-deploy backup (v13, newest #541) and what
  must be exported first are in `pj-agdev/.local/p19/notes.md`.

## Open limits

- Bodies are buffered JSON/base64 (no streaming); fine on this host, measured.
- Crash recovery was exercised by injected failures in tests, not by killing
  the deployed service.
- aghangout: no OS-native clipboard/Finder events tested; live limits and a
  live failed-send retry untested.
- Agents' handling of nodes (answers without performing nodes, a fetched copy
  placed in a tracked folder) was observed, recorded, and not changed in this
  episode.

## Where things are

- Contract and help: agdevworld `workflow_editor/docs/message-files.md`,
  `wfe conversation help` (*Files*); README_DEV *Files on a person's message*.
- Local evidence (ignored): `pj-agdev/.local/p19/` — notes (baseline,
  deployment, failure domain, recovery), build/deploy logs, proxy and memory
  measurements, agpc logs, screenshots, check scripts, export/restore folders;
  `pj-agdev/.local/devenv.md` *workflow_editor p19 on this host*.

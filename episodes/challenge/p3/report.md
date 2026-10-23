# Challenge p3 — final report: records of work

**Each challenge is now one directory, holding its statement, source
material and the work done on it.** People and agents can record an idea, a
failed experiment, a partial or a better solution as a **trial**
(`challenges/<id>/wedo/<name>/`) in any shape, with no outcome schema, and
no state ever changes as a result. The standard `summary.md` is optional.
When a summary is present, aghangout renders it; when it is not, aghangout
still shows the files.

## Review it (from any LAN computer)

1. Open `http://agstudio.local:8093/hangout/?challenge=ch-0003`. This is a
   fixture made by the deployed check. It shows source `voice.wav` and
   three trials:
   - `plain-stretch-failed`: a failed attempt with an image, a take and a
     link to the input. It was published from agpc.
   - `idea-separate-speech`: an idea only.
   - `files-only`: files with no summary.
2. Open `?challenge=ch-0003&at=a7fc0db`. This is an older
   revision with only the idea, and the page names what the latest revision
   adds.
3. Open `?challenge=ch-0002`. It is resolved and still takes trials:
   「＋ 取り組みを記録」 records one, and the state stays resolved.
4. Open `?challenge=ch-0001`. It is the real challenge after the move. Its
   evidence now lives at `challenges/ch-0001/evidence/`.

From an agent: `wfe challenge help`, `wfe challenge show ch-0003`, `wfe
challenge publish <id> --trial <name> <dir>`.

## Releases

| Component | Release |
| --- | --- |
| WFE | agdevworld `a252289` (code `73eb720`, migration script `d71e6f6`; challenge journal v2) |
| Repository layout | pj-challenges `3a8f4f9` (one migration commit) |
| aghangout | `a5088c5` |
| nginx (`/wfe/` body 100m) | agdevworld `a252289` |
| Guides | agautolab `1b65813`, pj-front `4784f9a`, gimmickimg context r3 |
| agpc worker | WFE `a2522894`, executor `1b65813` (nctl) |
| pj-agdev | `8be7bc1` |

## Steps

| Step | Report | Outcome |
| --- | --- | --- |
| 1 Reading and revisions | [report1](report1.md) | Discovery by directory, with or without a summary; custom and unspecified states; a revision covers every file, including direct Git commits; pinned reads are that commit's tree; the old layout still reads at old commits; the index includes trials; search covers trial summaries but not files |
| 2 Publication, CLI, HTTP | [report2](report2.md) | `publish`/`files`/`file`/`history --trial`; one commit per write through the journal; conflicts per path; path and symlink safeguards; limits and the 413; files served as untrusted content; on execution nodes, files stay on the node |
| 3 Migration | [report3](report3.md) | Inspected (ch-0001's evidence exposed the problem, so it stays at challenge scope and no trial was invented), scripted, rehearsed: 4/4 attachments and journal v2 |
| 4 aghangout | [report4](report4.md) | Rendered summaries, 資料, 取り組み, honest states, recording flow, pinned vs latest; Chromium and WebKit 324/324 |
| 5 Guidance and verification | [report5](report5.md) | Contract and help, wfrun guide, Front guide, context block, README_DEV; behaviour matrix |
| 6 Deployment | [report6](report6.md) | Cutover, live migration, checks 25/25 (studio) and 21/21 (agpc), recovery from the mirror identical to live |

## Completion criteria

| Criterion | Status |
| --- | --- |
| Each challenge's material is in one directory; each trial's evidence is in the trial | Met. `3a8f4f9`: nothing remains outside `challenges/<id>/`. The existing evidence is problem material, not trial material (report3). |
| Free-form trials can be saved, found and read without a summary, schema or workflow | Met: `files-only` and the page-recorded trial; search names matching trials. Trials have no run identity. |
| Standard summaries improve the UI; nonstandard records stay accessible and are not reinterpreted or normalized | Met: custom and unspecified states, unreadable YAML shown as written, field-only edits (tests and browser checks). |
| Publication, concurrency, historical references and Git recovery keep content and attribution across the change | Met: per-path conflicts, duplicates, interruptions (tests); pins verify (4/4); history spans the move; mirror recovery is identical. |
| The deployed LAN UI shows the whole flow, with documentation and evidence | Met: report6, README_DEV, the contract. |

## Known limits

- The page's recording form sends flat file names and forgets chosen files
  on reload; `wfe challenge publish` keeps directory trees. There is no
  general file editor.
- Files are never searched or interpreted. Only summaries are.
- Published journal rows from before p3 show their `paths` in the new
  layout. This is display only.
- Rolling back needs the v1 journal backup together with the old release
  (report6). Prefer forward repair.
- The check fixtures remain visible: `ch-0003` (open) and the two
  `p3-check`/`page-check` trials on `ch-0002`, all labelled as fixtures by
  "Omni Agent (challenge p3 check)".

Machine addresses, backups and commands are in the ignored
`pj-agdev/.local/challenge-p3/notes.md` and `pj-agdev/.local/devenv.md`.

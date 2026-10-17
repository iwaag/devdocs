# project_migration p6 — Step 4 report: portable input and external artifact references

Status: complete (2026-10-17). WFE can name shared folders in a request,
record exactly which files a run used, publish large outputs outside Git
without ever showing a partial copy, and bind a confirmation to exactly the
published bytes. The change is deployed and was exercised by a real agent
in a private area. The cross-host check ran against fixtures because the
Linux client's NFS mount was hung again; see "Limits". Raw evidence is in
the ignored `pj-agdev/.local/project-migration-p6/step4/`.

## Delivered

| Item | Where |
| --- | --- |
| Contract | agdevworld `workflow_editor/docs/shares.md` |
| Code | agdevworld `53efa4a` (shares), `45ffada` (fixes from the live trial) |
| Agent guide | agautolab `9104ca9`: wfrun guide, "Shared data" |
| Shared docs | `devdocs/README_DEV.md`, "Shared areas" |
| Deployed release | agdevworld `45ffada` (previous `c6390a2`; `53efa4a` was live for about 20 minutes in between). `wfe-release.sh deploy` ran `npm run check`: 376/376 |

## The contract in brief

1. **Reference.** `shared://<share>/<path>/` names a folder, and the same
   form without the trailing slash names a file. References are taken from
   the words of the input (braindump, request, Zulip post, delegation) and
   recorded in `input.references`. There is no upload form. Tracked records
   hold no host path.
2. **Per-host resolution.** Each host maps the logical share name to its own
   mount path in `<area>/shares.json` (ignored). A marker file at the share's
   root names the share, so an automount that did not happen reads as
   `unmounted`, not as an empty folder. The live area maps `agshare` to the
   server's backing folder, and the marker is in place.
3. **Input set** (`wfe run input`). The files a reference holds are recorded
   with path, size and sha256 in `inputs/i<n>.json` (in Git). By default they
   are staged into the run copy's `.local/inputs/i<n>/`: each file is hashed
   as it is copied, then hashed again on disk. A folder that changes while it
   is read is refused. A later check records `verified`, `changed` or
   `missing`. Every continued attempt re-checks its input sets (sizes) before
   its brief and records what it found, so a resume never silently uses a
   different set.
4. **External output** (`wfe run publish`). The files are copied to
   `shared://<share>/runs/<project>/<wf>/<run>/o<n>/` through a `.partial-*`
   folder, each file verified. The manifest is written inside, then the
   folder is renamed into place. A copy of the manifest goes into Git as
   `outputs/o<n>.json`. Copying happens outside the run's lock, so concurrent
   candidates do not block each other or the person's answers. A retry adopts
   a finished but unrecorded folder with the same content. A folder with
   other content is never touched; the next id is used.
5. **Confirmation** (`wfe run ask … --output <o>`). Every byte of the output
   is re-read against its manifest before the question is recorded. The
   manifest becomes a hashed subject, and the question keeps `{id, digest}`.
   `run.decide accepted` is refused if the output went missing, changed, or
   was superseded by a later output of the same candidate. Other candidates
   coexist.
6. **Review and download.** The run view has a **Shared data** section: what
   the request named, input sets and outputs, with this host's reach (a stat,
   never a hash). It has a Verify button for a full check and an "Open the
   files" link. The question card names the output it confirms.
   `GET api/shares/<share>/<path>` streams files with byte ranges and lists
   folders. Dot entries are never served, and the project file API is
   unchanged. `wfe share list|resolve|ls|verify` work without a run;
   `verify <manifest.json>` is the cross-host delivery check.

## Tests (plan item 8)

`test/shares.test.ts`, 12 tests, all in the release gate:

| Boundary | Result |
| --- | --- |
| Same logical reference, different physical mount paths | A manifest from one host verifies on another host whose shares.json maps a different path. A tampered manifest fails its own digest |
| Missing mount | No marker → `unmounted`; selection refused; nothing recorded. An unconfigured share → `unknown-share` |
| Changed input | A staged copy is immune. In place: a same-size content change is caught by a full check (a quick check misses it, as designed) and recorded as `changed`; a deleted file → `missing` |
| Interrupted publication | Leftover `.partial-*` folders are not content and are not served. A finished, unrecorded folder is adopted; one with altered files is refused (`output-altered`) |
| Concurrent candidates | Two parallel publishes get distinct ids and digests. A foreign `o1` folder was skipped, not overwritten |
| External-result revisions | Ask refused while the output was altered, allowed after repair. Accept refused while missing. A later output of the same candidate → `output-superseded`; another candidate does not supersede |
| Ordinary project artifacts | `artifact.attach` and loose-file confirmation are unchanged; no `outputs` appear |
| Download | 200 with full length, 206 with `content-range`, folder listing; markers, `.partial-*` and absent paths → 404 |
| CLI | `share list/resolve/verify`, `run inputs --verify-all --quick` |

## Live verification against the deployed service

| Path | Evidence |
| --- | --- |
| LAN download through the production service (port 8093) | Folder listing, a 16-byte range, and a full 64 MiB file whose sha256 matched the source. The marker and a `..` path returned 404 |
| Real agent (Sonnet profile) in a private scratch area, request naming a folder with two files (65.9 MiB) | The agent discovered `wfe run input`/`wfe share` from the brief and help, selected `i1` (staged), built the bundle under `.local/`, published `o1` (candidate `scene-v1`), registered a small presentation as a preview, and asked q1 with `--output o1` together with `--presentation p1`. After the answer, it recorded the decision: "confirming output o1 (c52b1c825cdb) and presentation p1" |
| Resume | The continued attempt recorded `input.observe` by the executor before its work |
| Git | The run commit held only manifests and records (pack size 0). The published hashes equalled the source inputs |
| Integrity on the real share | `wfe share verify` passed for both manifests. Appending one byte to a published file gave `changed — size changed`; after restoring it, a full re-check gave `verified` |
| UI (WebKit) | The Shared data section rendered with reach, last check, Verify and the download link |
| Cost | Two attempts: $0.81 + $0.23 = $1.04 |

The trial area, the share probes and the published trial output are gone.
The run records stay in the ignored evidence folder. Only the share marker
remains on the share.

### Defects found in the trial and fixed (45ffada)

- `wfe share verify … --quick` and `wfe run inputs --verify-all` were
  parsed as options that need a value.
- The download URL overflowed the side panel.
- The question card named the presentation but not the output it confirms.

## Implementation notes

- I took over uncommitted Step 4 work that had been idle since 2026-10-16
  (from before ex1 interrupted Step 4) and rebased it. I changed its design
  in two places. First, it hashed and copied multi-GB payloads while holding
  the run lock. Second, a retry facing an unrecorded folder with different
  content would have been refused forever.
- Not built: Google Drive, a storage-provider framework, retention/garbage
  collection, or a separate download server. `publicBase` in shares.json
  allows a separate download server later without a record change.

## Limits

1. **The Linux client's NFS mount was hung again at about 12:40 JST**
   (`findmnt` showed the mount; its TCP connection to the server sat in
   `FIN-WAIT-2` on an unprivileged port). This is Step 3's open limit 1:
   run-006's `noresvport` moved the connection off reserved port 850 but
   did not prevent the hang after the server's idle drop. `nctl drift`
   (observation 0.4 h old) still reported `agshare-client` converged. I did
   not repair it: it belongs to cagent's change path. Because of the hang,
   the cross-host test in this step ran against two fixture mounts, not the
   live client. Step 7's cross-host trial needs it fixed first.
2. A quick check (sizes) cannot see a same-size content change. Views and
   resume use quick checks to stay bounded; asking about an output and the
   Verify buttons read every byte.
3. Downloads go through the WFE service. That is adequate for hundreds of
   MB on the LAN. A dedicated file server is a `publicBase` away if needed.
4. The executor host that publishes must reach the share. A run moved to a
   host without the share sees `unmounted` and records it; it does not fall
   back to anything.

## Acceptance check

- **A request can name a folder:** yes, in prose; recorded from the words.
- **The run records exactly what it used:** yes, as an input set with
  per-file hashes and a digest, a staged copy, and resume-time checks.
- **A reviewer can reach and identify the delivered bytes without a heavy
  editor or a Git payload copy:** yes. The run view links the output on the
  share. The manifest (digest) is the confirmation subject. The decision
  names the output and its digest. Git holds only the manifest.

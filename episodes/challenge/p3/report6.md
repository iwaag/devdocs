# Challenge p3 — Step 6: deployment and demonstration

## Before

I inspected live state with nctl and the service; old local notes were not
taken as the current state:

- WFE served `21a094e`, with the challenge journal at v1 (3 operations, all
  published);
- aghangout served `9604695`;
- the agpc worker pinned WFE `21a094e` and executor `ba9e2c1`;
- the repository head was `6b72a13`, in the old layout;
- nctl status was ok, with drift converged=46 and drifting=2 (pre-existing
  inactive placements);
- no attempt was open.

## Cutover (2026-10-23, 21:32–21:40 JST)

1. I backed up the journal (v1) and recorded the repository head. The
   backup path is in the ignored notes.
2. I pushed agdevworld to GitHub and activated WFE `a252289` (built and
   checked: 538/538). Service and launcher are healthy; the journal is now
   v2.
3. I migrated the repository: commit `3a8f4f9`, pushed fast-forward to
   Gitea. The move plan is identical to the rehearsal. After `retry`, the
   index and the agpc mirror are at `3a8f4f9`.
4. I verified it live: all 4 existing attachments read in the old layout at
   `8176fd6`, with the same sha256. ch-0001's history spans the move, its
   creator is unchanged, and its links resolve.
5. aghangout `a5088c5` is deployed.
6. nginx was rebuilt with `client_max_body_size 100m` on `/wfe/`.
7. The guides went live: agautolab `1b65813` (pushed), pj-front `4784f9a`,
   and context block r3.
8. agpc was re-pinned through nctl, with WFE `a2522894` and executor
   `1b65813`. The playbook ended with ok=22, changed=4, and agpc's source is
   at `a252289`.
9. nctl was observed again: converged=46, drifting=2 (agobserver,
   cagent-snake-e2e). Both drifts were there before; none is new.

pj-agdev `8be7bc1` records the agdevworld and agautolab revisions.

## Demonstration (check actor "Omni Agent (challenge p3 check)")

No real challenge was claimed as solved. The check data is fixtures:

- `ch-0003`: "(fixture) challenge p3 deployed check: slow the speech in
  voice.wav to 0.8x…", with source `voice.wav`. Its trials:
  - `idea-separate-speech`: an idea only, from studio;
  - `files-only`: a nested `raw/take-a.wav` with no summary, from studio;
  - `plain-stretch-failed`: a summary, `take1.wav` and a spectrogram,
    linking `../../voice.wav`. It was **published from agpc** with files on
    agpc's disk, and read back there with `--out`, which wrote on agpc with
    an equal sha256.
- `ch-0002` (resolved fixture): one trial from the CLI and one recorded
  **through the page**. It stays resolved.

Deployed UI check (`.local/challenge-p3/step6/deployed.mjs`):

- **studio, full: 25/25.** It covers:
  - every trial listed, including the one without a summary;
  - source audio playable;
  - the failed trial rendered, with its image loading at the viewed
    revision;
  - the take served as `audio/wav`;
  - `../../voice.wav` resolving to the challenge's file;
  - downloads served as attachments under a sandbox CSP;
  - the trial's own history;
  - the idea-only trial and the files-only trial;
  - the revision `a7fc0db`, showing only its own trial, naming the later
    one, with links pinned to it;
  - ch-0001's evidence image loading at `8176fd6` (old layout) and now;
  - trial search;
  - the resolved fixture stays findable;
  - recording on the resolved challenge through the page, after which it
    stays resolved;
  - phone width;
  - no page errors.
- **From agpc (another LAN computer, Chrome), read-only: 21/21.**

## Recovery from Git

I cloned the **agpc backup mirror** into a new bare repository, pointed an
empty area at it, and synced. The reindexed challenges, states, revisions,
trials and files are **identical to the live service**, and the old pin
`ch-0001@8176fd6` has the same hash. The mirror and Gitea are at the same
head (`82201d5`). See `step6/recovery.log`.

## Migration of store and data, and rollback

- **Journal** `challenges.sqlite` v1 → v2: one column was added to
  `operations`, and the derived index was rebuilt. No open operation
  existed.
- **Repository**: one commit, `3a8f4f9`.
- **Rollback**:
  - Code alone cannot be rolled back, because `21a094e` refuses the v2
    journal.
  - A full rollback activates `21a094e`, copies the v1 backup over the
    journal (after exporting operations newer than #3), and reverts
    `3a8f4f9` in a clone, then pushes. Trials written since are kept in the
    history.
  - aghangout rolls back with `scripts/deploy.sh activate 9604695…`.
  - The worker pins go back through the same nctl batch.
  - Prefer forward repair.

  Commands are in the ignored `pj-agdev/.local/challenge-p3/notes.md` and
  `.local/devenv.md`.

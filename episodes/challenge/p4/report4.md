# Challenge p4 — step 4: deployment and demonstration

## Before (inspected live, 2026-10-23 22:45 JST)

- nctl status was ok. Drift: converged=46, drifting=2 (agobserver and
  cagent-snake-e2e, both pre-existing). aghangout's service placement was
  converged.
- WFE served `a2522894` (`wfe-release.sh status`), with no open attempts.
- aghangout served `4175292`.
- The agpc worker pinned WFE `a2522894` and executor `1b65813`, and both
  workers were available.

## Cutover

1. **WFE** `61e547a` (`wfe-release.sh deploy`): built, checked and healthy.
   The service and `bin/wfe` name the commit. There is no store change, so
   no backup or migration was needed.
2. **aghangout** `5e92078` (`scripts/deploy.sh`): `/hangout/` answers 200.
3. **ponderer guide** `380442b`, pushed to Gitea. Run copies take it from
   the workspace.
4. **agpc worker re-pinned through nctl**:
   - batch: `wfe_version` set to `61e547a9…`, executor unchanged;
   - applied with `nctl desired apply --yes` (update 1);
   - `render production`, then the `setup_wfe_worker` playbook: ok=22,
     changed=3;
   - on agpc the unit is active, and the source checkout is at `61e547a`.
5. **pj-agdev** `25314c7` records agdevworld `61e547a`.
6. **nctl again**: agpc and agstudio observed. Drift is still
   converged=46, drifting=2 (the same two).

## Demonstration

The check actor is "Omni Agent (challenge p4 check)". Both wishes say that
they are an Omni Agent operational check. Neither mentions a title.

**From studio (Chromium, `deployed.mjs`): 24/24.**

- **Room A** (`20261023-225438-sjbp91`): a new room with ponderer leading.
  - There is no title field, and focus is in the composer.
  - The wish was sent: 「家族で週末に遊べる、30分くらいで終わる短いボードゲームの案を一緒に考えたい」
    plus a second line.
  - The original title is that first line. It was readable **3.5 s** after
    Send, before anyone answered. The message is stored as written.
  - ponderer's **normal first turn** (`ponderer/conduct/run-023`, settled 46
    s after Send) replied and published its overview with
    `--title "家族で遊ぶ30分の短いボードゲーム案" --if-untitled`. WFE
    answered `display title "家族で遊ぶ30分の短いボードゲーム案"`.
  - The same display title appears in the room's heading, the aghangout
    list and **WFE Home**.
  - The room was created as an ordinary conversation so that Home, which
    lists only those, could show it. It was then recategorized `check`, with
    the reason recorded.
- **Room B** (`20261023-225540-6hcyrl`, created as a check): the person
  titled it before ponderer answered.
  - In 詳細 the person set 「人が先に付けた題（p4確認）」, with a reason,
    before any agent line.
  - ponderer's first turn (run-024) called
    `overview … --title "雨の日に子どもと家でできる遊び" --if-untitled`.
    WFE recorded the overview and answered: *Display title left as it is:
    Omni Agent (challenge p4 check) titled it at 2026-10-23T13:55:43.918Z
    ("人が先に付けた題（p4確認）").*
  - The only display-title record is the person's.
- No page errors.

**Archive**: both rooms were closed and archived (A `d1a22a0568d8`, B
`cf72ef1f7cb4`). In pj-ponderer, each `conversations/<id>/manifest.json`
has the same `displayTitle`. The transcript's `meta` records show who set
it: `ponderer-agent` with `ponderer/conduct/run-023` for A, and the check
actor for B.

**From agpc** (another LAN PC, Chrome, read-only): **5/5**.

- Room A shows ponderer's title.
- Room B shows the person's title.
- The 動作確認 list shows A by its title.
- A new room there asks for no title (nothing was sent).

Evidence is kept in ignored local files: scripts, logs, screenshots and
ponderer's commands with their outputs extracted from the attempt logs.

## Recovery

- **WFE**: `wfe-release.sh activate <area> a2522894…` (or `rollback`). It is
  safe both ways, because the store is unchanged. The older release ignores
  `ifUntitled`, so it would write such a title unconditionally.
- **aghangout**: `scripts/deploy.sh activate 41752925…`. The old page would
  ask for a title again. Outbox entries made by the new page carry their
  title and still deliver.
- **ponderer**: revert `380442b` in pj-ponderer.
- **agpc**: put the old `wfe_version` back in the batch, then run the same
  `desired apply`, `render production` and playbook.

The exact commands are in the ignored notes (`pj-agdev/.local/challenge-p4/notes.md`).

## Observations

- ponderer's first `overview` call in run-023 cited a message newer than
  its `--through` and was refused. The second call succeeded.
- In run-024 ponderer first tried a nonexistent `--text` option, then read
  the help and used `--purpose/--outcome/--unresolved`.

These are ordinary CLI refusals with clear messages. In both rooms the
title rule worked as written.

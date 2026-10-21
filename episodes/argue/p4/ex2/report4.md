# argue p4 ex2 — Step 4: deployed and verified from another LAN machine

## State before deployment (Nautobot/nctl)

- I refreshed the observation (`nctl observe agstudio`, success). `nctl
  drift --host agstudio` reported converged, both before and after.
- No service placement changed:
  - discussions run inside WFE (`agdevworld-web`);
  - aghangout is served from its release directory (`aghangout_web`, an
    HTTP check of `/hangout/`, still 200).

  Nautobot needed no update.
- Active work: none. There were no open attempts, and jobs were 573 done
  and 2 dropped, both before the Step 2 deploy and before this one.

## Deployed revisions

| Component | Revision | How |
| --- | --- | --- |
| WFE | agdevworld `8485dcd` (Step 2), then `5d6f511` (from `153bdad`) | `scripts/wfe-release.sh deploy`: the full check (488/488), then a service restart that reported healthy; store **v9** |
| aghangout | `c66bd02` (from `6210d30`) | `scripts/deploy.sh c66bd02`: unit tests in the clean export; `RELEASE.json` names the commit |
| pj-agdev pin | `2cd0f47` (agdevworld `5d6f511`) | pushed |
| pj-ponderer | `612a7b6` (guides) | Gitea main |
| devdocs | `README_DEV.md` *Reading discussions*, the *Discussions* client line | this commit |

## Backup and recovery

- **Backup**: `.local/agdev/.local/backups/argue-p4-ex2-predeploy-20261021-151205/`,
  taken before the v9 migration. It holds:
  - the conversation store at v8 and the exec store;
  - `conversations.json` and `registry.json`;
  - the archive writer's clone;
  - a pj-ponderer workspace mirror.
- **Rollback**: written in `pj-agdev/.local/argue-p4-ex2/notes.md`. Code
  and data must go back together, because `153bdad` refuses store v9.
  - **WFE**: re-activate `153bdad` and restore the v8 store copy. This
    loses #248–#249 and every overview and meta record written since.
  - **aghangout**: `scripts/deploy.sh activate 6210d30…`.
  - **The archive**: the records added in pj-ponderer are additive and
    need no undo.

## Bounded check through normal agent work

The check discussion is `argue-p4-ex2-check`, category `check`, created
with `--category check`.

- **Brief (#248)**: forge says in one line which kind of media it has been
  asked for most, and what it counted; ponderer asks, then concludes
  briefly. The brief does **not** mention the overview.
- **Run**: `ponderer/conduct/run-009`, 06:45:47–06:47:37Z.
  - It decided (activity 45), asked forge (activity 46, invitation 72),
    and forge answered #249: "Images, by a clear margin… 7 images, 3
    audio, 2 text", with the runs counted.
  - **On its own, it published an overview** at 06:46:55Z, by
    `ponderer-agent`, run `ponderer/conduct/run-009`, through #249 and
    citing #248 and #249, in Japanese:

    > 目的: [#248] forgeに、これまで最も多く依頼されたメディアの種類を記録から答えてもらう小さな確認。
    > 結論: [#249] forgeの記録では画像が最多（provide依頼12件中、画像7・音声3・テキスト2。…）

  - Its final decision (activity 47) ends "concluding".
- **Deviation**: no conclusion was **announced** in the discussion. The
  conclusion exists only in the decision record and the overview. I
  recorded this as annotation 3 (an evaluation, outside the speech) and
  did not correct it.
- The discussion is listed only under 動作確認. I closed it after the LAN
  check, and its archive is current (`8909b8b`).

## LAN evidence (agpc)

`.local/argue-p4-ex2/step4/lan-check.mjs` ran on agpc through Ansible as
`~/p15-check/lan-check-ex2.mjs` with Chrome. **36/36 passed**
(`step4/agpc-lan-check.txt`).

| Area | Result |
| --- | --- |
| Contract | counts per view `discussion {2 open, 4 closed}`, `check {2 open, 5 closed}` before closing the check; 6 + 7 = 13, so every discussion is in exactly one view; paging (`limit=2&offset=4`) works; every ordinary row has an overview citing messages and a display title; the check discussion's overview came from ponderer's run and was current |
| Discovery | the 議論 list shows the 6 ordinary rows by default, the two views show their counts, and no artwork is behind the list; every row says 結論; the three RTS×Jev experiments are told apart (1回目・実行委員会方式 / 2回目・Omniの助言つき / 3回目・ponderer進行); 終了 shows the 4 closed rows; the check is under 動作確認 |
| Direct link and sources | `?discussion=p16-s4-provenance` opens on its display title with 目的, 結論, 未解決 and 次に必要なこと; `#142` focuses forge's English original |
| Metadata after reload | the display title set in 詳細 on the check discussion persisted after a reload; messages, invitations and state were unchanged; the change is recorded with its reason |
| Still usable | an open discussion (`p16-s3-room`) shows all its speech, To: and the composer; 再生表示 still seats the members; a personal conversation with Front opens with its composer; WFE Home lists ordinary discussions by display title, with the checks counted apart; no page errors |
| Narrow (390 px) | the list and a room have no sideways scroll |

No message was posted to a real discussion from agpc. No person's reply
was pending anywhere, so no pending reply needed checking.

## Reading tasks: before and after

The Developer's screenshot is not stored with the episode. The reference
is my capture of the same deployed screen before the change
([before](shots/before-desktop-lobby.png), the 12-card grid). Task subject:
`argue-p4-trial`.

| Task | Before (6210d30) | After (c66bd02, from agpc) |
| --- | --- | --- |
| Identify the topic | card title cut at one line: 「rtsでjevを取り入れたAI同士の対戦実験（ar…」. It is indistinguishable from the two other cards with the same prefix | 「RTSにJevを取り入れたAI対戦実験（3回目・ponderer進行）」, in full in the list and as the room's heading |
| Its purpose | not shown. The card showed "Ponderer: **Conclusion: one match was played. The…" (the last message) | 目的: 「同じ目標の実験を、pondererの進行で行い、議論の進め方そのものを試す…」 `#234` |
| Its outcome | only by reading the last message or stepping through the player (the room opened at line 1 for a new reader) | 結論: 「1試合が実行され、標準ボット（席0）がJev駆動の側（席1）に勝った。1試合だけで強さについては何も言えず…」 `#238 #239 #240`, in the list (2 lines) and at the top of the room |
| What remains unresolved | not shown anywhere as such | 未解決: the review the brief asked for was not held; the "nothing owed" claim conflicts with the evaluator's note; further comparison is the Developer's call |
| Where to reply | the composer, below the player | the composer, unchanged, below the overview and the conversation. For a question to the person, 返信する in 次に必要なこと names it |

The agpc check read all five answers **with no panel opened**
(`with-panel` count 0) and **without guessing truncated text**:
`innerText` of the heading and the 目的, 結論 and 未解決 entries.

## Review links (from any LAN machine)

- The list: <http://agstudio.local:8093/hangout/>
- Checks: <http://agstudio.local:8093/hangout/?list=check>
- An entry with outcome and unresolved points:
  <http://agstudio.local:8093/hangout/?discussion=argue-p4-trial>
- Sources in another language:
  <http://agstudio.local:8093/hangout/?discussion=p16-s4-provenance>
  (click `#142`)
- The bounded check (overview by ponderer):
  <http://agstudio.local:8093/hangout/?discussion=argue-p4-ex2-check>
- WFE Home: <http://agstudio.local:8093/wfe/>
- Screenshots:
  - [deployed list](shots/after-deployed-desktop-list.png);
  - [checks](shots/after-deployed-desktop-checks.png);
  - [entry](shots/after-deployed-desktop-entry.png);
  - [narrow list](shots/after-deployed-narrow-list.png);
  - [narrow entry](shots/after-deployed-narrow-entry.png);
  - [agpc entry](shots/after-agpc-entry-p4-trial.png).

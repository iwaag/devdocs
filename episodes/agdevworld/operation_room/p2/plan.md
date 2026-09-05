# operation_room p2 実装計画 — 会話/タスク層ボード

p1の調査結果(p1/report.md)に基づき、会話/タスク層の状態ボードを完成させる。
この層だけが完全かつ正直に観測可能であり、stalled検知がこの画面の価値の中心。

スコープ外(p3以降): プロセス/バックエンド層のタイル(listener、ComfyUI、
ollama、launchd)、routine発火の応答確認、prompt_id結合。

破壊的フェーズ・非公開実験環境。制約は末尾の最小限のみ、他は実装者裁量。

## 全体像

```
agents の intro(ロスター) ─┐
Zulip 初回スイープ ─────────┼→ relay(agentroom)内の状態エンジン → /ops JSON
Zulip event queue(増分) ──┘            ↓
                           agdevworld operation room ビュー
```

状態は p1/report.md §2 の定義に従う:
`awaiting` / `acked` / `stalled`(閾値既定15分、設定可能に) / `done`(✔)、
加えて表示上の `unknown`。

## Step 1: intro契約の拡張(pyagag側・ENT相当の先行作業)

観測者が必要とするロスター(インスタンス名、巡回チャンネルとprefix)を
`#agents` の `intro-<instance>` に機械可読な形で含める。

- 観測側にロスターをハードコードしない。p1で推測ロスターが幻のstallを
  計66件出した(report §3 #9)。introは既に「契約」であり、挙動変更時の
  再投稿運用も確立しているので、鮮度維持はその運用に乗る。
- 実装場所: `pyagag/src/agag/intro.py`(intro生成)。各agentの巡回対象は
  `AgentSpec` にあるので、introへ自動転記できるはず。
- 形式は実装者裁量(fenced blockでも key: value 行でも)。ただし
  「人間向け紹介文の可読性を壊さない」「観測者がパースできる」の両立と、
  形式自体の文書化(introの中か devpolicy)だけは満たすこと。
- 全インスタンス(6体)にintroを再投稿させて完了。再投稿は各agentの
  既存機構で行い、observerや人手でintroを代筆しない。

## Step 2: 観測専用bot

- タスク層の全量スイープは183コールでHTTP 429に到達し(report §3 #10)、
  しかもagentと同一クォータを消費する。クォータ分離のため専用bot
  (例: opsroom-bot)を新設する。
- 作成はprovisioner credential(`AGAG_ZULIP_ADMIN_ENV` のパス)経由か
  Zulip管理UIか、どちらでも。credentialは `pj-agdev/.local/zulip/` の
  既存規約(0600のenvファイル)に合わせて置く。
- このbotは**読み取り専用の運用**とする。botの投稿はagentの有償runを誘発し
  会話を誤誘導する(既知の事故パターン)。例外は検証用に作る、
  どのagentも監視しない専用テストチャンネルへの投稿のみ。

## Step 3: relayの状態エンジン(`agdevworld/agentroom/`)

p1の使い捨てプローブ(`.local/opsprobe/`)のロジックを本実装に昇格させる。
別サービスは立てず、agent_roomのrelayに `/ops` エンドポイントを足す。

- **起動時**: introからロスターを読み、全量スイープで状態を初期化。
- **定常時**: Zulipのevent queue(`register` → `events` long-poll)で増分更新。
  新規メッセージとトピック変更(✔リネーム含む)を反映する。
  定常時に全量スイープを回さない。再スイープはキュー失効時の再同期のみ。
- 状態計算の必須部品:
  - `agag.selfnote` のパーサ(`parse_served`, `without_selfnotes`,
    `last_real_speaker`)。selfnoteを発話者に数えない不変条件を守る。
  - `RESOLVED_TOPIC_PREFIX`(`agag/zulip.py:88`)。トピックのキーは
    ✔を剥いだbare topicで持ち、リネーム前後を同一視する(p9事故の教訓)。
  - awaitingの2経路(owner route / mention route)はp1のプローブが
    実証済みの判定をそのまま使う(詳細は p1/report2.md 以降を参照)。
- `/ops` のJSONには各行の**provenance**を含める: 状態の根拠
  (最終実投稿のmessage idと時刻、対応するservedノートの有無、
  適用した判定経路)。UIはこれを表示するだけで済む形にする。
- relay自身の健全性も返す: event queueの生存と最終イベント時刻。
  キューが死んでいる間、データは `unknown` として配信する。
- 状態はインメモリでよい(agent_roomと同方針: スナップショットファイルは
  作らない)。再起動=再スイープ。stalled閾値は設定値(env等)にする。

## Step 4: フロントのoperation roomビュー

- agdevworldに新ビューを追加。PanelGridScene再利用か新規コンポーネントかは
  裁量(agent_roomでは再利用+`panelHeight`/`nameFontSize`追加で足りた)。
- 表示の必須要件は2つだけ:
  1. **unknownを第3の状態として明示的に描く**。relay不達・キュー死亡・
     ロスター取得失敗を「すべて正常」に見せない。p9の26分沈黙は観測上
     idleと同一だった — unknownをidle色で塗るボードはこの画面の存在理由を
     殺す。
  2. **各行にprovenanceを添える**(例: 「stalled — 最終実投稿から17分、
     servedノートなし」)。緑も赤もすべて痕跡からの推論なので、根拠を
     見せることが誤stallへの唯一の防御。
- stalledを最上位に並べる。グルーピング(agent別/プロジェクト別)は裁量。

## Step 5: 検証

- **スクリーンショットで見る**。agent_room step5の教訓(非可視チェックを
  全通過した視覚欠陥が3件)をそのまま適用する。CDP直叩きのドライバは
  step5で作った約40行の手法が使える。`--headless --screenshot` は
  PanelGridSceneの無限tweenで固まるので使わない。
- stalled検知の実証: 専用テストチャンネル(Step 2の例外)で人工的に
  「名指し後、served無し」の状況を作り、閾値経過でstalledに遷移し、
  返信・✔で解消することを確認する。
- 幻stallの再検査: p1で誤検知を出したFrontと autolab について、
  intro由来ロスターで誤検知が消えていることを確認する。
- event queue経路の確認: ✔リネームがdoneに反映されること(bare-topic
  キーイングの実地確認)。

## 罠(p1で実測済み、先に読むこと)

- Zulipホストが `.local` 名の場合、mDNSのAAAA未応答で名前解決が数秒
  止まることがある。遅いと感じたらIPv4強制かIP直指定。
- macOSのLocal Network許可はバイナリ単位。relayを新しいinterpreterや
  launchd配下で動かすときは、最初に疎通を確認する(p2ではZulipのみ
  だが、p3のComfyUI/ollamaで確実に踏む)。
- `#front` は公開チャンネル。観測botのsubscribe範囲を考えるとき、
  公開チャンネルは購読なしでも読める一方、mention検索の挙動が変わる点に
  注意(p1プローブの実装が先例)。

## 制約(最小限)

1. ロスターをobserver側にハードコードしない。introから読む。
2. 定常運用はevent queue駆動。全量スイープは起動時と再同期時のみ。
   観測は専用botのcredentialで行う。
3. unknownをidle/正常として描かない。
4. selfnoteを発話者に数えない(`agag.selfnote` を通す)。
5. 観測botはテストチャンネル以外に投稿しない。credentialはコミットしない。

それ以外(introの形式、状態エンジンのデータ構造、UIの見た目、
グルーピング、stalled閾値の初期値調整)はすべて実装者の裁量。

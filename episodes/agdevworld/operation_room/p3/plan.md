# operation_room p3 実装計画 — routine表示とagfrontチャット

braindump(p3/braindump.md)とそのレビューに基づく。スコープは
routine一覧・routineごとのagfrontチャット・選択中セッションの進行表示。
プロセス/バックエンド層の全面タイルはp4。braindump 7行目の
「実行中トピック情報のZulipへの集約投稿」は**採用しない** — p2のrelayが
既にイベント駆動でその集約を果たしており、Zulipへの書き戻しは古くなれる
第二の正本を作るだけ(in-system agent向けに価値があるなら別episodeで)。

前提: relayはlaunchd常駐(p2ex2)。コード反映は
`launchctl kickstart -k gui/$(id -u)/com.agdev.agentroom`。

## 全体像

```
schedule.json(ローカル読み) ─┐
#front routine-* topics ──────┼→ relay /routines, /routines/<name>  (読み: 観測bot)
#front front-routine-* fires ─┘
[selfnote][rootchat] の痕跡 ──→ relayがセッション木を内部再構成
チャット送信 ────────────────→ relay POST /chat  (書き: developer credential)
```

## Step 1: relayのroutine読み取り(`/routines`)

- **ロスター**: `#front` の `routine-*` トピック(常設依頼)。発火の会話は
  `front-routine-<name>`。どちらも観測botが既にevent queueで購読済みなので、
  新規コストはほぼ無い。
- **schedule.json**: `pj-agdev/.local/rtschedule/schedule.json` をrelayが
  **ローカルファイルとして直接読む**。:8093のroutine-guiをfetchしない —
  http.serverはCORSヘッダを返さないのでブラウザから直接読めず、relay経由に
  する必然性がある。パスはplistのenvで渡す(絶対パスを非ignoreファイルに
  書かない)。スキーマは `devenv/routine/dispatch.py:50-95`
  (`requests[]` / `events[]`、`fired_at`、7日pruning)。
- 各routineに出すもの: 名前、常設依頼文、次の発火予定/直近の発火、そして
  **直近の発火が応答されたか**(fireの投稿にFrontの返信が続いたか —
  旧p3の「routine-fire answers」をここで実装する)。

## Step 2: セッション木の再構成

「routine実行中セッション」= 1回の発火から派生した会話の木。
`front-routine-<name>` の発火を根に、`[selfnote][rootchat] <channel>/<topic>`
の痕跡をたどって、Frontが開いた/呼んだ先(workplan-、workrun-等)を接続する。

- relayはevent queueで全realmの投稿を見ているので、rootchatノートも
  受信済み。**表示はしないが、内部のリンク情報としては使ってよい**
  (「selfnoteを発話者に数えない」「ユーザーに見せない」の2不変条件は維持)。
- 各ノードには/opsの既存状態(awaiting/acked/stalled/done)をそのまま貼る。
  状態計算を二重実装しない。
- 一覧に出すセッションは**直近3件まで**(braindumpの指定)。それ以前は
  ドリルダウンでだけ見えれば十分。

## Step 3: チャット送信(`POST /chat`)

agdevworld初の書き込み経路。分離を固くする:

- **credentialは観測botと別**。`AGENTROOM_CHAT_ZULIP_ENV` 等でdeveloperの
  envを渡す。未設定ならチャットは読み取り専用になり、ビューがそう言う
  (OPSROOM_ZULIP_ENV無し→503と同じ縮退パターン)。観測botへの
  フォールバックはしない。
- **投稿先は `#front` のroutine関連トピックのみ**(relay側で制限する)。
  他agentのチャンネルへはGUIから直接書かない — 配車はFrontの仕事であり、
  Single Entranceの意味そのもの。
- developerとしての投稿がFrontの有償runを起動するのは仕様(それがチャット)。
  ただし**Zulipは長すぎる投稿を黙って切り詰める**(comfynotifyで実証済み)
  ので、入力長のガードを入れる。

## Step 4: フロントのroutineビュー

- routine一覧カード(Step 1の内容+「未応答の発火」があれば強調)。
- routineを選ぶと: そのセッション木(最大3セッション)と、
  `front-routine-<name>` の**履歴付きチャット**。チャットUIは死んでいる
  `chatPanel.ts` を今度こそ本来の姿(Zulip #frontの薄いラッパー)にする —
  README_DEVが「later phase」と約束していたのがこのフェーズ。
  履歴表示はselfnoteを除去した実投稿のみ。
- **選択中セッションの高頻度チェックは非Zulip信号だけ**に適用する。
  Zulip側はevent queueが既にリアルタイムなので何も足さない(429の教訓に
  逆行するポーリング追加は不可)。非Zulip信号とは当面、関与agentの
  in-flightハーネス検知(roleのworkspace dirに、それより新しい
  `run-NNNN.json` が無い= run進行中。p1の実証済みシグナル)。数秒間隔で
  relayが見るのは選択中セッションの関与agentに限る。
- unknown≠idle と provenance表示の2原則はこのビューにもそのまま適用。

## Step 5: 検証

- スクリーンショット(`.local/opsshot.mjs`)。視覚検証は4フェーズ連続で
  欠陥を捕捉している — 今回も必須。
- 読み取り側(一覧・セッション木・履歴)は実データで確認。
  過去のroutine発火が実在するので材料はある。
- **チャットの実往復テストは1回だけ、意図的に行う**: 実在のroutineトピック
  にGUIから短い質問を送り、Frontが応答し、それがチャット履歴に
  リアルタイムで現れるまで。これは実際に有償runを1回買う。自動テストや
  リトライでの送信連打はしない(送信系の単体テストはZulipをモックする)。
- launchd配下での schedule.json 読み取り確認(パス・権限)。

## 罠・ヒント

- Frontの返信は常にdeveloperとの `front-*` 会話に返る(p8契約)。発火
  トピックで会話していればそこに返るので、チャットビューはトピック単位で
  素直に読めばよい。
- `#front` は公開チャンネル。fire・返信とも観測botのevent queueに既に
  届いている(realm購読済み)。
- schedule.json は dispatcher が随時書き換える。読み取りは毎回読み直しで
  よい(小さいファイル。inotify等は不要)。
- チャット送信直後の自分の投稿は、event queue経由で戻ってくるのを待って
  履歴に出す(楽観表示するなら重複排除)。

## 制約(最小限)

1. 観測botはZulipに書かない。チャット書き込みは専用env(developer)経由で、
   フォールバック無し。
2. GUIからの投稿先は `#front` のroutine関連トピックのみ(relay側で強制)。
3. Zulipへのポーリング追加はしない。高頻度化は非Zulip信号かつ
   選択中セッション限定。
4. selfnoteは表示しない(内部のリンク再構成には使ってよい)。
5. 実投稿を伴うテストは手動・最小回数。credentialはコミットしない。

他(セッション木のデータ構造、チャットUIの見た目、in-flightポーリングの
間隔、ビューをopsビュー内のモードにするか独立ビューにするか)は実装者裁量。

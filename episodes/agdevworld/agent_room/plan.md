# agent_room 実装計画

braindump.md で挙げた「Zulip観測によるAgent状態表示」をagdevworldに追加する。
スナップショット(ファイルへの事前フェッチ)方式は取らず、リクエスト時にライブでZulipを読む。

実験用の非公開環境のため、破壊的変更・後方互換性の放棄は自由。以下に挙げる制約は
「守らないと壊れる/矛盾する」ものだけに絞ってあり、それ以外はすべて実装者の裁量。

## やること

1. Zulipをライブで読む小さなバックエンド(agdevworldには現状バックエンドが無い)
2. `#agents` の `intro-<instance>` トピックからAgent一覧を作る
3. `pj-<slug>` / `work-<slug>` チャンネル群から未解決(`✔ ` が付いていない)トピックの
   フラット一覧を作る
4. フロントに長方形カードで表示する新しいビュー

## 1. バックエンド

agdevworldは現在「pure frontend」(README_DEV.md参照、`modernize_agdevworld` p1で
assistantサービスごと削除済み)。ブラウザから直接Zulip REST APIを叩くのはAPIキーの
露出とCORSの問題があるので、小さな中継サーバーを新設する。

**モデルにすべき既存実装**: `pj-clusterintent/cagent/src/cagent_api/server.py` の
window server(stdlib `http.server` + `ThreadingHTTPServer`、ポート8790、
**無認証**)。cagentは node(mTLS)/human(bearer)/window(無認証)の3つの入口を
使い分けているが、agent_roomは読み取り専用・非公開環境なのでwindow相当の
無認証1本で十分。bearerトークンやmTLSは要らない。

**Zulip読み取りロジックは自作せず `pyagag`(`agag.zulip.ZulipClient`,
`pyagag/src/agag/zulip.py`)を再利用することを強く推奨**する。
理由:
- `RESOLVED_TOPIC_PREFIX = "✔ "` (zulip.py:88) が既に定義されていて、`channel_topics()`
  はresolved込みで返ってくるので自分でこの定数を再実装するとズレるリスクがある。
  実際に「✔ のリネームを見落として空トピックとして誤読し、ミッションが26分止まった」
  という事故が過去に起きている(devdocs/README_DEV.md:175-179)。ここは既存コードに
  乗るのが一番安全。
- レートリミット処理(`RateLimited`, `rate_limit_backoff`)も既に入っている。
- `channels()` / `channel_folders()` / `stream_id()` / `topic_history()` などが
  一通り揃っている。

Node/TSで直接Zulip REST APIを叩く選択肢もあり得るが、その場合は上記の
resolved-prefix処理とレートリミットを自分で再実装することになる点は認識しておくこと。
どちらを選ぶかは実装者の裁量。

**認証情報**: 専用のread-onlyな権限を持つZulip credentialは今のところ存在しない
(Zulipに読み取り専用APIキーという概念自体がない)。`pj-agdev/.local/zulip/*.env`
または各エージェントリポジトリの `.local/zulip.env` に既存botの認証情報がある
(`front-bot`, `autolab-*-bot`, `forge-bot`, `cagent-bot`, `developer` 等)。
非公開の実験環境なので、新しいbotを`agag init`で作らず既存のもの(例:
`developer.env` か `cagent-bot`)を使い回して構わない。専用botを新設したければ
それも良い。どちらでも良いので実装を止めない。**ただしcredentialファイル自体は
コミットしない**(既存の `.local/` gitignore運用に合わせる。styles.mdの
「ローカル環境情報を非ignoreファイルに出さない」ルールにも合致)。

**未解決トピック一覧を作るための列挙**: `pj-<slug>` チャンネル + その
`work-<slug>` 派生チャンネルを横断列挙するコードは現状どこにも存在しない
(作成/アーカイブ用のコードはあるが読み取り用の列挙は無い、
`pj-agdev/agautolab/src/agautolab/project_archive.py:100` あたりのネーミング規則が
参考になる)。ここは新規に書く必要がある。単純に `channels()` を全部見て
名前が `pj-` で始まるもの、および同じchannel folderに属する `work-` 始まりの
チャンネルを拾えば足りるはず。プロジェクトごとにグルーピングするか、
全部フラットにするかは実装者の裁量(フラットの方が早く終わる)。

**キャッシュ**: ファイルへのスナップショットは作らない、という制約は「バックエンド
プロセスの起動中だけ効くインメモリキャッシュ」までは禁止していない。Zulip API呼び出し
回数が気になるなら軽いin-memoryキャッシュを挟んでよい。必須ではない。

## 2. Agent一覧

`#agents` チャンネルの `intro-<instance>` トピック(append-only)をそのままソースにする。
これは「契約」(README_DEV.md:184)なので、GUIは加工・解釈しすぎず、投稿内容を
素直に見せるだけでよい。

- 表示するのは最新の紹介ポストの本文で十分(挙動変更時に再投稿される運用なので)。
  履歴も見たければ全ポストを併記しても良い。
- **`[selfnote]` および `[selfnote][served]` タグの行は非表示にする**こと。
  これはagent間の内部連絡用で、`chatlog.md`/`agentchat read` からも意図的に
  隠されている(README_DEV.md:156-161)。生Zulipメッセージをそのまま読むと
  混ざって出てくるので、フロントかバックエンドどちらかで一度は必ずフィルタする。
- harness/model/backendの情報は出さない(braindump時点の判断通り)。introに
  そもそも書かれていない設計だし、「Agent ≠ Model」方針(README_DEV.md:211-213)
  にも合っている。

## 3. 未解決ワーク一覧

- 「トピック命名規則がagentごとに違うので難しいかも」という braindump の懸念は
  半分だけ当たっている。**resolved/unresolvedの判定自体はZulip共通機能
  (`✔ ` プレフィックス)なので命名規則に関係なく一律にできる**。
  `workplan-` / `assetplan-` / `workrun-` のような意味付けの解釈はagentごとに
  違うので、そこは無理に統一しようとせず後回しでよい。
- 第一段階は「チャンネル名 + 生のトピック名」だけを出すフラットな一覧で十分
  (braindumpで質問していたカード表示の粒度にも合う)。
- どのagent/プロジェクトの分まで巡回対象にするかは実装者の裁量。まずは
  `pj-`プレフィックスのチャンネル全部で良いはず。

## 4. フロント表示

- 既存の `PanelGridScene`(`src/scenes/PanelGridScene.ts`)は `nodes` /
  `workspaces` / `autolab` / `tasks` の4ビューを設定駆動で共有している
  (`src/views.ts`, `src/viewSwitcher.ts`)。ここに5つ目のview設定として
  乗せるのが一番手数が少ないが、別コンポーネントとして新設しても構わない
  (ユーザー指示通りどちらでも良い)。
- カードに載せる情報: agent名/instance名、entrance(チャンネル名)、
  intro本文、未解決ワーク件数(バッジ)。クリックで未解決トピックの
  フラット一覧を開く、程度で十分。
- 既存の `chatPanel.ts` (`/api/chat` 宛、現在死んでいる)や `detailPopup.ts` の
  プロファイル編集導線には触れなくてよい。今回のスコープ外。

## 制約まとめ(最小限)

1. スナップショットファイル方式(`scripts/fetch-cluster-state.mjs` → `public/*.json`
   のようなビルド時/事前実行フェッチ)は今回使わない。バックエンドが都度ライブに読む。
2. `[selfnote]` 系の行はユーザー向け表示から除外する。
3. 未解決判定は `✔ ` プレフィックスの有無で行う(自前で別ルールを作らない)。
4. harness/model/backend情報は表示しない。
5. Zulip credentialファイルはコミットしない(既存の `.local/` 運用に合わせる)。

上記以外(バックエンドの実装言語、PanelGridScene再利用の是非、データの
グルーピング粒度、キャッシュの有無、UIの見た目)はすべて実装者の裁量に委ねる。

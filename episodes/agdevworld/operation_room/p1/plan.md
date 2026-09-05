# operation_room p1 実装計画(調査フェーズ)

p1は調査フェーズ。目的は「実行状態がどこまで検知できるか」を信号源ごとに確定させ、
operation_roomの設計方針を決める材料を揃えること。コードを書く場合は本機能実装では
なく実験・検証(使い捨てプローブ)として扱う。

## 成果物

report.md に以下を含める:

1. **信号源ごとの能力マトリクス**: 信号源 / 何がわかるか / 遅延・鮮度 /
   タスクへの帰属可否 / 利用に必要な追加作業
2. **状態定義の提案**: braindumpの [計画のみ/実行中/返答投稿待ち/完了] を
   観測可能性の調査結果に合わせて再定義したもの(検知不能な状態は縮退させる)
3. **できないことリスト**: 検知不能・帰属不能と判明したものの明示

## 計画時点で判明していること(調査不要、報告に転記してよい)

調査の重複を避けるため、計画作成時に確認済みの事実を先に置く。

### run記録は終了時にのみ書かれる
- `pyagag/src/agag/harness.py:870` `write_run_record()` が単発の `write_text` で
  書く。呼び出しは `agag/agent.py:271` 付近で、`run_harness()` が**返ってから**。
  つまり実行中のrunは `run-NNNN.json` を残さない。番号の予約すらされない
  (`agag/topics.py:102` `next_record_path()` は空き番号スキャン)。
- レコードに開始/終了時刻は無く `duration_ms` のみ。終了時刻はファイルmtimeが唯一。
- フィールド: `schema: ag.agent-run.v1`, `role, profile, harness, provider, model,
  duration_ms, cost_usd, usage, num_turns, transcript, outcome, failure` 等。
  → **完了状態とバックエンド帰属の情報源としては優秀、「実行中」検知には使えない。**
- 置き場所: 各リポジトリの `.local/agent/<role>/run-NNNN.json`。

### 実行中を示すロック/pid/statusファイルは存在しない
- `agag/` 以下に lockfile / pidfile / in-progress マーカーは無い。
- `agag/status.py:45` の `agag-status.json`(`agag.status.v1`)はリスナーの
  ポーリング健全性ファイルで、**ポーリング成功時にしか更新されない**。
  ハーネス実行がリスナーをブロックしている間は古くなるため、
  staleness は「busy」と「dead」を区別できない。観測者にとっての罠。

### forgeの生成ジョブ状態はファイル遷移で読める
- `agforge/src/agforge/assetrun_topic.py:87,152` —
  `.local/agentws/<work id>/generator/pending.json`(投入済み・未watch)が
  `start_watching()`(同:342)で `watching.json` に置き換わる。
  フィールドは `{prompt_id, note}` + watch時に `trigger`。
- これが**バックエンドジョブをタスク(work id)に帰属できる現状唯一の経路**。

### ComfyUI
- URL は `AGFORGE_COMFYUI_URL`(agforgeの `.local/.env`)。
- 既存の読み取りコード: `agforge/src/agforge/comfy_video.py:105`(`GET /queue` →
  `queue_running`/`queue_pending`)、同:152 と `comfy_async.py:74`
  (`GET /history/<prompt_id>`)。comfynotify側にも
  `comfynotify/src/comfynotify/comfy.py:16-34` に同等ヘルパ。
- SwarmUI は生きた参照コード無し(過去episodeの env 名のみ)。ComfyUI直で考えてよい。

### ollama
- base_url 設定は各ワークスペースの `.local/agents.local.toml`
  `[local.provider.ollama]`(パーサ `agag/agent_config.py:289`)、
  デフォルト `agag/agcode.py:34` = `http://localhost:11434`。
- **`/api/ps` を叩くコードはどこにも無い**(nctl系のヘルスプローブは
  `/v1/models` と `/api/tags` のみ)。セッション数観測は完全に新規。
- ollamaのセッションには依頼元情報が無いので、タスク帰属は原理的に不可能の見込み。

### routine一覧には既存のGUI seamがある
- 正本は `pj-agdev/.local/rtschedule/schedule.json`(Giteaリポジトリのclone)。
  スキーマは `pj-agdev/devenv/routine/dispatch.py:50-95`
  (`requests[{id, said_at, until, text}]` / `events[{id, at, from, kind, routine,
  fired_at, logical_at}]`)。
- routine名の一覧は `#front` の `routine-*` トピック、発火は
  `front-routine-<name>` トピックへの投稿(`devenv/routine/trigger.sh`)。
- **`com.agdev.routine-gui.plist.in` が既に `python3 -m http.server 8093` で
  rtschedule ディレクトリを配信している。** operation_roomはこれを流用するか
  置き換えるかを設計判断として報告に含めること。
- launchd常駐(`com.agdev.agforge-zulip` ほか各リスナー、comfy-notifier)は
  `launchctl list` が安価な「何が生きているか」の情報源。

### selfnoteのパーサは揃っている
- `pyagag/src/agag/selfnote.py` — `SELFNOTE_MARKER`(:56)、
  `parse_served()`(:176)→ `(Conversation, message id)`、`parse_rootchat`(:162)、
  `without_selfnotes()`(:210)、`last_real_speaker`(:224)。
- 不変条件(:42): selfnoteを「最後に話した人」に数えてはならない。
  観測系は必ず `without_selfnotes` を通してから活動を導出すること。

## 調査項目

上の既知事実で埋まらない部分。優先度順。

### A. Zulip由来のタスク状態の再構成(最重要)
- 「名指しされたがまだserveされていない呼び出し」を
  `[selfnote][served] <channel>/<topic> <message id>` と突き合わせて検出できるか、
  実データで検証する。✔リネーム越しの突き合わせが正しく動くかも含む
  (p9の26分停止事故の再発検知そのもの)。
- ここから [返答投稿待ち] と [stalled(名指し後X分経過で未serve)] が
  導出できるかを確認する。**stalled検知はこの画面の実用価値の中心**なので、
  4状態案に加えることを前提に検証する。
- 実験プローブの土台には agent_room の relay(`agdevworld/agentroom/room.py`)が
  使える。ただしp1では relay 本体に本実装を足さず、使い捨てスクリプトでよい。

### B. 「実行中」の近似検知の実験
run記録・ロックファイルが無い以上、候補は3つ。それぞれ使い捨てプローブで
検知率と誤検知を確認する:
1. transcript ファイル(`run_harness` がストリーム中に書く)の成長監視
2. ハーネスCLIプロセスの `pgrep`(claude / codex / agy / gemini / agcode)
3. `agag-status.json` の staleness(busy/dead 曖昧性込みで、他信号との併用前提)

結論が「実行中はイベント駆動では取れず、粗いポーリング近似のみ」でも
それで良い。設計方針(状態は痕跡から再構成する)の根拠として報告する。

### C. バックエンド活動のタスク帰属
- ComfyUI `/queue` の prompt_id と forge の `pending.json`/`watching.json` の
  突き合わせで「どのworkの生成が走っているか」を実際に出せるか検証。
- ollama は `/api/ps` を一度叩いてみて、何が返るか(モデル名・数だけか)を
  記録する。帰属不可能ならその旨を「できないことリスト」へ。

### D. 状態定義の再構成
A〜Cの結果を踏まえ、状態を2層に分けて定義し直す:
- **会話/タスク層**(Zulip導出): 計画のみ / 返答投稿待ち / stalled / 完了(✔)
- **プロセス/バックエンド層**(実観測): ハーネス実行近似、ComfyUIキュー深さ、
  ollamaセッション、launchd生存
単一の状態機械に無理に統合しない。両層を結ぶのは現状 forge の prompt_id 経路
だけである、という前提で設計方針を書く。

## 制約(最小限)

1. **プローブからZulipに投稿しない。** botの投稿はエージェントの有償runを
   誘発し、会話を誤誘導する(既知の事故パターン)。読み取り専用で行う。
   どうしても投稿実験が必要なら、どのエージェントも監視していない
   専用テストチャンネルを作って行う。
2. 実験コードは使い捨てと分かる場所に置く(`.local/` 配下や明示の
   `experiments/` 等)。本体コード(relay、PanelGridScene等)への本実装は
   p2以降に回す。
3. credentialや `.local` の実値を報告・コミットに書き写さない
   (env変数名・ファイルパスの言及は可)。

それ以外(プローブの言語、検証の順序、どこまで深掘りするか)は実装者の裁量。
調査の途中で「これは取れない」と早期に確定したものは、深追いせず
できないことリストに落として先へ進んでよい。

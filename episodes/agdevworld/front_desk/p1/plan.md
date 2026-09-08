# Front Desk p1

## 目的と前提

agfront と雑談し、その会話から仕事を依頼して結果を受け取れる、グラフィックノベル風の Phaser シーンを作る。雑談後に `ghtrends` を一回実行し、完了まで確認する。

- 背景は本ディレクトリの `bg.png`、左下の顔は `agfront.jpg`。画像を加工せず利用するところから始める。
- 表示する GUI は Phaser で描画する。下部の顔の右に会話テキスト、最下部に入力バー、開閉できる別領域に会話履歴を置く。
- ユーザーへの返答は絵文字を多用するギャル風。他エージェントへの投稿は通常の業務文体。
- 非公開の実験環境であり、後方互換性は不要。必要な箇所は置き換えてよい。細かな API、配置、クラス分割は実装者が決める。新しい認証基盤や汎用チャット基盤の整備は本フェーズの目的に含めない。

## 実装手順

### 1. Phaser シーンと入力

`agdevworld` に専用の `FrontDeskScene` を追加し、`/?view=frontdesk` などから開けるようにする。既存 dashboard からの導線も用意する。背景は全面を覆い、顔は縦横比を維持して表示する。

最新の返答、長文のページ送り、履歴の開閉・スクロール、送信状態を実装する。履歴を見ている間も入力中の文章を保持する。リンクなど、仕事の結果を確認するための情報も辿れるようにする。

日本語 IME と絵文字を最初に小さく検証する。見える入力バーは Phaser、IME・貼り付けの受け口には非表示 textarea を使ってよい。変換確定 Enter の誤送信、長文の欠落、絵文字の分断を確認する。演出や細かなレスポンシブ調整は実装者の裁量でよい。

### 2. character_talk とイベント駆動

`agfront/agents.toml` に `profiles.character_talk` と `roles.character_talk`、`agent/guides/character_talk/guide.md` を追加する。通常の front role から独立して設定できるようにし、初期のモデル・ツールは既存 front を参考にする。口調は profile 名ではなく新 guide で定義する。

会話トピックは `#front` の `front-desk-<会話ID>` を推奨する。既存の `front-` 監視を利用できる。新規会話と既存会話の再開を提供し、返答先の home トピックから role と guide を選ぶ。callback でも同じ選択処理を通す。

一回の実行は、現在の投稿を読み、必要な依頼とユーザーへの返答を行って終了する。相手の返答やユーザーの確認を待つためにハーネスを保持せず、次の Zulip 投稿で再開する。新 guide にはこの流れを明記し、既存 guide の `agentchat wait` 案内は引き継がない。必要な証拠の確認には `agentchat read --since` が使える。

他エージェントの発見・依頼・監督には既存の紹介文、`agentchat`、`rootchat`、`served` を使う。依頼先や作業手順をコードに固定する必要はない。ユーザーへの返答だけをキャラクター口調とし、他エージェントへの投稿は通常文体にする。

### 3. relay と会話の接続

`agdevworld/agentroom` に Front Desk の会話一覧・履歴取得・送信を追加する。Zulip を履歴の正本とし、既存のイベントキューから画面へ反映する。再読み込み・relay 再起動・解決済みトピックでも履歴を再取得できるようにする。

送信には既存の Developer 用資格情報を利用する。Front Desk の送信先検証、文字数確認、送信中の二重操作抑止を設ける程度でよい。送信結果が不明な場合はその旨を表示し、自動再送で同じ依頼を増やさない。資格情報とローカル環境情報は従来どおり ignored ファイルに置く。

`selfnote` は表示せず、ACK は返答と区別して受付状態に使う。画面の待機表示とエージェントの実行継続は別物として扱う。接続不調は画面で分かるようにし、取得できない状態を完了や無活動と断定しない。

### 4. 検証・反映

- `npm run build`、変更に対応する agfront と relay のテストを実行する。特に直接投稿と callback の両方で character_talk が選ばれること、通常 front との区別、履歴復元、送信先と二重送信を確認する。
- ブラウザで画像、長文、履歴、日本語 IME、絵文字、画面サイズ変更を確認する。web を再ビルドし、変更した常駐プロセスを再起動して反映を確認する。
- Front Desk から雑談を数往復し、続けて「ghtrends を一回実行して、完了まで見届けて」と依頼する。必要な会話上の確認にも同じ画面から答える。
- 他エージェントへの依頼後に Front の実行が終了し、相手の mention で再開して元の画面へ報告することを記録する。ユーザー向けとエージェント向けの文体も実際の投稿で確認する。
- ghtrends の成果物と index の更新・コミット、作業完了、Front Desk への結果報告まで確認する。受付や「依頼した」という返答だけでは完了にしない。
- `report.md` に結果、会話・作業の参照、残課題を簡潔に残す。変更を commit/push する。ローカル固有の証拠は ignored 領域に置く。

## 調査で得たヒント

- `src/main.ts` は入口を分岐しており、Phaser は world view のときだけ読み込む。専用シーンは `PanelGridScene` へ押し込まず独立させると扱いやすい。
- `agfront/src/agfront/zulip_listener.py` の `serve`、`front_prompt`、`run_front` は現在 front 固定。`handle_mention` は `rootchat_home` で元会話を特定して同じ `serve` を呼ぶので、ここに共通の role/guide 選択を置ける。`run_role` には profile 指定もある。
- 既存 listener は実行後に到着した投稿も再確認する。callback の処理済み記録は `note_served` が担う。別の待機ループや会話台帳を追加する前にこの経路を利用する。
- relay の `chat.py` は現在ルーチン専用。`server.py` と `ops.py` が API とイベントキューの主な接続点。ルーチン用の制約を緩めるだけで済ませず、Front Desk の用途に合う入口を作る。
- 実際のルーチン名は `ghtrends`。`routine-ghtrends` には Front の報告も混在するため、standing request は単純な最新投稿ではなく、既存 `routines.py` と同様に依頼者の最新投稿を読む。
- ghtrends の依頼は、未収録の trending repository 一件を選び、GitHub API の数値を使って `main/repos/` の要約と `main/index.md` を同じコミットで更新するもの。実行時は実際の standing request を読み直す。調査時の最新 run は検証のみで、作業の成功例ではない。
- 調査時点では `nctl status` は正常、対象ホストの drift は converged、relay は live、チャット資格情報は設定済み。実装時のサービス状態は `pj-clusterintent/nctl` または Nautobot で確認する。現時点では pj-clusterintent のコード変更は想定しない。
- 画面確認には既存の ignored `.local/opsshot.mjs` が参考になる。手順とサービス反映方法は `agdevworld/README_DEV.md` と `pj-agdev/.local/devenv.md` を参照する。環境メモには過去の記述も残るため、現在のコードと観測を優先する。

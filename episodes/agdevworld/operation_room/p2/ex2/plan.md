# operation_room p2ex2 — p3前の残件掃討

p2が残した3つの小粒残件を、p3(routine表示+チャット)に入る前に片付ける。
いずれも独立しており、順不同で並行してよい。

## A. done行confirm追補の実施

計画済み: `../confirm/plan.md` をそのまま実施する(relayの
`POST /ops/confirm` 全消し、done行限定、インメモリ記録、
「N rows open」カウントのdone除外、テストベッドでの一巡検証)。

## B. relayのlaunchd常駐化

手動起動をやめる。再起動のたびに再構築へ241コール掛かる状態エンジンを
relayが抱えた今、常駐化の根拠はagent_room時代より強い。p3のチャット常用の
前提でもある。

- **テンプレの先例**: `pj-agdev/devenv/launchd/` に `com.agdev.*.plist.in` が
  一式ある(`routine-gui`(http.server常駐)、`agfront-zulip` 等)。同じ流儀で
  `com.agdev.agentroom.plist.in` を足し、生成済みplistと実値は従来通り
  ignored領域に置く(絶対パスを非ignoreファイルに書かない)。
- **launchdの既知の罠**(いずれもこの環境で実測済みの先例あり):
  - plistに `PATH` を明示すること。cagent-apiの先例: 無いと `uv` が
    見つからず全滅する。
  - 環境変数(`AGENTROOM_ZULIP_ENV` / `OPSROOM_ZULIP_ENV` /
    `AGENTROOM_STALLED_SECONDS`)はplistで渡す。credential値は書かず
    envファイルのパスだけを書く。
  - ZulipホストをmDNSの `.local` 名で引く場合、AAAA未応答で数秒止まる
    ことがある。遅ければIPv4強制かIP/localhost直指定。
  - macOSのLocal Network許可はバイナリ単位。launchd配下のinterpreterで
    最初に `serve.sh check` 相当の疎通を確認してから常駐化を完了とする
    (p3でComfyUI/ollamaに触る前の予行にもなる)。
- **運用**: コード反映は `launchctl kickstart -k gui/$(id -u)/<label>`。
  ログは `.local/` 配下へ。`KeepAlive` で落ちたら上がる、程度で十分
  (再起動コストは241コールだが、頻発しなければ許容。頻発するようなら
  その時に考える)。
- **検証**: `launchctl list` にpid、`curl /healthz`、Macの再ログイン後も
  `/ops` が生きていること、event queue失効→再登録が無人で回ること。

## C. agping-agstudio1 の退役

p2レポートの判断事項。プロジェクト実体がどのマシンにも無い以上、
ボードのためだけにfixtureを再生成するのは本末転倒なので、**退役**を選ぶ
(再生成したくなったら runsmoke1 の記録から `agag init` でいつでも作れる)。

- `#agents` の `intro-agping-agstudio1` トピックを✔ resolveする。
  これはこのepisode唯一の意図的なZulip書き込みであり、**developer
  credentialで行う**。観測bot(Opsroom Observer)は今後も書かない。
- resolve後にopsボードとagent_roomビューの両方を確認する。relayや
  agent_roomが「✔済みintroトピック=退役」を実装していない場合、
  amberのunknownカードが残り続けるので、そのときは
  「resolved introはロスター/一覧から除外する」小修正を両ビュー分入れる
  (bare-topicキーイングの流儀に合わせ、✔を剥いで同一視した上で
  resolved フラグで除外)。
- agpingのZulip botアカウント自体を無効化するかは裁量(残っていても
  ボードに出なくなれば実害はない)。

## スコープ外(記録のみ)

- p2レポートのDeus Ex Machina note由来のENT候補
  「introが古いと告げられたらagent自身が再投稿する」は本episodeでは
  扱わない。
- プロセス/バックエンド層のタイルとroutineチャットはp3以降。

## 制約(最小限)

1. 観測botはZulipに書かない(Cのresolveはdeveloper credentialで)。
2. credential実値・絶対ローカルパスを非ignoreファイルに書かない。
3. confirm実施分は `../confirm/plan.md` の制約に従う。

他はすべて実装者裁量。3件とも report.md は1本にまとめてよい。
